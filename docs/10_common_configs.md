# Give label and tain to master nodes

```bash
kubectl taint node master-1 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint node master-2 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint node master-3 node-role.kubernetes.io/control-plane:NoSchedule


kubectl label node master-1 node-role.kubernetes.io/control-plane=
kubectl label node master-2 node-role.kubernetes.io/control-plane=
kubectl label node master-3 node-role.kubernetes.io/control-plane=
```

# Install Ingress Controller by helm chart
I chose nginx ingress controller. Firstly we need to install helm . 
Reference: https://helm.sh/docs/intro/install/

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm show values ingress-nginx/ingress-nginx > ingress-nginx.yaml
helm install ingress-nginx ingress-nginx/ingress-nginx --values=ingress-nginx.yml --create-namespace

```

# Install Cert manager by helm chart

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.11.0 \
--set installCRDs=true
kubectl apply -f cluster-issuer.yaml 
```

# Install kubernetes monitoring by helm chart

## Add helm repo and update
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Save values to file and update
```
helm show values prometheus-community/kube-prometheus-stack > kube-prometheus-stack.yaml
```

## Install
```
helm install prometheus prometheus-community/kube-prometheus-stack --values kube-prometheus-stack.yaml -n monitoring --create-namespace
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --values kube-prometheus-stack.yaml -n monitoring
```

## Create ingress for grafana
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: monitoring.asilbek.com
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  rules:
  - host: monitoring.asilbek.com
    http:
      paths:
      - backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - monitoring.asilbek.com
    secretName: backend-ingress-tls
```