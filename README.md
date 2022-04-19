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

# Change app-access-test IP address
Change the IP addresses in `./manifest/app-access-test` to what you are using.

_This is tested in single kubernetes node in linode_
_Ideally, you should use a hostname_. 

# Create all VCS resources
`argocd app create cluster-root -f argo/root.yml`

# Access Argo Workflow
`kubectl -n argoworkflow port-forward deployment/argo-server 8082:2746`
`https://localhost:8082`

# Maintenance

## Adding new Argo App or K8s resource?
Add the entry at `./argo/apps`

## K8s Configuration Update?
Git commit and push it, ArgoCD will pick it up.
You can also manually hit the "sync" button in argo app

## Test
`http://$IP/smoke`

This should return the default nginx landing page

## Note
`istio-ingressgateway` service is moved to app-access-test.. i think argocd shines with kustomize
ArgoCD complains because the k8s resource is defined in 2 applicatiion
which can potential troublesome for user because he/she has multiple places that can modify the file. 
With kustomize, it aligns to what ArgoCD wants, DRY code.


apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EUXhPREUyTWpRME1Gb1hEVE15TURReE5URTJNalEwTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTU1KClBRYW9Bd2IrSlQyY2NxRjhIUVV1cTlrbFgzWlBFVHQyWVFXYWtzN3FRUXdvMXVOREd4U3h2MEtEQWJ1Z3JIWXUKYUM4WmhYN3BIL0xBUVI3aW8wUlBUR3RtY1U2RE11bTdVSWtSbGdpTHZQZE5tUmJSY00raldUR2ZRQXlmc2R0dAp4TktlMFVnVnMyZlRoSFZNdEpMdXZpQmhJY21GS2hENDlRalpmSWhQeW50SHYyQW82ZEJnbktzMVJjUEhqN295Cm12MkFGVUdGd3FYaHA4aE5waktUWmRNcjZRWldDa0Z6ZXRxU0hEK2Ryc0NVekptN0hQQnlkSGFDV1lRK0FGay8KQ3FHa0xra0k5ZDlEWTF0QnduMUkydmt5czNLOC9xNkdpZ2kzQVZhMFBueEZldDBWY0d2NU1wd0pIMVhQSHliWAptTjdkMXdCb2FrbFNWV2p1NG5rQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZEZ0IvT3RsZHNkdms5NlpDMnArUC9EZXNCWnZNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQWlqcEFleUxQbVFHWVZuM3haWQpWL3dNMjRxZ25MaDROWVRNSXlmU3lPdkJGSmJMLzdvak04NUtFRkJqbWxnTFE5T0Q3K2gveTJiK3IvUERkYjhNClpJQUxzcWQxbWdpSUR0Rk1BSFExN0hqdXZDL1p6U0o4RVh1bDVZdm51Y0VNQ0FCVC96bjNxcU9DR2t5amZtUVcKMDVnVFZLTm5WVDQvNkRZTmRrcjJxRUZJRmRkbktrWUFVem9wbnltVmpnZER5eEllU2I4M0hzdzJhZUlsRkR3TQpyTHBocUtsWEw3b0Nib2w2clIvNkNqaVJxcVdIajU0ajB3K29TOFlVUW5OQ2hKWksrMnFLZGFmUUtmc0FrelZ0CjZzVnZkK0wrOXhFQzZZdGRrcksvdUQ5c0Y3bVBKWllpQ0ZqMzNERm9Qc2xWRXJQdHNTa3libnM4amJlb2VzcE4Kdk1vPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  server: https://e9a35095-1b3c-497f-891d-147c0590d051.ap-south-2.linodelke.net:443
  name: lke58062

users:
- name: lke58062-admin
  user:
  as-user-extra: {}
  token: eyJhbGciOiJSUzI1NiIsImtpZCI6ImE3R2psVHRqZDVTUnFDMVRKNjE4SmJyWGx5VEJibWRZZFIxWmN3bGxZcEkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJsa2UtYWRtaW4tdG9rZW4tNnd2ajIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibGtlLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMjM0YmJhZjItMmI3ZC00ZmI3LTgzODgtYjliNmNkNTYxOTgyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmxrZS1hZG1pbiJ9.WG2P8OqrHg0G97TE3QsjGXOTC7_x3_ASd4AKRUsEA8D1ubAsC01A1_Os7PtmWlzzobMzPeI7ucMlrRpWW8J7vokhxgQXYwhg2hwJb3p5FwZ7MaSDa3I_krAHf5UM3cZ6AyM0BwEChRpJzyXv7BtHybNCrVz66OVsb1go_4m3vvXRPRAWPL-_15QjJe09cmjAAvfR5SWroLHxH2-rDq1Bv5UVYfqZq1hVDzFO1RkVWi2yePj--lCLohHpkvJxepg4JPi-YidNTey2hwGF-rF3b_DeEq2kJbq6JIpW73qXnIjY6PaZ_M8IQq9GoP4735GJp4hlnOoz20P4ZgLnVmO3SQ

contexts:
- context:
  cluster: lke58062
  namespace: default
  user: lke58062-admin
  name: lke58062-ctx

current-context: lke58062-ctx
