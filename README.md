# Delphi Mercado Pago Middleware

> REST payment middleware for Mercado Pago built with Delphi, Horse and RESTRequest4Delphi, designed around layered architecture, explicit dependencies and testable application boundaries.

**English** | [Português (Brasil)](README.pt-BR.md)

![Delphi](https://img.shields.io/badge/Delphi-10%2B-C71A36)
![Horse](https://img.shields.io/badge/Horse-REST%20Server-2F4F4F)
![RESTRequest4Delphi](https://img.shields.io/badge/HTTP%20Client-RESTRequest4Delphi-4B6EAF)
![dataset--serialize](https://img.shields.io/badge/Serialization-dataset--serialize-6A5ACD)
![Mercado Pago](https://img.shields.io/badge/Integration-Mercado%20Pago-009EE3)
![Architecture](https://img.shields.io/badge/Architecture-Layered-555555)

---

## About the project

**Delphi Mercado Pago Middleware** is a backend middleware project designed to isolate Delphi applications from the external contracts and infrastructure concerns of Mercado Pago.

The project focuses on building a maintainable payment integration with clear architectural boundaries, explicit dependencies and controlled access to external services.

Instead of coupling the client application directly to Mercado Pago APIs, the middleware introduces an internal REST contract between the application and the payment infrastructure.

```text
Delphi Application
        |
        | HTTP / JSON
        v
Delphi Middleware
        |
        v
Horse
        |
        v
Routes
        |
        v
Controllers
        |
        v
Services
        |
        +-------------------+
        |                   |
        v                   v
    Gateways           Repositories
        |
        v
RESTRequest4Delphi
        |
        | HTTPS / JSON
        v
   Mercado Pago
```

The external Mercado Pago contract remains isolated from the application's internal contract whenever practical.

---

## Goals

This project is being developed as a practical implementation of software engineering principles applied to a real-world financial integration.

The main goals are:

* provide a stable REST boundary between Delphi applications and Mercado Pago;
* isolate external payment contracts behind gateways;
* keep business rules independent from HTTP frameworks;
* apply explicit dependency inversion through interfaces;
* improve testability by replacing external dependencies in unit tests;
* centralize external HTTP communication;
* protect private credentials inside the middleware;
* treat payment operations with appropriate security and idempotency concerns;
* evolve the integration without unnecessarily coupling client applications to external API changes.

---

## Tech stack

| Technology             | Responsibility                            |
| ---------------------- | ----------------------------------------- |
| **Delphi 10+**         | Middleware implementation                 |
| **Horse**              | REST server and HTTP infrastructure       |
| **RESTRequest4Delphi** | Outbound HTTP communication               |
| **dataset-serialize**  | `TDataSet` serialization when appropriate |
| **Mercado Pago**       | External payment platform                 |

The project targets **Delphi 10 or newer**.

Library-specific APIs are validated against the version actually used by the project before implementation.

---

## Architecture

The project follows a layered architecture based on **MVC + Service Layer**, with infrastructure concerns kept outside application rules.

```text
Route
  |
  v
Controller
  |
  v
Service
  |
  +---------------------+
  |                     |
  v                     v
Gateway             Repository
  |                     |
  v                     v
External API         Persistence
```

### Routes

Routes belong to the Horse HTTP infrastructure.

Responsibilities:

* define HTTP routes;
* associate HTTP methods with controllers;
* attach required HTTP middlewares.

Routes must not contain business rules, persistence logic or Mercado Pago integration code.

### Controllers

Controllers adapt HTTP requests to the application layer.

Responsibilities:

* receive HTTP input;
* perform structural input handling;
* create request DTOs;
* invoke services;
* translate application results into HTTP responses.

Controllers do not directly communicate with Mercado Pago.

### Services

Services implement application use cases and business rules.

Responsibilities:

* business validation;
* orchestration;
* decision making;
* coordination between gateways and repositories.

Services remain independent from `THorseRequest` and `THorseResponse`.

### Gateways

Gateways isolate external integrations.

```text
Service
   |
   v
IPaymentGateway
   |
   v
Mercado Pago Gateway
   |
   v
RESTRequest4Delphi
   |
   v
Mercado Pago
```

Mercado Pago-specific HTTP details belong to this boundary.

### Repositories

When persistence is required, access is exposed through repository contracts.

```text
Service
   |
   v
Repository Interface
   |
   v
Repository Implementation
   |
   v
Database
```

No persistence technology is assumed until it is explicitly defined by the project.

---

## Internal vs external contracts

A core architectural principle of this project is keeping the internal middleware API independent from Mercado Pago contracts whenever practical.

```text
Client Application
       |
       v
Internal Request DTO
       |
       v
Service
       |
       v
Mapper
       |
       v
Mercado Pago DTO
       |
       v
Gateway
       |
       v
Mercado Pago
```

The middleware is responsible for translating between internal application contracts and external payment contracts.

This reduces the impact of future external API changes on client applications.

---

## Engineering principles

The project is guided by the following engineering practices:

### SOLID

SOLID principles are applied pragmatically, with emphasis on:

* Single Responsibility Principle;
* Dependency Inversion Principle;
* small and cohesive interfaces;
* explicit dependencies;
* low coupling;
* high cohesion.

The goal is maintainability and testability, not unnecessary abstraction.

### Interface-oriented design

Application boundaries are exposed through interfaces where this provides concrete architectural value.

Typical dependency direction:

```text
Controller -> IService
Service    -> IGateway
Service    -> IRepository
```

Concrete external integrations remain behind abstractions.

### Dependency Injection

Dependencies are preferably provided explicitly through constructors.

The project avoids hidden dependencies through mutable global state or Service Locator patterns.

### Clean Code

The codebase prioritizes:

* clear naming;
* cohesive classes;
* small responsibilities;
* explicit dependencies;
* minimum duplication;
* no unnecessary abstractions;
* incremental refactoring;
* minimal technically correct changes.

### TDD and testability

New behavior and relevant refactoring are designed with testability in mind.

Preferred development flow:

```text
Requirement
    |
    v
Scenarios
    |
    v
Tests
    |
    v
Implementation
    |
    v
Refactoring
```

External Mercado Pago services must not be required for unit tests.

External dependencies should be replaceable by fakes or test implementations.

---

## Security principles

Payment integration is treated as a security-sensitive backend responsibility.

The project establishes the following principles:

* Mercado Pago private credentials remain in the middleware;
* secrets must not be committed to source control;
* `Authorization` headers must not be exposed in logs;
* Access Tokens and Client Secrets must never be logged;
* CVV and sensitive payment data must never be logged;
* TLS validation must not be disabled as a workaround;
* logs must be sanitized;
* environments must be separated;
* retries of financial operations must not be performed blindly;
* idempotency must be handled according to the official API contract;
* webhook authenticity must be validated according to current Mercado Pago documentation.

---

## Idempotency

Financial operations require special handling because a network failure does not necessarily mean the remote operation was not processed.

The project distinguishes:

```text
New logical operation
```

from:

```text
Retry of the same logical operation
```

When idempotency is required by the selected Mercado Pago API, the implementation must follow the current official contract.

A retry must not automatically create a new financial operation without evaluating the previous request state.

---

## Correlation and observability

Where applicable, requests should use a `CorrelationId` across the middleware flow:

```text
Client
  |
  v
Horse
  |
  v
Controller
  |
  v
Service
  |
  v
Gateway
  |
  v
External Service
```

`CorrelationId` and `IdempotencyKey` represent different concepts and must not be treated as interchangeable identifiers.

Logs should provide enough diagnostic information without exposing sensitive data.

---

## Error handling

The middleware should preserve proper HTTP semantics.

External Mercado Pago failures must not automatically be exposed directly to consumers.

The application boundary is responsible for translating technical failures into an appropriate internal error contract while preserving sanitized diagnostic information.

Large duplicated `try/except` blocks across controllers should be avoided when centralized HTTP exception handling is appropriate.

---

## Data serialization

`dataset-serialize` is used only when `TDataSet` ↔ JSON conversion is an appropriate infrastructure concern.

It is not intended to replace:

* DTOs;
* entities;
* domain models;
* mappers.

When business rules operate on persisted data, the preferred direction is:

```text
TDataSet
   |
   v
Mapper
   |
   v
DTO / Entity
   |
   v
Service
```

---

## Mercado Pago integration

Mercado Pago APIs are treated as external contracts that may evolve independently from this middleware.

Before implementing or changing an integration, the project requires validation of the current official documentation regarding, when applicable:

* product/API being used;
* endpoint;
* HTTP method;
* authentication;
* required headers;
* request contract;
* response contract;
* idempotency;
* webhook rules;
* statuses;
* error responses;
* environment requirements.

No Mercado Pago product is assumed automatically.

The actual payment flow must be selected according to the business requirement and the current official documentation.

Official documentation:

https://www.mercadopago.com.br/developers/

---

## Project structure

The conceptual organization follows responsibilities rather than framework convenience:

```text
MercadoPago
|
+-- Contracts
+-- Routes
+-- Controllers
+-- Services
+-- Gateways
+-- DTO
+-- Mappers
+-- Middlewares
+-- Serialization
+-- Config
+-- Exceptions
+-- Tests
```

This represents the intended architectural organization.

Directories and units should only be introduced when they have an actual responsibility in the implementation.

---

## Design decisions

### Why a middleware?

Directly coupling a desktop or business application to an external payment provider makes external API changes propagate through the client codebase.

The middleware creates a controlled boundary:

```text
Application Contract
        !=
Mercado Pago Contract
```

This allows infrastructure-specific concerns to remain isolated from business-facing consumers.

### Why gateways?

Gateways prevent external HTTP contracts from spreading throughout controllers and services.

They provide a single architectural boundary for external payment communication.

### Why interfaces?

Interfaces make dependencies explicit and allow external implementations to be replaced during automated tests.

### Why Service Layer?

The Service Layer keeps use cases and business decisions independent from Horse and HTTP-specific objects.

---

## Development rules

Some fundamental rules of the project are:

```text
Route      != Business Rule
Controller != Mercado Pago Client
Service    != HTTP Framework
DTO        != Infrastructure
Repository != Controller
Gateway    == External Integration Boundary
```

And specifically:

```text
Horse                -> HTTP infrastructure
RESTRequest4Delphi   -> outbound HTTP infrastructure
dataset-serialize    -> TDataSet serialization infrastructure
Mercado Pago         -> external dependency
```

Business rules must remain as independent as practical from all four.

---

## Roadmap

The project evolves incrementally.

Planned engineering concerns include:

* [ ] internal API contract evolution;
* [ ] application authentication and authorization;
* [ ] Mercado Pago gateway implementations according to validated use cases;
* [ ] persistence where required by business requirements;
* [ ] standardized application error contract;
* [ ] correlation and structured logging;
* [ ] idempotency handling where required;
* [ ] webhook processing and authenticity validation where applicable;
* [ ] unit tests for services and business rules;
* [ ] integration tests for HTTP boundaries;
* [ ] environment configuration strategy;
* [ ] automated quality and build validation.

Items in this roadmap represent planned engineering work and must not be interpreted as already implemented functionality.

---

## Current status

This repository is under active development.

Features, endpoints and Mercado Pago products are documented as implemented only after their contracts and behavior are validated against the actual source code and the current official documentation.

The project deliberately avoids presenting planned capabilities as completed features.

---

## References

### Mercado Pago

Official developer documentation:

https://www.mercadopago.com.br/developers/

### Horse

https://github.com/HashLoad/horse

### RESTRequest4Delphi

https://github.com/viniciussanchez/RESTRequest4Delphi

### dataset-serialize

https://github.com/viniciussanchez/dataset-serialize

---

## Disclaimer

This is an independent software project and is not an official Mercado Pago SDK.

Mercado Pago is an external platform integrated by the middleware according to its official developer documentation.

---

## Contributing

The project currently prioritizes architectural consistency, security, testability and incremental evolution.

Contributions should preserve the existing architectural boundaries and avoid introducing infrastructure dependencies into business rules.

---

## License

This project is licensed under the **MIT License**.

* [MIT License](LICENSE)
