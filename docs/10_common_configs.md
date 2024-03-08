## Give label and tain to master nodes

```bash
kubectl taint node master-1 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint node master-2 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint node master-3 node-role.kubernetes.io/control-plane:NoSchedule


kubectl label node master-1 node-role.kubernetes.io/control-plane=
kubectl label node master-2 node-role.kubernetes.io/control-plane=
kubectl label node master-3 node-role.kubernetes.io/control-plane=
```

## Install Ingress Controller
I chose nginx ingress controller. Firstly we need to install helm . 
Reference: https://helm.sh/docs/intro/install/

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm show values ingress-nginx/ingress-nginx > ingress-nginx.yaml
helm install ingress-nginx ingress-nginx/ingress-nginx --values=ingress-nginx.yml --create-namespace

```

Install Cert manager 

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.11.0 \
--set installCRDs=true
kubectl apply -f cluster-issuer.yaml 
```