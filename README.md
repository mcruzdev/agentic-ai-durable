# agentic-ai-durable

Agentic AI Durable is a Kubernetes-based application stack that combines the newsletter-drafter service with Redis and Kafka, plus Prometheus and Grafana for observability.

## Architecture

The application includes:
- **newsletter-drafter**: A Quarkus-based microservice (3 replicas)
- **Redis**: Standalone instance for caching and data storage
- **Kafka**: Single-node broker with topic auto-creation enabled
- **Prometheus**: Metrics collection with scrape config for Quarkus `/q/metrics`
- **Grafana**: Visualization with pre-provisioned datasource and dashboard

## Prerequisites

Before you begin, ensure you have the following tools installed:

- **KinD** (Kubernetes in Docker): https://kind.sigs.k8s.io/docs/user/quick-start/
- **kubectl**: https://kubernetes.io/docs/tasks/tools/
- **Helm** (v3+): https://helm.sh/docs/intro/install/
- **Docker**: https://docs.docker.com/get-docker/

Verify installations:
```bash
kind --version
kubectl version --client
helm version
docker --version
```

## Quick Start with KinD

### 1. Create a KinD Cluster

Create a local Kubernetes cluster:

```bash
kind create cluster --name agentic-ai
```

Verify the cluster is running:
```bash
kubectl cluster-info --context kind-agentic-ai
kubectl get nodes
```

### 2. Install Helm Dependencies

Navigate to the Helm chart directory and fetch dependencies:

```bash
cd k8s/agentic-ai-stack
helm dependency update
```

### 3. Install the Application Stack

Install the chart into your KinD cluster:

```bash
helm install agentic-ai . -n agentic-ai --create-namespace
```

This command:
- Creates the `agentic-ai` namespace
- Deploys Redis (1 standalone instance)
- Deploys Prometheus and Grafana
- Deploys Kafka (single node)
- Deploys the newsletter-drafter service (3 replicas)
- Creates all required RBAC resources (ServiceAccount, Role, RoleBinding)

### 4. Verify Installation

Check the status of all resources:

```bash
# List all pods in the agentic-ai namespace
kubectl get pods -n agentic-ai

# Wait for all pods to be ready
kubectl wait --for=condition=ready pod \
  -l app.kubernetes.io/name=newsletter-drafter \
  -n agentic-ai \
  --timeout=300s

# Check service status
kubectl get svc -n agentic-ai

# View resource overview
kubectl get all -n agentic-ai
```

### 5. Access the Application

#### Newsletter Drafter Service

Forward the newsletter-drafter service to localhost:

```bash
kubectl port-forward svc/newsletter-drafter 8080:80 -n agentic-ai
```

The service will be available at: `http://localhost:8080`

Health checks:
- Liveness: `http://localhost:8080/q/health/live`
- Readiness: `http://localhost:8080/q/health/ready`
- Startup: `http://localhost:8080/q/health/started`

#### Grafana Dashboard

Access Grafana for observability:

```bash
kubectl port-forward svc/agentic-ai-grafana 3000:80 -n agentic-ai
```

Navigate to: `http://localhost:3000`

The chart preconfigures:
- Prometheus datasource (auto-provisioned)
- `Newsletter Drafter Overview` dashboard (auto-imported)

Get Grafana admin password:

```bash
kubectl get secret agentic-ai-grafana -n agentic-ai -o jsonpath="{.data.admin-password}" | base64 --decode && echo
```

Check Prometheus target status:

```bash
kubectl port-forward svc/agentic-ai-prometheus-server 9090:80 -n agentic-ai
```

Then open `http://localhost:9090/targets` and verify the `newsletter-drafter` job is `UP`.

#### Redis

Connect to Redis for data operations:

```bash
kubectl port-forward svc/agentic-ai-redis 6379:6379 -n agentic-ai
```

Redis CLI connection:
```bash
redis-cli -h localhost -p 6379
```

### 6. View Logs

Stream logs from the newsletter-drafter deployment:

```bash
kubectl logs -f deployment/newsletter-drafter -n agentic-ai
kubectl logs -f deployment/newsletter-drafter -n agentic-ai --all-containers=true
```

View logs from a specific pod:
```bash
kubectl logs <pod-name> -n agentic-ai
```

### 7. Execute Commands in Pods

Access a pod's shell:

```bash
kubectl exec -it <pod-name> -n agentic-ai -- /bin/sh
```

Execute a single command:
```bash
kubectl exec <pod-name> -n agentic-ai -- command
```

## Configuration

### Customizing Deployment

You can override default values during installation:

```bash
helm install agentic-ai . \
  -n agentic-ai \
  --create-namespace \
  --set newsletterDrafter.replicaCount=5 \
  --set redis.architecture=replication
```

For more customization options, see `k8s/agentic-ai-stack/values.yaml`.

### Environment Variables

The newsletter-drafter service has the following key environment variables:
- `KUBERNETES_NAMESPACE`: Injected from pod metadata
- `QUARKUS_REDIS_HOSTS`: Redis connection string (auto-injected)
- `KAFKA_BOOTSTRAP_SERVERS`: Kafka bootstrap URL (auto-injected)
- `OPENAI_API_KEY`: Optional (value or secret reference)

## Cleanup

### Remove the Application Stack

```bash
helm uninstall agentic-ai -n agentic-ai
```

### Delete the KinD Cluster

```bash
kind delete cluster --name agentic-ai
```

## Troubleshooting

### Pods not starting

Check pod events:
```bash
kubectl describe pod <pod-name> -n agentic-ai
```

### View chart rendered manifests

Dry-run to see what will be deployed:
```bash
helm template agentic-ai . -n agentic-ai
```

### Check resource requests/limits

View pod resources:
```bash
kubectl top pods -n agentic-ai
```

### RBAC Issues

Verify ServiceAccount and permissions:
```bash
kubectl get serviceaccount -n agentic-ai
kubectl get role -n agentic-ai
kubectl get rolebinding -n agentic-ai
```

## Additional Resources

- [KinD Documentation](https://kind.sigs.k8s.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [Quarkus Documentation](https://quarkus.io/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/grafana/)