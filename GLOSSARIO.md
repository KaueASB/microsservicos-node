# Glossário - Microsserviços

## A

### **API Gateway**

- Ponto de entrada único para todas as requisições de clientes em uma arquitetura de microsserviços. Atua como proxy reverso, roteando requisições para os serviços apropriados e fornecendo funcionalidades transversais como autenticação, rate limiting e logging.

### **AMQP (Advanced Message Queuing Protocol)**

- Protocolo de mensageria que permite comunicação assíncrona entre aplicações através de filas de mensagens. Utilizado para desacoplar serviços em arquiteturas distribuídas.

### **Auto-instrumentação**

- Processo automatizado de adicionar observabilidade (traces, métricas, logs) ao código sem modificações manuais extensivas, utilizando bibliotecas como OpenTelemetry.

### **AWS ECS (Elastic Container Service)**

- Serviço de orquestração de containers da Amazon Web Services que permite executar e escalar aplicações containerizadas.

## B

### **Broker de Mensagens**

- Sistema intermediário que facilita a comunicação entre diferentes serviços através do envio e recebimento de mensagens. Exemplos: RabbitMQ, Apache Kafka.

### **Business Logic**

- Conjunto de regras e processos que definem como os dados são criados, armazenados e modificados em um sistema, representando as regras de negócio específicas.

## C

### **Container**

- Unidade de software que empacota código e suas dependências para que a aplicação execute de forma rápida e confiável em diferentes ambientes computacionais.

### **CORS (Cross-Origin Resource Sharing)**

- Mecanismo que permite que recursos de uma página web sejam acessados por outra página de um domínio diferente, controlando o acesso entre origens distintas.

### **Circuit Breaker**

- Padrão de design que previne falhas em cascata em sistemas distribuídos, interrompendo chamadas para serviços que estão falhando.

## D

### **Docker**

- Plataforma de containerização que permite empacotar aplicações e suas dependências em containers leves e portáveis.

### **Docker Compose**

- Ferramenta para definir e executar aplicações Docker multi-container usando arquivos YAML para configuração.

### **Distributed Tracing**

- Técnica de observabilidade que rastreia requisições através de múltiplos serviços em uma arquitetura distribuída, permitindo identificar gargalos e problemas de performance.

### **Drizzle ORM**

- Object-Relational Mapping (ORM) TypeScript-first que fornece uma interface type-safe para interagir com bancos de dados.

## E

### **Event-Driven Architecture**

- Padrão arquitetural onde componentes se comunicam através da produção e consumo de eventos, promovendo baixo acoplamento.

### **ECS Fargate**

- Mecanismo de computação serverless para containers que permite executar containers sem gerenciar servidores ou clusters.

## F

### **Fastify**

- Framework web rápido e eficiente para Node.js, focado em performance e baixo overhead, com suporte nativo a TypeScript.

### **Fault Tolerance**

- Capacidade de um sistema continuar operando corretamente mesmo quando alguns de seus componentes falham.

## G

### **Gateway Pattern**

- Padrão que fornece um ponto de entrada único para um sistema, abstraindo a complexidade interna e fornecendo uma interface simplificada.

### **Grafana**

- Plataforma de observabilidade que permite visualizar, consultar e alertar sobre métricas e logs de sistemas distribuídos.

## H

### **Health Check**

- Endpoint ou mecanismo que verifica se um serviço está funcionando corretamente, usado por load balancers e orquestradores.

### **Horizontal Scaling**

- Estratégia de escalabilidade que adiciona mais instâncias de um serviço para lidar com maior carga.

## I

### **Infrastructure as Code (IaC)**

- Prática de gerenciar e provisionar infraestrutura através de código, permitindo versionamento e automação.

### **Idempotência**

- Propriedade de operações que podem ser executadas múltiplas vezes sem alterar o resultado além da primeira execução.

## J

### **Jaeger**

- Sistema de distributed tracing open-source usado para monitorar e troubleshoot transações em arquiteturas de microsserviços.

## K

### **Kong**

- API Gateway open-source que fornece funcionalidades como roteamento, autenticação, rate limiting e observabilidade.

## L

### **Load Balancer**

- Componente que distribui requisições de entrada entre múltiplas instâncias de um serviço para otimizar utilização de recursos.

### **Loose Coupling**

- Princípio de design onde componentes têm dependências mínimas entre si, facilitando manutenção e evolução independente.

## M

### **Message Queue**

- Sistema que permite comunicação assíncrona entre serviços através do armazenamento temporário de mensagens.

### **Microsserviços**

- Padrão arquitetural que estrutura uma aplicação como uma coleção de serviços pequenos, independentes e fracamente acoplados.

### **Microservice Pattern**

- Conjunto de práticas e padrões para implementar arquiteturas de microsserviços eficazes.

## N

### **Node.js**

- Runtime JavaScript construído no motor V8 do Chrome, permitindo execução de JavaScript no servidor.

## O

### **OpenTelemetry (OTEL)**

- Framework de observabilidade que fornece APIs, bibliotecas e ferramentas para coletar, processar e exportar dados de telemetria.

### **Orchestration**

- Coordenação automatizada de múltiplos serviços ou containers para trabalhar juntos como um sistema coeso.

## P

### **PostgreSQL**

- Sistema de gerenciamento de banco de dados relacional open-source conhecido por sua robustez e conformidade com padrões SQL.

### **Pulumi**

- Plataforma de Infrastructure as Code que permite definir infraestrutura usando linguagens de programação familiares.

### **Publisher-Subscriber Pattern**

- Padrão de mensageria onde publishers enviam mensagens sem conhecer os subscribers, e subscribers recebem mensagens de interesse.

## Q

### **Queue**

- Estrutura de dados FIFO (First In, First Out) usada para armazenar mensagens temporariamente entre serviços.

## R

### **RabbitMQ**

- Broker de mensagens open-source que implementa o protocolo AMQP, facilitando comunicação assíncrona entre aplicações.

### **Resilience**

- Capacidade de um sistema se recuperar de falhas e continuar operando sob condições adversas.

### **REST API**

- Interface de programação que segue os princípios REST (Representational State Transfer) para comunicação entre sistemas.

## S

### **Service Discovery**

- Mecanismo que permite serviços encontrarem e se comunicarem uns com os outros dinamicamente em uma arquitetura distribuída.

### **Service Mesh**

- Camada de infraestrutura dedicada para facilitar comunicação serviço-a-serviço com recursos como load balancing, criptografia e observabilidade.

### **Span**

- Unidade básica de trabalho em distributed tracing que representa uma operação individual dentro de um trace.

## T

### **Trace**

- Representação de uma requisição completa através de múltiplos serviços em um sistema distribuído.

### **TypeScript**

- Superset do JavaScript que adiciona tipagem estática, melhorando a qualidade e manutenibilidade do código.

## U

### **Upstream/Downstream**

- Termos que descrevem a direção do fluxo de dados: upstream refere-se à origem dos dados, downstream ao destino.

## V

### **Vertical Scaling**

- Estratégia de escalabilidade que aumenta os recursos (CPU, memória) de uma instância existente.

## W

### **Webhook**

- Método de comunicação onde uma aplicação fornece informações para outras aplicações em tempo real através de callbacks HTTP.

## Z

### **Zod**

- Biblioteca TypeScript-first para validação e parsing de schemas, garantindo type safety em runtime.

### **Zero Downtime Deployment**

- Estratégia de deployment que permite atualizações de aplicação sem interrupção do serviço.
