# Newsletter Drafter

Quarkus service that drafts and reviews newsletters using an agentic workflow.

## Prerequisites

- Java 21+
- Maven (or use `./mvnw`)
- kubectl

## Run locally (dev mode)

From this newsletter-drafter directory:

```bash
./mvnw quarkus:dev
```

Useful local endpoints:

- API: <http://localhost:8080>
- Quarkus Dev UI: <http://localhost:8080/q/dev/>
- Health: <http://localhost:8080/q/health>
- Metrics: <http://localhost:8080/q/metrics>


## Building Kubernetes manifests

This command only build Kubernetes manifests, it does not deploy the image to Kubernetes cluster.

```shell
./mvnw clean package -Dquarkus.kubernetes.deploy=false
```

## Building and pushing the image

This command:

1. Build the application
2. Push the container image to a container registry 
3. And finally deploy your application into the Kubernetes cluster.

```shell
./mvnw clean package -Pkind -Dquarkus.container-image.push=true -Dquarkus.container-image.image=<username>/newsletter-drafter:latest
```

## Kubernetes deployment (manifests only)

---
**NOTE**

Before deploying, edit `/k8s/agentic-ai-manifests/openai-secret.yaml` and set put `api-key` value.

---

This project uses plain Kubernetes manifests.

The newsletter service image is already defined in the manifests. In normal usage, you only need to apply the manifests.

Deploy all infrastructure and app resources:

```bash
kubectl apply -f /k8s/agentic-ai-manifests
```

Verify:

```bash
kubectl get pods
kubectl get svc
```

Port-forward:

```bash
kubectl port-forward -n default svc/durable-flow-demo 8080:80
```

Optional port-forward commands for observability and dependencies:

```bash
kubectl port-forward -n default svc/grafana 3000:3000
kubectl port-forward -n default svc/prometheus 9090:9090
kubectl port-forward -n default svc/redis 6379:6379
kubectl port-forward -n default svc/kafka 9092:9092
```

Remove resources:

```bash
kubectl delete -f /k8s/agentic-ai-manifests
```

## Notes

- For production, do not keep secrets in manifests. Use Kubernetes Secrets for values like `OPENAI_API_KEY`.
- Service and dependency manifests live in `k8s/agentic-ai-manifests`.
