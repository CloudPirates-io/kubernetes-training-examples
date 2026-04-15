# Kubernetes Networking

This directory demonstrates Kubernetes networking concepts: Service types and the Gateway API for HTTP routing.

## Directory Structure

### basics/
Service types and their use cases using an nginx deployment.
- **ClusterIP**: Internal cluster communication via DNS name
- **NodePort**: External access via static port on each node
- **LoadBalancer**: External access with cloud/on-premises load balancer
- **ExternalName**: CNAME mapping to external services

### gateway/
Gateway API HTTP routing with kgateway.
- **Path-Based Routing**: Different URL paths → same or different backends
- **Host-Based Routing**: Different hostnames → different backends (virtual hosting)

---

## Setup: Install kgateway

> Skip this if kgateway is already running in the cluster.

### TL/DR

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/experimental-install.yaml
helm upgrade -i kgateway-crds oci://cr.kgateway.dev/kgateway-dev/charts/kgateway-crds \
  --create-namespace --namespace kgateway-system \
  --version v2.3.0-main \
  --set controller.image.pullPolicy=Always
helm upgrade -i kgateway oci://cr.kgateway.dev/kgateway-dev/charts/kgateway \
  --namespace kgateway-system \
  --version v2.3.0-main \
  --set controller.image.pullPolicy=Always \
  --set controller.extraEnv.KGW_ENABLE_GATEWAY_API_EXPERIMENTAL_FEATURES=true
```


## 1. Install Gateway API CRDs

The experimental channel is required for some Gateway API features used in these guides.

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/experimental-install.yaml
```

## 2. Install kgateway CRDs

```bash
helm upgrade -i kgateway-crds oci://cr.kgateway.dev/kgateway-dev/charts/kgateway-crds \
  --create-namespace --namespace kgateway-system \
  --version v2.3.0-main \
  --set controller.image.pullPolicy=Always
```

## 3. Install kgateway

The `KGW_ENABLE_GATEWAY_API_EXPERIMENTAL_FEATURES` flag enables experimental Gateway API features.

```bash
helm upgrade -i kgateway oci://cr.kgateway.dev/kgateway-dev/charts/kgateway \
  --namespace kgateway-system \
  --version v2.3.0-main \
  --set controller.image.pullPolicy=Always \
  --set controller.extraEnv.KGW_ENABLE_GATEWAY_API_EXPERIMENTAL_FEATURES=true
```

## 4. Verify the installation

```bash
kubectl get pods -n kgateway-system
```

Expected output:

```
NAME                        READY   STATUS    RESTARTS   AGE
kgateway-5495d98459-46dpk   1/1     Running   0          19s
```

---

## Part 1: Service Types

### Step 1: Deploy the namespace and nginx

```bash
kubectl apply -f basics/00-namespace.yaml
kubectl apply -f basics/01-nginx-deployment.yaml

# Verify
kubectl get pods -n networking
```

### Step 2: ClusterIP (internal access)

```bash
kubectl apply -f basics/02-clusterip-service.yaml

# Verify the service has a cluster IP
kubectl get service nginx-clusterip -n networking

# Access from inside the cluster (e.g., via a temporary pod)
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -n networking -- \
  curl http://nginx-clusterip.networking.svc.cluster.local
```

**Key point:** ClusterIP is only reachable from within the cluster. DNS name: `<service>.<namespace>.svc.cluster.local`

### Step 3: NodePort (external access without load balancer)

```bash
kubectl apply -f basics/03-nodeport-service.yaml

# Verify the NodePort is assigned
kubectl get service nginx-nodeport -n networking

# Access from outside the cluster
curl http://<node-ip>:32730
```

**Key point:** NodePort opens the same port on every node. Range: 30000–32767.

### Step 4: LoadBalancer (cloud/on-premises load balancer)

```bash
kubectl apply -f basics/04-loadbalancer-service.yaml

# Watch for EXTERNAL-IP to be assigned (cloud only)
kubectl get service nginx-loadbalancer -n networking -w
```

**Key point:** On bare-metal clusters without a LoadBalancer controller (MetalLB, Cilium, etc.), `EXTERNAL-IP` stays `<pending>`.

### Step 5: ExternalName (DNS alias to external service)

```bash
kubectl apply -f basics/05-external-service.yaml

# Verify
kubectl get service external-database -n networking

# From inside the cluster, this resolves as a CNAME to example.com
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -n networking -- \
  curl http://external-database.networking.svc.cluster.local
```

**Key point:** No pods are selected — Kubernetes DNS returns a CNAME record. No proxying occurs.

---

## Part 2: Gateway API Routing

### Step 1: Deploy the Gateway

```bash
kubectl apply -f gateway.yaml

# Verify the Gateway is accepted and its Service is a NodePort
kubectl get gateway -n kgateway-system
kubectl get service -n kgateway-system
```

Expected Gateway status: `ACCEPTED` / `PROGRAMMED`

### Step 2: Deploy the echoserver

```bash
kubectl apply -f gateway/00-namespace.yaml
kubectl apply -f gateway/01-echoserver.yaml

# Verify
kubectl get pods -n gateway-api
kubectl get service -n gateway-api
```

The echo server returns the full request (headers, path, body) — useful for confirming which route matched.

### Step 3: Path-Based Routing

```bash
kubectl apply -f gateway/02-path-match.yaml

# Verify the HTTPRoute is attached to the Gateway
kubectl get httproute -n gateway-api

# Test: request with matching path
curl http://127.0.0.1:30080/echoserver
# Returns echo server response

# Test: request with non-matching path
curl http://127.0.0.1:30080/other
# Returns 404 - no route matches
```

**What's happening:** The HTTPRoute matches any request whose path starts with `/echoserver` and forwards it to the echoserver service.

### Step 4: Host-Based Routing (Virtual Hosting)

```bash
kubectl apply -f gateway/03-host-match.yaml

# Verify both routes
kubectl get httproute -n gateway-api

# Test: request with matching hostname
curl http://echoserver.127-0-0-1.nip.io:30080/
# Returns echo server response

# Test: request without matching hostname
curl http://127.0.0.1:30080/
# Returns 404 - hostname does not match
```

**What's happening:** The HTTPRoute matches only requests for `echoserver.127-0-0-1.nip.io`. nip.io provides wildcard DNS — `127-0-0-1.nip.io` always resolves to `127.0.0.1`.

> For managed training environments: replace `127-0-0-1.nip.io` with your cluster hostname (e.g., `clusterXX.training.cloudpirates.io`).

---

## Cleanup

```bash
# Remove gateway examples
kubectl delete -f gateway/
kubectl delete -f gateway.yaml

# Remove service type examples
kubectl delete namespace networking
```
