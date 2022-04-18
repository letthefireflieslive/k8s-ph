# Kubernetes PH Cluster

The whole PH K8s cluster configuration managed by ArgoCD (gitops).
This is meant to be managed by DevOps team.

# Prerequisite
- fresh kubernetes cluster

# Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.3.3/manifests/install.yaml
```

# Access ArgoCD
`kubectl -n argocd port-forward svc/argocd-server 8081:80`

# Get argo admin password
`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

the user name is `admin`

# Login to Argo
`argocd login localhost:8081` or via `http://localhost:8081`

# Create all VCS resources
`argocd app create root -f argo/root.yml`

# Maintenance

## Adding new Argo App or K8s resource?
Add the entry at `./argo/apps`

## K8s Configuration Update?
Git commit and push it, ArgoCD will pick it up.
You can also manually hit the "sync" button in argo app

