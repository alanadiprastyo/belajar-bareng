# Belajar Bareng - GitOps ArgoCD

## Requirement 
- Installed kubectl command-line tool.
- Have a kubeconfig file (default location is ~/.kube/config).

1. Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Download Argocd CLI

```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```
or we can install argocd cli use homebrew
```bash
brew install argocd
```

3. Expose Api server from Argocd

By default argocd not exposed api to external cluster. but we can expose to access api use some techniques

    - update type load balancer on services
    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

    ```
    - port forwarding
    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    Note: the api server can access to localhost:8080
    - Ingress
    we can follow the [ingress documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/)


4. Login using CLI

The initial password for admin account is automatic generate by sistem when we apply the manifest install argocd. where the password is store to secret `argocd-initial-admin-secret `
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

login to argocd use username `admin` and password from above
```
argocd login IP_ARGOCD_SERVER
```
after login we can update the password

```bash
argocd account update-password
```

5. access use UI

use load balancer / ingress / port forwarding to access to argocd server `https://ipaddress-argocd`

6. Create application from repository

For example we can use repo from my gitlab `https://gitlab.com/alandiprastyo/argocd-app-config` where we define the application which to deploy on kubernetes

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://gitlab.com/alandiprastyo/argocd-app-config.git
    targetRevision: HEAD
    path: dev
  destination: 
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```
apply the `application.yaml`
```bash
kubectl create -f application.yaml
```

7. Manage application use argocd cli

show all application on your argocd
```
argocd app list
```
get the spesific application
```bash
 argocd app get myapp-argo-application

Name:               myapp-argo-application
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          myapp
URL:                https://34.135.189.214/applications/myapp-argo-application
Repo:               https://gitlab.com/alandiprastyo/argocd-app-config.git
Target:             HEAD
Path:               dev
SyncWindow:         Sync Allowed
Sync Policy:        Automated (Prune)
Sync Status:        Synced to HEAD (be2b619)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME           STATUS  HEALTH   HOOK  MESSAGE
       Service     myapp      myapp-service  Synced  Healthy        service/myapp-service created
apps   Deployment  myapp      myapp          Synced  Healthy        deployment.apps/myapp created
```

Ref:
- https://www.weave.works/technologies/gitops/
- https://radar.cncf.io/
- https://argoproj.github.io/