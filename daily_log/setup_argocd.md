# ArgoCD Setup

- ArgoCD has been installed.
- It has been installed on the master node.

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get po -n argocd  -w

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

kubectl exec -it -n argocd deployment/argocd-server -- /bin/bash

argocd login localhost:8080

argocd account update-password
```
