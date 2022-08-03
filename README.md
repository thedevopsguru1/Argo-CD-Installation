# Argo-CD-Installation
Requirements
Installed kubectl command-line tool.
Have a kubeconfig file (default location is ~/.kube/config).

## 1. Install Argo CD
### Create a namespace named argocd for Argo CD
```
kubectl create namespace argocd
```
### Installing Argo CD into the target cluster
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## 2. Service Type Load Balancer

### Change the argocd-server service type to LoadBalancer:
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
## 3. Port Forwarding
### Kubectl port-forwarding can also be used to connect to the API server without exposing the service.
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
### The API server can then be accessed using https://localhost:8080
