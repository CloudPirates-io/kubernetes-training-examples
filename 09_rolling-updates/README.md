# Deployment Strategies

This directory demonstrates three Kubernetes deployment strategies using a demo application that displays its current color and shows real-time request distribution.

## Demo Application

The demo app provides:
- **Interactive Dashboard**: Visual display of request distribution across versions
- **GET /color Endpoint**: Returns current version color (blue, green, red, purple)
- **Health Probes**: `/readyz` (readiness) and `/livez` (liveness)

Available versions:
- `blue` - V1 (initial version)
- `green` - V2 (working update)
- `red` - V3 (intentionally broken probes for rollback demo)
- `purple` - V4 (canary version)

Access the dashboard: http://rollingupdates.127-0-0-1.nip.io:8080 (or your training environment hostname)

---

## Directory Structure

### 1. rolling-updates/
Kubernetes native rolling update strategy with zero-downtime deployments.
- Gradually replaces old pods with new pods
- Automatic health check validation
- Built-in rollback capability

### 2. canary/
Progressive traffic splitting for safer deployments.
- **Header-Based Canary**: Route specific users to new version
- **Weight-Based Canary**: Route percentage of traffic to new version

---

## Training Workflow

### Prerequisites

```bash
# Apply namespace
kubectl apply -f rolling-updates/00-namespace.yaml

# Verify namespace
kubectl get namespace rolling-updates
```

---

## Part 1: Rolling Updates

### Step 1: Deploy V1 (Blue)

```bash
# Deploy initial version (blue)
kubectl apply -f rolling-updates/

# Verify deployment
kubectl get deployment -n rolling-updates
kubectl get pods -n rolling-updates

# Access the dashboard
# Open: http://rollingupdates.127-0-0-1.nip.io:8080
# Or test: curl http://rollingupdates.127-0-0-1.nip.io:8080/color
```

### Step 2: Scale to 10 Replicas

Scale up to see rolling update behavior more clearly:

```bash
# Scale deployment
kubectl scale deployment/demo-app --replicas=10 -n rolling-updates

# Verify all pods are ready
kubectl get pods -n rolling-updates
```

### Step 3: Rolling Update to Green

Trigger rolling update by changing the image:

```bash
# Update to green version
kubectl set image deployment/demo-app demo-app=harbor.cloudpirates.io/training/rollouts-demo:green -n rolling-updates

# Watch rollout progress (in separate terminal)
kubectl rollout status deployment/demo-app -n rolling-updates

# Watch pods being replaced
kubectl get pods -n rolling-updates -w

# Check dashboard - see blue pods gradually replaced by green
# Test: curl http://rollingupdates.127-0-0-1.nip.io:8080/color
```

**What's happening:**
- maxSurge: 25% = 3 new green pods created
- maxUnavailable: 25% = 2 old blue pods terminated
- Process repeats until all pods are green

### Step 4: Rolling Update to Red (Broken Probes)

Update to version with failing health probes to demonstrate automatic rollout pause:

```bash
# Update to red version (has broken probes)
kubectl set image deployment/demo-app demo-app=harbor.cloudpirates.io/training/rollouts-demo:red -n rolling-updates

# Watch rollout - will pause when probes fail
kubectl rollout status deployment/demo-app -n rolling-updates

# Check pods - some red pods will show as NotReady
kubectl get pods -n rolling-updates

# Check events to see probe failures
kubectl describe deployment/demo-app -n rolling-updates
```

**What's happening:**
- Red pods fail readiness probes
- Kubernetes stops the rollout automatically
- Green pods remain available, serving traffic
- No downtime despite broken deployment

### Step 5: Rollback to Previous Version

```bash
# Rollback to last working version (green)
kubectl rollout undo deployment/demo-app -n rolling-updates

# Watch rollback progress
kubectl rollout status deployment/demo-app -n rolling-updates

# Verify all pods are green again
kubectl get pods -n rolling-updates
```

### Step 6: Explore Rollout History

```bash
# View all revisions
kubectl rollout history deployment/demo-app -n rolling-updates

# Output example:
# REVISION  CHANGE-CAUSE
# 1         <none>
# 3         <none>
# 4         <none>

# Rollback to specific revision (if needed)
kubectl rollout undo deployment/demo-app --to-revision=2 -n rolling-updates
```

**Key Concepts:**
- **Revision**: Sequential number for each rollout (1, 2, 3, 4...)
- **Version**: Application version (blue, green, red...)
- One version can have multiple revisions (e.g., blue could be revision 1 and 4)
- `--to-revision` allows precise rollback to any previous state


Blue (Version 1) -> Green (Version 2) -> Red (Version 3) -> "Rollback"

Revision 1       ->       Revision 2  ->    Revision 3   -> "Rollback"
                          Revision 4             <-

---

## Part 2: Canary Deployments

### Step 7: Prepare for Canary Testing

Scale down to 5 replicas for clearer canary demonstration:

```bash
# Scale down V1 deployment
kubectl scale deployment/demo-app --replicas=5 -n rolling-updates

# Verify scaling
kubectl get pods -n rolling-updates
```

### Step 8: Deploy Canary Version (Purple)

Deploy V2 alongside V1:

```bash
# Deploy V2 deployment and service
kubectl apply -f canary/01-deployment-v2.yaml
kubectl apply -f canary/02-service-v2.yaml

# Verify both versions running
kubectl get deployments -n rolling-updates
kubectl get pods -n rolling-updates -l app=demo-app      # V1 (green) - 5 pods
kubectl get pods -n rolling-updates -l app=demo-app-v2   # V2 (purple) - 5 pods
```

### Step 9: Header-Based Canary Testing

Route specific requests to canary version using HTTP headers:

```bash
# Deploy header-based canary HTTPRoute
kubectl apply -f canary/03-header-match.yaml

# Verify httproutes
kubectl get httproute -n rolling-updates

# Test normal traffic (goes to V1/green)
curl http://rollingupdates.127-0-0-1.nip.io:8080/color
# Output: green

# Test canary traffic (goes to V2/purple)
curl -H "x-version: 2" http://rollingupdates.127-0-0-1.nip.io:8080/color
# Output: purple

# Open dashboard - normal traffic shows only green
# Add header in browser dev tools to see purple
```

**Use Case:** Beta testing, internal testing, specific user segments

### Step 10: Switch to Weight-Based Canary

Route percentage of traffic to canary version:

```bash
# Remove header-based canary route first
kubectl delete httproute/demo-app-header -n rolling-updates

# Apply weight-based canary HTTPRoute (replaces base demo-app route with 90/10 split)
kubectl apply -f canary/04-canary-weight.yaml

# Test traffic distribution
for i in {1..20}; do curl http://rollingupdates.127-0-0-1.nip.io:8080/color; echo ""; done

# Output: ~10% purple, ~90% green

# Check dashboard - see ~10% traffic to purple
```

**Progressive Rollout:**
```bash
# Increase canary to 50%
kubectl patch httproute demo-app -n rolling-updates \
  --type='json' -p='[{"op": "replace", "path": "/spec/rules/0/backendRefs/0/weight", "value": 50},{"op": "replace", "path": "/spec/rules/0/backendRefs/1/weight", "value": 50}]'

# Increase canary to 75%
kubectl patch httproute demo-app -n rolling-updates \
  --type='json' -p='[{"op": "replace", "path": "/spec/rules/0/backendRefs/0/weight", "value": 25},{"op": "replace", "path": "/spec/rules/0/backendRefs/1/weight", "value": 75}]'

# Complete rollout to 100% - switch main route to V2 only
kubectl patch httproute demo-app -n rolling-updates \
  --type='json' -p='[{"op": "replace", "path": "/spec/rules/0/backendRefs/0/name", "value": "demo-app-v2"},{"op": "remove", "path": "/spec/rules/0/backendRefs/1"}]'
```

---

## Blue/Green Deployment (Manual Demo)

Blue/Green is demonstrated manually by switching the main HTTPRoute backend:

```bash
# Current state: HTTPRoute points to demo-app (green)
kubectl get httproute/demo-app -n rolling-updates -o yaml | grep "name: demo-app"

# Switch to V2 (purple) instantly
kubectl patch httproute demo-app -n rolling-updates \
  --type='json' -p='[{"op": "replace", "path": "/spec/rules/0/backendRefs/0/name", "value": "demo-app-v2"}]'

# All traffic instantly switches to purple
# Check dashboard - immediate switch from green to purple

# Rollback if issues (instant)
kubectl patch httproute demo-app -n rolling-updates \
  --type='json' -p='[{"op": "replace", "path": "/spec/rules/0/backendRefs/0/name", "value": "demo-app"}]'
```

---

## Cleanup

```bash
# Delete all resources
kubectl delete namespace rolling-updates

# Or delete individually
kubectl delete -f rolling-updates/
kubectl delete -f canary/
```

## Additional Resources

- [Kubernetes Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Gateway API HTTPRoute Traffic Splitting](https://gateway-api.sigs.k8s.io/guides/traffic-splitting/)
- [Deployment Strategies Comparison](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/)
