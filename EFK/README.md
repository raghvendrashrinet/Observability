1.Create Namespace
  kubectl create namespace logging  

  Note : increase cluster capacity  as more capacity is required :  
         az aks scale --resource-group "rg1" --name "aks-solar-system" --node-count 2

2.Install Elasticsearch on AKS  
 - [Elastic Search Documentation](https://www.elastic.co/blog/getting-started-with-the-azure-integration-enhancement)  
 - [ElasticSearch and Kibana Installation on Azure](https://www.elastic.co/blog/elasticsearch-and-kibana-deployments-on-azure)  
 - [Elastic Stack Helm Chart](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/managing-deployments-using-helm-chart)
 
The main change here is the storageClassName. On Azure, managed-csi is the standard SSD-backed storage.
```
helm install elasticsearch --set replicas=1 --set volumeClaimTemplate.storageClassName=managed-csi --set persistence.labels.enabled=true  elastic/elasticsearch -n logging
```
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
  output :  
       1. Watch all containers come up.
         $ kubectl get pods --namespace=logging -l release=kibana -w
       2. Retrieve the elastic user's password.
         $ kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
       3. Retrieve the kibana service account token.
         $ kubectl get secrets --namespace=logging kibana-kibana-es-token -ojsonpath='{.data.token}' | base64 -d

6) Install Fluent Bit
Open your fluentbit-values.yaml.

Update the HTTP_Passwd with the password you retrieved in Step 4.

Deploy:

helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging


Access Kibana: Open your browser and go to http://<AZURE_PUBLIC_IP>:5601.

Azure Storage: You can verify the persistent volumes created by Elasticsearch in the Azure Portal under your MC_ resource group (the infrastructure group for your AKS nodes).

### Deploy app onaks to monitor

helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging
