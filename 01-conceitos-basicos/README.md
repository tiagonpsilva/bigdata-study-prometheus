# Conceitos Básicos do Prometheus

## Fundamentos

O Prometheus é um sistema de monitoramento e alerta de código aberto, originalmente desenvolvido no SoundCloud. Suas principais características são:

- Modelo de dados multidimensional (séries temporais identificadas por nome de métrica e pares chave-valor)
- Linguagem de consulta flexível (PromQL)
- Pull model para coleta de métricas
- Push model suportado via gateway intermediário
- Descoberta de alvos via service discovery
- Múltiplos modos de visualização: gráficos, dashboards e APIs

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

## Modelo de Dados

### Tipos de Métricas

1. **Counter**
```yaml
# Exemplo de counter
http_requests_total{method="POST", endpoint="/api/data"} 2423
```

2. **Gauge**
```yaml
# Exemplo de gauge
cpu_temperature_celsius{core="0"} 45.2
```

3. **Histogram**
```yaml
# Exemplo de histogram
http_request_duration_seconds_bucket{le="0.1"} 24054
http_request_duration_seconds_bucket{le="0.5"} 33444
http_request_duration_seconds_bucket{le="1.0"} 34500
```

4. **Summary**
```yaml
# Exemplo de summary
rpc_duration_seconds{quantile="0.99"} 3.2
rpc_duration_seconds{quantile="0.95"} 1.4
rpc_duration_seconds_sum 8953.2
rpc_duration_seconds_count 27644
```

## PromQL Básico

### Queries Básicas

1. **Seleção Instantânea**
```promql
http_requests_total
```

2. **Range Vector**
```promql
http_requests_total[5m]
```

3. **Filtros por Labels**
```promql
http_requests_total{status="200", method="GET"}
```

### Operadores

1. **Aritméticos**
```promql
# Taxa de erros
sum(rate(http_requests_total{status=~"5.."}[5m])) / 
sum(rate(http_requests_total[5m])) * 100
```

2. **Lógicos**
```promql
# Serviços com alta latência
http_request_duration_seconds > 0.5
```

3. **Agregação**
```promql
# Total de requests por método
sum(http_requests_total) by (method)
```

### Funções Comuns

1. **rate()**
```promql
# Taxa de requests por segundo nos últimos 5 minutos
rate(http_requests_total[5m])
```

2. **increase()**
```promql
# Aumento no número de erros na última hora
increase(http_errors_total[1h])
```

3. **histogram_quantile()**
```promql
# 95º percentil da latência
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

## Exercícios Práticos

1. **Consulta Básica**
```promql
# Liste todas as métricas relacionadas a CPU
cpu_usage_percent

# Filtre por núcleo específico
cpu_usage_percent{core="0"}
```

2. **Agregações**
```promql
# Média de uso de CPU por host
avg(cpu_usage_percent) by (host)

# Máximo uso de memória por serviço
max(memory_usage_bytes) by (service)
```

3. **Taxas e Tendências**
```promql
# Taxa de crescimento de erros
rate(error_count_total[5m])

# Previsão de uso de disco em 4 horas
predict_linear(disk_usage_bytes[1h], 4 * 3600)
```

## Recursos Adicionais

- [Documentação Oficial do Prometheus](https://prometheus.io/docs/introduction/overview/)
- [PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
- [Best Practices](https://prometheus.io/docs/practices/naming/)
- [Prometheus Blog](https://prometheus.io/blog/) 