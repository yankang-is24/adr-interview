# ADR: Establishing a Compile-Time Enforced API Contract Between Backend and Frontend

**Status:** Proposed  
**Date:** 2026-03-01  

---

## Context

Currently:

- Backend exposes APIs via controllers.
- The API specification is implicitly defined in:
  - Jira tickets
  - Slack/messages
  - Backend controller implementation
- The full-stack engineer manually checks backend implementation when integrating frontend changes.

### Problems Observed

- Breaking API changes are detected late (runtime or QA).
- No single source of truth for API contracts.
- Documentation is fragmented and quickly outdated.
- Onboarding requires manual knowledge transfer.
- Future external consumers (e.g., mobile apps, partners) would struggle to rely on undocumented APIs.

---

## Goals

1. Detect breaking changes at **compile time** where possible.
2. Improve API documentation quality and consistency.
3. Enable safe API evolution.
4. Support additional internal or external consumers.
5. Avoid excessive operational overhead.

---

## Non-Goals

- Redesigning the entire backend architecture.
- Introducing heavy infrastructure unless justified.

---

## Decision Drivers

- Developer productivity
- Compile-time safety
- Backward compatibility management
- Tooling ecosystem maturity
- Operational complexity
- Scalability to multiple consumers

---

## Options Considered

---

### Option A – Status Quo (Controllers as Contract)

Backend controllers implicitly define the API.  
Frontend manually aligns with backend implementation.

#### Pros
- No additional tooling
- Simple setup
- Flexible backend development

#### Cons
- No compile-time guarantees across services
- High risk of silent breaking changes
- No authoritative documentation
- Poor scalability to additional consumers
- Knowledge siloed in implementation

#### Assessment
Does not meet compile-time safety or documentation goals.

---

### Option B – Shared Type Definitions Package

Create a shared TypeScript package (e.g., `@company/api-types`) defining request/response DTOs.  
Both frontend and backend import the same types.

#### Pros
- Compile-time safety (especially in monorepo)
- Easy to implement
- Minimal tooling overhead
- Strong FE/BE alignment

#### Cons
- Works best only in TypeScript ecosystems
- Couples frontend and backend release cycles
- No formal API documentation
- Weak support for non-TS external consumers
- No explicit versioning strategy

#### Assessment
Improves compile-time detection but limited long-term scalability.

---

### Option C – OpenAPI (Contract-First or Code-First)

Define APIs using an OpenAPI specification (YAML/JSON).  
Generate:

- Backend controller interfaces or validators
- Frontend typed client
- API documentation site
- Mock server

Breaking changes detected via:

- Schema diff tools in CI
- Type generation failures
- Contract validation tests

#### Pros
- Single source of truth
- Language-agnostic
- Enables typed client generation
- Built-in documentation (Swagger UI)
- Supports external consumers
- Allows automated breaking-change detection

#### Cons
- Additional tooling and maintenance
- Requires governance discipline
- Slightly slower iteration initially

#### Assessment
Strong candidate balancing safety, documentation, and ecosystem support.

---

### Option D – GraphQL Schema as Contract

Define a GraphQL schema as the API contract.  
Frontend consumes via generated typed queries.

#### Pros
- Strong type system
- Client-driven data selection
- Excellent code generation ecosystem
- Breaking changes detectable via schema diffing
- Self-documenting schema

#### Cons
- Architectural shift required
- Higher operational complexity
- Potential overkill for simple REST use cases
- Learning curve

#### Assessment
Provides strong guarantees but may be disproportionate to current needs.

---

### Option E – Consumer-Driven Contract Testing (e.g., Pact)

Frontend defines expected API interactions.  
Backend verifies compliance in CI.

#### Pros
- Detects breaking changes automatically
- Encourages explicit expectations
- Works across different stacks

#### Cons
- Does not provide authoritative documentation
- Adds CI complexity
- Requires explicit contract format
- Can become brittle if poorly maintained

#### Assessment
Useful complement to Options C or D but insufficient as standalone solution.

---

## Decision

Adopt **Option C – OpenAPI as the canonical API contract**, with the following approach:

1. Maintain an OpenAPI specification as the single source of truth.
2. Generate:
   - Backend interfaces or request/response validators.
   - Frontend typed API client.
3. Enforce breaking-change detection in CI via:
   - Schema diff checks.
   - Compatibility validation tools.
4. Publish auto-generated API documentation.
5. Consider introducing consumer-driven contract testing later as an enhancement.

---

## Consequences

### Positive

- Breaking changes detectable during build/CI.
- Clear centralized documentation.
- Easier onboarding.
- Enables future external consumers.
- Improves long-term scalability.

### Negative

- Increased initial setup cost.
- Requires governance and review discipline.
- Slightly slower iteration during early adoption.

---

## Migration Plan (High-Level)

1. Pilot with one critical endpoint.
2. Introduce OpenAPI spec alongside existing controllers.
3. Add code generation and CI validation.
4. Gradually migrate remaining endpoints.

---

## Open Questions

- Contract-first or code-first approach?
- API versioning strategy (URL vs header-based)?
- Backward compatibility guarantees and deprecation policy?
- Should runtime schema validation be enforced in production?
