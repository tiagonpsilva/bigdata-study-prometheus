# Exemplos Práticos

## 1. Monitoramento de Aplicação Python

### app.py
```python
from prometheus_client import Counter, Gauge, Histogram, start_http_server
import random
import time

# Métricas
REQUEST_COUNT = Counter('app_requests_total', 'Total de requisições')
REQUEST_LATENCY = Histogram('app_request_latency_seconds', 'Latência das requisições')
USERS_ONLINE = Gauge('app_users_online', 'Usuários online')

def process_request():
    # Simula processamento
    REQUEST_COUNT.inc()
    with REQUEST_LATENCY.time():
        time.sleep(random.uniform(0.1, 0.5))
    
    # Atualiza usuários online
    USERS_ONLINE.set(random.randint(10, 100))

def main():
    # Inicia servidor de métricas
    start_http_server(8000)
    
    while True:
        process_request()
        time.sleep(1)

if __name__ == '__main__':
    main()
```

### prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'python-app'
    static_configs:
      - targets: ['localhost:8000']
```

## 2. Monitoramento de Stack Spark

### docker-compose.yml
```yaml
version: '3.8'

services:
  spark-master:
    image: bitnami/spark:3.4
    environment:
      - SPARK_MODE=master
    ports:
      - "8080:8080"
      - "7077:7077"
    
  spark-worker:
    image: bitnami/spark:3.4
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
    depends_on:
      - spark-master

  prometheus:
    image: prom/prometheus:v2.45.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:10.0.3
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
```

### prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spark'
    static_configs:
      - targets: ['spark-master:8080']
    metrics_path: /metrics/prometheus
```

## 3. Monitoramento de Stack Kafka

### docker-compose.yml
```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      
  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: kafka

  kafka-exporter:
    image: danielqsj/kafka-exporter:latest
    command:
      - '--kafka.server=kafka:29092'
    ports:
      - "9308:9308"
    depends_on:
      - kafka

  prometheus:
    image: prom/prometheus:v2.45.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:10.0.3
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
```

### prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka-exporter:9308']
```

## 4. Monitoramento de Stack Airflow

### docker-compose.yml
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=airflow
      - POSTGRES_PASSWORD=airflow
      - POSTGRES_DB=airflow
    ports:
      - "5432:5432"

  airflow-webserver:
    image: apache/airflow:2.7.1
    depends_on:
      - postgres
    environment:
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
      - AIRFLOW__CORE__LOAD_EXAMPLES=no
    ports:
      - "8080:8080"
    command: webserver
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  airflow-scheduler:
    image: apache/airflow:2.7.1
    depends_on:
      - postgres
    environment:
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
      - AIRFLOW__CORE__LOAD_EXAMPLES=no
    command: scheduler

  prometheus:
    image: prom/prometheus:v2.45.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:10.0.3
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
```

### prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'airflow'
    static_configs:
      - targets: ['airflow-webserver:8080']
    metrics_path: /metrics
```

## Exercícios Práticos

1. **Aplicação Python**
   - Execute o exemplo Python
   - Visualize métricas no Prometheus
   - Crie dashboard no Grafana

2. **Stack Spark**
   - Inicie o ambiente Spark
   - Configure métricas JMX
   - Monitore jobs e executors

3. **Stack Kafka**
   - Configure Kafka Exporter
   - Monitore tópicos e consumers
   - Crie alertas para lag

4. **Stack Airflow**
   - Habilite métricas do Airflow
   - Monitore DAGs e tasks
   - Implemente SLAs

## Recursos Adicionais

- [Python Client](https://github.com/prometheus/client_python)
- [Spark Monitoring](https://spark.apache.org/docs/latest/monitoring.html)
- [Kafka Exporter](https://github.com/danielqsj/kafka_exporter)
- [Airflow Metrics](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/metrics.html) 