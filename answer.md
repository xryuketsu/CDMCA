### 1. **How do you implement Validation in ASP.NET MVC?**

I typically use **Data Annotations** like `[Required]`, `[Range]`, etc., for basic validation. For more complex, layered, or shared business rules, I use **FluentValidation**, which integrates well with ASP.NET, supports dependency injection, and promotes a clean separation of concerns.

✅ *Library alternative:*

* `FluentValidation` – clean, expressive, and testable validation.

---

### 2. **What are Filters in ASP.NET MVC? What does ActionFilterAttribute do?**

Filters allow code to run **before or after controller actions**, useful for cross-cutting concerns like logging, authorization, or caching.
`ActionFilterAttribute` is a base class used to create **custom action filters**, where you can override methods like `OnActionExecuting` and `OnResultExecuted` to inject behavior around action execution.

---

### 3. **What is the difference between const, readonly, and static in C#?**

* `const`: Value is known at compile-time and cannot be changed.
* `readonly`: Value is assigned at runtime (constructor or inline) and cannot be modified afterward.
* `static`: Belongs to the class itself, not instances. Shared across all instances.

```csharp
const string AppName = "MyApp";              // Compile-time
readonly DateTime CreatedAt = DateTime.Now;  // Runtime
static int Counter = 0;                      // Shared state
```

---

### 4. **Explain IEnumerable, IQueryable, and List<T> in C#?**

* `IEnumerable<T>`: For in-memory collections, supports forward-only iteration.
* `IQueryable<T>`: Used for LINQ-to-SQL/EF Core, builds DB queries and supports deferred execution.
* `List<T>`: A concrete collection with full in-memory data and indexing support.

---

### 5. **What are RESTful status codes and their meanings for 204, 401, 403?**

* **204 No Content** – Successful request, no body returned.
* **401 Unauthorized** – Authentication required or failed (e.g., missing/invalid token).
* **403 Forbidden** – Authenticated but lacks permission for the requested resource.

---

### 6. **What is an Idempotent operation in REST API? Give an example.**

An **idempotent** operation returns the same result regardless of how many times it's called.
Example:

```http
PUT /users/42
{
  "email": "user@example.com"
}
```

Calling this multiple times with the same data won’t change the final state after the first update.

---

### 7. **What is JWT (JSON Web Token), and what is it used for?**

JWT is a **compact, stateless token format** used for **authentication and authorization**. It contains encoded user data (claims), is signed for integrity, and is typically used in client-facing APIs (e.g., web/mobile apps). It’s passed using the `Authorization: Bearer <token>` header.

✅ *When to use:*

* **Use JWT** for client-facing applications where user identity and roles need to be securely passed between services.

❗ **However**, in **microservice-to-microservice (machine-to-machine)** communication, I consider **JWT overkill** due to unnecessary token parsing and validation cost.

🔐 In those cases, I prefer using **API Keys** secured via **custom middleware**, often combined with IP whitelisting or header signatures for internal traffic. It’s faster, simpler, and easier to manage within a trusted internal network.

✅ *Library/custom options:*

* `Microsoft.AspNetCore.Authentication.JwtBearer`
* Custom API key middleware for internal services

---

### 8. **How do you handle pagination in REST APIs? Give an example**

I use **Entity Framework Core with LINQ**, applying `.Skip()` and `.Take()` for efficient paging:

```csharp
var paged = await _context.Employees
    .Where(e => e.IsActive)
    .OrderBy(e => e.LastName)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

Example:

```http
GET /employees?page=2&pageSize=10
```

Returns:

```json
{
  "data": [...],
  "totalCount": 125,
  "page": 2,
  "pageSize": 10
}
```

📌 For large datasets or complex queries, I use **Stored Procedures** or `FromSqlRaw()` for better performance.

🔗 *Live example in my current project:*
[https://github.com/xryuketsu/CyraSoft.Hris](https://github.com/xryuketsu/CyraSoft.Hris)

---

### 9. **What is CQRS, and how is it implemented in .NET?**

**CQRS (Command Query Responsibility Segregation)** separates reads (queries) and writes (commands) to simplify code and scale independently.

In .NET, I use:

* `MediatR` – for separating commands and queries.
* `FluentValidation` – for input validation.
* **Custom Mapping** – for DTO ↔ Entity transformation.

While AutoMapper is common, I prefer **custom mappers** when building projects from scratch. Custom mappers:
* Offer full control over property mapping
* Reduce magic and unexpected behavior
* Align better with clean architecture and domain boundaries

Mapping is done via explicit methods like `ToDto()` or `ToEntity()` in mapping services or static helpers.

---

### 10. **How do you cancel an asynchronous operation in C#?**

Use `CancellationToken`, which allows you to request cancellation of a long-running task gracefully.

```csharp
public async Task RunAsync(CancellationToken ct)
{
    while (!ct.IsCancellationRequested)
    {
        await Task.Delay(500);
    }
}
```

---

### 11. **How do Microservices communicate with each other?**

Microservices communicate via:

* **RESTful HTTP** – simple and human-readable.
* **gRPC** – fast, compact binary format for high-performance needs.
* **Message brokers** – like RabbitMQ, Kafka, for async/event-driven flows.
* **Service bus** – such as Azure Service Bus, NATS, or MassTransit for scalable messaging.

Choice depends on latency, coupling, consistency needs, and message frequency.

---

### 12. **What are the challenges in Microservices architecture?**

Some common challenges in microservices architecture include:

* **Service discovery and orchestration**
* **Data consistency across services**
* **Distributed logging and tracing**
* **Authentication and secure service-to-service communication**
* **Latency and fault tolerance**
* **Complex CI/CD and deployment pipelines**

To address these in my current stack, I use **Azure-native tools**, including:

* **Azure API Management (APIM)** – as the **API gateway** to handle:

  * Centralized routing
  * Request throttling
  * Caching
  * Security policies (API keys, tokens)
  * Analytics and monitoring
* **Azure Monitor and Application Insights** – for **centralized logging, diagnostics, and tracing**
* **Azure DevOps** – for CI/CD pipelines and release management across services

---

### 💡 Additional Notes:

> When working on a **personal project** or leading architecture decisions, I follow my own scalable, modular clean architecture pattern focused on:
>
> * Domain-Driven Design
> * Layered separation of concerns
> * Unit-testable and flexible service boundaries

🔗 My public clean architecture template:
[https://github.com/xryuketsu/CDMCA](https://github.com/xryuketsu/CDMCA)
(Clean Domain-Driven Modular-Centric Architecture)

🔧 Currently applied in my personal HRIS project:
[https://github.com/xryuketsu/CyraSoft.Hris](https://github.com/xryuketsu/CyraSoft.Hris)
