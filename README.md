# AKS Cluster Setup with Prometheus and Grafana 

This guide provides a complete CLI command sequence to create an AKS cluster and deploy Prometheus and Grafana for monitoring.

## 1. Create AKS Cluster

| Bash | PowerShell | CMD |
|------|------------|-----|
| RESOURCE_GROUP="myResourceGroup"<br>AKS_NAME="myAKSCluster"<br>LOCATION="eastus" | $RESOURCE_GROUP = "myResourceGroup"<br>$AKS_NAME = "myAKSCluster"<br>$LOCATION = "eastus" | set RESOURCE_GROUP=myResourceGroup<br>set AKS_NAME=myAKSCluster<br>set LOCATION=eastus |

```
# Create resource group
az group create --name myResourceGroup --location eastus       -- Bash
az group create --name $RESOURCE_GROUP --location $LOCATION    -- Powershell
az group create --name %RESOURCE_GROUP% --location %LOCATION%  -- CMD


# Create AKS cluster
az aks create --resource-group $RESOURCE_GROUP --name $AKS_NAME --node-count 1 --node-vm-size Standard_B2s --generate-ssh-keys


# Get cluster credentials
az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME
```

## 2. Install Prometheus and Grafana

```bash
# Add Helm repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
 Generates YAML and Applies it safely Idempotent command,Works if namespace exists or not

# Install kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
  ✅ This deploys Prometheus, Grafana, Alertmanager, Node Exporter, and kube-state-metrics in the monitoring namespace.

Pods created:   kubectl get pods -n monitoring

Example output:
   prometheus-kube-prometheus-prometheus
   prometheus-grafana
   prometheus-node-exporter
   prometheus-kube-state-metrics
Architecture Created
 Kubernetes Cluster
        │
        ▼
 Node Exporter (node metrics)
        │
        ▼
 Prometheus (scrapes metrics)
        │
        ▼
 Grafana (visual dashboards)


```
## 3. Access Grafana      

```bash
# Get admin password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
 username: admin
 password: <decoded above>
Powershell
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" |
% { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
# Port-forward to access Grafana Dashboard , To access from your system
kubectl port-forward -n monitoring svc/prometheus-grafana 8080:80

# For production deployment
   Internet
     │
     ▼
  Ingress Controller
     │
     ▼
  Grafana Service

