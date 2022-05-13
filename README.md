# grafana-loki-setup

Grafana Loki Setup Guide in Kubernetes

## Single Tenent Model (Using DaemonSet for Log Collector Client(Fluent bit))

1. Install Grafana loki-distributed chart in loki namespace. It will install loki's mircoservice components.

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki-distributed -n loki --create-namespace --values=sample-values.yaml
```

2. Install Fluent bit for collecting log with proper loki service name.

```bash
helm upgrade --install fluent-bit grafana/fluent-bit -n loki \
                      --set loki.serviceName=loki-loki-distributed-distributor.loki.svc
```

3. Port forward grafana service & add loki data source. After that from `Explore` section in grafana UI, logs can be examined.

Sample:

![sample-log](./static/grafana-logs.png)


4. For getting alert configure alert_manager url in `.values.loki.config` section & add necessary rules in `.values.ruler.directories` sections.

Sample rules file:
```yaml
rules.yaml:
  groups:
  - name: should_fire
    rules:
      - alert: KubeDBOperatorHighFailedError
        expr: sum by (container) (rate({namespace="kubedb"} |= "Failed"[1m])) > 0.0001
        for: 1m
        labels:
            severity: warning
        annotations:
            summary: High failed error
  - name: test-rule
    rules: 
      - alert: loki-test-rule
        annotations: 
          message: "testing loki rules"
        expr: 1+1
        for: 1m
        labels: 
          severity: critical
```

Alert in AlertManager UI:

![alert-manager-sample-ui](./static/alert-manager-sample.png)

## Multi Tenent Model (Promtail log collector as a Sidecar)


