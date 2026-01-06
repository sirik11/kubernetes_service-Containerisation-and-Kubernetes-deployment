# Kubernetes ML Inference Service

This directory contains artifacts to containerise the ONNX inference engine and deploy it on Kubernetes.  It supports auto‑scaling via the HorizontalPodAutoscaler, readiness/liveness probes and Prometheus/Grafana monitoring.

## Dockerfile

A multi‑stage Dockerfile builds the C++ inference binary from source and produces a minimal runtime image.  Replace the `model.onnx` file with your quantised model.  To build locally and push to a registry:

```bash
cd kubernetes_service
# copy or build your inference binary into onnx_inference_engine
# Build the image
podman build -t yourrepo/inference-service:latest .
# Push to your registry (DockerHub/Quay/ECR)
podman push yourrepo/inference-service:latest
```

The runtime image expects to find `/models/model.onnx` and exposes metrics on port 8080.

## Kubernetes manifests

- **deployment.yaml** – defines a `Deployment` running the inference service.  It includes resource requests/limits, environment variables and health probes.  Liveness probes restart the container if it deadlocks, readiness probes prevent traffic until the model is ready, and startup probes disable other probes until the application has started【784770933933275†L873-L907】.
- **service.yaml** – exposes the Deployment via a `Service`.  Change the type to `LoadBalancer` or `NodePort` for external access.
- **hpa.yaml** – configures a HorizontalPodAutoscaler to scale replicas between 1 and 5 based on CPU utilisation.  The scaling formula used by the controller is \(\text{desired replicas} = \frac{\text{current util}}{\text{target util}} \times \text{current replicas}\)【419351093937699†L286-L307】.
- **grafana_dashboard.json** – a Grafana dashboard definition with panels for latency, throughput, replica count and node pressure.  Node pressure metrics use PSI values (`psi_cpu_some_avg10`, `psi_memory_some_avg10`) to measure how often tasks are stalled due to CPU or memory contention【756160371554875†L902-L917】.

Deploy the manifests:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

Ensure the **metrics server** or a **Prometheus adapter** is installed in your cluster so that the HPA can read CPU metrics.  Then apply the dashboard JSON to Grafana.

## Probes and metrics

You may need to implement a `/healthz` endpoint in your application.  In the provided C++ sample, metrics are exposed at `/metrics` on port 8080.  You can update the liveness and readiness probes to use `/metrics` or implement a separate health endpoint.

To visualise metrics, deploy Prometheus and Grafana (e.g. using the [`kube-prometheus-stack`](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) Helm chart) and import `grafana_dashboard.json`.

