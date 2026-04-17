---
name: laravel-layered-architecture
description: Enforces layered architecture in Laravel projects with domain layer isolation, dependency direction rules, and layer responsibilities. Use when creating new classes (Services, Actions, Repositories, ValueObjects, Aggregates), moving code between layers, or reviewing architecture in a project with a `domain/src/` directory. Invoke automatically whenever a new class is being placed and the project has this layered structure.
user-invocable: false
---

# Laravel Layered Architecture

## Layers (top to bottom)

**Presentation** (`app/Commands/`, `app/Http/`, routes, views)
Handles user interaction. Delegates to application services. Never contains business logic.

**Application** (`app/Services/`)
Orchestrates workflows between presentation, domain, and infrastructure. Translates external input into domain objects. No business rules here -- only coordination.

**Domain** (`domain/src/`)
Core business logic. Separate PSR-4 root, zero framework dependencies. Contains aggregates, value objects, domain services, events, actions, repositories (interfaces only).

**Infrastructure** (`app/Aggregates/`, `app/Repositories/`, `app/Providers/`)
Implements domain interfaces. Framework adapters, persistence, external service integrations. Wires domain contracts to concrete implementations via service providers.

## Dependency Direction

Strict top-down only: Presentation -> Application -> Domain <- Infrastructure.

- Domain depends on nothing (no `use App\...` or `use Illuminate\...` in `domain/src/`). This keeps the core business logic testable without booting the framework and makes it portable if the framework ever changes.
- Application depends on domain, never on presentation.
- Infrastructure implements domain interfaces (dependency inversion). The domain defines what it needs; infrastructure decides how to fulfil it.
- Presentation depends on application services, never directly on domain aggregates.

## Domain Layer Rules

- Lives in `domain/src/` with its own `Domain\` namespace (separate PSR-4 autoload root in composer.json).
- Tests live in `domain/tests/`.
- Internal structure mirrors bounded contexts or aggregate roots: `domain/src/Aggregates/ContextName/`.
- Each aggregate context contains: Actions (commands), Events, Entities, ValueObjects, Services, Repositories (interfaces), Exceptions.
- Value objects are immutable. Enforce invariants in constructors/factory methods. Immutability means callers can share and cache them safely without defensive copying.
- Domain services contain logic that doesn't belong to a single aggregate.
- Repository interfaces live in the domain. Implementations live in `app/`.

## Application Layer Rules

- Each service gets its own directory under `app/Services/ServiceName/`.
- Define contracts (interfaces) alongside implementations.
- Adapters for external libraries live here (e.g., `PhpSpreadsheetAdapter`).
- Service-specific exceptions in `Exceptions/` subdirectory.

## When Creating New Code

New business concept or rule -> `domain/src/`. Ask: "Does this exist without Laravel?" If yes, it's domain.

New CLI command or HTTP endpoint -> `app/Commands/` or `app/Http/`. Inject application services, don't touch domain directly.

New external integration (API client, file reader, database) -> `app/Services/` with a contract interface. If the domain needs it, define the interface in `domain/src/` and implement in `app/`.

New aggregate -> Create under `domain/src/Aggregates/Name/` with Actions/, Events/, ValueObjects/, Repositories/ subdirectories. Infrastructure adapters go in `app/Aggregates/Name/`.

## Anti-patterns

- Importing `Illuminate\*` in domain code.
- Business logic in commands, controllers, or service providers.
- Domain objects depending on application services.
- Skipping the application layer (presentation calling domain directly).
- Concrete infrastructure classes in domain signatures (use interfaces).
