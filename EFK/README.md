1.Create Namespace
  kubectl create namespace logging

2.Install Elasticsearch on AKS
The main change here is the storageClassName. On Azure, managed-csi is the standard SSD-backed storage.

helm install elasticsearch --set replicas=1 --set volumeClaimTemplate.storageClassName=managed-csi --set persistence.labels.enabled=true  elastic/elasticsearch -n logging

4) Retrieve Credentials
The process for getting the password remains exactly the same as the AWS version:
bash
 # Get Username
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
# Get Password
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d

Powershell
 # Get Username
 $secret = kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}'
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret))
# Get Password
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}')))


5) Install Kibana
Azure will provision a Public IP for the Kibana service.

Bash
helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging
Tip: Use kubectl get svc -n logging -w to wait for the EXTERNAL-IP to appear.


6) Install Fluent Bit
Open your fluentbit-values.yaml.

Update the HTTP_Passwd with the password you retrieved in Step 4.

Deploy:

helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging
