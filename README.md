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



