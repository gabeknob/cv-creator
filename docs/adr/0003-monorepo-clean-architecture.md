# ADR 0003: Monorepo and Clean Architecture

## Status

Accepted

## Context

The project is a monorepo. It should support a web app, an API app, a worker app, and shared backend layers. The architecture should be explicit enough for learning and portfolio value.

## Decision

Use a Bun workspace monorepo with layer-based packages and separate runnable apps.

Initial tree:

```txt
apps/
  web/
  api/
  worker/

packages/
  domain/
  application/
  dynamodb/
  canonical-tags/
  cv-renderer/
  object-storage/
  queue/
  llm/
  config/
```

Future packages can include:

```txt
packages/
  vector-qdrant/
  custom-cv-layout/
```

Dependency direction:

```txt
domain
  no framework, database, GraphQL, NestJS, or SDK imports

application
  depends on domain
  contains use cases, ports, repository contracts, and application DTOs

adapter packages
  depend on application and domain
  implement ports

apps/api
  depends on application and adapter packages
  exposes GraphQL resolvers, REST controllers, guards, and presenters

apps/worker
  depends on application and adapter packages
  processes queues using NestJS application context
```

Use abstract classes for application ports so NestJS can use them as dependency-injection tokens.

Use generic repository base contracts where useful, but keep domain-specific repository methods explicit.

Use presenters consistently at the delivery boundary.

## Rationale

The API and worker are delivery mechanisms for the same application. They should not import from each other, but they should share the application layer.

Layer-based packages make Clean Architecture easier to learn because dependency direction is visible. Internal folders can still be organized by domain area.

Separate adapter packages make external dependencies visible. DynamoDB, object storage, queues, LLM providers, and future vector stores can be swapped or inspected independently.

The shared CV renderer package owns semantic CV document rendering rules used by both the web preview and backend export path. Keeping it as a shared package protects preview/export parity without coupling the API app to the web app.

## Consequences

Benefits:

- Clean dependency boundaries.
- Use cases can be called by both API and worker.
- Worker stays separate without becoming a microservice.
- NestJS dependency injection can wire concrete adapters behind abstract ports.
- Portfolio readers can see architectural intent.

Tradeoffs:

- More files and package configuration.
- More ceremony for simple operations.
- Requires discipline to prevent app-to-app imports.

## Rules

- Apps must not import from each other.
- Domain must not import frameworks or infrastructure SDKs.
- Application must not import delivery concerns such as GraphQL resolvers or REST controllers.
- Infrastructure packages implement application ports.
- Presenters adapt use case output into GraphQL or REST response shapes.
