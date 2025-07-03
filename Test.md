# Contract-Driven Modular Clean Architecture (CDMCA)

A scalable, testable, and maintainable architecture pattern for .NET-based enterprise applications, optimized for strong separation of concerns, parallel front-end/back-end development, and long-term codebase stability.

> ⚠️ **Framework Compatibility**: This architecture is designed specifically for use with **ASP.NET Core Web API**, **Blazor WebAssembly**, and **ASP.NET Core MVC**. While the core principles are adaptable, it is not directly portable to frontend stacks like React or Angular unless bridged via API layers.

---

## 🧱 Layered Architecture Overview

```
Solution/
│
├── Contracts/       # DTOs, Request/Response models, Service Interfaces, Connector Interfaces
├── Core/            # Business logic (implements Contracts interfaces)
├── DataLayer/       # Infrastructure: EF Core, Repositories, UoW
├── API/             # ASP.NET Core REST API layer
├── WebApp/          # Frontend project (e.g. Blazor, ASP.NET Core MVC)
├── Shared/          # Optional common utilities/constants
└── Tests/           # Unit tests for Core (and optionally API/Contracts)
```

> ✅ Architecture supports evolution to CQRS, event sourcing, SignalR, microservices, and versioned contract packaging (NuGet) without refactor.

---

## 🎯 Architectural Principles

### ✅ Modular Separation

* Each layer has a single responsibility.
* No cross-layer references except as defined by dependency rules.

### ✅ Contract-Driven Communication

* `Contracts` defines all DTOs, interfaces, and wrappers.
* Frontend/backend can independently implement or consume contracts.

### ✅ Backend-Agnostic Frontend

* Frontend uses "connectors" that implement service interfaces.
* No raw HTTP/gRPC logic in UI components — inject and use.
* Ideal for fast solo development and consistent mocking.

### ✅ Predictable API Contracts

* Uses `BaseRequest<T>` and `BaseResponse<T>` to standardize data exchange.
* Optional metadata (paging, error codes, success flags, etc.)
* Enables consistent parallel frontend/backend work.

### ✅ Testability & Reusability

* Core logic is testable without infrastructure.
* Connectors and services use DI and interfaces.
* Contracts can be reused across platforms: Function Apps, desktop apps, mobile (MAUI), workers.

---

## 📚 Folder/Layer Responsibilities

### 1. **Contracts/**

* Defines:

  * `Dto/` (shared models)
  * `Commands/` (e.g., `Request<T>`, `Response<T>`)
  * `Interfaces/` (service/connector interfaces)
  * `Connector/` (default frontend connector implementations)

* Used by: Core, API, WebApp, Workers, Function Apps, or any consumer

* Must not reference any other project

* Supports NuGet packaging for reuse across microservices

### 2. **Core/**

* Implements service logic defined in `Contracts.Interfaces`
* Depends only on Contracts (not API or DataLayer)
* Can be used by:

  * REST APIs
  * gRPC services
  * Background services
  * CLI/Desktop Apps

> ✅ Core logic can be reused, wrapped, or replaced by CQRS handlers if needed.

### 3. **DataLayer/**

* EF Core + Repository Pattern
* Implements data access interfaces (e.g., `IUserRepository`, `IUnitOfWork`)
* Core interacts via DI — infrastructure swap is safe
* Can be replaced with:

  * CQRS read/write models
  * Event-sourced repositories
  * Dapper, Mongo, or 3rd-party APIs

### 4. **API/**

* Thin ASP.NET Core layer (Controllers only)
* Maps HTTP endpoints to Core service methods or CQRS handlers
* Handles:

  * Auth
  * Middleware
  * Swagger
  * WebSockets (SignalR if needed)

> ✅ API can evolve to expose gRPC, SignalR, GraphQL, or REST — Core remains unchanged.

### 5. **WebApp/**

* Consumes Contracts only
* Uses Connectors that implement Contracts service interfaces
* Typed, predictable interaction — no raw HTTP
* Can be:

  * Blazor WebAssembly
  * ASP.NET Core MVC
  * MAUI/desktop/mobile frontends
  * CLI frontend

### 6. **Tests/**

* Unit tests for Core and optionally API/Contracts
* Mock service interfaces via Contracts
* Frontend can test logic using connector mocks

---

## 🔄 Development Flow

1. Frontend dev defines a new `Request<T>` + `Response<T>` + Connector method in `Contracts`
2. Backend dev implements the interface in `Core`
3. API exposes the method by injecting the Core service or a CQRS handler
4. No layer breaks if additions follow conventions

---

## 📌 Conventions

* 🔒 `Contracts` must remain stable (only additive changes allowed)
* ✅ Support semantic versioning if consumed externally
* 🚫 Never rename or remove existing fields/methods without coordination
* ✅ Extend DTOs via inheritance or composition
* ✅ Place per-feature folders under `Dto`, `Commands`, `Connectors`, etc.
* 🧪 Core services should have complete unit test coverage

---

## ⚙️ CQRS/Event Sourcing Compatibility

> This architecture is **CQRS-ready** — you can wrap or replace Core services with `ICommandHandler<T>` and `IQueryHandler<T>` patterns.

* **Core Layer** can be partially reused or bypassed
* **DataLayer** can be replaced with event stores
* **Contracts** layer holds all CQRS commands/queries/events
* **API** can route through command/query dispatchers

> ✅ You can evolve without breaking consumers or rewriting everything.

---

## 🛰️ Microservices Strategy

* Deploy each service with:

  * `Contracts`, `Core`, `DataLayer`, `API` — no WebApp needed
* Contracts can be published as shared NuGet packages for cross-service use
* Supports:

  * REST, gRPC, queues (Azure Service Bus, RabbitMQ)
  * Background workers
  * Azure Functions or Linux containers
* Each microservice is deployable independently

---

## 🧩 Sample Folder Structures

### 🔹 **Solo / Small Team Monolith**

```
Solution/
│
├── Contracts/
│   ├── Dto/
│   │   └── Payments/MoneyDto.cs
│   ├── Commands/
│   │   └── Payments/CreatePaymentRequest.cs
│   ├── Interfaces/
│   │   └── Payments/IPaymentService.cs
│   └── Connectors/
│       └── Payments/PaypalConnector.cs
│
├── Core/
│   └── Services/
│       └── Payments/
│           ├── PaypalPayService.cs
│           └── MayaPayService.cs
│
├── DataLayer/
│   └── Repositories/
│       └── Payments/
│           └── EfPaymentRepository.cs
│
├── API/
│   └── Controllers/
│       └── PaymentsController.cs
│
├── WebApp/
│   └── Pages/
│       └── PaymentsPage.razor
```

---

### 🔹 **Large Team Structure (Multi-feature)**

```
Solution/
├── Contracts/
│   ├── Dto/Orders/
│   ├── Dto/Payments/
│   ├── Commands/Orders/
│   ├── Commands/Payments/
│   ├── Interfaces/Orders/
│   └── Interfaces/Payments/
│
├── Core/
│   ├── Services/Orders/
│   └── Services/Payments/
│
├── DataLayer/
│   ├── Repositories/Orders/
│   └── Repositories/Payments/
│
├── API/
│   ├── Controllers/Orders/
│   └── Controllers/Payments/
```

---

### 🔹 **Microservice Example (Payments Service Only)**

```
PaymentsService/
│
├── Contracts/                      # Shared or internal NuGet
├── Core/
│   └── Payments/
│       └── PaymentProcessor.cs
├── DataLayer/
│   └── Payments/
│       └── PaymentRepository.cs
├── API/
│   └── Controllers/
│       └── PaymentsController.cs
└── Worker/
    └── BackgroundProcessor.cs      # Optional
```

---

## ✅ Summary

| Feature                   | Benefit                                 |
| ------------------------- | --------------------------------------- |
| Contract-Driven           | Predictable APIs, safe parallel dev     |
| Layered Structure         | Separation of concerns, modular scale   |
| DI Everywhere             | Easy mocking, flexible infrastructure   |
| Typed Frontend Connectors | Minimal boilerplate, no raw HTTP        |
| Reusable Core             | Supports REST, gRPC, desktop reuse      |
| CQRS/Event Source Ready   | Wrap, replace, or evolve infrastructure |
| Microservice Compatible   | Independent deployment and scaling      |
| Future-Proofing           | Migrate without rewriting               |

---

## 📁 Optional Enhancements

* Add gRPC or WebSocket transport by injecting Core in a new host
* Codegen SDKs (NSwag, etc.) if dynamic typing is needed
* Use source generators or CLI tools


to scaffold base structure

* Publish Contracts as internal NuGet packages for microservice consumption

---

> 🧠 **Architecture by Ronjun Cajilig** — evolved over 4+ years through real-world full-stack development in .NET Core, Blazor, and Azure. Designed for maintainability, agility, and silent evolution in solo, startup, or enterprise teams.

---
