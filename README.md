# 🧠 Estudo do Prometheus

Este repositório contém explicações detalhadas e exemplos práticos dos conceitos fundamentais do Prometheus, focando em monitoramento, métricas, alertas e visualização de dados com práticas de observabilidade moderna.

## 📋 Índice de Conceitos

1. **📊 [Conceitos Básicos](./01-conceitos-basicos/README.md)** - Fundamentos, arquitetura e componentes do Prometheus
2. **⚙️ [Instalação e Configuração](./02-instalacao-configuracao/README.md)** - Setup do ambiente, configurações e service discovery
3. **📈 [Métricas e Instrumentação](./03-metricas-instrumentacao/README.md)** - Tipos de métricas, exporters e instrumentação de código
4. **🚨 [Alertas](./04-alertas/README.md)** - Regras de alerta, Alertmanager e notificações
5. **🔄 [Integração de Dados](./05-integracao-dados/README.md)** - PromQL, federation e remote storage
6. **📊 [Dashboards](./06-dashboards/README.md)** - Visualização com Grafana e console templates
7. **💡 [Exemplos](./07-exemplos/README.md)** - Cases práticos e integrações

## 🌟 Objetivo

Este repositório tem como objetivo proporcionar um entendimento prático do Prometheus, com explicações claras e exemplos de casos de uso reais. Cada conceito é explorado em detalhes, com configurações funcionais e boas práticas de observabilidade.

## ⚙️ Pré-requisitos

- Docker e Docker Compose
- Conhecimento básico de Linux
- Familiaridade com conceitos de monitoramento
- Go (opcional, para instrumentação personalizada)

## 🚀 Como Usar

1. Clone o repositório
```bash
git clone https://github.com/tiagonpsilva/bigdata-study-prometheus.git
```

2. Navegue até o diretório do projeto
```bash
cd bigdata-study-prometheus
```

3. Inicie o ambiente com Docker Compose
```bash
docker-compose up -d
```

4. Acesse o Prometheus
```
http://localhost:9090
```

## 🐳 Ambiente de Desenvolvimento

### Docker Compose

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
```

## 🔍 Estrutura do Projeto

```
bigdata-study-prometheus/
├── 01-conceitos-basicos/       # Fundamentos e arquitetura
├── 02-instalacao-configuracao/ # Setup e configurações
├── 03-metricas-instrumentacao/ # Métricas e instrumentação
├── 04-alertas/                # Configuração de alertas
├── 05-integracao-dados/       # PromQL e integrações
├── 06-dashboards/            # Visualizações Grafana
└── 07-exemplos/             # Cases práticos
```

Cada diretório contém:
- README com explicações detalhadas
- Exemplos práticos de configuração
- Exercícios para prática
- Links para documentação adicional

## 🎯 Exemplos Práticos

O diretório [exemplos](exemplos/README.md) contém configurações completas para diferentes casos de uso:

1. Monitoramento de Aplicação Web
   - Métricas de latência e throughput
   - Alertas de disponibilidade
   - Dashboard de performance

2. Monitoramento de Infraestrutura
   - Métricas de CPU, memória e disco
   - Alertas de capacidade
   - Dashboard de recursos

3. Monitoramento de Containers
   - Métricas do Docker/Kubernetes
   - Alertas de estado dos containers
   - Dashboard de cluster

## 📝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para:
- Reportar bugs
- Sugerir melhorias
- Enviar pull requests

## 📚 Recursos Adicionais

- [Documentação Oficial do Prometheus](https://prometheus.io/docs/)
- [Prometheus Blog](https://prometheus.io/blog/)
- [Grafana Labs Blog](https://grafana.com/blog/)
- [PromCon Talks](https://prometheus.io/community/)

## 📜 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.