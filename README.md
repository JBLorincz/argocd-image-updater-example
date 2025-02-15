# Steps to use
## Initial steps 
1. Create a kubernetes cluster (minikube, microk8s, whatever you want)
2. [Install ArgoCD to that cluster](https://argo-cd.readthedocs.io/en/stable/getting_started/)
3. [Ensure you have the ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
4. [Install Argocd image updater to that cluster](https://argocd-image-updater.readthedocs.io/en/stable/install/installation/)
## Git steps
5. Create a [access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) in github with the `repo` scope and permissions to access your forked repo. Remember the token!
6. Fork this repository. ArgoCD will need a way to modify the repository itself to achieve the GitOps design. The access token will let it do that.
7. Using the Argo CLI, Register your repository like so: `argocd repo add <PATH TO YOUR FORK HERE> --username accesstoken --password <accesstoken>` NOTE: use your access token as the password here (username can be any nonempty string). Path to your fork should look something like `https://github.com/jblorincz/argocd-image-updater-example`
8. In Argo, create a git secret with your access token to let image updater make updates.
```
kubectl -n argocd create secret generic git-creds \
  --from-literal=username=accesstoken \
  --from-literal=password=<accesstoken>
```
9. Using the Argo CLI, apply the main app:
```
argocd app create apps \
    --dest-namespace argocd \
    --dest-server https://kubernetes.default.svc \
    --repo <PATH TO YOUR FORK HERE> \
    --path image-updater-example  
argocd app sync apps
```

10. Watch as two apps get deployed, a `develop` and a `production` app. both will spawn one pod using the `nginx:stable-alpine` tag. develop is hooked up to argo image updater, while production is not. Both pods will be using the same image at first, but wait (at least a few hours) and you will see that the develop app the image gets automatically updated as new updates are made to nginx.


11. Check for `argo-image-updater` to make a commit to your Github repo, modifying the file at `example-app/.argocd-source-develop.yaml`. This is where ArgoCD Image Updater will write the image tag to use, overriding the setting in the `values.yaml` to ensure that the image with the most recent digest is being applied for the image tag.


Read more about the ArgoCD Image Updater here: [https://argocd-image-updater.readthedocs.io/en/stable/](https://argocd-image-updater.readthedocs.io/en/stable/)
