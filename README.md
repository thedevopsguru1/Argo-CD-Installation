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
# We have three options: Load balancer or Port forwarding or Ingress
## 2. Service Type Load Balancer(Only for work)

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
## 4. Login Using The CLI
### The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using kubectl:
### Please make sure that you run this command only on git bash
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
![image](https://user-images.githubusercontent.com/107158398/182578029-b1309b9e-7317-4b54-876f-c47285614dcd.png)

### Go to the argo CD UI and as username put admin. 
### the password will be the one you got from the previous command.
![image](https://user-images.githubusercontent.com/107158398/182578162-59320f6d-93f2-4e05-b5d0-d03867d3e367.png)
# Thanks guys
# To Uninstall Argo CD from a k8s
## Use this command:
```
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
# Add Ingress for our argoCD
### No TLS
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
 ```
 ### With TLS
 ```
 apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/ingress.class: nginx
   # kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    # If you encounter a redirect loop or are getting a 307 response code
    # then you need to force the nginx ingress to connect to the backend using HTTPS.
    #
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-secret # do not change, this is provided by Argo CD
 ```
