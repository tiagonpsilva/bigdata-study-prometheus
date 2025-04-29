# Alertas no Prometheus

## Configuração do Alertmanager

### alertmanager.yml
```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager@example.com'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'team-emails'
  routes:
    - match:
        severity: critical
      receiver: 'team-slack'
      continue: true
    - match:
        severity: warning
      receiver: 'team-emails'

receivers:
  - name: 'team-slack'
    slack_configs:
      - channel: '#alerts'
        title: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}"
        text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"
        send_resolved: true

  - name: 'team-emails'
    email_configs:
      - to: 'team@example.com'
        send_resolved: true
```

## Regras de Alerta

### rules/node_alerts.yml
```yaml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: avg(rate(node_cpu_seconds_total{mode="system"}[5m])) by (instance) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% for 5 minutes"

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 90%"

      - alert: DiskSpaceRunningOut
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} < 0.1
        for: 1h
        labels:
          severity: critical
        annotations:
          summary: "Disk space running out on {{ $labels.instance }}"
          description: "Less than 10% disk space available"
```

### rules/data_stack_alerts.yml
```yaml
groups:
  - name: data_stack_alerts
    rules:
      # Spark Alerts
      - alert: SparkJobFailure
        expr: spark_jobs_total{status="failed"} > 0
        for: 1m
        labels:
          severity: critical
          team: data
        annotations:
          summary: "Spark job failure detected"
          description: "Spark job has failed in the last minute"

      # Kafka Alerts
      - alert: KafkaHighConsumerLag
        expr: kafka_consumer_lag > 10000
        for: 10m
        labels:
          severity: warning
          team: data
        annotations:
          summary: "High Kafka consumer lag"
          description: "Consumer group {{ $labels.group }} has lag > 10k messages"

      # Airflow Alerts
      - alert: AirflowTaskFailure
        expr: airflow_dag_failed_tasks_total > 0
        for: 5m
        labels:
          severity: critical
          team: data
        annotations:
          summary: "Airflow task failure"
          description: "DAG {{ $labels.dag_id }} has failed tasks"

      # PostgreSQL Alerts
      - alert: PostgreSQLHighConnections
        expr: pg_stat_activity_count > 100
        for: 5m
        labels:
          severity: warning
          team: data
        annotations:
          summary: "High number of PostgreSQL connections"
          description: "More than 100 active connections"
```

## Roteamento

### Exemplo de Roteamento Complexo
```yaml
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default-receiver'
  routes:
    # Rota para time de dados
    - match_re:
        service: ^(spark|kafka|airflow|postgres)$
      receiver: 'data-team'
      routes:
        - match:
            severity: critical
          receiver: 'data-team-pager'
          continue: true
        - match:
            severity: warning
          receiver: 'data-team-slack'

    # Rota para infraestrutura
    - match:
        team: infra
      receiver: 'infra-team'
      group_by: ['alertname', 'instance']
      group_wait: 1m
      routes:
        - match:
            severity: critical
          receiver: 'infra-team-pager'
```

## Notificações

### Templates
```yaml
templates:
  - '/etc/alertmanager/template/*.tmpl'

receivers:
  - name: 'data-team-slack'
    slack_configs:
      - channel: '#data-alerts'
        title: |
          [{{ .Status | toUpper }}] {{ .CommonLabels.alertname }}
        text: |
          *Alert:* {{ .CommonLabels.alertname }}
          *Severity:* {{ .CommonLabels.severity }}
          *Service:* {{ .CommonLabels.service }}
          
          *Description:* {{ .CommonAnnotations.description }}
          
          *Affected Instances:*
          {{- range .Alerts }}
            - {{ .Labels.instance }}
          {{- end }}
```

## Exercícios Práticos

1. **Configuração Básica**
   - Configure Alertmanager
   - Defina regras simples
   - Teste notificações

2. **Alertas para Stack de Dados**
   - Implemente alertas para Spark
   - Configure monitoramento de Kafka
   - Monitore jobs do Airflow

3. **Roteamento Avançado**
   - Configure múltiplos receivers
   - Implemente roteamento por severidade
   - Use templates personalizados

4. **Integração com Canais**
   - Configure Slack
   - Configure Email
   - Configure PagerDuty

## Recursos Adicionais

- [Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Alertmanager Configuration](https://prometheus.io/docs/alerting/latest/configuration/)
- [Notification Templates](https://prometheus.io/docs/alerting/latest/notification_examples/)
- [Best Practices](https://prometheus.io/docs/practices/alerting/) 