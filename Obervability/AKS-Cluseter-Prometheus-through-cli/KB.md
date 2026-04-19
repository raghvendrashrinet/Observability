## In the kube-prometheus-stack setup:  

1. ### Host/Node metrics: Node Exporter –
    ``` deployed as a DaemonSet, collects CPU, memory, disk, and network usage from each AKS node. ```

 
2. ### Kubernetes infrastructure (Pods, Nodes, Deployments, etc.):
   ```kube-state-metrics – queries the Kubernetes API and exposes metrics about object states (e.g., pod status, resource requests). ```
 
3. ### Applications hosted on the cluster: Custom application exporters
   ``` (e.g., Prometheus client libraries in your app) or sidecar exporters. Prometheus scrapes metrics from the /metrics endpoint exposed by your application if a ServiceMonitor is configured. ```

kube-prometheus-stack, which automatically installs:
  Prometheus
  Grafana
  Alertmanager
  Node Exporter
  kube-state-metrics

So the metrics are already being collected.kubectl port-forward -n monitoring svc/prometheus-grafana 8080:80
Step 1 — Open Grafana
 Open in browser: http://localhost:8080
Step 2 — Check Data Source
 Inside Grafana: Connections → Data Sources
 You should see: Prometheus
 Click it → Test connection
If it says Data source is working, everything is fine.

Step 3 — Open Prebuilt Dashboards

The Helm chart already installs many dashboards.

Go to:

Dashboards → Browse

Step 3 — Open Prebuilt Dashboards

The Helm chart already installs many dashboards.

Go to:

Dashboards → Browse

You will see folders like:

Kubernetes / Compute Resources
Kubernetes / Networking
Kubernetes / API server
Nodes
Pods

Important dashboards:

Kubernetes / Compute Resources / Cluster
Kubernetes / Compute Resources / Node
Kubernetes / Compute Resources / Pod

Step 4 — View Node Metrics (Node Exporter)

Open dashboard:

Nodes

Now you will see metrics coming from:

Node Exporter

Metrics include:

CPU usage
Memory usage
Disk usage
Network traffic
Load average

Example query inside Grafana panel:

node_cpu_seconds_total

This is the node exporter metric.

Step 5 — View Kubernetes Metrics

Open dashboard:

Kubernetes / Compute Resources / Pod

Metrics come from:

kube-state-metrics

You will see:

Pod CPU usage
Pod memory usage
Pod restarts
Container usage

Example metric:

kube_pod_container_status_restarts_total
Step 6 — Verify Metrics Directly in Prometheus

You can also check raw metrics.

Run:

kubectl port-forward svc/prometheus-kube-prometheus-prometheus -n monitoring 9090

Open:

http://localhost:9090

Now in Prometheus search box try:

node_cpu_seconds_total

Step 7 — Understand Metric Flow

Your cluster currently works like this:

Kubernetes Node
      │
      ▼
Node Exporter
      │
      ▼
Prometheus (scrapes metrics)
      │
      ▼
Grafana (visualizes)

And for Kubernetes objects:

Kubernetes API
      │
      ▼
kube-state-metrics
      │
      ▼
Prometheus
      │
      ▼
Grafana
