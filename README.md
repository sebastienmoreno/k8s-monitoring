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

## Deploy Kubernetes dashboard

The Kubernetes dashboard will run on 8081 local port, and 30001 node port in internal.

```
kubectl create ns kubernetes-dashboard
helm install kubernetes-dashboard --namespace kubernetes-dashboard helm-charts/dashboard
```

**Open dashboard:**
```
open http://localhost:8081
```

## Deploy Prometheus Grafana

The Grafana dashboard will run on 8082 local port, and 30002 node port in internal.

```
kubectl create namespace monitoring
helm install monitoring --namespace monitoring -f helm-values/monitoring-values.yaml stable/prometheus-operator
```

> Doc:
> https://github.com/helm/charts/tree/master/stable/prometheus-operator
> For information:
> - Grafana default values: https://github.com/helm/charts/blob/master/stable/grafana/values.yaml
> - Prometheus default value: https://github.com/helm/charts/blob/master/stable/prometheus/values.yaml

**Open Grafana:**
```
open http://localhost:8082
```

## Deploy Elastic Stack (ELK)

The Kibana dashboard will run on 8083 local port, and 30003 node port in internal.

**Install ELK:**
```
kubectl create namespace logging
helm install logging --namespace logging -f helm-values/logging-values.yaml stable/elastic-stack
```

**Open Kibana:**
```
open http://localhost:8083
```

> For information:
> - see Kibana values: https://github.com/helm/charts/blob/master/stable/kibana/values.yaml
> - see ELK default values: https://github.com/helm/charts/blob/master/stable/elastic-stack/values.yaml

## Test with custom services

```
kubectl create namespace test
kubectl -n test run --image=smoreno/python-provider:latest provider --port=8080
kubectl -n test run --image=smoreno/python-client:latest client --port=8080
kubectl -n test expose deployment client --port=8080 --name=client
kubectl -n test expose deployment provider --port=8080 --name=provider
```
