# Kubernetes PH Cluster

The whole PH K8s cluster configuration managed by ArgoCD (gitops).
This is meant to be managed by DevOps team.

# Prerequisite
- fresh kubernetes cluster
- (optional) argocd CLI
- (optional) argo (workflow) CLI

# Install ArgoCD & Sealed Secret
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.3.3/manifests/install.yaml
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.17.5/controller.yaml
kubectl wait pods --for=condition=Ready --all-namespaces --all
```

# Expose & Access ArgoCD
`while true; do kubectl -n argocd port-forward svc/argocd-server 8081:80; done`

Access
`localhost:8081` at the browser

# Get argo admin password
`export ARGOCDPW=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo)`

# Login to Argo
`argocd login --insecure --username admin --password $ARGOCDPW localhost:8081`

# Update IP address
`export IP=[YOURIP]`
```
sed -i -e "s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/$IP/g" manifest/app-access-test/install.yml
sed -i -e "s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/$IP/g" manifest/argo-events/example1.yml
sed -i -e "s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/$IP/g" manifest/argo-events/example2.yml
git add .
git commit -m "Update IPs"
git push
```

## Create sealed secrets
```
export DOCKER_USERNAME=[CHANGEME]
export DOCKER_PASSWORD=[CHANGEME]
export GITHUB_PERSONAL_ACCESS_TOKEN=[CHANGEME]
```
```
kubectl -n argoevents create secret docker-registry container-reg-creds --docker-username=$DOCKER_USERNAME --docker-password=$DOCKER_PASSWORD --docker-server="https://index.docker.io/v1/" --dry-run=client -o yaml | kubeseal -o yaml > manifest/argo-events/container-registry-creds.yml
kubectl -n argoevents create secret generic github-access --from-literal=token=$GITHUB_PERSONAL_ACCESS_TOKEN --dry-run=client -o yaml | kubeseal -o yaml > manifest/argo-events/github-pat.yml
git add . 
git commit -m "Add container registry and github PAT creds"
git push
```

# Create all VCS resources
```
argocd proj create cluster-tools -f argo/project.yml
argocd app create cluster-root -f argo/root.yml
```

# Expose and Access Argo Workflow
`while true; do kubectl -n argoworkflow port-forward deployment/argo-server 8082:2746; done`

Access at
`https://localhost:8082`

# Expose Argo Webhook Event
`while true; do kubectl -n argoevents port-forward $(kubectl -n argoevents get pod -l eventsource-name=github-sb1 -o name) 12001:12001; done`

# Maintenance

## Adding new Argo App or K8s resource?
Add the entry at `./argo/apps`

## K8s Configuration Update?
Git commit and push it, ArgoCD will pick it up.
You can also manually hit the "referesh"/"sync" button in argo app

# Test

## Test App access
`http://$IP/smoke`
This should return the default nginx landing page

## Test argo event webhook
`curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http:///$IP:12001/argoevent`
This should return the message success

You can also check `kubectl -n argoevents get workflows | grep "webhook"`

## Note
`istio-ingressgateway` service is moved to app-access-test.. i think argocd shines with kustomize
ArgoCD complains because the k8s resource is defined in 2 applicatiion
which can potential troublesome for user because he/she has multiple places that can modify the file. 
With kustomize, it aligns to what ArgoCD wants, DRY code.

