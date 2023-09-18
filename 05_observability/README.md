
# Metrics Server
To use the metrics server we created a customized (non prod) config

Based on your current cluster setup (KIND or Managed Training Environment) you need to install
different folders to your cluster:

### KIND Install Command
```
kubectl apply -k ./metrics-server-kind
```

### Managed Training Environment Install Command
```
kubectl apply -k ./metrics-server-managed
```
