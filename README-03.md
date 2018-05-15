
### Deploy Grafana docker image to Kubernets cluster

It is very short manual, non configurable how to in this part.

```
kubectl run --image=grafana/grafana grafana --port 3000
kubectl expose deployment grafana --port 3000 --name=grafana --type=NodePort
```

To access Grafana on `<public master node ip>:30002` the noePort should be set expilicitely
```
kubectl -n default edit service grafana
```
add parameter spec:ports:nodePort: 30002 to config and save.

Login to Grafana. (admin:admin)

Create a Grafana datasource with this settings:

name: DS_PROMETHEUS
type: prometheus
url: http://<public master node ip>:30002
access: direct


Import [Kubernetes cluster monitoring (via Prometheus)](https://grafana.com/api/dashboards/315/revisions/3/download) manually
or use curl

```
curl 'http://admin:admin@<public master node ip>:30002/api/dashboards/import' -H 'Content-Type: application/json;charset=UTF-8' '@conf/grafana-dashboard.json'
```

And enjoy the pics :)

**Sources:**
1. [Example Prometheus Monitoring](https://github.com/RisingStack/example-prometheus-nodejs)
2. [Kubernetes cluster monitoring (via Prometheus)](https://grafana.com/dashboards/315)
3. [grafana dashboard json](https://grafana.com/api/dashboards/315/revisions/3/download)
4. [Grafana Labs: Export and Import](http://docs.grafana.org/reference/export_import/)
