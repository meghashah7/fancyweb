# fancyweb
fancyweb

Pre requisite:

On the laptop a single node cluster was set up using kind.
the configuration file for kind is : cluster.yaml
Important commands:
kind create cluster --config=config.yaml

Installed kubectl
Installed metrics-server for the hpa to work
command: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


Next run
kubectl create -f fancweb.yaml

fancyweb.yaml has a deployment which uses the image fancyweb:latest
This image was built using main.go.
Corresponding Dockerfiles can be found in fancyweboperator repository.

Then a service called fancyweb-server of type NodePort is used to expose the deployment.
This service can be accessed at http://NODEIP:PORT on the laptop:
We can get the Node IP using the below command:
kubectl get nodes -o wide
We can get the PORT using the below command
kubectl get svc -o wide

Now in order for the HPA to work we need metrics-server to be deployed, which can be done using components.yaml
Next in order to support Auto Scaling HPA called fancyweb-operator is created.

The load testing can be done for autoscaling using the below command 

kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://nodeip:serviceport; done"

PROMETHEUS and GRAFANA setup:
I used helm to install prometheus and grafana
Commands:

helm repo update
helm install prometheus stable/prometheus-operator --namespace monitoring


Next in order for prometheus to scrape the fancyweb-server, I created a ServiceMonitor with the correct labels: servicemonitor.yaml

Once all this is done I did port-forwarding to access prometheus as well as grafana:
kubectl port-forward pod/prometheus-prometheusoperator-kube-pr-prometheus-0 9090:9090
kubectl port-forward -n monitoring pod/prometheus-grafana-5dbdf669c8-86tkb 3000:3000

I could access the following endpoints:

http://NodeIP:30000 : fancyweb page 
http://NodeIP:30000/prometheus : the metrics page which was exposed by the main.go for prometheus scraping
http://localhost9090 : prometheus application
http://localhost:3000 : Grafana application

In order to get the credential for grafana, I had to do kubectl get secret -n monitoring and identify the grafana userid and password which was in base64 encoded format.

From prometheus I used the query http_requests_total which gave  me the total number of requests served by the fancyweb-server
From grafana, I have set up the datasource as prometheus and then I created the new dashboard with new panel and PromQL http_requests_total, it gave me a graph of total requests served in real time.
I had setup the alerts in alert page on the same dashboard. ( I have not tested the alerts as I did not configure SMTP)
In order to see the number of pods of fanyweb-server running, I browsed to the built-in Dashboard : Namespace(Workload)



