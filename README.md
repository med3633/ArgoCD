# ArgoCD
- USE Kind (Kubernetes IN Docker) est principalement utilisé pour les tests locaux et le développement.
- ArgoCD tool for Gitops ( single src of truth )
- ArgoCD specified in CD and in kuberntes cluster while jenkins is a tool for CI/CD
click to see releases of kind
```bash
https://github.com/kubernetes-sigs/kind/releases
```
# ingress
```bash
kind: cluster
apiversion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
```bash
kind create cluster --config=cluster.yml
```
```bash
kubectl cluster-info --context kind-kind
```
# Install ArgoCD
## Install Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
### install Helm repo
```bash
helm repo add argo https://argoproj.github.io/argo-helm
```
### Crete ns ArgoCD 
```bash
kubectl create namespace argocd
```
### hart of ArgoCD
```bash
helm install argocd argo/argo-cd
```
# nginx-ingress controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/deploy/static/provider/kind/deploy.yaml
```
# update helm to use this ingress controller ==> SSL termination have access to ingress controller 
- nb: argocd is the name of the chart
```bash
helm upgrade argocd --set configs.params."server\.insecure"=true --set server.ingress.enabled=true --set server.ingress.ingressClassName="nginx" -n argocd argo/argo-cd
```
# create deployement for nginx
```bash
vi overlays/kustomization.yaml
```
```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: kustomization
resources:
  - ../base
namespace: default
patches:
- target:
    kind: Deployment
    name: nginx-deployment
  patch: |-
    apiVersion: apps/vi
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 2
```
## connect to git
settings > Repository  > connect repo 
## do Application to argoCD has all instruction of ArgoCD 
```bash
mkdir argo-cd
```
```bash
cd argo-cd
```
## manifest of ArgoCD (Application because argoCD have Application Controller )
- nginx.yml
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: ''
# say this path do only deployement
    path: overlays
# bransh of deploy
    targetRevision: main
# any cluster k8s would deploy
  destination:
# API srver of target cluster
    server: 'https://kubernetes.default.svc'
    namespace: default
# synchronize any change detect on repo we will apply on cluster k8s auto or manually
  syncPolicy:
    automated:
# rollback if state on cluste updated it not the same on git repo
      selfHeal: true
# if dlete manifest on repo  delete this manifest on k8s
      prune: true
```
```bash
kubectl apply -f nginx.yml
```
# change manifest on other branch
```bash
git checkout -b feature/decrease-replicas
```

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: kustomization
resources:
  - ../base
namespace: default
patches:
- target:
    kind: Deployment
    name: nginx-deployment
  patch: |-
    apiVersion: apps/vi
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 4
```
```bash
git add overlays/kustomization.yaml
```
```bash
git commit -m "Decrease the replicas count"
```
```bash
git push
```
# to accelare synch 
```bash
argocd app sync nginx
```










