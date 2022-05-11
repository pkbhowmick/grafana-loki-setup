# grafana-loki-setup

Grafana Loki Setup Guide in Kubernetes

## Single Tenent Model (Using DaemonSet)

1. Install Grafana loki-stack in loki namespace. It will install loki(StatefulSet) and promtail(DaemonSet) by default.

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki-stack grafana/loki-stack -n loki --create-namespace
```

2. Check default values file for grafana/loki-stack. By default only loki and promtail is enabled.

```bash
helm show values grafana/loki-stack
```

Values file:

```yaml
loki:
  enabled: true
  isDefault: true

promtail:
  enabled: true
  config:
    lokiAddress: http://{{ .Release.Name }}:3100/loki/api/v1/push

fluent-bit:
  enabled: false

grafana:
  enabled: false
  sidecar:
    datasources:
      enabled: true
      maxLines: 1000
  image:
    tag: 8.3.5

prometheus:
  enabled: false
  isDefault: false

filebeat:
  enabled: false
  filebeatConfig:
    filebeat.yml: |
      # logging.level: debug
      filebeat.inputs:
      - type: container
        paths:
          - /var/log/containers/*.log
        processors:
        - add_kubernetes_metadata:
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"
      output.logstash:
        hosts: ["logstash-loki:5044"]

logstash:
  enabled: false
  image: grafana/logstash-output-loki
  imageTag: 1.0.1
  filters:
    main: |-
      filter {
        if [kubernetes] {
          mutate {
            add_field => {
              "container_name" => "%{[kubernetes][container][name]}"
              "namespace" => "%{[kubernetes][namespace]}"
              "pod" => "%{[kubernetes][pod][name]}"
            }
            replace => { "host" => "%{[kubernetes][node][name]}"}
          }
        }
        mutate {
          remove_field => ["tags"]
        }
      }
  outputs:
    main: |-
      output {
        loki {
          url => "http://loki:3100/loki/api/v1/push"
          #username => "test"
          #password => "test"
        }
        # stdout { codec => rubydebug }
      }
```

