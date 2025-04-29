# Dashboards com Grafana

## Configuração do Grafana

### docker-compose.yml
```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:10.0.3
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped

volumes:
  grafana_data:
```

### Datasource Provisioning
```yaml
# datasources.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

## Dashboards para Stack de Dados

### 1. Apache Spark Dashboard
```json
{
  "title": "Spark Overview",
  "panels": [
    {
      "title": "Active Applications",
      "type": "stat",
      "targets": [
        {
          "expr": "sum(spark_app_status{status='running'})"
        }
      ]
    },
    {
      "title": "Memory Usage",
      "type": "graph",
      "targets": [
        {
          "expr": "sum(spark_executor_memory_used_bytes) by (application_id)"
        }
      ]
    },
    {
      "title": "Failed Tasks",
      "type": "timeseries",
      "targets": [
        {
          "expr": "sum(rate(spark_stage_failed_tasks[5m])) by (application_id)"
        }
      ]
    }
  ]
}
```

### 2. Apache Kafka Dashboard
```json
{
  "title": "Kafka Overview",
  "panels": [
    {
      "title": "Messages In Per Second",
      "type": "graph",
      "targets": [
        {
          "expr": "sum(rate(kafka_server_brokertopicmetrics_messagesinpersec[5m])) by (topic)"
        }
      ]
    },
    {
      "title": "Consumer Lag",
      "type": "heatmap",
      "targets": [
        {
          "expr": "sum(kafka_consumer_group_lag) by (group, topic)"
        }
      ]
    },
    {
      "title": "Under-Replicated Partitions",
      "type": "stat",
      "targets": [
        {
          "expr": "sum(kafka_server_replicamanager_underreplicatedpartitions)"
        }
      ]
    }
  ]
}
```

### 3. Apache Airflow Dashboard
```json
{
  "title": "Airflow Overview",
  "panels": [
    {
      "title": "DAG Success Rate",
      "type": "gauge",
      "targets": [
        {
          "expr": "sum(airflow_dag_status{status='success'}) / sum(airflow_dag_status) * 100"
        }
      ]
    },
    {
      "title": "Task Duration",
      "type": "graph",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, sum(rate(airflow_task_duration_seconds_bucket[5m])) by (le, task_id))"
        }
      ]
    },
    {
      "title": "Failed Tasks by DAG",
      "type": "table",
      "targets": [
        {
          "expr": "sum(airflow_task_status{status='failed'}) by (dag_id)"
        }
      ]
    }
  ]
}
```

### 4. PostgreSQL Dashboard
```json
{
  "title": "PostgreSQL Overview",
  "panels": [
    {
      "title": "Active Connections",
      "type": "stat",
      "targets": [
        {
          "expr": "sum(pg_stat_activity_count{state='active'})"
        }
      ]
    },
    {
      "title": "Transaction Duration",
      "type": "graph",
      "targets": [
        {
          "expr": "max(pg_stat_activity_max_tx_duration) by (datname)"
        }
      ]
    },
    {
      "title": "Tuple Operations",
      "type": "graph",
      "targets": [
        {
          "expr": "rate(pg_stat_database_tup_fetched[5m])",
          "legend": "Fetched"
        },
        {
          "expr": "rate(pg_stat_database_tup_inserted[5m])",
          "legend": "Inserted"
        },
        {
          "expr": "rate(pg_stat_database_tup_updated[5m])",
          "legend": "Updated"
        }
      ]
    }
  ]
}
```

## Templates e Variáveis

### Dashboard Variables
```json
{
  "templating": {
    "list": [
      {
        "name": "cluster",
        "type": "query",
        "query": "label_values(spark_app_status, cluster)"
      },
      {
        "name": "application",
        "type": "query",
        "query": "label_values(spark_app_status{cluster=\"$cluster\"}, application_id)"
      },
      {
        "name": "interval",
        "type": "interval",
        "options": ["5m", "15m", "1h", "6h", "12h", "24h"]
      }
    ]
  }
}
```

### Reusable Panels
```json
{
  "panels": [
    {
      "title": "Resource Usage",
      "type": "row",
      "panels": [
        {
          "title": "CPU Usage",
          "type": "graph",
          "repeat": "instance"
        },
        {
          "title": "Memory Usage",
          "type": "graph",
          "repeat": "instance"
        }
      ]
    }
  ]
}
```

## Exportação

### Grafana API
```bash
# Exportar dashboard
curl -H "Authorization: Bearer YOUR_API_KEY" \
     http://localhost:3000/api/dashboards/uid/DASHBOARD_UID \
     > dashboard.json

# Importar dashboard
curl -X POST \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d @dashboard.json \
     http://localhost:3000/api/dashboards/db
```

## Exercícios Práticos

1. **Dashboard Básico**
   - Configure Grafana
   - Adicione Prometheus como fonte
   - Crie dashboard simples

2. **Stack de Dados**
   - Dashboard para Spark
   - Dashboard para Kafka
   - Dashboard para Airflow

3. **Templates**
   - Use variáveis
   - Crie templates reutilizáveis
   - Implemente filtros dinâmicos

4. **Exportação**
   - Exporte dashboards
   - Versione no Git
   - Automatize provisioning

## Recursos Adicionais

- [Grafana Documentation](https://grafana.com/docs/)
- [Dashboard Best Practices](https://grafana.com/docs/grafana/latest/best-practices/best-practices-for-creating-dashboards/)
- [Provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/)
- [API Reference](https://grafana.com/docs/grafana/latest/http_api/) 