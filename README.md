# Kubernetes Training Examples

Hands-on examples for the [CloudPirates Kubernetes introduction training](https://www.cloudpirates.io/solutions/training/kubernetes-introduction). They take you from your first Pod to rolling updates, canary releases and autoscaling, and stay useful as a reference after the training.

## How this repository works

- **Follow the numbers.** The directories `01_` to `10_` build on each other in the order of the training. Later examples assume you know the earlier ones; for example, autoscaling in `10_scaling/` needs the Metrics Server from `05_observability/`.
- **Read the README first.** Every directory has a README that explains the concepts and walks you through the exercise step by step.
- **Apply files in order.** Most manifests are numbered (`00-namespace.yaml`, `01-...`, `02-...`). Apply them one by one to see what each step changes, or apply a whole (sub)directory with `kubectl apply -f <directory>/`.
- **Read the comments.** The YAML files are commented inline, explaining what each field does and what to try next.
- **Break things.** Some examples are meant to fail, such as exceeding a memory limit or rolling out a version with failing probes, so you can see how Kubernetes reacts.

You need a Kubernetes cluster: use your managed training environment, or set up a local one as described in [Getting Started](#getting-started).

## Directory Structure

### [00_the-hard-way/](00_the-hard-way/)
**Kubernetes The Hard Way** - Manual cluster setup from scratch without automation tools.
- Step-by-step guide for building a Kubernetes cluster manually
- Understanding core components: etcd, API server, controller manager, scheduler, kubelet
- Only used during the training as example files
- See [README.md](00_the-hard-way/README.md)

### [01_basics/](01_basics/)
**Kubernetes Fundamentals** - Core building blocks and pod patterns.
- Basic Pod definition and configuration
- Init Containers - Initialization tasks before main container starts
- Sidecar Containers - Helper containers running alongside main application
- Resource limits and requests explanation

### [02_workload_resources/](02_workload_resources/)
**Workload Resources** - Different types of workload controllers for various use cases.
- **Deployment** - Stateless applications with rolling updates and scaling
- **ReplicaSet** - Maintains specified number of pod replicas
- **StatefulSet** - Stateful applications with persistent storage (Redis example with volumeClaimTemplates)
- **DaemonSet** - Runs pods on all/selected nodes (Prometheus node exporter example)
- **Job** - One-time batch processing tasks
- **CronJob** - Scheduled recurring tasks

### [03_probes/](03_probes/)
**Health Checks** - Pod health monitoring and lifecycle management.
- **HTTP Probes** - Health checks via HTTP endpoints (startup, readiness, liveness)
- **TCP Probes** - TCP socket connection checks (Redis example)
- **Exec Probes** - Command execution health checks (PostgreSQL example)
- Best practices for probe configuration

### [04_resources/](04_resources/)
**Resource Management** - CPU and memory resource control.
- **QoS Classes** - Guaranteed, Burstable, BestEffort quality of service
- **CPU Management** - CPU requests and limits
- **Memory Management** - Memory requests and limits
- Impact on pod scheduling and eviction

### [05_observability/](05_observability/)
**Metrics and Monitoring** - Cluster and application observability.
- Metrics Server installation and configuration
- `kubectl top nodes` - Node resource usage
- `kubectl top pods` - Pod resource consumption
- Foundation for autoscaling and monitoring

### [06_config-management/](06_config-management/)
**Configuration and Secrets** - Application configuration and sensitive data management.
- **ConfigMaps** - Non-sensitive configuration data (environment variables, volume mounts)
- **Secrets** - Sensitive data storage (base64 encoded, stringData)
- **PodDisruptionBudgets** - Maintain availability during voluntary disruptions
- **SealedSecrets** - GitOps-safe encrypted secrets with comprehensive guide

### [07_storage/](07_storage/)
**Persistent Storage** - Data persistence and volume management.
- **Special Volumes** - emptyDir (ephemeral), hostPath (node filesystem)
- **Manual Provisioning** - PersistentVolume and PersistentVolumeClaim workflow
- **Dynamic Provisioning** - Automatic volume provisioning with StorageClasses

### [08_networking/](08_networking/)
**Networking and Gateway API** - Service discovery and external access.
- **Service Types**
  - ClusterIP - Internal cluster communication
  - NodePort - External access via node IP and port
  - LoadBalancer - Cloud/on-premises load balancer integration
  - ExternalName - DNS CNAME record for external services
- **Gateway API** - HTTP routing with kgateway (path-based and virtual host-based)
  - Shared public Gateway (`public-gateway.yaml`) exposed via NodePort on port 8080
  - HTTPRoutes attach to the Gateway from any namespace

### [09_rolling-updates/](09_rolling-updates/)
**Deployment Strategies** - Safe application updates and traffic management.
- **Rolling Updates** - Kubernetes native zero-downtime deployments
  - Update images with `kubectl set image`
  - Rollout history and revision management
  - Rollback to previous versions
  - Demo with failing probes to show automatic rollback protection
- **Canary Deployments** - Gradual rollout with traffic splitting
  - Header-based routing (route specific users to new version)
  - Weight-based routing (percentage-based traffic split)
  - HTTPRoute-based traffic splitting via the Gateway API
- Demo application with `/color` endpoint and interactive dashboard on port 8080

### [10_scaling/](10_scaling/)
**Autoscaling** - Scale workloads automatically based on load.
- **HorizontalPodAutoscaler** - CPU-based scaling (1-10 replicas at 50% utilization)
- Example Deployment and Service to scale
- k6 load test script to generate traffic
- Requires the Metrics Server from `05_observability/`

## Getting Started

### Prerequisites
- A Kubernetes cluster: either your managed training environment or a local kind cluster (see below)
- kubectl CLI tool installed
- Basic understanding of containers and YAML

### Local Demo Cluster

To run the examples locally, set up a kind cluster with our training setup: [CloudPirates-io/kind-cluster-training-setup](https://github.com/CloudPirates-io/kind-cluster-training-setup). Follow the instructions in that repository.

### Usage

Clone this repository:

```bash
git clone https://github.com/CloudPirates-io/kubernetes-training-examples.git
cd kubernetes-training-examples
```

Each directory contains ready-to-use YAML manifests. Apply them to your cluster:

```bash
# Create namespace
kubectl apply -f <directory>/00-namespace.yaml

# Apply all resources in a directory
kubectl apply -f <directory>/

# Watch resources being created
kubectl get all -n <namespace> --watch

# Clean up
kubectl delete namespace <namespace>
```

### Namespace Convention

Most directories follow a consistent naming pattern where **namespace = directory name**:
- `01_basics/` → namespace: `default`
- `02_workload_resources/` → namespace: `workload-resources`
- `03_probes/` → namespace: `probes`
- `04_resources/` → namespace: `resources`
- `06_config-management/` → namespace: `config-management`
- `07_storage/` → namespace: `storage`
- `08_networking/` → namespace: `networking`
- `09_rolling-updates/` → namespace: `rolling-updates`
- `10_scaling/` → namespace: `scaling`

Exceptions: `04_resources/qos/` uses `qos-example`, `06_config-management/pdb/` uses `pdb-examples` and `08_networking/gateway/` uses `gateway-api`.
