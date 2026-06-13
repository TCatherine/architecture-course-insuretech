## Установка prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install prometheus prometheus-community/prometheus \
  -n scaletestapp \
  --set server.persistentVolume.enabled=false \
  --set alertmanager.enabled=false \
  --set prometheus-node-exporter.enabled=false \
  --set prometheus-pushgateway.enabled=false \
  --set kube-state-metrics.enabled=false

helm install prometheus-adapter prometheus-community/prometheus-adapter \
  -n scaletestapp \
  --set prometheus.url=http://prometheus-server.scaletestapp.svc.cluster.local \
  --set prometheus.port=80
```

Добавляем правило в ConfigMap prometheus-adapter
```yaml
- seriesQuery: 'http_requests_total{namespace!="",pod!=""}'
    resources:
    overrides:
        namespace: {resource: "namespace"}
        pod: {resource: "pod"}
    name:
    matches: "^(.*)_total$"
    as: "${1}_per_second" # Превратит "http_requests_total" в "http_requests_per_second"
    metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
    ```