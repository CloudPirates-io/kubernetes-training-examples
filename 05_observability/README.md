
# Metrics Server

The Metrics Server collects resource metrics from Kubelets and exposes them through the Kubernetes API server for use by monitoring tools and autoscalers (HPA, VPA).

To use the metrics server we created a customized (non production) config.

Based on your current cluster setup (KIND or Managed Training Environment) you need to install
different files to your cluster:

### KIND Install Command
```bash
kubectl apply -k ./metrics-server-kind
```

### Managed Training Environment Install Command
```bash
kubectl apply -k ./metrics-server-managed
```

## Using Metrics Server

After installation, wait a few moments for the metrics server to start collecting data. Then you can use the following commands:

### kubectl top nodes

Displays resource usage (CPU and memory) for all nodes in the cluster:

```bash
kubectl top nodes
```

**Example output:**
```
NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
control-plane        250m         12%    1024Mi          25%
worker-1             150m         7%     768Mi           19%
worker-2             180m         9%     896Mi           22%
```

- **CPU(cores)**: Current CPU usage in millicores (1000m = 1 core)
- **CPU%**: Percentage of allocatable CPU being used
- **MEMORY(bytes)**: Current memory usage
- **MEMORY%**: Percentage of allocatable memory being used

### kubectl top pods

Displays resource usage for pods in the current namespace:

```bash
kubectl top pods
```

**Show pods from all namespaces:**
```bash
kubectl top pods -A
```

**Show pods from a specific namespace:**
```bash
kubectl top pods -n kube-system
```

**Sort by CPU or memory:**
```bash
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory
```

**Show containers within pods:**
```bash
kubectl top pods --containers
```

**Example output:**
```
NAME                          CPU(cores)   MEMORY(bytes)
nginx-deployment-abc123       5m           32Mi
backend-api-def456            50m          128Mi
database-ghi789               100m         512Mi
```

- **CPU(cores)**: Current CPU usage in millicores
- **MEMORY(bytes)**: Current memory usage

