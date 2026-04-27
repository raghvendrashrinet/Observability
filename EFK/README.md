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

### Deploy app on aks to monitor
   Use log-Generator manifest in this directory 
    $kubectl apply -f log-generator.yaml

To check your logs in Kibana on Azure, you need to follow these three steps to tell Kibana where to find the data that Fluent Bit has been sending to Elasticsearch.

1. Access the Kibana UI
First, get the URL for your Kibana instance. Since you used type: LoadBalancer on Azure, it will have a Public IP.

Bash
kubectl get svc kibana -n logging
Copy the EXTERNAL-IP and open it in your browser at port 5601 (e.g., http://20.xx.xx.xx:5601).

2. Create a "Data View" (Index Pattern)
Kibana needs to know which "index" (folder) in Elasticsearch to look at. Fluent Bit usually creates indices starting with fluent-bit- or logstash-.

Log in using the credentials you retrieved earlier (Username: elastic).

Click the Hamburger Menu (☰) in the top left.

Scroll down to Management and click Stack Management.

On the left sidebar, click Data Views (in older versions, this is called Index Patterns).

Click Create data view.

Name/Index Pattern: Enter fluent-bit-* or simply *.

Tip: You should see a success message on the right saying "The index pattern matches 1 source." If you don't see this, Fluent Bit hasn't successfully sent any logs yet.

Timestamp field: Select @timestamp from the dropdown.

Click Save data view to Kibana.

3. View the Logs in "Discover"
Now that the connection is made, you can actually see the data:

Click the Hamburger Menu (☰) again.

Click Discover (usually the very first option under Analytics).

Select the Data View you just created from the dropdown on the left.

Search for your App: In the search bar at the top, type:
kubernetes.labels.app : "log-generator"

You should now see the random logs from your test container appearing in the list.

🛠️ Troubleshooting: "No results match your search"
If you don't see logs:

Check Time Range: Ensure the time picker (top right) is set to "Last 15 minutes."

Check Fluent Bit Logs: Run kubectl logs -n logging -l app.kubernetes.io/name=fluent-bit and look for errors. If it says 401 Unauthorized, your password in the fluentbit-values.yaml is wrong.

Check Elasticsearch: Run kubectl get pods -n logging to make sure the Elasticsearch pod is "Running" and not "Pending" (which usually means Azure is still creating the disk).

Once you see the logs, would you like to know how to create a simple bar chart to visualize which pods are sending the most errors?
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging



1. Create the Data View
In Kibana, when it asks for the Index Pattern, type this exactly:

logstash-*

Why the asterisk? Logs are created daily (e.g., logstash-2026.04.27, logstash-2026.04.28). The * tells Kibana to look at all of them as one single stream of data.







If you see logstash-2026.04.27 in your list of indices, congratulations! This means the entire pipeline is working perfectly.

The reason it says "Logstash" instead of "Fluent Bit" is that Fluent Bit, by default, uses the Logstash-compatible output format to make it easier for Kibana to understand.



2. Locate your "log-generator" Data
Once the Data View is saved, go to the Discover tab. You are going to see a "firehose" of data from the entire cluster. To find only your specific test app, use the KQL (Kibana Query Language) search bar at the top:

Plaintext
kubernetes.labels.app : "log-generator"
3. What to look for in the "node" data
Since you mentioned "node," you can now see exactly which Azure VM node your pod is running on.

In the left-hand sidebar of the Discover page, search for the field kubernetes.host.

Click the + (plus) icon next to it.

Kibana will add a column showing you the internal Azure private IP or hostname of the node.

🚀 Your First "Analysis" Task
Since you have the logs flowing, try this to see the power of the EFK stack:

In the search bar, type level : "ERROR".

Hit Enter.

Kibana will instantly filter thousands of logs to show you only the simulated "Errors" from your log-generator.

How does it feel to see your first live logs on Azure? Would you like to try making a pie chart of the log levels (Info vs Error) next?
