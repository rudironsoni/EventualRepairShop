# Eventual Repair Shop

[![.NET](https://img.shields.io/badge/.NET-7.0-512BD4)](https://dotnet.microsoft.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

A microservices-based repair shop management system demonstrating Event-Driven Architecture (EDA), Event Sourcing, and CQRS patterns using .NET.

## 📋 Table of Contents

- [About](#about)
- [Architecture](#architecture)
- [Services](#services)
- [Technologies](#technologies)
- [Getting Started](#getting-started)
- [Development](#development)
- [API Documentation](#api-documentation)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## 🎯 About

EventualRepairShop is a distributed system designed to manage repair shop operations including customer management, repair order processing, scheduling, and inventory/warehouse management. The system demonstrates modern architectural patterns and best practices for building scalable, resilient microservices.

## 🏗 Architecture

The system follows a microservices architecture with the following key principles:

- **Event-Driven Architecture (EDA)**: Services communicate asynchronously via events
- **Event Sourcing**: Domain events are stored as the source of truth
- **CQRS**: Separate read and write models for optimal performance
- **Domain-Driven Design (DDD)**: Rich domain models with clear bounded contexts

### System Overview

```
┌─────────────────┐
│    WebAPI       │  ← REST API Gateway
└────────┬────────┘
         │
    ┌────┴─────┬──────────┬───────────┐
    │          │          │           │
┌───▼────┐ ┌──▼─────┐ ┌──▼──────┐ ┌─▼────────┐
│Repair  │ │Schedule│ │Customer │ │Warehouse │
│Order   │ │        │ │         │ │          │
└───┬────┘ └───┬────┘ └───┬─────┘ └────┬─────┘
    │          │           │            │
    └──────────┴───────────┴────────────┘
                    │
              ┌─────▼──────┐
              │  RabbitMQ  │  ← Message Bus
              └─────┬──────┘
                    │
              ┌─────▼──────┐
              │ SQL Server │  ← Event Store
              └────────────┘
```

## 🔧 Services

### RepairOrder Service
Manages the lifecycle of repair orders from creation to completion.

**Responsibilities:**
- Place new repair orders
- Track repair order status
- Manage repair order items
- Emit repair order events

### Scheduling Service
Handles appointment scheduling for repairs.

**Responsibilities:**
- Register repair schedules
- Manage scheduled dates
- Track customer and device information
- Coordinate with repair orders

### Customer Service
Manages customer information and interactions.

**Status:** ⚠️ Under Development

### Warehouse Service
Manages inventory and parts for repairs.

**Status:** ⚠️ Under Development

### WebAPI
REST API gateway providing external access to the system.

**Features:**
- Versioned APIs (v1)
- Swagger/OpenAPI documentation
- FluentValidation integration
- CORS support

## 🛠 Technologies

### Backend
- **.NET 7.0** - Application framework
- **C# 11** - Programming language
- **ASP.NET Core** - Web API framework
- **Entity Framework Core 7.0** - ORM for event store

### Messaging & Events
- **MassTransit 8.0** - Distributed application framework
- **RabbitMQ** - Message broker
- **Event Sourcing** - Domain event persistence

### Data Storage
- **SQL Server** - Event store database
- **Entity Framework Core** - Data access

### Cross-Cutting Concerns
- **Serilog** - Structured logging
- **FluentValidation** - Input validation
- **Swashbuckle** - API documentation
- **CorrelationId** - Distributed tracing

### Development Tools
- **Docker & Docker Compose** - Containerization
- **Visual Studio / Rider** - IDEs

## 🚀 Getting Started

### Prerequisites

- [.NET 7.0 SDK](https://dotnet.microsoft.com/download/dotnet/7.0) or later
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Visual Studio 2022](https://visualstudio.microsoft.com/) or [JetBrains Rider](https://www.jetbrains.com/rider/) (optional)
- [Git](https://git-scm.com/)

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/rudironsoni/EventualRepairShop.git
   cd EventualRepairShop
   ```

2. **Start infrastructure services**
   ```bash
   docker-compose -f docker-compose.Development.Infrastructure.yaml up -d
   ```

   This will start:
   - SQL Server (port 1433)
   - RabbitMQ (port 5672, management UI on 15672)

3. **Restore NuGet packages**
   ```bash
   dotnet restore
   ```

4. **Build the solution**
   ```bash
   dotnet build
   ```

5. **Apply database migrations**
   ```bash
   # RepairOrder service
   cd src/Services/RepairOrder/Infrastructure.EventStore
   dotnet ef database update
   
   # Scheduling service
   cd ../../Scheduling/Infrastructure.EventStore
   dotnet ef database update
   ```

6. **Run the services**
   
   In separate terminal windows:
   
   ```bash
   # WebAPI
   cd src/Web/WebAPI
   dotnet run
   
   # RepairOrder Worker
   cd src/Services/RepairOrder/WorkerService
   dotnet run
   
   # Scheduling Worker
   cd src/Services/Scheduling/WorkerService
   dotnet run
   ```

7. **Access the API**
   - Swagger UI: http://localhost:5000/swagger (adjust port as needed)
   - RabbitMQ Management: http://localhost:15672 (guest/guest)

## 💻 Development

### Project Structure

```
EventualRepairShop/
├── src/
│   ├── Contracts/              # Shared contracts and DTOs
│   ├── Services/
│   │   ├── RepairOrder/
│   │   │   ├── Application/    # Use cases and services
│   │   │   ├── Domain/         # Domain models and events
│   │   │   ├── Infrastructure.EventStore/
│   │   │   ├── Infrastructure.MessageBus/
│   │   │   └── WorkerService/  # Background worker
│   │   ├── Scheduling/
│   │   ├── Customer/           # Under development
│   │   └── Warehouse/          # Under development
│   └── Web/
│       └── WebAPI/             # API Gateway
├── test/                       # Test projects (to be added)
├── docker-compose.*.yaml       # Docker compositions
└── EventualRepairShop.sln
```

### Building

```bash
# Build all projects
dotnet build

# Build specific project
dotnet build src/Web/WebAPI/WebAPI.csproj

# Build in Release mode
dotnet build -c Release
```

### Running Tests

⚠️ **Test projects are not yet implemented.**

```bash
# Run all tests (when available)
dotnet test

# Run with coverage
dotnet test /p:CollectCoverage=true
```

### Database Migrations

```bash
# Add new migration
cd src/Services/RepairOrder/Infrastructure.EventStore
dotnet ef migrations add MigrationName

# Update database
dotnet ef database update

# Revert migration
dotnet ef database update PreviousMigrationName
```

### Code Style

This project follows:
- Microsoft C# coding conventions
- StyleCop rules (where configured)
- Nullable reference types enabled

## 📖 API Documentation

When running the WebAPI, interactive API documentation is available via Swagger UI:

**Development:** http://localhost:[port]/swagger

### Example Endpoints

#### RepairOrders API
- `POST /api/v1/repair-orders` - Place a new repair order
- `GET /api/v1/repair-orders/{id}` - Get repair order details

#### Schedulings API
- `POST /api/v1/schedulings` - Register a new schedule
- `GET /api/v1/schedulings/{id}` - Get schedule details

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guidelines](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

### Quick Start for Contributors

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

This project was heavily influenced by knowledge gathered from:

- [**Kauan Schumacher**](https://github.com/kauanschumacher) - For introducing me to complex concepts of Event-Driven Architecture and teaching me a lot
- [**Antonio Falcão Jr.**](https://github.com/AntonioFalcaoJr) - For the incredible [EventualShop](https://github.com/AntonioFalcaoJr/EventualShop) project that inspired many concepts here

This is my way to say thanks for what I've learned from both of you.

## 📞 Support

For questions, issues, or suggestions:
- Open an [Issue](https://github.com/rudironsoni/EventualRepairShop/issues)
- Start a [Discussion](https://github.com/rudironsoni/EventualRepairShop/discussions)

---

**Note:** This project is under active development. Some services (Customer, Warehouse) are not yet fully implemented.