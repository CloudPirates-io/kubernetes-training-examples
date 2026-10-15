# Scaling

Kubernetes can adjust the number of Pods automatically to match load.

## HorizontalPodAutoscaler (HPA)

The HPA compares observed metrics with a target and changes a Deployment's replica count:

```
desiredReplicas = ceil(currentReplicas * currentMetric / targetMetric)
```

`04-hpa.yaml` scales `example-app` between 1 and 10 replicas, targeting 50% average CPU utilization. Utilization is measured relative to the container's **CPU request**, so requests must be set.

## Prerequisites

- Metrics Server installed (see `05_observability/`)
- CPU requests on the target Deployment

## Files

- `01-namespace.yaml`, `02-deployment.yaml`, `03-service.yaml` - the app to scale (namespace `scaling`)
- `04-hpa.yaml` - the autoscaler
- `k6-loadtest.js` - k6 script generating load against `http://localhost:8081`
