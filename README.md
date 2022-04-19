# Kubernetes PH Cluster

The whole PH K8s cluster configuration managed by ArgoCD (gitops).
This is meant to be managed by DevOps team.

# Prerequisite
- fresh kubernetes cluster
- (optional) argocd CLI
- (optional) argo (workflow) CLI

# Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.3.3/manifests/install.yaml
```

# Access ArgoCD
`kubectl -n argocd port-forward svc/argocd-server 8081:80`

`localhost:8081` at the browser

# Get argo admin password
`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

the user name is `admin`

# Login to Argo
`argocd login localhost:8081`

# Change app-access-test IP address
Change the IP addresses in `./manifest/app-access-test` to what you are using.

_This is tested in single kubernetes node in linode_
_Ideally, you should use a hostname_. 

# Create all VCS resources
```
argocd proj create cluster-tools -f argo/project.yml
argocd app create cluster-root -f argo/root.yml
```

# Get Argo Workflow Token
`SECRET=$(kubectl get sa argo-workflow-admin -o=jsonpath='{.secrets[0].name}')
ARGO_TOKEN="Bearer $(kubectl get secret $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"
echo $ARGO_TOKEN`

# Access Argo Workflow
`kubectl -n argoworkflow port-forward deployment/argo-server 8082:2746`
`https://localhost:8082`

# Expose Argo Webhook Event
`kubectl -n argoevents port-forward $(kubectl -n argoevents get pod -l eventsource-name=webhook -o name) 12000:12000 &`

# Maintenance

## Adding new Argo App or K8s resource?
Add the entry at `./argo/apps`

## K8s Configuration Update?
Git commit and push it, ArgoCD will pick it up.
You can also manually hit the "sync" button in argo app

# Test

## Test App access
`http://$IP/smoke`

## Test argo event webhook
`curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http://localhost:12000/example`

This should return the default nginx landing page

## Note
`istio-ingressgateway` service is moved to app-access-test.. i think argocd shines with kustomize
ArgoCD complains because the k8s resource is defined in 2 applicatiion
which can potential troublesome for user because he/she has multiple places that can modify the file. 
With kustomize, it aligns to what ArgoCD wants, DRY code.

