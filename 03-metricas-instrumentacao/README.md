# Métricas e Instrumentação no Prometheus

## Tipos de Métricas

### 1. Counter
Métricas que só aumentam (exceto em reset).

```python
from prometheus_client import Counter

requests_total = Counter('http_requests_total', 
                        'Total de requisições HTTP',
                        ['method', 'endpoint'])

# Incrementando
requests_total.labels(method='POST', endpoint='/data').inc()
```

### 2. Gauge
Métricas que podem aumentar ou diminuir.

```python
from prometheus_client import Gauge

cpu_usage = Gauge('cpu_usage_percent', 
                  'Uso de CPU em porcentagem',
                  ['core'])

# Definindo valor
cpu_usage.labels(core='0').set(45.2)

# Incrementando/Decrementando
cpu_usage.labels(core='0').inc()
cpu_usage.labels(core='0').dec()
```

### 3. Histogram
Distribuição de valores em buckets.

```python
from prometheus_client import Histogram

request_duration = Histogram('http_request_duration_seconds',
                           'Duração das requisições HTTP',
                           ['method'],
                           buckets=[0.1, 0.5, 1.0, 2.0, 5.0])

# Observando valores
request_duration.labels(method='GET').observe(0.7)
```

### 4. Summary
Similar ao Histogram, mas com quantis calculados no cliente.

```python
from prometheus_client import Summary

request_latency = Summary('http_request_latency_seconds',
                         'Latência das requisições HTTP',
                         ['method'],
                         quantiles=[0.5, 0.9, 0.99])

# Observando valores
request_latency.labels(method='GET').observe(0.3)
```

## Exporters

### Node Exporter
Métricas do sistema operacional.

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### PostgreSQL Exporter
```yaml
scrape_configs:
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']
    params:
      database: ['postgres']
```

### Kafka Exporter
```yaml
scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka-exporter:9308']
```

## Client Libraries

### Python
```python
from prometheus_client import start_http_server, Counter, Gauge
import random
import time

# Métricas
requests = Counter('app_requests_total', 'Total de requisições')
in_progress = Gauge('app_requests_in_progress', 'Requisições em andamento')
queue_size = Gauge('app_queue_size', 'Tamanho da fila de processamento')

# Decorator para tracking
def track_requests(fn):
    def wrapper(*args, **kwargs):
        in_progress.inc()
        requests.inc()
        try:
            return fn(*args, **kwargs)
        finally:
            in_progress.dec()
    return wrapper

# Exemplo de uso
@track_requests
def process_request():
    time.sleep(random.random())

if __name__ == '__main__':
    # Expõe métricas na porta 8000
    start_http_server(8000)
    
    while True:
        process_request()
        queue_size.set(random.randint(0, 100))
        time.sleep(1)
```

### Java (Spring Boot)
```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Counter;
import org.springframework.stereotype.Component;

@Component
public class MetricsService {
    private final Counter requestCounter;
    private final Gauge queueSize;

    public MetricsService(MeterRegistry registry) {
        this.requestCounter = Counter.builder("app_requests_total")
            .description("Total de requisições")
            .register(registry);
            
        this.queueSize = Gauge.builder("app_queue_size", queue, Queue::size)
            .description("Tamanho da fila")
            .register(registry);
    }

    public void recordRequest() {
        requestCounter.increment();
    }
}
```

## Best Practices

### 1. Nomenclatura
```plaintext
# Bom
http_requests_total
process_cpu_seconds_total
node_memory_usage_bytes

# Ruim
requests
cpu
memory
```

### 2. Labels
```plaintext
# Bom - Cardinalidade controlada
http_requests_total{status="200", method="GET"}
process_cpu_seconds{core="0"}

# Ruim - Alta cardinalidade
http_requests_total{ip="192.168.1.1", user_id="12345"}
```

### 3. Tipos de Métricas
```plaintext
# Use Counter para:
- Número de requisições
- Erros
- Tasks completadas

# Use Gauge para:
- Temperatura
- Memória em uso
- Threads ativas

# Use Histogram para:
- Latência
- Tamanho de resposta
- Duração de tasks
```

## Exemplos para Stack de Dados

### 1. Apache Spark
```python
from prometheus_client import Counter, Gauge

# Métricas de Jobs
spark_jobs_total = Counter('spark_jobs_total', 
                          'Total de jobs Spark',
                          ['status'])

# Métricas de Executors
spark_executors = Gauge('spark_executors_active',
                       'Número de executors ativos')

# Métricas de Memória
spark_memory = Gauge('spark_memory_bytes',
                    'Uso de memória',
                    ['type'])
```

### 2. Apache Kafka
```python
from prometheus_client import Gauge, Counter

# Métricas de Mensagens
kafka_messages = Counter('kafka_messages_total',
                        'Total de mensagens',
                        ['topic'])

# Métricas de Consumer Lag
kafka_consumer_lag = Gauge('kafka_consumer_lag',
                          'Consumer lag',
                          ['group', 'topic'])
```

### 3. Apache Airflow
```python
from prometheus_client import Counter, Gauge, Histogram

# Métricas de DAGs
dag_success_total = Counter('airflow_dag_success_total',
                           'Total de DAGs com sucesso',
                           ['dag_id'])

# Duração de Tasks
task_duration = Histogram('airflow_task_duration_seconds',
                         'Duração das tasks',
                         ['task_id', 'dag_id'])
```

## Exercícios Práticos

1. **Instrumentação Básica**
   - Crie um serviço web simples
   - Adicione métricas de requests
   - Exponha endpoint de métricas

2. **Exporters**
   - Configure Node Exporter
   - Configure PostgreSQL Exporter
   - Visualize métricas no Prometheus

3. **Custom Metrics**
   - Implemente métricas de negócio
   - Use diferentes tipos de métricas
   - Adicione labels relevantes

4. **Integração com Stack de Dados**
   - Configure métricas para Spark
   - Configure métricas para Kafka
   - Configure métricas para Airflow

## Recursos Adicionais

- [Client Libraries](https://prometheus.io/docs/instrumenting/clientlibs/)
- [Exporters and Integrations](https://prometheus.io/docs/instrumenting/exporters/)
- [Metric Types](https://prometheus.io/docs/concepts/metric_types/)
- [Metric Best Practices](https://prometheus.io/docs/practices/naming/) 