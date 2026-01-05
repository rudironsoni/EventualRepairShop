# EventualRepairShop Architecture

## Table of Contents

- [Overview](#overview)
- [Architectural Patterns](#architectural-patterns)
- [System Architecture](#system-architecture)
- [Service Details](#service-details)
- [Data Flow](#data-flow)
- [Technology Stack](#technology-stack)
- [Design Decisions](#design-decisions)

## Overview

EventualRepairShop is built using a microservices architecture following Domain-Driven Design (DDD) principles, Event-Driven Architecture (EDA), Event Sourcing, and Command Query Responsibility Segregation (CQRS) patterns.

### Key Characteristics

- **Distributed System**: Services are loosely coupled and independently deployable
- **Event-Driven**: Asynchronous communication via domain events
- **Eventually Consistent**: Services maintain their own data stores
- **Resilient**: Fault isolation between services
- **Scalable**: Services can be scaled independently

## Architectural Patterns

### 1. Domain-Driven Design (DDD)

Each microservice represents a bounded context with:
- **Aggregates**: Consistency boundaries (e.g., RepairOrder, Scheduling)
- **Entities**: Objects with identity
- **Value Objects**: Immutable objects without identity
- **Domain Events**: Capture state changes

### 2. Event Sourcing

Instead of storing current state, we store all domain events:

```
Traditional Approach:
[Current State] → Latest row in database

Event Sourcing:
[Event 1] → [Event 2] → [Event 3] → [Current State]
```

**Benefits:**
- Complete audit trail
- Time travel (reconstruct state at any point)
- Event replay for debugging
- New projections from historical data

**Implementation:**
- Events stored in SQL Server event store
- Each aggregate has its own event stream
- Events are immutable once stored

### 3. CQRS (Command Query Responsibility Segregation)

Separate models for writes (commands) and reads (queries):

```
Commands (Write)           Queries (Read)
     ↓                           ↑
[Aggregate Root]            [Read Model]
     ↓                           ↑
[Event Store]  → Events → [Projections]
```

**Benefits:**
- Optimized read and write models
- Independent scaling
- Complex queries without impacting write performance

### 4. Event-Driven Architecture

Services communicate via asynchronous events through RabbitMQ:

```
Service A              RabbitMQ            Service B
   ↓                      ↓                   ↓
[Publish Event] → [Message Queue] → [Consume Event]
```

**Benefits:**
- Loose coupling
- Temporal decoupling (services don't need to be available simultaneously)
- Scalability
- Resilience

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    External Clients                      │
│              (Web, Mobile, Third-party)                  │
└────────────────────┬────────────────────────────────────┘
                     │ HTTPS/REST
┌────────────────────▼────────────────────────────────────┐
│                       WebAPI                             │
│              (API Gateway / BFF)                         │
│    - REST endpoints                                      │
│    - Request validation                                  │
│    - Command publishing                                  │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┬──────────────┬────────────┐
        │                         │              │            │
┌───────▼────────┐  ┌─────────▼────────┐  ┌────▼──────┐  ┌─▼────────┐
│  RepairOrder   │  │   Scheduling     │  │  Customer │  │ Warehouse│
│    Service     │  │    Service       │  │  Service  │  │ Service  │
│                │  │                  │  │           │  │          │
│ - Domain       │  │ - Domain         │  │ - Domain  │  │ - Domain │
│ - Application  │  │ - Application    │  │ - App     │  │ - App    │
│ - Infra        │  │ - Infra          │  │ - Infra   │  │ - Infra  │
└────────┬───────┘  └────────┬─────────┘  └─────┬─────┘  └────┬─────┘
         │                   │                   │             │
         └───────────────────┴───────────────────┴─────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
      ┌───────▼────────┐          ┌────────▼────────┐
      │   RabbitMQ     │          │   SQL Server    │
      │ (Message Bus)  │          │  (Event Store)  │
      │                │          │                 │
      │ - Exchanges    │          │ - Event Streams │
      │ - Queues       │          │ - Per Service   │
      └────────────────┘          └─────────────────┘
```

### Service Layer Architecture

Each microservice follows Clean Architecture / Onion Architecture:

```
┌─────────────────────────────────────────────────────────┐
│                    WorkerService                         │
│              (Entry Point / Host)                        │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                  Application Layer                       │
│   - Use Cases (Interactors)                             │
│   - Application Services                                │
│   - Command/Event Handlers                              │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                   Domain Layer                           │
│   - Aggregates (Business Logic)                         │
│   - Entities                                             │
│   - Value Objects                                        │
│   - Domain Events                                        │
│   - Domain Services                                      │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│               Infrastructure Layer                       │
│   - Event Store (EF Core)                               │
│   - Message Bus (MassTransit)                           │
│   - External Services                                    │
└─────────────────────────────────────────────────────────┘
```

**Dependency Rule:** Dependencies point inward. Domain has no dependencies on outer layers.

## Service Details

### RepairOrder Service

**Bounded Context:** Managing repair orders and their lifecycle

**Aggregates:**
- `RepairOrder` - Root aggregate for repair operations

**Domain Events:**
- `RepairOrderPlaced`
- `RepairOrderStarted`
- `RepairOrderCompleted`
- `RepairOrderCancelled`
- `RepairItemAdded`

**Commands:**
- `PlaceRepairOrder`
- `StartRepair`
- `CompleteRepair`
- `CancelRepairOrder`
- `AddRepairItem`

**Responsibilities:**
- Accept new repair orders
- Track repair progress
- Manage repair items/parts
- Update repair status
- Emit domain events for cross-service coordination

### Scheduling Service

**Bounded Context:** Managing repair appointments and schedules

**Aggregates:**
- `Scheduling` - Root aggregate for appointments

**Domain Events:**
- `SchedulingRegistered`
- `SchedulingConfirmed`
- `SchedulingRescheduled`
- `SchedulingCancelled`

**Commands:**
- `RegisterScheduling`
- `ConfirmScheduling`
- `RescheduleAppointment`
- `CancelScheduling`

**Responsibilities:**
- Register new appointments
- Manage scheduling conflicts
- Track customer availability
- Coordinate with repair orders

### Customer Service

**Bounded Context:** Customer information and relationship management

**Status:** 🚧 Under Development

**Planned Features:**
- Customer registration
- Contact information management
- Customer history
- Preferences and notes

### Warehouse Service

**Bounded Context:** Inventory and parts management

**Status:** 🚧 Under Development

**Planned Features:**
- Parts inventory tracking
- Stock levels
- Parts ordering
- Inventory allocation for repairs

### WebAPI Service

**Responsibilities:**
- API Gateway / Backend for Frontend (BFF)
- REST endpoint exposure
- Request validation (FluentValidation)
- Command routing to appropriate services
- API documentation (Swagger/OpenAPI)
- CORS handling
- Correlation ID propagation

## Data Flow

### Command Flow (Write Operation)

```
1. Client → HTTP POST → WebAPI
2. WebAPI validates request
3. WebAPI publishes command to RabbitMQ
4. Service consumes command
5. Service loads aggregate from event store
6. Aggregate executes business logic
7. Aggregate produces domain events
8. Service stores events in event store
9. Service publishes integration events to RabbitMQ
10. Other services react to integration events
```

### Query Flow (Read Operation)

```
1. Client → HTTP GET → WebAPI
2. WebAPI queries read model
3. Read model returns data
4. WebAPI returns response
```

### Event Flow

```
┌──────────────┐
│ Service A    │
│ Aggregate    │
└──────┬───────┘
       │ Produces
       ▼
┌──────────────┐
│ Domain Event │
│ (stored)     │
└──────┬───────┘
       │ Published as
       ▼
┌──────────────┐
│ Integration  │
│ Event        │
└──────┬───────┘
       │ Via RabbitMQ
       ▼
┌──────────────┐
│ Service B    │
│ Consumer     │
└──────────────┘
```

## Technology Stack

### Core Framework
- **.NET 7.0** - Application framework
- **C# 11** - Programming language

### Data & Messaging
- **SQL Server** - Event store persistence
- **Entity Framework Core 7** - ORM for event store
- **RabbitMQ** - Message broker
- **MassTransit 8** - Distributed application framework

### Cross-Cutting
- **Serilog** - Structured logging
- **FluentValidation** - Input validation
- **Swashbuckle** - API documentation
- **CorrelationId** - Request tracking

### Infrastructure
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration

## Design Decisions

### Why Event Sourcing?

**Pros:**
- Complete audit trail of all changes
- Time travel and debugging capabilities
- Ability to create new projections from history
- Natural fit for event-driven architecture

**Cons:**
- Increased complexity
- Schema evolution challenges
- Eventual consistency

**Decision:** Benefits outweigh costs for a repair shop system where audit trail and event replay are valuable.

### Why CQRS?

**Pros:**
- Optimized read and write models
- Independent scaling
- Simplified complex domain logic

**Cons:**
- Additional complexity
- Eventual consistency between write and read models

**Decision:** Worth it for this system to separate complex write operations from read queries.

### Why Microservices?

**Pros:**
- Independent deployability
- Technology flexibility
- Team autonomy
- Fault isolation

**Cons:**
- Distributed system complexity
- Network latency
- Data consistency challenges

**Decision:** Appropriate for this domain which has clear bounded contexts (orders, scheduling, inventory).

### Why RabbitMQ?

**Alternatives considered:** Apache Kafka, Azure Service Bus, AWS SNS/SQS

**Decision:** RabbitMQ chosen for:
- Mature and stable
- Good MassTransit integration
- Easy local development
- Lower operational complexity than Kafka
- No cloud vendor lock-in

### Why SQL Server for Event Store?

**Alternatives considered:** EventStoreDB, PostgreSQL, MongoDB

**Decision:** SQL Server chosen for:
- Team familiarity
- Transactional guarantees
- Good tooling support
- ACID compliance for events

## Future Considerations

### Short Term
- Implement comprehensive test suite
- Add API versioning
- Implement authentication/authorization
- Add health checks and monitoring

### Medium Term
- Implement read model projections
- Add caching layer (Redis)
- Implement saga pattern for long-running processes
- Add API rate limiting

### Long Term
- Migrate to .NET 8/9 LTS
- Consider Kubernetes for orchestration
- Implement service mesh (Istio/Linkerd)
- Add distributed tracing (OpenTelemetry/Jaeger)
- Implement CQRS read models with dedicated databases

## References

- [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)
- [Implementing Domain-Driven Design by Vaughn Vernon](https://www.oreilly.com/library/view/implementing-domain-driven-design/9780133039900/)
- [Building Microservices by Sam Newman](https://www.oreilly.com/library/view/building-microservices-2nd/9781492034018/)
- [Event Sourcing Pattern](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)
- [EventualShop by Antonio Falcão Jr.](https://github.com/AntonioFalcaoJr/EventualShop)
