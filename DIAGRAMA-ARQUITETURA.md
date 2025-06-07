# Diagrama de Arquitetura - Microsserviços Node.js

## Visão Geral da Arquitetura

```mermaid
graph TB
    %% Cliente e API Gateway
    Client[👤 Cliente] --> ALB[🔄 AWS Application Load Balancer]
    ALB --> Kong[🚪 Kong API Gateway<br/>Port: 8000]
    
    %% Microsserviços
    Kong --> Orders[📦 Orders Service<br/>Port: 3333<br/>Node.js + Fastify]
    Kong --> Invoices[🧾 Invoices Service<br/>Port: 3334<br/>Node.js + Fastify]
    
    %% Bancos de Dados
    Orders --> OrdersDB[(🗄️ Orders Database<br/>PostgreSQL<br/>Neon Cloud)]
    Invoices --> InvoicesDB[(🗄️ Invoices Database<br/>PostgreSQL<br/>Neon Cloud)]
    
    %% Message Broker
    Orders --> RabbitMQ[🐰 RabbitMQ<br/>Port: 5672<br/>Management: 15672]
    RabbitMQ --> Invoices
    
    %% Observabilidade
    Orders --> OTEL[📊 OpenTelemetry]
    Invoices --> OTEL
    OTEL --> Grafana[📈 Grafana Cloud<br/>Metrics & Alerts]
    OTEL --> Jaeger[🔍 Jaeger<br/>Distributed Tracing<br/>Port: 16686]
    
    %% Infraestrutura AWS
    subgraph "AWS ECS Fargate"
        Orders
        Invoices
        Kong
    end
    
    subgraph "Observability Stack"
        Jaeger
        Grafana
        OTEL
    end
    
    %% Styling
    classDef service fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef database fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef infrastructure fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef observability fill:#fff3e0,stroke:#e65100,stroke-width:2px
    
    class Orders,Invoices,Kong service
    class OrdersDB,InvoicesDB database
    class ALB,RabbitMQ infrastructure
    class Jaeger,Grafana,OTEL observability
```

## Fluxo de Dados Detalhado

```mermaid
sequenceDiagram
    participant C as Cliente
    participant ALB as Load Balancer
    participant K as Kong Gateway
    participant O as Orders Service
    participant R as RabbitMQ
    participant I as Invoices Service
    participant ODB as Orders DB
    participant IDB as Invoices DB
    participant T as Tracing (Jaeger)
    participant G as Grafana

    %% Criação de Pedido
    C->>ALB: POST /orders {amount: 29.99}
    ALB->>K: Forward request
    K->>O: Route to Orders Service
    
    %% Processamento no Orders Service
    activate O
    O->>T: Start trace span
    O->>ODB: INSERT order
    ODB-->>O: Order created (UUID)
    O->>R: Publish OrderCreated event
    O->>T: Add trace attributes
    O->>G: Send metrics
    O-->>K: 201 Created
    deactivate O
    
    K-->>ALB: Response
    ALB-->>C: 201 Created
    
    %% Processamento Assíncrono
    R->>I: Consume OrderCreated event
    activate I
    I->>T: Continue trace span
    I->>IDB: INSERT invoice
    IDB-->>I: Invoice created
    I->>R: ACK message
    I->>G: Send metrics
    I->>T: End trace span
    deactivate I
    
    %% Observabilidade
    T->>G: Export trace data
    G->>G: Process metrics & alerts
```

## Arquitetura de Deployment

```mermaid
graph TB
    %% GitHub e CI/CD
    Dev[👨‍💻 Developer] --> GitHub[📚 GitHub Repository]
    GitHub --> Actions[⚙️ GitHub Actions<br/>CI/CD Pipeline]
    
    %% Build Process
    Actions --> Build[🔨 Build & Test]
    Build --> DockerBuild[🐳 Docker Build]
    DockerBuild --> ECR[📦 AWS ECR<br/>Container Registry]
    
    %% Infrastructure
    Actions --> Pulumi[🏗️ Pulumi<br/>Infrastructure as Code]
    Pulumi --> AWS[☁️ AWS Cloud]
    
    %% AWS Resources
    subgraph "AWS Infrastructure"
        VPC[🌐 VPC<br/>Virtual Private Cloud]
        ALB[⚖️ Application Load Balancer]
        ECS[🚢 ECS Fargate Cluster]
        SG[🛡️ Security Groups]
        CW[📊 CloudWatch]
        
        VPC --> ALB
        VPC --> ECS
        VPC --> SG
        ECS --> CW
    end
    
    %% Services Deployment
    ECR --> ECS
    ECS --> OrdersTask[📦 Orders Task Definition]
    ECS --> InvoicesTask[🧾 Invoices Task Definition]
    ECS --> KongTask[🚪 Kong Task Definition]
    
    %% External Services
    OrdersTask --> NeonOrders[(🗄️ Neon PostgreSQL<br/>Orders DB)]
    InvoicesTask --> NeonInvoices[(🗄️ Neon PostgreSQL<br/>Invoices DB)]
    
    %% Observability
    OrdersTask --> GrafanaCloud[📈 Grafana Cloud]
    InvoicesTask --> GrafanaCloud
    KongTask --> GrafanaCloud
    
    %% Styling
    classDef cicd fill:#e3f2fd,stroke:#0277bd,stroke-width:2px
    classDef aws fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef service fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef external fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    
    class GitHub,Actions,Build,DockerBuild,Pulumi cicd
    class AWS,VPC,ALB,ECS,SG,CW,ECR aws
    class OrdersTask,InvoicesTask,KongTask service
    class NeonOrders,NeonInvoices,GrafanaCloud external
```

## Estrutura de Dados e Contratos

```mermaid
erDiagram
    %% Orders Service Schema
    CUSTOMERS_ORDERS {
        string id PK
        string name
        string email
        timestamp created_at
    }
    
    ORDERS {
        string id PK
        string customer_id FK
        integer amount
        enum status
        timestamp created_at
    }
    
    %% Invoices Service Schema
    CUSTOMERS_INVOICES {
        string id PK
        string name
        string email
        timestamp created_at
    }
    
    INVOICES {
        string id PK
        string order_id
        timestamp created_at
    }
    
    %% Message Contracts
    ORDER_CREATED_MESSAGE {
        string orderId
        number amount
        object customer
        string customer_id
    }
    
    %% Relationships
    CUSTOMERS_ORDERS ||--o{ ORDERS : "has many"
    ORDERS ||--|| ORDER_CREATED_MESSAGE : "generates"
    ORDER_CREATED_MESSAGE ||--|| INVOICES : "creates"
```

## Fluxo de Observabilidade

```mermaid
graph LR
    %% Services
    Orders[📦 Orders Service] --> OTEL_Orders[📊 OpenTelemetry<br/>Auto-Instrumentation]
    Invoices[🧾 Invoices Service] --> OTEL_Invoices[📊 OpenTelemetry<br/>Auto-Instrumentation]
    Kong[🚪 Kong Gateway] --> Kong_Logs[📝 Kong Logs]
    
    %% Instrumentation Details
    OTEL_Orders --> HTTP_Traces[🌐 HTTP Traces]
    OTEL_Orders --> DB_Traces[🗄️ Database Traces]
    OTEL_Orders --> AMQP_Traces[🐰 RabbitMQ Traces]
    OTEL_Orders --> Custom_Spans[⭐ Custom Spans]
    
    OTEL_Invoices --> HTTP_Traces2[🌐 HTTP Traces]
    OTEL_Invoices --> DB_Traces2[🗄️ Database Traces]
    OTEL_Invoices --> AMQP_Traces2[🐰 RabbitMQ Traces]
    
    %% Export Destinations
    HTTP_Traces --> Grafana[📈 Grafana Cloud]
    DB_Traces --> Grafana
    AMQP_Traces --> Grafana
    Custom_Spans --> Grafana
    
    HTTP_Traces2 --> Grafana
    DB_Traces2 --> Grafana
    AMQP_Traces2 --> Grafana
    
    Kong_Logs --> Grafana
    
    %% Local Development
    OTEL_Orders --> Jaeger[🔍 Jaeger<br/>Local Tracing]
    OTEL_Invoices --> Jaeger
    
    %% Monitoring Outputs
    Grafana --> Dashboards[📊 Dashboards]
    Grafana --> Alerts[🚨 Alerts]
    Grafana --> SLOs[🎯 SLOs/SLIs]
    
    Jaeger --> Trace_Analysis[🔍 Trace Analysis]
    Jaeger --> Performance_Debug[⚡ Performance Debug]
    
    %% Styling
    classDef service fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef telemetry fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef output fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    
    class Orders,Invoices,Kong service
    class OTEL_Orders,OTEL_Invoices,HTTP_Traces,DB_Traces,AMQP_Traces,Custom_Spans,HTTP_Traces2,DB_Traces2,AMQP_Traces2,Kong_Logs telemetry
    class Grafana,Jaeger,Dashboards,Alerts,SLOs,Trace_Analysis,Performance_Debug output
```

## Padrões de Comunicação

```mermaid
graph TB
    %% Synchronous Communication
    subgraph "Comunicação Síncrona"
        Client_Sync[👤 Cliente] --> Kong_Sync[🚪 Kong Gateway]
        Kong_Sync --> Orders_Sync[📦 Orders Service]
        Kong_Sync --> Invoices_Sync[🧾 Invoices Service]
        
        Orders_Sync --> OrdersDB_Sync[(🗄️ Orders DB)]
        Invoices_Sync --> InvoicesDB_Sync[(🗄️ Invoices DB)]
    end
    
    %% Asynchronous Communication
    subgraph "Comunicação Assíncrona"
        Orders_Async[📦 Orders Service] --> RabbitMQ_Async[🐰 RabbitMQ<br/>Exchange: orders<br/>Queue: orders]
        RabbitMQ_Async --> Invoices_Async[🧾 Invoices Service]
        
        %% Message Flow
        Orders_Async -.->|Publish| OrderCreated[📨 OrderCreated Event]
        OrderCreated -.->|Route| RabbitMQ_Async
        RabbitMQ_Async -.->|Consume| Invoices_Async
        Invoices_Async -.->|ACK| RabbitMQ_Async
    end
    
    %% Health Checks
    subgraph "Health Monitoring"
        ALB_Health[⚖️ Load Balancer] -.->|Health Check| Orders_Health[📦 /health]
        ALB_Health -.->|Health Check| Invoices_Health[🧾 /health]
        ALB_Health -.->|Health Check| Kong_Health[🚪 /health]
    end
    
    %% Styling
    classDef sync fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef async fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef health fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    
    class Client_Sync,Kong_Sync,Orders_Sync,Invoices_Sync,OrdersDB_Sync,InvoicesDB_Sync sync
    class Orders_Async,RabbitMQ_Async,Invoices_Async,OrderCreated async
    class ALB_Health,Orders_Health,Invoices_Health,Kong_Health health
```

## Estratégia de Deployment

```mermaid
graph TB
    %% Development Environment
    subgraph "Ambiente de Desenvolvimento"
        Dev_Docker[🐳 Docker Compose]
        Dev_Services[📦 Services Locais]
        Dev_RabbitMQ[🐰 RabbitMQ Local]
        Dev_Jaeger[🔍 Jaeger Local]
        
        Dev_Docker --> Dev_Services
        Dev_Docker --> Dev_RabbitMQ
        Dev_Docker --> Dev_Jaeger
    end
    
    %% Staging Environment
    subgraph "Ambiente de Staging"
        Stage_ECS[🚢 ECS Fargate]
        Stage_ALB[⚖️ Application LB]
        Stage_RDS[🗄️ RDS PostgreSQL]
        Stage_Grafana[📈 Grafana Cloud]
        
        Stage_ALB --> Stage_ECS
        Stage_ECS --> Stage_RDS
        Stage_ECS --> Stage_Grafana
    end
    
    %% Production Environment
    subgraph "Ambiente de Produção"
        Prod_ECS[🚢 ECS Fargate<br/>Multi-AZ]
        Prod_ALB[⚖️ Application LB<br/>Multi-AZ]
        Prod_RDS[🗄️ Neon PostgreSQL<br/>High Availability]
        Prod_Grafana[📈 Grafana Cloud<br/>Full Monitoring]
        Prod_AutoScaling[📈 Auto Scaling]
        
        Prod_ALB --> Prod_ECS
        Prod_ECS --> Prod_RDS
        Prod_ECS --> Prod_Grafana
        Prod_ECS --> Prod_AutoScaling
    end
    
    %% Deployment Pipeline
    GitHub[📚 GitHub] --> Actions[⚙️ GitHub Actions]
    Actions --> Test[🧪 Tests]
    Test --> Build[🔨 Build Images]
    Build --> Deploy_Stage[🚀 Deploy Staging]
    Deploy_Stage --> Deploy_Prod[🚀 Deploy Production]
    
    Deploy_Stage --> Stage_ECS
    Deploy_Prod --> Prod_ECS
    
    %% Styling
    classDef dev fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef stage fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef prod fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    classDef pipeline fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    
    class Dev_Docker,Dev_Services,Dev_RabbitMQ,Dev_Jaeger dev
    class Stage_ECS,Stage_ALB,Stage_RDS,Stage_Grafana stage
    class Prod_ECS,Prod_ALB,Prod_RDS,Prod_Grafana,Prod_AutoScaling prod
    class GitHub,Actions,Test,Build,Deploy_Stage,Deploy_Prod pipeline
```

---

## Legenda dos Diagramas

### Símbolos Utilizados
- 👤 **Cliente**: Usuário final ou sistema externo
- 🚪 **API Gateway**: Ponto de entrada único (Kong)
- 📦 **Orders Service**: Microsserviço de pedidos
- 🧾 **Invoices Service**: Microsserviço de faturas
- 🐰 **RabbitMQ**: Message broker para comunicação assíncrona
- 🗄️ **Database**: Bancos de dados PostgreSQL
- 📊 **OpenTelemetry**: Framework de observabilidade
- 📈 **Grafana**: Plataforma de monitoramento
- 🔍 **Jaeger**: Sistema de distributed tracing
- ⚖️ **Load Balancer**: Balanceador de carga AWS
- 🚢 **ECS**: Amazon Elastic Container Service
- 🐳 **Docker**: Containerização
- ⚙️ **CI/CD**: Pipeline de integração e deployment

### Cores dos Componentes
- **Azul**: Serviços de aplicação
- **Roxo**: Bancos de dados
- **Verde**: Infraestrutura
- **Laranja**: Observabilidade
- **Vermelho**: Produção
- **Amarelo**: Staging

Estes diagramas capturam a arquitetura completa do projeto, desde o desenvolvimento local até o deployment em produção, incluindo todos os aspectos de observabilidade, comunicação entre serviços e estratégias de deployment.