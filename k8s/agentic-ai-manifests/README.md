# Agentic AI Manifests (No Helm)

This directory contains plain Kubernetes manifests (Kustomize) for:
- Redis
- Kafka (single node, KRaft, PLAINTEXT, topic auto-creation enabled)
- Prometheus
- Grafana integrated with Prometheus
- Preloaded Grafana custom dashboard

## Deploy

```bash
kubectl apply -k k8s/agentic-ai-manifests
```

## Verify

```bash
kubectl get pods -n agentic-ai
kubectl get svc -n agentic-ai
```

## Access services

Grafana:

```bash
kubectl port-forward -n agentic-ai svc/grafana 3000:3000
```

Prometheus:

```bash
kubectl port-forward -n agentic-ai svc/prometheus 9090:9090
```

Redis:

```bash
kubectl port-forward -n agentic-ai svc/redis 6379:6379
```

Kafka:

```bash
kubectl port-forward -n agentic-ai svc/kafka 9092:9092
```

## Kafka bootstrap for applications in-cluster

Use:

```text
kafka.agentic-ai.svc.cluster.local:9092
```

## Delete

```bash
kubectl delete -k k8s/agentic-ai-manifests
```
