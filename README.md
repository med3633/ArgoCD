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
