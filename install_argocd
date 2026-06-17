# Install Argocd
<br>
Install ArgoCD on k8s cluster
<br>
<br>

```sh
kubectl create namespace argocd
```
```sh
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
<br>
Kubectl port-forward can also be used to connect to the API server without exposing the service.
<br>
<br>

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
<br>
The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using the argocd CLI:
<br>
<br>

```sh
argocd admin initial-password -n argocd
```
