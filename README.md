# kubernetes_service-Containerisation-and-Kubernetes-deployment

Purpose: This directory shows how to package the inference engine into a Docker image, define Kubernetes resources and provide a Grafana dashboard for observability.
Key files and their roles:
Dockerfile – Multi‑stage build:
First stage installs build tools, copies the onnx_inference_engine source and uses CMake to compile the binary.
Second stage creates a small runtime image and copies the inference binary and your model.onnx into /models/. It exposes port 8080 for metrics.
deployment.yaml – A Deployment manifest:
Runs the container with resource requests and limits.
Sets environment variables (e.g. MODEL_PATH), and defines liveness, readiness and startup probes. Liveness probes restart the container if it deadlocks, readiness probes remove it from service until the model is ready, and startup probes delay other checks until the application has started
kubernetes.io
.
You can adjust the probes’ path to /metrics or implement a /healthz endpoint.
service.yaml – Exposes the deployment via a ClusterIP service on port 80. Change the type to LoadBalancer or NodePort for external access.
hpa.yaml – Defines a HorizontalPodAutoscaler with minReplicas: 1, maxReplicas: 5 and a CPU target of 60 %. The controller uses the formula desiredReplicas = (currentUtil ÷ targetUtil) × currentReplicas
komodor.com
 to scale.
grafana_dashboard.json – Provides a Grafana dashboard with four panels: latency quantiles, throughput, replica count and node pressure (PSI). PSI metrics such as psi_cpu_some_avg10 and psi_memory_some_avg10 show how often tasks are stalled due to CPU or memory contention
kubernetes.io
.
README.md – Explains:
How to build and push the Docker image.
What each manifest does and how to apply them (kubectl apply -f ...).
Why a metrics server or Prometheus adapter is needed for the HPA to function.
How to load the Grafana dashboard and interpret the panels.
Getting started:
Make sure Docker or Podman is installed and that you have access to a Kubernetes cluster.
Build and push the image:
cd projects/kubernetes_service
podman build -t yourrepo/inference-service:latest .
podman push yourrepo/inference-service:latest
Apply the manifests:
kubectl apply -f deployment.yaml -f service.yaml -f hpa.yaml
Install Prometheus and Grafana (e.g. using the kube-prometheus-stack Helm chart) and import grafana_dashboard.json to visualise latency, throughput and node pressure.
