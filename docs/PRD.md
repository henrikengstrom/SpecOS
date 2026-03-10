# Product Requirements Document: SpecOS

**Version:** 0.1 (Concept)
**Status:** Draft
**Author:** Henrik Engström
**Date:** 2026

---

# 1. Overview

## Product Name

**SpecOS — Specification Operating System for AI Agents**

## Vision

SpecOS converts **human intent into deterministic, machine-readable specifications** that autonomous AI agents can reliably execute.

SpecOS is designed as a companion to [SpecKit (SDD)](https://github.com/github/spec-kit). SpecKit defines the Spec-Driven Development methodology and workflow — intent capture, planning, task breakdown, and implementation. SpecOS provides the **formal specification language and compiler** that makes SpecKit's specifications machine-verifiable and compilable into deterministic artifacts. See [SpecOS + SpecKit Integration](speckit-integration.md) for the full integration design.

Today most AI development relies on prompts. Prompts are ambiguous, unstable, and difficult to validate. SpecOS introduces a **formal intermediate representation (IR) for intent** that sits between human ideas and AI execution.

Instead of:

```
Prompt → Code
```

SpecOS enables:

```
Intent → Specification → Agents → Verified Implementation
```

The specification becomes the **source of truth** for systems built with AI.

---

# 2. Problem Statement

AI coding tools have dramatically improved code generation but still fail at **capturing requirements accurately**.

Most failures occur because:

- Requirements are incomplete
- Edge cases are not specified
- Business rules are implicit
- State transitions are undefined

Prompt-based development hides these issues.

SpecOS solves this by:

- extracting constraints
- formalizing state transitions
- defining invariants
- generating tests automatically

This produces a **deterministic specification graph** that AI agents can implement with far greater reliability.

---

# 3. Why Not TLA+, Alloy, or Z?

Formal specification languages like **TLA+**, **Alloy**, and **Z** already exist. SpecOS is not trying to replace them. It occupies a different point in the design space:

| Dimension | TLA+ / Alloy / Z | SpecOS |
|---|---|---|
| **Target audience** | Researchers, verification engineers | Application developers |
| **Learning curve** | Steep (weeks to months) | Minimal (minutes to hours) |
| **Scope** | General-purpose formal verification | API-driven backend systems (v1) |
| **AI integration** | Not designed for AI agents | Core design goal |
| **Output** | Proofs, counterexamples | API schemas, tests, agent contracts |
| **Generation** | Hand-written by specialists | Generated from natural language |

SpecOS intentionally **trades theoretical completeness for practical adoption**. The goal is not to prove system correctness in the academic sense, but to give AI agents unambiguous, structured contracts that are far more reliable than prompts.

Developers who need full formal verification should use TLA+ or Alloy. Developers who need a better specification layer for AI-assisted development should use SpecOS.

---

# 4. Goals

SpecOS aims to:

1. Convert natural language intent into structured specifications
2. Detect ambiguous or missing requirements
3. Provide agents with deterministic contracts instead of prompts
4. Automatically generate validation tests
5. Serve as the specification layer for AI development platforms

---

# 5. Target Users

## Primary

**Software Developers** building:

- APIs
- Backend systems
- Internal tools
- Automation systems

## Secondary

- Product managers working with engineering teams
- AI agent platforms
- Developer tool companies

---

# 6. Product Principles

SpecOS is designed around five principles.

### 1. Specification First

Specifications are the source of truth, not prompts or code.

### 2. Deterministic Execution

Agents receive structured contracts instead of ambiguous instructions. Note: determinism begins **at the specification level**. The natural-language-to-spec step is inherently non-deterministic; SpecOS ensures that once a spec is finalized, execution against it is fully deterministic.

### 3. Constraint Discovery

The system actively identifies missing rules and edge cases.

### 4. Composability

Specifications form graphs that can be reused and extended.

### 5. Model Independence

SpecOS works with any AI model or agent platform.

---

# 7. Runtime Model

SpecOS v1 ships as a **CLI tool** with the following modes of operation:

### CLI

```
specos init                        # Initialize a SpecOS project
specos validate spec.specos        # Validate a specification file
specos compile spec.specos         # Compile to output artifacts
specos test spec.specos            # Generate and run tests
specos export openapi spec.specos  # Export to OpenAPI format
```

### Library

SpecOS also exposes a **programmatic API** for embedding in other tools:

```
specos::compile("spec.specos") -> CompilationResult
```

### Integration Points

- **CI pipelines:** `specos validate` as a pre-merge check
- **AI agent platforms:** agents consume compiled spec artifacts
- **Editor extensions:** syntax highlighting and validation for `.specos` files
- **MCP server (future):** expose SpecOS as a tool for AI coding assistants

SpecOS is **not** a hosted service. Specifications live in the repository alongside code.

---

# 8. Core Architecture

SpecOS consists of five primary components connected as follows:

```
                     ┌─────────────────────-─┐
                     │    Developer / User   │
                     └────────-──┬─────────-─┘
                                 │ natural language / DSL
                                 ▼
                  ┌─────────────────────────────-┐
                  │    Intent Capture Engine     │
                  │  (ambiguity detection,       │
                  │   constraint discovery,      │
                  │   ontology-driven expansion) │
                  └──────────────┬─────────────-─┘
                                 │ structured intent
                                 ▼
                  ┌────────────────────────────-─┐
                  │   Specification Generator    │
                  │  (produces SpecOS DSL)       │
                  └──────────────┬─────────────-─┘
                                 │ .specos files
                                 ▼
              ┌─────────────────────────────────-────-─┐
              │       Specification Validator          │
              │  (missing transitions, contradictions, │
              │   undefined states, incomplete rules)  │
              └──────────────────┬───────────────-───-─┘
                                 │ validated spec
                                 ▼
              ┌──────────────────────────────────────┐
              │         Specification Graph (DAG)    │
              │  ┌────────┐ ┌────────┐ ┌──────────-┐ │
              │  │Entities│→│  State │→│Constraints│ │
              │  │        │ │Machines│ │           │ │
              │  └────┬───┘ └────┬───┘ └─────┬────-┘ │
              │       │          │           │       │
              │       └─────────-┴───────────┘       │
              │                  │                   │
              │           ┌──────┴──────┐            │
              │           │  Operations │            │
              │           └─────────────┘            │
              └──────────────────┬───────────────────┘
                                 │
                   ┌─────-───────┼────────────┐
                   ▼             ▼            ▼
            ┌────────────┐┌──────────-┐┌─────────────┐
            │ Constraint ││   Test    ││    Agent    │
            │ Compiler   ││ Generator ││  Contract   │
            │            ││           ││   Emitter   │
            └──────┬─────┘└─────-┬──-─┘└─────-┬──-───┘
                   │             │            │
                   ▼             ▼            ▼
           ┌-────────────┐┌─────────-─┐┌─────────-───┐
           │ OpenAPI     ││   Tests   ││   Agent     │
           │ JSON Schema ││           ││  Contracts  │
           │ DB Models   ││           ││             │
           └-────────────┘└──────────-┘└───────────-─┘
```

The **Specification Graph** is the central data structure — a Directed Acyclic Graph where nodes represent entities, state machines, constraints, and operations, and edges represent dependencies between them.

---

# 9. Specification Model

Specifications combine four formal components.

## 9.1 Types

Defines the structure of data.

Example:

```
User
  id: UUID
  email: string
  created_at: timestamp
```

This layer resembles:

- TypeScript
- GraphQL schemas
- protobuf

---

## 9.2 State Machines

Defines allowed state transitions for entities.

Example:

```
Subscription

states:
  trial
  active
  canceled

transitions:
  trial -> active
  active -> canceled
```

This provides deterministic behavior modeling based on automata theory.

---

## 9.3 Constraints

Defines logical rules that must always hold.

Example:

```
refund_amount <= payment_amount
cancellation_date >= start_date
```

Constraints prevent invalid states and enforce business logic.

---

## 9.4 Behavioral Properties

Defines expected behavior for testing and validation.

Example:

```
properties:

canceling twice is idempotent
upgrading does not reduce balance
```

These properties enable automatic test generation.

---

# 10. Core Components

## 10.1 Intent Capture Engine

> **Note:** Intent capture is primarily handled by SpecKit via `/speckit.specify` and `/speckit.clarify`. SpecOS does not replicate this workflow. However, when SpecOS is used standalone (without SpecKit), it provides a lightweight conversational interface for gathering requirements. When used with SpecKit, SpecOS consumes the clarified `spec.md` as input.

The engine performs:

- ambiguity detection
- constraint discovery
- domain inference (via the Spec Ontology Layer)

Example interaction:

User:

"Build a subscription billing system"

System:

"Should cancellations be immediate or at the end of the billing cycle?"

---

## 10.2 Specification Generator

Transforms captured intent into the SpecOS DSL.

Example:

```
entity Subscription

billing_cycle: monthly
refund_policy: prorated

state_machine:
  trial -> active
  active -> canceled
```

---

## 10.3 Specification Validator

Validates specifications before code generation.

Detects:

- missing transitions
- logical contradictions
- undefined states
- incomplete constraints

---

## 10.4 Constraint Compiler

Compiles specifications into artifacts usable by AI agents.

Outputs include:

- API schemas
- agent contracts (see section 10.6)
- validation rules
- test suites

---

## 10.5 Test Generator

Automatically produces:

- edge case tests
- property tests
- fuzz tests

This ensures agent implementations comply with the specification.

---

## 10.6 Agent Contract Format

The most critical output of the Constraint Compiler is the **agent contract** — the structured artifact that AI agents receive instead of a prompt. An agent contract is a JSON document that fully describes what the agent must implement.

Example agent contract for `cancel_subscription`:

```json
{
  "contract": "cancel_subscription",
  "version": "1.0.0",
  "entity": "Subscription",
  "operation": {
    "name": "cancel_subscription",
    "type": "state_transition",
    "input": {
      "subscription_id": { "type": "uuid", "required": true }
    },
    "output": {
      "subscription_id": { "type": "uuid" },
      "status": { "type": "enum", "value": "canceled" },
      "canceled_at": { "type": "timestamp" }
    },
    "preconditions": [
      { "field": "Subscription.status", "operator": "==", "value": "active" }
    ],
    "effects": [
      { "field": "Subscription.status", "from": "active", "to": "canceled" },
      { "field": "Subscription.canceled_at", "set": "now()" }
    ],
    "error_cases": [
      {
        "condition": "Subscription.status != active",
        "error": "INVALID_STATE",
        "message": "Subscription is not active"
      },
      {
        "condition": "Subscription not found",
        "error": "NOT_FOUND",
        "message": "Subscription does not exist"
      }
    ]
  },
  "constraints": [
    { "rule": "canceled_at >= created_at" },
    { "rule": "status == canceled => canceled_at != null" }
  ],
  "properties": [
    { "name": "idempotent", "description": "Calling cancel twice produces the same result" }
  ],
  "tests": {
    "happy_path": "Active subscription transitions to canceled",
    "edge_cases": [
      "Cancel already-canceled subscription returns same result",
      "Cancel trial subscription returns error"
    ]
  }
}
```

This contract gives an AI agent everything it needs: input/output schemas, preconditions, state transitions, error handling rules, constraints, and expected test cases. No ambiguity, no prompt interpretation required.

---

# 11. Example Workflow

## Step 1

Developer describes system intent.

Example:

"Create a subscription billing API with monthly plans and prorated refunds."

## Step 2

SpecOS extracts requirements and generates specification.

## Step 3

Validator detects missing rules.

Example warning:

"Cancellation behavior not defined"

## Step 4

Developer clarifies rules.

## Step 5

SpecOS generates:

- specification
- API contract
- tests

## Step 6

AI agents implement the system.

---

# 12. Example Specification

```
entity Subscription

fields:
  id: UUID
  user_id: UUID
  status: enum(trial, active, canceled)

state_machine:
  trial -> active
  active -> canceled

constraints:
  cancellation_date >= created_at

properties:
  cancel_subscription is idempotent
```

---

# 13. Initial Product Scope (V1)

The first version of SpecOS focuses on **API development**.

Features:

- specification generation for APIs
- state machine modeling
- automatic test generation
- OpenAPI export

Integration targets:

- GitHub
- CI pipelines
- AI coding tools

---

# 14. Success Metrics

### Specification Completeness

Percentage of systems with no missing constraints.

### Bug Reduction

Reduction in production bugs related to business logic.

### Developer Adoption

Number of repositories using SpecOS in CI.

### Specification Reuse

Reuse of specification components across projects.

---

# 15. Long-Term Vision

SpecOS becomes the **standard specification layer for AI-driven software development**.

Future capabilities include:

- autonomous system simulation
- consensus agent validation
- formal verification
- cross-agent interoperability

The specification layer becomes the interface between human intent and autonomous software construction.

---

# 16. Spec Ontology Layer (v0.1)

The Spec Ontology Layer defines the **core vocabulary of software systems** used by SpecOS. It allows the system to infer missing requirements, normalize developer intent, and generate richer specifications.

Ontology concepts represent reusable primitives that can be specialized inside project specifications.

The v0.1 ontology focuses on **API‑driven backend systems**, providing a minimal but powerful foundation.

---

## 16.1 Core Ontology Concepts (v0.1)

### Identity & Access

```
User
Account
Role
Permission
Session
Token
```

These represent authentication and authorization primitives.

Relationships:

```
User belongs_to Account
User has Session
Role grants Permission
Token authenticates Session
```

---

### Resource Modeling

```
Resource
Entity
Collection
Identifier
Metadata
```

These define structured data objects within systems.

Relationships:

```
Resource has Identifier
Collection contains Resource
Resource has Metadata
```

---

### State & Lifecycle

```
State
Transition
Event
Action
Workflow
```

These represent automata‑style behavior modeling.

Relationships:

```
Event triggers Transition
Transition changes State
Workflow contains States
Action executes during Transition
```

---

### API Systems

```
API
Endpoint
Request
Response
Schema
Operation
```

These concepts model API behavior.

Relationships:

```
API exposes Endpoint
Endpoint accepts Request
Endpoint returns Response
Operation executes Endpoint
```

---

### Reliability & Control

```
RetryPolicy
RateLimit
Timeout
Idempotency
Queue
```

These represent operational guarantees.

Relationships:

```
Endpoint uses RateLimit
Operation follows RetryPolicy
Operation requires Idempotency
Queue buffers Operation
```

---

## 16.2 Example Ontology Concept

Example concept definition used internally by SpecOS:

```
Concept: Endpoint

properties:
  method
  path
  request_schema
  response_schema

constraints:
  endpoint must belong_to API

relationships:
  accepts: Request
  returns: Response
```

---

## 16.3 Ontology‑Driven Requirement Discovery

When developers describe intent, SpecOS uses the ontology to infer missing elements.

Example input:

"Build an API for managing subscriptions"

Ontology expansion suggests concepts:

```
Subscription
Plan
Invoice
Payment
CancellationPolicy
RetryPolicy
```

SpecOS then asks clarification questions such as:

- "Should failed payments be retried?"
- "Can subscriptions be canceled immediately or at billing cycle end?"

---

## 16.4 Ontology Extension

Developers can extend the ontology with project‑specific concepts.

Example:

```
Concept: TeamSubscription

extends: Subscription

properties:
  team_id
  seat_count
```

These extensions inherit constraints and relationships from parent concepts.

---

## 16.5 Future Ontology Modules

Future versions of SpecOS may add domain modules including:

- Billing Systems
- Authentication
- Workflow Engines
- Data Pipelines
- Event Processing
- Commerce Systems

Each module introduces specialized concepts, constraints, and behavioral templates.

---

## 16.6 Design Principles for the Ontology

The SpecOS ontology should follow several rules:

1. Concepts must be composable
2. Relationships must be explicitly typed
3. Constraints must be machine‑verifiable
4. Domains should remain modular
5. Concepts should map cleanly to implementation primitives

---

## 16.7 Long‑Term Vision

Over time the ontology becomes a **knowledge graph of software architecture patterns**.

This enables SpecOS to:

- infer missing requirements
- suggest system architectures
- generate more accurate specifications
- guide AI agents toward correct implementations

The ontology ultimately becomes the **semantic backbone of AI‑driven software development**.

---

# 17. SpecOS DSL v0.1

The SpecOS DSL is the human-editable language used to define machine-readable specifications. It is designed to be:

- structured enough for deterministic execution
- simple enough for everyday developer use
- expressive enough to capture state, constraints, and behavior

The v0.1 DSL is intentionally narrow and optimized for **API-driven backend systems**.

---

## 17.1 DSL Design Goals

The DSL should:

1. Be readable in plain text
2. Be easy to generate from natural language
3. Be easy to validate mechanically
4. Map directly to test generation and agent execution
5. Avoid the complexity of formal verification languages such as TLA+

The intended developer experience should feel closer to:

- Terraform
- OpenAPI
- Prisma schema

than to theorem provers or academic specification languages.

---

## 17.2 Core Constructs

The v0.1 DSL supports the following primary constructs.

### 1. system

Defines the overall service or bounded context.

### 2. entity

Defines core domain objects.

### 3. state_machine

Defines states and allowed transitions.

### 4. operation

Defines externally visible behavior such as API actions.

### 5. constraints

Defines invariants and validation rules.

### 6. properties

Defines expected behaviors for test generation.

### 7. relationships

Defines connections between entities (see section 17.5).

### 8. errors

Defines failure modes and error handling behavior (see section 17.6).

---

## 17.3 Example DSL

```specos
system SubscriptionAPI {
  entity User {
    fields {
      id: uuid
      email: string
      created_at: timestamp
    }
  }

  entity Subscription {
    fields {
      id: uuid
      user_id: uuid
      status: enum(trial, active, canceled)
      created_at: timestamp
      canceled_at: timestamp?
    }

    relationships {
      belongs_to User via user_id
    }

    state_machine {
      trial -> active
      active -> canceled
    }

    constraints {
      canceled_at >= created_at
      status == canceled => canceled_at != null
    }

    properties {
      cancel_subscription is idempotent
    }
  }

  operation create_subscription {
    input {
      user_id: uuid
      plan_id: uuid
    }

    output {
      subscription_id: uuid
      status: enum(trial, active)
    }

    errors {
      user_not_found: "User does not exist"
      already_subscribed: "User already has an active subscription"
    }
  }

  operation cancel_subscription {
    input {
      subscription_id: uuid
    }

    output {
      subscription_id: uuid
      status: enum(canceled)
      canceled_at: timestamp
    }

    preconditions {
      Subscription.status == active
    }

    effects {
      Subscription.status -> canceled
      Subscription.canceled_at -> now()
    }

    errors {
      not_found: "Subscription does not exist"
      invalid_state: "Subscription is not active"
    }
  }
}
```

---

## 17.4 Semantics of the DSL

### system

Represents a bounded context or deployable service.

Example:

```specos
system BillingAPI { ... }
```

---

### entity

Represents a persistent domain object.

Example:

```specos
entity Invoice {
  fields {
    id: uuid
    amount: integer
    status: enum(open, paid, void)
  }
}
```

---

### fields — nullable types

Field types can be marked as **nullable** with the `?` suffix. A nullable field may contain a value or be absent/null.

```specos
fields {
  canceled_at: timestamp?    # nullable — may be null
  created_at: timestamp      # required — must always have a value
}
```

The `?` suffix can be applied to any type: `string?`, `uuid?`, `integer?`, `timestamp?`, `enum(...)? `.

Constraints can reference nullable fields to enforce conditional presence:

```specos
constraints {
  status == canceled => canceled_at != null
}
```

---

### state_machine

Represents valid lifecycle transitions.

Example:

```specos
state_machine {
  open -> paid
  open -> void
}
```

This allows SpecOS to validate transitions and generate state-based tests.

---

### operation

Represents an externally callable action, usually mapping to an API endpoint, command, or workflow trigger.

Operations should include both **input** and **output** blocks. When an operation has `effects` that change entity state, the `output` block should reflect the resulting entity state so consumers know what to expect.

Example:

```specos
operation pay_invoice {
  input {
    invoice_id: uuid
  }

  output {
    invoice_id: uuid
    status: enum(paid)
    paid_at: timestamp
  }

  preconditions {
    Invoice.status == open
  }

  effects {
    Invoice.status -> paid
  }

  errors {
    not_found: "Invoice does not exist"
    already_paid: "Invoice has already been paid"
    invalid_state: "Invoice has been voided"
  }
}
```

If an operation omits the `output` block, the compiler emits a warning and infers a minimal output from the `effects` block.

---

### constraints

Represents invariants that must always hold.

Example:

```specos
constraints {
  refund_amount <= payment_amount
}
```

Constraints are machine-validated and also used for test generation.

---

### properties

Represents expected behavioral guarantees.

Example:

```specos
properties {
  pay_invoice is idempotent
}
```

Properties can be compiled into regression tests and property-based tests.

---

## 17.5 Relationship Modeling

Entities in real systems are connected. The DSL supports explicit relationship declarations inside entity blocks.

### Syntax

```specos
entity Subscription {
  fields {
    id: uuid
    user_id: uuid
    plan_id: uuid
  }

  relationships {
    belongs_to User via user_id
    belongs_to Plan via plan_id
    has_many Invoice
  }
}
```

### Supported relationship types

| Keyword | Meaning |
|---|---|
| `belongs_to Entity via field` | This entity references another entity through a foreign key field |
| `has_one Entity` | This entity owns exactly one of another entity |
| `has_many Entity` | This entity owns zero or more of another entity |

### Validation

The compiler validates relationships:

- Referenced entities must exist in the same `system`
- `via` fields must be defined in the `fields` block
- `via` field types must match the referenced entity's identifier type
- Circular `belongs_to` chains are flagged as warnings

### Why explicit relationships matter

Without relationships, the specification is a collection of isolated entities. Relationships enable:

- referential integrity constraints
- cascade behavior modeling (e.g., deleting a user cascades to subscriptions)
- richer agent contracts that understand entity graphs
- more accurate test generation (e.g., create a User before creating a Subscription)

---

## 17.6 Error and Failure Modeling

Real systems fail. The DSL provides an `errors` block inside operations to explicitly define failure modes.

### Syntax

```specos
operation charge_subscription {
  input {
    subscription_id: uuid
  }

  output {
    charge_id: uuid
    amount: integer
    status: enum(succeeded)
  }

  errors {
    not_found: "Subscription does not exist"
    payment_failed: "Payment method was declined"
    already_charged: "Subscription already charged for this period"
  }

  on_error payment_failed {
    retry: 3
    backoff: exponential
    fallback: Subscription.status -> past_due
  }
}
```

### Error block semantics

Each entry in `errors` defines a named failure case with a human-readable description. These are compiled into:

- error response schemas in OpenAPI output
- negative test cases in test generation
- error handling rules in agent contracts

### on_error blocks

Optional `on_error` blocks define compensation or retry behavior for specific errors. Supported fields:

- `retry`: number of retry attempts
- `backoff`: retry strategy (`fixed`, `linear`, `exponential`)
- `fallback`: state transition to apply after all retries are exhausted

### Validation

The compiler checks:

- `on_error` blocks reference errors defined in the `errors` block
- `fallback` state transitions target valid states
- Operations that mutate state should define at least one error case (warning)

---

## 17.7 Compilation Targets

The SpecOS DSL is not an end in itself. It compiles into downstream artifacts.

Potential outputs include:

- OpenAPI specifications
- JSON schemas
- database models
- agent task contracts
- unit tests
- integration tests
- fuzz tests

This makes the DSL the intermediate representation between intent and implementation.

---

## 17.8 Validation Rules

The SpecOS DSL compiler should detect:

- undefined entities
- invalid field references
- unreachable states
- contradictory constraints
- operations with missing outputs
- state transitions to undefined targets
- undefined relationship targets
- `on_error` referencing undefined errors
- operations that mutate state without defining error cases

Example compiler error:

```text
ERROR: state 'past_due' referenced in transition but not defined
```

---

## 17.9 Natural Language to DSL Workflow

A major purpose of the DSL is to serve as the output of the Intent Capture Engine.

Example:

Developer input:

"Create an API where users can start a subscription trial and later cancel it."

Generated DSL draft:

```specos
system SubscriptionAPI {
  entity Subscription {
    fields {
      id: uuid
      user_id: uuid
      status: enum(trial, active, canceled)
    }

    state_machine {
      trial -> active
      active -> canceled
    }
  }
}
```

SpecOS would then ask follow-up questions for missing elements such as:

- How long does a trial last?
- Can a subscription be canceled during trial?
- Is cancellation immediate or end-of-cycle?
- What happens if the user is not found?

---

## 17.10 Why a Custom DSL

The DSL exists because existing formats only solve parts of the problem.

- OpenAPI models endpoints but not rich business behavior
- ORM schemas model data but not workflow logic
- Markdown specs are readable but not executable
- TLA+ / Alloy are powerful but not designed for AI agents and have steep learning curves

SpecOS needs a format that unifies:

- structure
- behavior
- constraints
- error handling
- relationships
- testable properties

in a syntax approachable by everyday developers and easily generated by AI.

---

## 17.11 Specification Versioning and Evolution

Specifications evolve as systems change. SpecOS supports explicit versioning to manage this.

### Version declarations

```specos
system SubscriptionAPI {
  version "2.0.0"

  # ... entities, operations, etc.
}
```

### Compatibility checking

When a specification changes, the compiler can compare against a previous version and report:

```text
specos diff spec-v1.specos spec-v2.specos

BREAKING:
  - Removed state 'trial' from Subscription.state_machine
  - Removed field 'trial_end' from Subscription.fields

COMPATIBLE:
  - Added state 'paused' to Subscription.state_machine
  - Added field 'paused_at' to Subscription.fields
  - Added operation 'pause_subscription'
```

### Evolution rules

The compiler classifies changes as **breaking** or **compatible**:

| Change | Classification |
|---|---|
| Add new entity | Compatible |
| Add new field (nullable) | Compatible |
| Add new state | Compatible |
| Add new transition | Compatible |
| Add new operation | Compatible |
| Remove field | Breaking |
| Remove state | Breaking |
| Remove transition | Breaking |
| Change field type | Breaking |
| Add new field (required) | Breaking |
| Add new constraint | Potentially breaking |

### Migration hints

For breaking changes, the compiler can suggest migration steps:

```text
MIGRATION HINT: Removed state 'trial'
  - Existing entities in state 'trial' need a migration path
  - Consider adding a transition: trial -> active (to migrate existing records)
```

### Why versioning matters

If the specification is the source of truth, evolving it safely is essential. Without versioning:

- Agent implementations may break silently when specs change
- There is no way to assess the impact of a change before applying it
- Teams cannot coordinate changes across services that share spec dependencies

---

## 17.12 Design Constraints for v0.1

The first DSL version should avoid:

- advanced theorem-proving syntax
- arbitrary user scripting
- full programming language semantics
- distributed systems modeling

The goal of v0.1 is practical adoption, not theoretical completeness.

---

## 17.13 Long-Term DSL Evolution

Future versions may support:

- modular imports
- domain-specific concept inheritance
- event-driven systems
- temporal constraints
- policy definitions
- multi-service interactions

The v0.1 DSL is intended to be the smallest useful language that can support real developer workflows.
