# Contributing to EventualRepairShop

First off, thank you for considering contributing to EventualRepairShop! It's people like you that make this project better.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Project Structure](#project-structure)

## Code of Conduct

This project and everyone participating in it is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check the existing issues to avoid duplicates. When you create a bug report, include as many details as possible:

- **Use a clear and descriptive title**
- **Describe the exact steps to reproduce the problem**
- **Provide specific examples** to demonstrate the steps
- **Describe the behavior you observed** and what you expected
- **Include screenshots** if relevant
- **Provide your environment details** (.NET version, OS, etc.)

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion:

- **Use a clear and descriptive title**
- **Provide a detailed description** of the suggested enhancement
- **Explain why this enhancement would be useful**
- **List any similar features** in other projects if applicable

### Your First Code Contribution

Unsure where to begin? Look for issues labeled:

- `good first issue` - Simple issues perfect for newcomers
- `help wanted` - Issues where we need community help
- `documentation` - Documentation improvements

### Pull Requests

1. Fork the repository and create your branch from `main`
2. Follow the development setup instructions
3. Make your changes following our coding standards
4. Add or update tests as appropriate
5. Ensure all tests pass
6. Update documentation as needed
7. Submit your pull request

## Development Setup

### Prerequisites

- .NET 7.0 SDK or later
- Docker Desktop
- Git
- Your preferred IDE (Visual Studio, Rider, or VS Code)

### Setup Steps

```bash
# Clone your fork
git clone https://github.com/YOUR-USERNAME/EventualRepairShop.git
cd EventualRepairShop

# Add upstream remote
git remote add upstream https://github.com/rudironsoni/EventualRepairShop.git

# Start infrastructure
docker-compose -f docker-compose.Development.Infrastructure.yaml up -d

# Restore packages
dotnet restore

# Build solution
dotnet build

# Run tests (when available)
dotnet test
```

## Coding Standards

### C# Guidelines

- Follow [Microsoft's C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- Use meaningful variable and method names
- Keep methods focused and small
- Write XML documentation for public APIs
- Use nullable reference types appropriately
- Prefer explicit types over `var` when type is not obvious

### Architecture Guidelines

- **Domain Layer**: Pure domain logic, no infrastructure dependencies
- **Application Layer**: Use cases and business workflows
- **Infrastructure Layer**: External concerns (databases, message bus)
- **DDD Principles**: Maintain aggregate boundaries and domain events
- **Event Sourcing**: Store all domain events in event store
- **CQRS**: Separate command and query responsibilities

### Code Example

```csharp
namespace RepairOrder.Domain.Aggregates;

/// <summary>
/// Represents a repair order aggregate root.
/// </summary>
public sealed class RepairOrder : AggregateRoot
{
    private RepairOrder() { } // For EF Core

    /// <summary>
    /// Places a new repair order.
    /// </summary>
    /// <param name="customer">The customer information.</param>
    /// <param name="device">The device to be repaired.</param>
    /// <param name="description">Repair description.</param>
    /// <returns>A new repair order instance.</returns>
    public static RepairOrder Place(
        Customer customer,
        Device device,
        string description)
    {
        ArgumentNullException.ThrowIfNull(customer);
        ArgumentNullException.ThrowIfNull(device);
        ArgumentException.ThrowIfNullOrEmpty(description);
        
        var repairOrder = new RepairOrder();
        // Implementation...
        return repairOrder;
    }
}
```

## Commit Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

### Examples

```
feat(repair-order): add cancel repair order functionality

Implement the ability to cancel repair orders with proper
event sourcing and domain event emission.

Closes #123
```

```
fix(scheduling): correct date validation logic

The previous validation was allowing past dates which
should not be permitted for new schedules.
```

## Pull Request Process

1. **Update Documentation**: Update README.md or other docs with details of changes if needed
2. **Add Tests**: Include unit and/or integration tests for your changes
3. **Follow Code Style**: Ensure your code follows project conventions
4. **Update Dependencies**: Document any new dependencies in the PR description
5. **One Feature Per PR**: Keep pull requests focused on a single concern
6. **Write Good Commit Messages**: Follow the commit guidelines above
7. **Respond to Feedback**: Be responsive to code review comments

### PR Description Template

```markdown
## Description
Brief description of what this PR does

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Changes Made
- List specific changes
- One item per line

## Testing
Describe testing performed

## Checklist
- [ ] My code follows the project's coding standards
- [ ] I have added tests that prove my fix/feature works
- [ ] I have updated the documentation accordingly
- [ ] My changes generate no new warnings
- [ ] All tests pass locally
```

## Project Structure

```
EventualRepairShop/
├── src/
│   ├── Contracts/              # Shared contracts
│   ├── Services/               # Microservices
│   │   ├── [ServiceName]/
│   │   │   ├── Application/    # Use cases
│   │   │   ├── Domain/         # Domain model
│   │   │   ├── Infrastructure.EventStore/
│   │   │   ├── Infrastructure.MessageBus/
│   │   │   └── WorkerService/
│   └── Web/
│       └── WebAPI/             # API Gateway
└── test/                       # Test projects
```

### Adding a New Service

When adding a new microservice:

1. Follow the existing structure (Application, Domain, Infrastructure layers)
2. Implement domain aggregates with event sourcing
3. Add message consumers for cross-service communication
4. Include database migrations for event store
5. Add API endpoints in WebAPI if needed
6. Document the service in README.md

## Questions?

Feel free to:
- Open an issue for questions
- Start a discussion on GitHub Discussions
- Reach out to project maintainers

## Recognition

Contributors will be recognized in our README.md and release notes.

Thank you for contributing to EventualRepairShop! 🎉
