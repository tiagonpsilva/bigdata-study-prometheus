# Estudo do Prometheus

Este repositório contém materiais de estudo sobre o Prometheus, com foco especial em monitoramento de stacks de dados.

## Estrutura do Repositório

1. [Conceitos Básicos](./01-conceitos-basicos/README.md)
   - Fundamentos do Prometheus
   - Arquitetura
   - Modelo de dados
   - PromQL básico

2. [Instalação e Configuração](./02-instalacao-configuracao/README.md)
   - Instalação via Docker
   - Configuração básica
   - Service discovery
   - Retenção e storage

3. [Métricas e Instrumentação](./03-metricas-instrumentacao/README.md)
   - Tipos de métricas
   - Exporters
   - Client libraries
   - Best practices

4. [Alertas](./04-alertas/README.md)
   - Alertmanager
   - Regras de alerta
   - Roteamento
   - Notificações

5. [Integração com Stack de Dados](./05-integracao-dados/README.md)
   - Apache Spark
   - Apache Kafka
   - Apache Airflow
   - PostgreSQL/MySQL
   - Elasticsearch
   - MongoDB

6. [Dashboards](./06-dashboards/README.md)
   - Grafana
   - Visualizações
   - Templates
   - Exportação

7. [Exemplos](./exemplos/README.md)
   - Cases práticos
   - Docker Compose
   - Configurações
   - Dashboards

## Pré-requisitos

- Docker e Docker Compose
- Conhecimento básico de monitoramento
- Familiaridade com sistemas distribuídos
- Conhecimento básico de SQL e PromQL

## Como Usar

1. Clone o repositório
```bash
git clone https://github.com/tiagonpsilva/bigdata-study-prometheus.git
```

2. Navegue até o diretório de exemplos
```bash
cd bigdata-study-prometheus/exemplos
```

3. Execute o ambiente de laboratório
```bash
docker-compose up -d
```

4. Acesse:
   - Prometheus: http://localhost:9090
   - Grafana: http://localhost:3000
   - AlertManager: http://localhost:9093

## Contribuição

Sinta-se à vontade para contribuir com este repositório através de Pull Requests.

## Licença

Este projeto está sob a licença MIT. 