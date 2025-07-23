
# Contract-Driven Modular Clean Architecture (CDMCA)

A scalable, testable, and maintainable architecture pattern for .NET-based enterprise applications, optimized for strong separation of concerns, parallel front-end/back-end development, and long-term stability.

> ⚠️ Supports both .NET and non-.NET frontends (e.g., Blazor, Angular, React). .NET clients use typed **Connectors**, while non-.NET apps interact directly with the API.
>
> 🔧 **Note:**
> This architecture is intentionally **kept minimal and clean** to serve as a **starting point**. It is fully **extensible** based on project requirements.
>
> ✅ **API Layer can be implemented using**:
>
> * **REST** (e.g., ASP.NET Web API)
> * **gRPC** for high-performance internal or external service communication
>
> ✅ **Azure-based Sample Usage**:
>
> * **API Layer**: Azure App Service or Azure Kubernetes Service (AKS)
> * **Data Layer**: Azure SQL, Cosmos DB, or Table Storage
> * **Identity/Auth**: Azure Active Directory B2C or Azure Entra ID
> * **Messaging**: Azure Service Bus or Azure Event Grid
> * **Logging & Tracing**: Azure Monitor, Application Insights, Log Analytics
> * **Secrets & Config**: Azure Key Vault and Azure App Configuration
> * **CI/CD**: Azure DevOps Pipelines with Infrastructure-as-Code (Bicep or Terraform)
>
> You can also integrate:
>
> * **API Gateways** (e.g., Azure API Management, YARP)
> * **BFF** (Backend-for-Frontend) layers for client-specific optimization
>
> CDMCA is ideal for evolving between:
>
> * **Modular Monolith** → **Hybrid Composable** → **Microservices**
>
---

## 🧱 Layered Architecture Overview

```
Solution/
│
├── Contracts/        # DTOs, Request/Response models, Service Interfaces
│   └── Interfaces/   # Service interfaces (shared across API, Core, Connectors)
│   └── Dto/          # Shared models across frontend/backend
│   └── Commands/     # Request/Response objects inheriting BaseRequest/BaseResponse
│
├── Core/             # Business logic implementing Contracts.Interfaces
│
├── DataLayer/        # EF Core, Dapper, or other persistence implementations
│   └── Repositories/ # Implements abstractions (e.g., IUserRepository)
│
├── API/              # ASP.NET Core REST API Layer (controllers, middleware, auth)
│
├── Connector/        # Typed SDK (implements service interfaces using HTTP)
│
├── WebApp/           # .NET Frontend (e.g., Blazor, MVC) consuming Connectors
│
├── Shared/           # Common utilities: constants, extensions, filters
│
├── Tests/            # Unit tests for Core, Contracts, and Connectors
│
└── NonDotNetClients/ # Angular, React, Flutter (uses OpenAPI/Swagger)
```

---

## 📚 Layer Responsibilities

| Layer          | Responsibility                                                                 |
|----------------|----------------------------------------------------------------------------------|
| **Contracts**  | Defines service interfaces, DTOs, and commands. Core and Connector implement these. |
| **Core**       | Business rules and application services. Implements Contracts.Interfaces.        |
| **DataLayer**  | EF Core/Dapper repositories. Used only by Core.                                 |
| **API**        | ASP.NET Core controllers, middleware, auth. Calls Core.                         |
| **Connector**  | Typed SDK clients. Implements Contracts.Interfaces via HTTP.                    |
| **WebApp**     | .NET frontends (Blazor, MVC) injecting connectors.                              |
| **NonDotNetClients** | JavaScript/mobile UIs calling API via HTTP.                                   |
| **Shared**     | Utilities reused across all layers.                                             |

---

## 🔁 Architecture Usage Scenarios

### 🔷 1. Microservices Architecture

```
Contracts (User/Payroll/Attendance)
   ↑         ↑         ↑
Core     API Layer    Connector
   ↑                      ↑
DataLayer            WebApp (.NET UI)
   ↑
Database             JS / Flutter → HTTP → API
```

**Use Case**: Large enterprise system split into autonomous microservices with separate APIs and connectors.

**Pros**:
- Fully independent deployment per domain
- Independent scaling per module
- Contracts enforce service boundaries

**Cons**:
- Requires advanced DevOps and coordination
- Multiple repositories and duplicated infra
- Versioning between services needs governance

---

### 🔷 2. Modular Monolith

```
Contracts (Shared across modules)
   ↑         ↑         ↑
User Core   API Layer   Connector
   ↑                      ↑
DataLayer            Blazor / MVC
   ↑
Database             React → HTTP → API
```

**Use Case**: Internal tools or SaaS platforms with multiple domain modules in a single deployable app.

**Pros**:
- Centralized deployment and config
- Easy to share services and contracts
- Compatible with .NET and non-.NET frontends

**Cons**:
- Domain boundaries can blur
- Harder to extract later
- Shared database must be carefully managed

---

### 🔷 3. Hybrid Composable Architecture

```
Contracts (Interfaces + DTOs)
   ↑         ↑         ↑
Core     API Layer    Connector
   ↑                      ↑
DataLayer            Blazor WASM
   ↑
Database             React / Angular → HTTP → API
```

**Use Case**: Mixed-client architecture with shared backend (Core + API + DataLayer) used by both internal (.NET) and external (non-.NET) consumers.

**Pros**:
- Supports multiple UI types without duplicated logic
- Contracts used end-to-end in .NET
- OpenAPI supports external integrations

**Cons**:
- Contracts must evolve slowly and carefully
- Requires versioning for backward compatibility
- Slightly more testing coordination

---

## 📌 When to Use CDMCA

| Scenario                               | Recommended |
|----------------------------------------|-------------|
| Internal enterprise app (Blazor/MVC)   | ✅ Yes       |
| Mobile frontend (Flutter/React Native) | ✅ Yes       |
| API consumed by multiple teams         | ✅ Yes       |
| Startup MVP with fast iteration        | ❌ Overhead  |
| Single-stack SPA without reusability   | ❌ Use simpler SPA-API combo |

---

## ✅ Summary

| Architecture         | Deployment | Best For                         | Pros                                   | Cons                                  |
|----------------------|------------|-----------------------------------|----------------------------------------|----------------------------------------|
| Microservices         | Distributed | Enterprise domains, large teams  | Scale, isolation, autonomy             | Complex infra, team coordination       |
| Modular Monolith      | Single App | Mid-sized platforms               | Simpler dev, centralized control       | Tighter coupling, shared database risk |
| Hybrid/Composable     | Mixed      | Internal + public clients         | Multi-UI, shared logic, OpenAPI        | Careful versioning, mixed test paths   |

---
