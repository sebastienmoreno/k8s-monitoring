
## Deploy Prometheus Grafana

```
kubectl create namespace monitoring
helm dependency build helm-charts/monitoring/
helm install monitoring --namespace monitoring -f helm-values/monitoring/values.yaml helm-charts/monitoring
```

> Doc:
> https://github.com/helm/charts/tree/master/stable/prometheus-operator

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