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
* All connector interfaces mirror backend service interfaces in Contracts.
* No raw HTTP logic in UI components.

### ✅ Predictable API Contracts

* Uses `BaseRequest<T>` and `BaseResponse<T>` to standardize data exchange.
* Allows optional metadata like paging, error codes, success flags, etc.

### ✅ Testability & Reusability

* Core logic is testable without infrastructure.
* Connectors can be mocked using the same interface contracts.

---

## 📚 Folder/Layer Responsibilities

### 1. **Contracts/**

* Defines:

  * `Dto/` (shared models)
  * `Commands/` (e.g., `Request<T>`, `Response<T>`)
  * `Interfaces/` (service/connector interfaces)
  * `Connector/` (default frontend connector implementations)
* Used by: Core, API, WebApp
* Must not reference any other project

### 2. **Core/**

* Implements service logic defined in `Contracts.Interfaces`
* Depends only on Contracts (not API or DataLayer)
* Reusable in REST, gRPC, CLI, or desktop apps

### 3. **DataLayer/**

* EF Core + Repository Pattern
* Implements data access interfaces (e.g., `IUserRepository`, `IUnitOfWork`)
* Used by Core through injected abstractions

### 4. **API/**

* Thin ASP.NET Core layer (Controllers only)
* Maps HTTP endpoints to Core service methods
* Handles auth, filters, middleware, Swagger
* Injects services via DI

### 5. **WebApp/**

* Consumes Contracts
* Injects Connectors (that implement service interfaces)
* Uses only standard base requests/responses

### 6. **Tests/**

* Unit tests for Core and optionally Contracts/Connectors
* Frontend logic can use connector mocking for isolation

---

## 🔄 Development Flow

1. Frontend dev defines a new `Request<T>` + `Response<T>` + Connector method in `Contracts`
2. Backend dev implements the interface in `Core`
3. API exposes the method by injecting the core service
4. No layer breaks if additions follow conventions

---

## 📌 Conventions

* 🔒 `Contracts` must remain stable (only additive changes allowed)
* 🚫 Never rename or remove existing fields/methods without coordination
* ✅ Prefer extending models with inheritance or composition
* 🧪 Core services should have complete unit test coverage
* 🔄 Optional: version Contracts if external teams consume them

---

## ✅ Summary

| Feature                   | Benefit                               |
| ------------------------- | ------------------------------------- |
| Contract-Driven           | Predictable APIs, safe parallel dev   |
| Layered Structure         | Separation of concerns, modular scale |
| DI Everywhere             | Easy mocking, flexible infrastructure |
| Typed Frontend Connectors | Minimal boilerplate, no raw HTTP      |
| Reusable Core             | Supports REST, gRPC, desktop reuse    |

---

## 📁 Optional Enhancements

* Add gRPC or WebSocket transport by injecting Core in a new host
* Codegen SDKs (NSwag, etc.) if dynamic typing is needed
* Use source generators or CLI tools to scaffold base structure

---

> Architecture by Ronjun Cajilig — evolved over 4 years through enterprise-grade solo development and full-stack implementation using .NET Core, Blazor, and Azure.
