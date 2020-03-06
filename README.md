KUBERNETES MONITORING
=====================

Kubernetes Demonstration stack for monitoring and logs.

# Requirements

You should first check the [requirements](requirements.md) to install before launching the demo.

Add the stable chart repo:
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo udpate
```


# Demos

## Start local k3d Kubernetes

Start a local k3d stack, exposing 3 nodeports.

```
k3d create --publish 8081:30001@k3d-k3s-default-worker-0 --publish 8082:30002@k3d-k3s-default-worker-0 --publish 8083:30003@k3d-k3s-default-worker-0 --publish 8080:80 --workers 2
```


## Deploy Elastic Stack (ELK)

**Install ELK:**
```
kubectl create namespace elk
helm install --namespace elk --name elk -f helm-values/elk/values.yaml stable/elastic-stack
```

**Open Kibana:**
```
open http://localhost:8081
```

> For information:
> - see Kibana values: https://raw.githubusercontent.com/helm/charts/master/stable/kibana/values.yaml
> - see ELK default values: https://github.com/helm/charts/blob/master/stable/elastic-stack/values.yaml

## Deploy Prometheus Grafana

```
kubectl create namespace monitoring
helm dependency build helm-charts/monitoring/
helm install monitoring --namespace monitoring -f helm-values/monitoring/values.yaml helm-charts/monitoring
```

**Open Grafana:**
```
open http://localhost:8082
```

**Get admin password:**
```
kubectl get secret --namespace monitoring monitoring-grafana-secret -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 --decode ; echo
```


> For information:
> - Grafana default values: https://github.com/helm/charts/blob/master/stable/grafana/values.yaml
> - Prometheus default value: https://github.com/helm/charts/blob/master/stable/prometheus/values.yaml