# Instalação e Configuração do Prometheus

## Instalação via Docker

### Docker Compose Básico

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.45.0
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.25.0
    container_name: alertmanager
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    ports:
      - "9093:9093"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.0.3
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3000:3000"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

## Configuração Básica

### prometheus.yml
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  # Exemplo para Spark
  - job_name: 'spark'
    static_configs:
      - targets: ['spark-master:8080']
    metrics_path: '/metrics'

  # Exemplo para Kafka
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka-broker:7071']

  # Exemplo para Airflow
  - job_name: 'airflow'
    static_configs:
      - targets: ['airflow-webserver:8080']
    metrics_path: '/admin/metrics'
```

### alertmanager.yml
```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'

route:
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        title: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}"
        text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"
```

## Service Discovery

### Kubernetes Service Discovery
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

### File-based Service Discovery
```yaml
scrape_configs:
  - job_name: 'file-sd'
    file_sd_configs:
      - files:
        - 'targets/*.json'
        refresh_interval: 5m
```

Exemplo de arquivo targets/services.json:
```json
[
  {
    "targets": ["service1:9100", "service2:9100"],
    "labels": {
      "env": "production",
      "job": "node"
    }
  }
]
```

## Retenção e Storage

### Configuração de Retenção
```yaml
command:
  - '--storage.tsdb.path=/prometheus'
  - '--storage.tsdb.retention.time=30d'
  - '--storage.tsdb.retention.size=50GB'
```

### Compactação
```yaml
command:
  - '--storage.tsdb.min-block-duration=2h'
  - '--storage.tsdb.max-block-duration=6h'
```

## Exercícios Práticos

1. **Configuração Básica**
   - Configure o Prometheus para monitorar a si mesmo
   - Adicione o Node Exporter
   - Verifique as métricas na interface web

2. **Service Discovery**
   - Configure descoberta baseada em arquivo
   - Adicione e remova targets dinamicamente
   - Observe a atualização automática

3. **Alerting**
   - Configure alertas básicos
   - Teste diferentes canais de notificação
   - Implemente agrupamento de alertas

4. **Storage**
   - Configure diferentes períodos de retenção
   - Monitore o uso de disco
   - Implemente backup dos dados

## Recursos Adicionais

- [Documentação de Configuração](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)
- [Service Discovery](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config)
- [Storage](https://prometheus.io/docs/prometheus/latest/storage/)
- [Best Practices](https://prometheus.io/docs/practices/storage/) 