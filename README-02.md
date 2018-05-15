
### Deploy Prometheus configurated to monitor Kubernets

All entities will be imported into `default` namespace

Phrometheus will be accessable on <public master node ip>:30001

```
kubectl create -f conf/prometheus-cluster-role.yaml
kubectl create -f conf/prometheus-config-map.yaml
kubectl create -f conf/prometheus-deployment.yaml
kubectl create -f conf/prometheus-service.yaml
```

**Sources:**
1. [How To Setup Prometheus Monitoring On Kubernetes Cluster](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)
