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
helm install prometheus prometheus-community/kube-prometheus-stack --values kube-prometheus-stack.yaml
```

## Create ingress for grafana
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: monitoring.zafarsaidov.uz
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: monitoring.zafarsaidov.uz
    http:
      paths:
      - backend:
          service:
            name: prometheus-grafana
            port:
              number: 3000
        path: /
        pathType: Prefix
```