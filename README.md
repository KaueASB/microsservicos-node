# Desafio Microsserviços Node.js

## 📋 Índice

- [Visão Geral](#-visão-geral)
- [Arquitetura](#arquitetura)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Microsserviços](#-microsserviços)
- [Infraestrutura](#infraestrutura)
- [Observabilidade](#-observabilidade)
- [Configuração e Execução](#configuração-e-execução)
- [API Endpoints](#-api-endpoints)
- [Deployment](#-deployment)
- [Monitoramento](#-monitoramento)
- [Troubleshooting](#-troubleshooting)
- [Contribuição](#-contribuição)

## 🎯 Visão Geral

Este projeto implementa uma arquitetura de microsserviços completa usando Node.js, demonstrando as melhores práticas para sistemas distribuídos modernos. O sistema simula um fluxo de e-commerce simplificado onde pedidos são criados e faturas são geradas automaticamente através de comunicação assíncrona.

### Objetivos do Projeto

- **Demonstrar arquitetura de microsserviços**: Implementação prática de serviços independentes e fracamente acoplados
- **Comunicação assíncrona**: Uso de message brokers para desacoplar serviços
- **Observabilidade completa**: Implementação de distributed tracing, métricas e logs
- **Infrastructure as Code**: Automação completa do deployment usando Pulumi
- **Containerização**: Uso de Docker para portabilidade e consistência
- **API Gateway**: Ponto de entrada único com Kong
- **Escalabilidade**: Deployment em AWS ECS Fargate para auto-scaling

## Arquitetura

### Visão Arquitetural

O sistema é composto por:

1. **API Gateway (Kong)**: Ponto de entrada único, roteamento e políticas transversais
2. **Microsserviço Orders**: Gerenciamento de pedidos e publicação de eventos
3. **Microsserviço Invoices**: Processamento de faturas baseado em eventos
4. **Message Broker (RabbitMQ)**: Comunicação assíncrona entre serviços
5. **Bancos de Dados**: PostgreSQL independente para cada serviço
6. **Observabilidade**: Jaeger para tracing e Grafana para métricas
7. **Load Balancer**: AWS Application Load Balancer para distribuição de carga

### Padrões Arquiteturais Implementados

- **Database per Service**: Cada microsserviço possui seu próprio banco de dados
- **Event-Driven Architecture**: Comunicação através de eventos assíncronos
- **API Gateway Pattern**: Ponto de entrada único para clientes externos
- **Circuit Breaker**: Implementado via observabilidade para detectar falhas
- **Health Check Pattern**: Endpoints de saúde para monitoramento
- **Distributed Tracing**: Rastreamento de requisições através dos serviços

## Tecnologias Utilizadas

### Backend

- **Node.js 22**: Runtime JavaScript com suporte a TypeScript nativo
- **TypeScript**: Tipagem estática para maior robustez
- **Fastify**: Framework web de alta performance
- **Drizzle ORM**: ORM type-safe para PostgreSQL
- **Zod**: Validação de schemas e type safety

### Mensageria

- **RabbitMQ**: Message broker AMQP
- **amqplib**: Cliente Node.js para RabbitMQ

### Observabilidade

- **OpenTelemetry**: Framework de observabilidade
- **Jaeger**: Distributed tracing
- **Grafana**: Visualização de métricas e logs

### Infra

- **Docker**: Containerização
- **Kong**: API Gateway
- **PostgreSQL**: Banco de dados relacional
- **AWS ECS Fargate**: Orquestração de containers serverless
- **AWS Application Load Balancer**: Balanceamento de carga
- **Pulumi**: Infrastructure as Code

### DevOps

- **GitHub Actions**: CI/CD pipeline
- **Docker Compose**: Orquestração local
- **Neon**: PostgreSQL como serviço

## 📁 Estrutura do Projeto

```bash
desafio-microsservicos-nodejs/
├── app-orders/                # Microsserviço de Pedidos
│   ├── src/
│   │   ├── broker/            # Configuração RabbitMQ
│   │   │   ├── channels/      # Definição de canais
│   │   │   └── messages/      # Handlers de mensagens
│   │   ├── db/                # Configuração do banco
│   │   │   ├── migrations/    # Migrações do banco
│   │   │   └── schema/        # Esquemas Drizzle
│   │   ├── http/              # Servidor HTTP
│   │   └── tracer/            # Configuração OpenTelemetry
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── app-invoices/              # Microsserviço de Faturas
│   ├── src/
│   │   ├── broker/            # Configuração RabbitMQ
│   │   │   ├── channels/      # Definição de canais
│   │   │   └── subscriber.ts  # Consumidor de eventos
│   │   ├── db/                # Configuração do banco
│   │   │   ├── migrations/    # Migrações do banco
│   │   │   └── schema/        # Esquemas Drizzle
│   │   └── http/              # Servidor HTTP
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── contracts/                 # Contratos compartilhados
│   └── messages/              # Definições de mensagens
│
├── docker/                    # Configurações Docker
│   └── kong/                  # Configuração Kong Gateway
│
├── infra/                     # Infrastructure as Code
│   ├── src/
│   │   ├── cluster.ts         # Configuração ECS
│   │   ├── images/            # Definições de imagens Docker
│   │   ├── services/          # Definições de serviços
│   │   └── load-balancer.ts   # Configuração ALB
│   └── index.ts               # Ponto de entrada Pulumi
│
├── .github/workflows/         # CI/CD GitHub Actions
├── docker-compose.yml         # Orquestração local completa
└── README.md
```

## 🔧 Microsserviços

### Orders Service (Porta 3333)

**Responsabilidades:**

- Criação e gerenciamento de pedidos
- Validação de dados de entrada
- Persistência no banco de dados
- Publicação de eventos `OrderCreated`
- Distributed tracing com OpenTelemetry

**Endpoints:**

- `GET /health` - Health check
- `POST /orders` - Criação de pedidos

**Tecnologias específicas:**

- Fastify com validação Zod
- Drizzle ORM com PostgreSQL
- RabbitMQ para publicação de eventos
- OpenTelemetry para observabilidade

**Fluxo de Criação de Pedido:**

1. Recebe requisição POST com valor do pedido
2. Valida dados usando Zod
3. Gera UUID único para o pedido
4. Persiste no banco PostgreSQL
5. Inicia span de tracing personalizado
6. Publica evento `OrderCreated` no RabbitMQ
7. Retorna status 201

### Invoices Service (Porta 3334)

**Responsabilidades:**

- Consumo de eventos `OrderCreated`
- Geração automática de faturas
- Persistência de faturas no banco
- Health check para monitoramento

**Endpoints:**

- `GET /health` - Health check

**Tecnologias específicas:**

- Fastify para servidor HTTP
- Drizzle ORM com PostgreSQL
- RabbitMQ consumer para eventos
- OpenTelemetry para observabilidade

**Fluxo de Processamento:**

1. Consome evento `OrderCreated` do RabbitMQ
2. Processa dados do pedido
3. Gera fatura correspondente
4. Persiste fatura no banco PostgreSQL
5. Acknowledges a mensagem processada

## Infraestrutura

### Arquitetura AWS

A infraestrutura é provisionada na AWS usando Pulumi com os seguintes componentes:

#### ECS Cluster

- **Fargate**: Computação serverless para containers
- **Auto Scaling**: Escalabilidade automática baseada em métricas
- **Service Discovery**: Descoberta automática de serviços

#### Application Load Balancer

- **Health Checks**: Monitoramento contínuo da saúde dos serviços
- **Target Groups**: Agrupamento lógico de instâncias
- **Multi-AZ**: Distribuição em múltiplas zonas de disponibilidade

#### Networking

- **VPC**: Rede virtual privada isolada
- **Subnets**: Sub-redes públicas e privadas
- **Security Groups**: Controle de acesso granular

#### Observabilidade e Monitoramento

- **Grafana Cloud**: Métricas e alertas
- **OpenTelemetry**: Coleta de telemetria
- **Jaeger**: Distributed tracing

### Configuração Local

Para desenvolvimento local, o projeto usa Docker Compose com:

- **RabbitMQ**: Message broker com interface de gerenciamento
- **Jaeger**: Distributed tracing local
- **Kong**: API Gateway com admin interface
- **PostgreSQL**: Bancos de dados independentes (via Neon)

## 📊 Observabilidade

### Distributed Tracing

**OpenTelemetry Configuration:**

- Auto-instrumentação para HTTP, Fastify, PostgreSQL e AMQP
- Spans customizados para operações críticas
- Atributos contextuais para debugging
- Exportação para Grafana Cloud via OTLP

**Instrumentações Ativas:**

- `http`: Requisições HTTP de entrada e saída
- `fastify`: Framework web
- `pg`: Queries PostgreSQL
- `amqplib`: Operações RabbitMQ

### Métricas e Alertas

**Grafana Cloud Integration:**

- Métricas de performance de aplicação
- Métricas de infraestrutura AWS
- Alertas baseados em SLIs/SLOs
- Dashboards customizados

### Health Checks

Cada serviço expõe endpoint `/health` para:

- Load balancer health checks
- Kubernetes liveness/readiness probes
- Monitoramento de disponibilidade

## Configuração e Execução

### Pré-requisitos

- Node.js 22+
- Docker e Docker Compose
- Conta AWS (para deployment)
- Pulumi CLI (para infraestrutura)

### Execução Local

1. **Clone o repositório:**

```bash
git clone <repository-url>
cd desafio-microsservicos-nodejs
```

1. **Configure variáveis de ambiente:**

```bash
# app-orders/.env
DATABASE_URL=postgresql://orders_owner:password@host/orders
BROKER_URL=amqp://admin:admin@localhost:5672

# app-invoices/.env
DATABASE_URL=postgresql://invoices_owner:password@host/invoices
BROKER_URL=amqp://admin:admin@localhost:5672
```

3. **Inicie a infraestrutura local:**

```bash
docker-compose up -d
```

4. **Execute as migrações:**

```bash
cd app-orders && npm run db:migrate
cd ../app-invoices && npm run db:migrate
```

5. **Inicie os serviços:**

```bash
# Terminal 1 - Orders Service
cd app-orders && npm run dev

# Terminal 2 - Invoices Service
cd app-invoices && npm run dev
```

### Execução com Docker

```bash
# Build e execução completa
docker-compose up --build

# Execução em background
docker-compose up -d

# Logs dos serviços
docker-compose logs -f orders
docker-compose logs -f invoices
```

## 🌐 API Endpoints

### Orders Service

#### Criar Pedido

```http
POST http://localhost:3333/orders
Content-Type: application/json

{
  "amount": 29.99
}
```

**Resposta:**

```http
HTTP/1.1 201 Created
```

#### Health Check (Orders)

```http
GET http://localhost:3333/health
```

**Resposta:**

```http
HTTP/1.1 200 OK
Content-Type: text/plain

OK
```

### Invoices Service

#### Health Check (Invoices)

```http
GET http://localhost:3334/health
```

**Resposta:**

```http
HTTP/1.1 200 OK
Content-Type: text/plain

OK
```

### Via API Gateway (Kong)

Quando executando com Kong:

```http
POST http://localhost:8000/orders
Content-Type: application/json

{
  "amount": 29.99
}
```

## 🚀 Deployment

### Infraestrutura AWS

1. **Configure credenciais AWS:**

```bash
aws configure
```

2. **Instale Pulumi:**

```bash
curl -fsSL https://get.pulumi.com | sh
```

1. **Deploy da infraestrutura:**

```bash
cd infra
pulumi up
```

### CI/CD Pipeline

O projeto inclui GitHub Actions para:

1. **Build e Test**: Validação de código e testes
2. **Docker Build**: Construção de imagens Docker
3. **Infrastructure Deploy**: Deploy da infraestrutura via Pulumi
4. **Application Deploy**: Deploy dos microsserviços

**Workflow Triggers:**

- Push para branch `main`
- Pull requests
- Tags de release

### Variáveis de Ambiente Produção

```bash
# Orders Service
DATABASE_URL=postgresql://user:pass@host/orders
BROKER_URL=amqp://user:pass@rabbitmq:5672
OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-prod.grafana.net/otlp
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic <token>
OTEL_SERVICE_NAME=orders

# Invoices Service
DATABASE_URL=postgresql://user:pass@host/invoices
BROKER_URL=amqp://user:pass@rabbitmq:5672
OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-prod.grafana.net/otlp
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic <token>
OTEL_SERVICE_NAME=invoices
```

## 📈 Monitoramento

### Métricas Importantes

**Application Metrics:**

- Request rate (req/s)
- Response time (p95, p99)
- Error rate (%)
- Database connection pool usage

**Infrastructure Metrics:**

- CPU utilization
- Memory usage
- Network I/O
- Container health

**Business Metrics:**

- Orders created per minute
- Invoice processing time
- Message queue depth

### Alertas Configurados

- **High Error Rate**: > 5% em 5 minutos
- **High Response Time**: p95 > 1s em 5 minutos
- **Service Down**: Health check failures
- **Queue Backlog**: > 100 mensagens pendentes

### Dashboards

1. **Application Overview**: Métricas gerais dos serviços
2. **Infrastructure**: Métricas de AWS ECS e ALB
3. **Business KPIs**: Métricas de negócio
4. **Distributed Tracing**: Análise de traces

## 🔍 Troubleshooting

### Problemas Comuns

#### Serviço não inicia

```bash
# Verificar logs
docker-compose logs orders
docker-compose logs invoices

# Verificar conectividade do banco
docker-compose exec orders npm run db:check
```

#### RabbitMQ não conecta

```bash
# Verificar status do RabbitMQ
docker-compose ps broker

# Acessar interface de gerenciamento
open http://localhost:15672
# Usuário: guest, Senha: guest
```

#### Traces não aparecem no Jaeger

```bash
# Verificar configuração OpenTelemetry
docker-compose logs jaeger

# Verificar se traces estão sendo enviados
curl http://localhost:14268/api/traces
```

### Debugging

#### Logs Estruturados

```bash
# Logs em tempo real
docker-compose logs -f

# Logs específicos de um serviço
docker-compose logs -f orders

# Logs com timestamp
docker-compose logs -t orders
```

#### Database Debugging

```bash
# Conectar ao banco Orders
docker-compose exec orders npm run db:studio

# Executar query manual
docker-compose exec orders npm run db:query "SELECT * FROM orders"
```

#### Message Queue Debugging

```bash
# Verificar filas no RabbitMQ
curl -u guest:guest http://localhost:15672/api/queues

# Verificar mensagens pendentes
curl -u guest:guest http://localhost:15672/api/queues/%2F/orders
```

### Performance Tuning

#### Database Optimization

- Connection pooling configurado
- Índices otimizados para queries frequentes
- Query analysis com EXPLAIN

#### Application Optimization

- Fastify configurado para alta performance
- Validação Zod otimizada
- Memory management com Node.js 22

#### Infrastructure Optimization

- ECS Fargate com auto-scaling
- ALB com health checks otimizados
- CloudWatch metrics para scaling decisions

## 🤝 Contribuição

### Padrões de Código

- **TypeScript**: Tipagem estrita habilitada
- **ESLint**: Linting automático
- **Prettier**: Formatação consistente
- **Conventional Commits**: Padrão de commits

### Processo de Contribuição

1. Fork do repositório
2. Criar branch feature (`git checkout -b feature/nova-funcionalidade`)
3. Commit das mudanças (`git commit -m 'feat: adiciona nova funcionalidade'`)
4. Push para branch (`git push origin feature/nova-funcionalidade`)
5. Abrir Pull Request

### Testes

```bash
# Executar testes unitários
npm test

# Executar testes de integração
npm run test:integration

# Coverage report
npm run test:coverage
```

### Documentação

- Manter README atualizado
- Documentar APIs com OpenAPI/Swagger
- Comentários em código complexo
- Exemplos de uso atualizados

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 🔗 Links Úteis

- [Documentação Fastify](https://www.fastify.io/)
- [Drizzle ORM Docs](https://orm.drizzle.team/)
- [OpenTelemetry Node.js](https://opentelemetry.io/docs/instrumentation/js/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials.html)
- [Kong Gateway](https://docs.konghq.com/)
- [Pulumi AWS](https://www.pulumi.com/docs/clouds/aws/)

---

**Desenvolvido** com ❤️ para demonstrar as melhores práticas em arquitetura de microsserviços com Node.**js**
