# Integração com Stack de Dados

## Apache Spark

### Configuração do Spark Metrics
```properties
# metrics.properties
*.sink.prometheus.class=org.apache.spark.metrics.sink.PrometheusSink
*.sink.prometheus.port=9091
*.sink.prometheus.period=10
*.sink.prometheus.unit=seconds
```

### Métricas Importantes
```promql
# Jobs e Stages
spark_job_active_tasks
spark_stage_failed_tasks
spark_stage_processing_time_ms

# Executors
spark_executor_active_tasks
spark_executor_memory_used_bytes
spark_executor_cpu_time_total

# Shuffle
spark_shuffle_bytes_written
spark_shuffle_records_written
spark_shuffle_fetch_wait_time
```

### Exemplo de Dashboard
```yaml
- name: "Spark Overview"
  panels:
    - title: "Active Jobs"
      expr: sum(spark_job_active_tasks)
    - title: "Failed Tasks"
      expr: sum(spark_stage_failed_tasks)
    - title: "Memory Usage"
      expr: sum(spark_executor_memory_used_bytes) by (executor_id)
```

## Apache Kafka

### JMX Exporter Config
```yaml
lowercaseOutputName: true
rules:
  - pattern: kafka.server<type=(.+), name=(.+)><>Value
    name: kafka_server_$1_$2
  - pattern: kafka.controller<type=(.+), name=(.+)><>Value
    name: kafka_controller_$1_$2
```

### Métricas Essenciais
```promql
# Brokers
kafka_server_replicamanager_underreplicatedpartitions
kafka_server_replicamanager_isrexpandspersec
kafka_server_replicamanager_isrshrinkspersec

# Topics
kafka_server_brokertopicmetrics_messagesinpersec
kafka_server_brokertopicmetrics_bytesinpersec
kafka_server_brokertopicmetrics_bytesoutpersec

# Consumer Groups
kafka_consumer_group_lag
kafka_consumer_group_offset
```

### Exemplo de Alertas
```yaml
groups:
  - name: kafka_alerts
    rules:
      - alert: KafkaUnderReplicatedPartitions
        expr: kafka_server_replicamanager_underreplicatedpartitions > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Kafka under-replicated partitions"
```

## Apache Airflow

### Configuração do Airflow
```python
# airflow_config.py
PROMETHEUS_METRICS_PORT = 9102
PROMETHEUS_METRICS_ENABLED = True
```

### Métricas Principais
```promql
# DAGs
airflow_dag_status{status="success"}
airflow_dag_status{status="failed"}
airflow_dag_processing_time_seconds

# Tasks
airflow_task_status{status="running"}
airflow_task_duration_seconds
airflow_task_failure_count
```

### Exemplo de Grafana Dashboard
```json
{
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
      "type": "heatmap",
      "targets": [
        {
          "expr": "rate(airflow_task_duration_seconds_bucket[5m])"
        }
      ]
    }
  ]
}
```

## PostgreSQL

### Configuração do Exporter
```yaml
# postgres_exporter.yml
datasource:
  host: localhost
  port: 5432
  user: prometheus
  password: password
  database: postgres
  sslmode: disable
```

### Métricas Críticas
```promql
# Conexões
pg_stat_activity_count
pg_stat_activity_max_tx_duration

# Performance
pg_stat_database_tup_fetched
pg_stat_database_tup_inserted
pg_stat_database_tup_updated
pg_stat_database_tup_deleted

# Bloqueios
pg_locks_count
pg_stat_activity_count{state="active"}
```

### Exemplo de Regras
```yaml
groups:
  - name: postgresql
    rules:
      - alert: PostgreSQLHighConnections
        expr: pg_stat_activity_count > pg_settings_max_connections * 0.8
        for: 5m
        labels:
          severity: warning
```

## Elasticsearch

### Configuração do Exporter
```yaml
# elasticsearch_exporter.yml
es:
  uri: http://localhost:9200
  all: true
  indices: true
  cluster: true
```

### Métricas Importantes
```promql
# Cluster Health
elasticsearch_cluster_health_status
elasticsearch_cluster_nodes_total

# Índices
elasticsearch_indices_docs_total
elasticsearch_indices_store_size_bytes

# JVM
elasticsearch_jvm_memory_used_bytes
elasticsearch_jvm_gc_collection_seconds
```

## MongoDB

### Configuração do Exporter
```yaml
# mongodb_exporter.yml
mongodb:
  uri: "mongodb://localhost:27017"
  direct_connect: true
  collect_all: true
```

### Métricas Principais
```promql
# Operações
mongodb_op_counters_total
mongodb_mongod_op_latencies_latency_total

# Conexões
mongodb_connections
mongodb_connections_current

# Replicação
mongodb_replset_member_state
mongodb_replset_member_health
```

## Exercícios Práticos

1. **Spark Monitoring**
   - Configure métricas do Spark
   - Crie dashboard básico
   - Implemente alertas de performance

2. **Kafka Observability**
   - Configure JMX Exporter
   - Monitore consumer lag
   - Alerte sobre partições sub-replicadas

3. **Airflow Tracking**
   - Habilite métricas do Airflow
   - Monitore duração de tasks
   - Configure alertas de falha

4. **Database Monitoring**
   - Configure PostgreSQL Exporter
   - Monitore performance de queries
   - Implemente alertas de conexão

## Recursos Adicionais

- [Spark Metrics](https://spark.apache.org/docs/latest/monitoring.html)
- [Kafka Monitoring](https://kafka.apache.org/documentation/#monitoring)
- [Airflow Metrics](https://airflow.apache.org/docs/apache-airflow/stable/logging-monitoring/metrics.html)
- [PostgreSQL Exporter](https://github.com/prometheus-community/postgres_exporter) 