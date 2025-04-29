# Conceitos Básicos do Prometheus

## Fundamentos

O Prometheus é um sistema de monitoramento e alerta de código aberto, originalmente desenvolvido no SoundCloud. Suas principais características são:

- **Modelo de dados multidimensional**: séries temporais identificadas por nome de métrica e pares chave-valor (labels)
- **PromQL**: Linguagem de consulta flexível e poderosa para análise de dados
- **Pull model**: Coleta de métricas via HTTP, com intervalo configurável
- **Push model**: Suportado via Pushgateway para casos específicos
- **Service discovery**: Descoberta automática de alvos via integração com várias plataformas
- **Visualização**: Suporte nativo a gráficos e integração com Grafana
- **Alertas**: Sistema robusto de alertas com AlertManager
- **Armazenamento**: Base de dados local eficiente e suporte a armazenamento remoto

## Arquitetura

```plaintext
                   ┌─────────────────┐
                   │    Prometheus   │
                   │    Server      │
┌─────────────┐    │               │    ┌─────────────┐
│  Exporters  │◄───┤  Scraping     │    │ AlertManager│
└─────────────┘    │               │───►└─────────────┘
                   │  Storage      │
┌─────────────┐    │               │    ┌─────────────┐
│  Jobs/Apps  │◄───┤  PromQL      │───►│   Grafana   │
└─────────────┘    └─────────────────┘    └─────────────┘
```

### Componentes Principais

1. **Prometheus Server**
   - Coleta de métricas (scraping)
   - Armazenamento
   - Processamento de regras
   - Interface web local

2. **Exporters**
   - Node Exporter (métricas do sistema)
   - MySQL Exporter
   - PostgreSQL Exporter
   - Redis Exporter
   - JMX Exporter
   - Blackbox Exporter (probes HTTP/HTTPS)

3. **AlertManager**
   - Gerenciamento de alertas
   - Agrupamento
   - Roteamento
   - Silenciamento
   - Inibição

4. **Pushgateway**
   - Suporte a jobs batch
   - Métricas de curta duração
   - Casos onde pull não é viável

## Modelo de Dados

### Tipos de Métricas

1. **Counter**
   - Valores que só aumentam
   - Reset apenas ao reiniciar
```yaml
# Exemplo de counter - total de requests HTTP
http_requests_total{method="POST", endpoint="/api/data"} 2423

# Exemplo de counter - bytes transmitidos
network_bytes_transmitted_total{interface="eth0"} 1.32e+9
```

2. **Gauge**
   - Valores que podem subir ou descer
   - Medições instantâneas
```yaml
# Exemplo de gauge - temperatura CPU
cpu_temperature_celsius{core="0"} 45.2

# Exemplo de gauge - uso de memória
memory_usage_bytes{instance="server-01"} 2.31e+9
```

3. **Histogram**
   - Distribuição de valores
   - Buckets configuráveis
```yaml
# Exemplo de histogram - duração de requests
http_request_duration_seconds_bucket{le="0.1"} 24054
http_request_duration_seconds_bucket{le="0.5"} 33444
http_request_duration_seconds_bucket{le="1.0"} 34500
http_request_duration_seconds_sum 53423.21
http_request_duration_seconds_count 34500
```

4. **Summary**
   - Similar ao histogram
   - Cálculo de quantis no cliente
```yaml
# Exemplo de summary - latência RPC
rpc_duration_seconds{quantile="0.99"} 3.2
rpc_duration_seconds{quantile="0.95"} 1.4
rpc_duration_seconds_sum 8953.2
rpc_duration_seconds_count 27644
```

## PromQL Básico

### Queries Básicas

1. **Seleção Instantânea**
```promql
# Valor atual da métrica
http_requests_total

# Com filtros de labels
http_requests_total{job="api-server", instance="10.0.0.1:9090"}
```

2. **Range Vector**
```promql
# Valores dos últimos 5 minutos
http_requests_total[5m]

# Valores da última hora com step de 1m
http_requests_total[1h:1m]
```

3. **Filtros por Labels**
```promql
# Match exato
http_requests_total{status="200"}

# Regex match
http_requests_total{status=~"5.."}

# Negação
http_requests_total{status!="200"}
```

### Operadores

1. **Aritméticos**
```promql
# Taxa de erros em porcentagem
sum(rate(http_requests_total{status=~"5.."}[5m])) / 
sum(rate(http_requests_total[5m])) * 100

# Uso de memória em GB
memory_usage_bytes / 1024 / 1024 / 1024
```

2. **Lógicos**
```promql
# Serviços com alta latência
http_request_duration_seconds > 0.5

# Combinação de condições
up == 0 or (cpu_usage_percent > 90 and memory_usage_percent > 80)
```

3. **Agregação**
```promql
# Total de requests por método
sum(http_requests_total) by (method)

# Média de CPU por host
avg(cpu_usage_percent) by (host)

# Top 3 processos por uso de memória
topk(3, memory_usage_bytes)
```

### Funções Comuns

1. **rate() e irate()**
```promql
# Taxa média de requests por segundo nos últimos 5 minutos
rate(http_requests_total[5m])

# Taxa instantânea
irate(http_requests_total[5m])
```

2. **increase()**
```promql
# Aumento no número de erros na última hora
increase(http_errors_total[1h])

# Bytes transmitidos nos últimos 30 minutos
increase(network_bytes_transmitted_total[30m])
```

3. **histogram_quantile()**
```promql
# 95º percentil da latência
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# 99º percentil por endpoint
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint))
```

## Exercícios Práticos

1. **Monitoramento Básico**
```promql
# Verifique o status dos targets
up

# Liste todas as métricas de CPU por núcleo
cpu_usage_percent{job="node_exporter"}

# Calcule o uso total de memória em GB
sum(memory_usage_bytes) / 1024^3
```

2. **Análise de Performance**
```promql
# Taxa de requests por segundo por endpoint
rate(http_requests_total[5m])

# Latência média por serviço
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

# Top 5 endpoints mais lentos
topk(5, avg(http_request_duration_seconds) by (endpoint))
```

3. **Alertas e Previsões**
```promql
# Serviços com alta taxa de erros (>5%)
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service) / 
sum(rate(http_requests_total[5m])) by (service) * 100 > 5

# Previsão de esgotamento de disco em 4 horas
predict_linear(disk_usage_bytes[1h], 4 * 3600) > disk_total_bytes
```

## Configuração Básica

### prometheus.yml
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'app'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['app:8080']
```

## Recursos Adicionais

- [Documentação Oficial do Prometheus](https://prometheus.io/docs/introduction/overview/)
- [PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
- [Best Practices de Monitoramento](https://prometheus.io/docs/practices/naming/)
- [Prometheus Blog](https://prometheus.io/blog/)
- [Awesome Prometheus](https://github.com/roaldnefs/awesome-prometheus)
- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)