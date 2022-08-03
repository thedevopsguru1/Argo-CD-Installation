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
## 4. Login Using The CLI
### The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using kubectl:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
## 5. Creating Apps Via UI
Open a browser to the Argo CD external UI, and login by visiting the IP/hostname in a browser and use the credentials set in step 4.

After logging in, click the + New App button as shown below:

### new app button

Give your app the name guestbook, use the project default, and leave the sync policy as Manual:

app information

Connect the https://github.com/argoproj/argocd-example-apps.git repo to Argo CD by setting repository url to the github repo url, leave revision as HEAD, and set the path to guestbook:

connect repo

For Destination, set cluster URL to https://kubernetes.default.svc (or in-cluster for cluster name) and namespace to default:

destination

After filling out the information above, click Create at the top of the UI to create the guestbook application:

destination

7. Sync (Deploy) The Application¶
Syncing via CLI¶
Once the guestbook application is created, you can now view its status:


$ argocd app get guestbook
Name:               guestbook
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://10.97.164.88/applications/guestbook
Repo:               https://github.com/argoproj/argocd-example-apps.git
Target:
Path:               guestbook
Sync Policy:        <none>
Sync Status:        OutOfSync from  (1ff8a67)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME          STATUS     HEALTH
apps   Deployment  default    guestbook-ui  OutOfSync  Missing
       Service     default    guestbook-ui  OutOfSync  Missing
The application status is initially in OutOfSync state since the application has yet to be deployed, and no Kubernetes resources have been created. To sync (deploy) the application, run:


argocd app sync guestbook
This command retrieves the manifests from the repository and performs a kubectl apply of the manifests. The guestbook app is now running and you can now view its resource components, logs, events, and assessed health status.

Syncing via UI¶
guestbook app view app
