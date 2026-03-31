# Agentic AI Manifests & Newsletter Drafter Demo

This repository contains the infrastructure manifests and the Quarkus Serverless Workflow application (`newsletter-drafter`) for the Agentic AI demo.

The infrastructure is pre-configured for Kubernetes (KIND recommended) and includes:
- **Redis**
- **Kafka** (single node, KRaft, PLAINTEXT, auto-creation enabled)
- **Prometheus**
- **Grafana** (integrated with Prometheus + preloaded custom dashboards)

## Prerequisites
* A running Kubernetes cluster (e.g., KIND running via Docker/Colima).
* `kubectl` CLI installed and configured.
* Java 17+ and Maven (to build the Quarkus application).

---

## 1. Deploy Infrastructure

First, spin up the backing services required by the application.

kubectl apply -f k8s/agentic-ai-manifests

Wait a moment for the infrastructure pods to initialize:
kubectl get pods -w

## 2. Configure Application Secrets

Before deploying the Quarkus application, you must provide your OpenAI API key. The application is configured to look for a Kubernetes Secret named `openai-credentials`.

Run this command, replacing `<YOUR_ACTUAL_KEY>` with your real OpenAI API key:

kubectl create secret generic openai-credentials --from-literal=api-key="<YOUR_ACTUAL_KEY>"

*(Note: The ConfigMap for Redis and Kafka connections is automatically generated and applied during the Quarkus build process).*

## 3. Build & Deploy the Application

Build the Quarkus application and deploy it to your cluster using the `kind` profile. This profile automatically bundles the Kubernetes manifests, generates the `infra-config` ConfigMap, and applies them to the cluster.

# From the root of your Quarkus project
./mvnw clean install -Dquarkus.profile=kind

## 4. Verify the Deployment

Check that all pods are in a `Running` state and that the application's health/readiness checks have passed:

kubectl get pods

Verify that the Durable Workflows lease mechanism is working correctly and a leader has been elected:

kubectl get lease

*You should see `flow-pool-leader-newsletter-drafter` and the associated member leases.*

---

## 5. Access Services (Port Forwarding)

If you are running this locally (e.g., on a Mac using Colima + KIND), you can access the services in your browser by forwarding the ports.

Open separate terminal tabs for the services you want to access:

### The Application (Newsletter Drafter)
kubectl port-forward svc/newsletter-drafter 8080:80

* **Access at:** http://localhost:8080

### Observability & Metrics
**Grafana:**
kubectl port-forward svc/grafana 3000:3000

* **Access at:** http://localhost:3000

**Prometheus:**
kubectl port-forward svc/prometheus 9090:9090

* **Access at:** http://localhost:9090

### Backend Infrastructure (For debugging)
**Redis:**
kubectl port-forward svc/redis 6379:6379

**Kafka:**
kubectl port-forward svc/kafka 9092:9092

*(Note: For in-cluster applications communicating with Kafka, use the bootstrap server: `kafka.default.svc.cluster.local:9092`)*

---

## 6. Teardown

To clean up your cluster, delete the application and the infrastructure manifests:

# Delete the application
kubectl delete -f target/kubernetes/kind.yml

# Delete the infrastructure
kubectl delete -f k8s/agentic-ai-manifests

# Clean up the secret
kubectl delete secret openai-credentials