# FocusFlow - Architecture Decision Log (ADR)

> This document records the key architectural and design decisions made for the FocusFlow Task Management application.

---

## ADR-001: Frontend Technology - Blazor Server

**Status:** Accepted  
**Date:** 2026-02-04

### Context
The application requires a rich, component-based UI with potential real-time collaboration features. Two Blazor hosting models were considered: Blazor Server and Blazor WebAssembly.

### Decision
We will use **Blazor Server** for the frontend application.

### Rationale
1. **Real-Time Collaboration Potential:** Task management applications benefit from live updates when team members change task statuses or assignments. Blazor Server's built-in SignalR foundation makes real-time features natural and efficient to implement.

2. **Smaller Payload & Faster Initial Load:** Users reach the task list faster since only a minimal client bundle is required. The entire .NET runtime remains on the server, reducing initial download size.

3. **Optimized for Authenticated, Data-Heavy Scenarios:** FocusFlow will frequently query projects, tasks, and dashboards. Keeping database connections server-side reduces latency and simplifies data access patterns.

### Consequences
- Requires persistent SignalR connection per user session
- Server resources scale with concurrent user count
- Application will consist of multiple Blazor pages/components for different features (Projects, Tasks, Dashboard, etc.)

---

## ADR-002: Backend Architecture - Clean/Onion Architecture

**Status:** Accepted  
**Date:** 2026-02-04

### Context
The project requirements specify layered/onion architecture with SOLID design principles. A clear separation of concerns is needed to ensure maintainability, testability, and adherence to dependency inversion.

### Decision
We will implement **Clean Architecture** following the **Onion Architecture** pattern with four distinct layers.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        PRESENTATION                              │
│                    (API Controllers, Blazor)                     │
└───────────────────────────┬─────────────────────────────────────┘
                            │ depends on
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                       INFRASTRUCTURE                             │
│         (EF Core, Identity, Repository Implementations,          │
│                    External Services)                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │ depends on
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                        APPLICATION                               │
│          (Use Cases, Services, DTOs, Validation, CQRS)          │
└───────────────────────────┬─────────────────────────────────────┘
                            │ depends on
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                          DOMAIN                                  │
│       (Entities, Value Objects, Domain Events, Interfaces)       │
│                                                                  │
│                    *** NO DEPENDENCIES ***                       │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Responsibility |
|-------|----------------|
| **Domain** | Core business entities, value objects, domain events, and repository interfaces. Has no external dependencies. |
| **Application** | Orchestrates use cases, defines DTOs, validation logic, and application services. Depends only on Domain. |
| **Infrastructure** | Implements interfaces defined in Domain/Application. Contains EF Core DbContext, Identity configuration, and external service integrations. |
| **Presentation** | API controllers, middleware, and Blazor UI components. Entry point for user interaction. |

### Consequences
- Dependencies always point inward toward the Domain layer
- Domain layer remains pure and framework-agnostic
- Infrastructure concerns are isolated and easily replaceable
- High testability through dependency injection and interface abstractions

---

## ADR-003: Solution & Project Structure

**Status:** Accepted  
**Date:** 2026-02-04

### Context
A clear project structure is needed to organize the codebase according to the chosen architecture, facilitate team collaboration, and support independent testing of each layer.

### Decision
We will organize the solution into a single `FocusFlow.sln` with separate projects per architectural layer, including a shared contracts project.

### Solution Structure Diagram

```
FocusFlow.sln
│
├── src/
│   ├── FocusFlow.Domain/              # Core domain layer
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Events/
│   │   └── Interfaces/
│   │
│   ├── FocusFlow.Application/         # Application/Use case layer
│   │   ├── Services/
│   │   ├── DTOs/
│   │   ├── Validation/
│   │   └── Interfaces/
│   │
│   ├── FocusFlow.Infrastructure/      # Infrastructure layer
│   │   ├── Persistence/
│   │   ├── Identity/
│   │   ├── Repositories/
│   │   └── Services/
│   │
│   ├── FocusFlow.Shared/              # Shared contracts & DTOs
│   │   ├── Contracts/
│   │   └── Models/
│   │
│   ├── FocusFlow.Api/                 # ASP.NET Core Web API
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   └── Configuration/
│   │
│   └── FocusFlow.Web/                 # Blazor Server UI
│       ├── Pages/
│       ├── Components/
│       └── Services/
│
└── tests/
    ├── FocusFlow.Domain.Tests/
    ├── FocusFlow.Application.Tests/
    ├── FocusFlow.Infrastructure.Tests/
    └── FocusFlow.Api.Tests/           # Integration tests
```

### Rationale
- **FocusFlow.Shared:** Contains DTOs and contracts shared between the API and Blazor client, ensuring type safety and reducing duplication.
- **Parallel test projects:** Each layer has a corresponding test project enabling isolated unit testing.
- **Clear separation:** Developers can work on different layers independently.

### Consequences
- Multiple projects increase initial setup complexity
- Clear boundaries enforce architectural constraints
- Shared project enables contract-first development between API and UI

---

## ADR-004: Development Methodology - Pragmatic Test-Driven Development

**Status:** Accepted  
**Date:** 2026-02-04

### Context
The project requirements specify test-driven C# backend development. A TDD approach ensures code quality, design clarity, and confidence in refactoring.

### Decision
We will follow a **Pragmatic TDD** approach, creating test projects alongside implementation projects and writing tests before or simultaneously with production code.

### Workflow

Start with the Domain layer and move outward (Domain → Application → Infrastructure → API), creating test projects alongside implementation projects.

### Development Order
1. `FocusFlow.Domain` + `FocusFlow.Domain.Tests`
2. `FocusFlow.Application` + `FocusFlow.Application.Tests`
3. `FocusFlow.Infrastructure` + `FocusFlow.Infrastructure.Tests`
4. `FocusFlow.Api` + `FocusFlow.Api.Tests`
5. `FocusFlow.Web` (with component tests as applicable)

### Consequences
- Higher initial development overhead for writing tests
- Improved code quality and design through test pressure
- Comprehensive test suite enables confident refactoring
- Documentation of expected behavior through test cases

---

## Decision Summary

| ID | Decision | Status |
|----|----------|--------|
| ADR-001 | Blazor Server for frontend | ✅ Accepted |
| ADR-002 | Clean/Onion Architecture (4 layers) | ✅ Accepted |
| ADR-003 | Single solution with layered projects + Shared | ✅ Accepted |
| ADR-004 | Pragmatic TDD, layer-by-layer | ✅ Accepted |

---

*This is a living document. New decisions will be appended as the project evolves.*
