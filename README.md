# SpecOS — Specification Operating System for AI Agents

SpecOS converts **human intent into deterministic, machine-readable specifications** that autonomous AI agents can reliably execute.

Instead of `Prompt → Code`, SpecOS enables:

```
Intent → Specification → Agents → Verified Implementation
```

The specification becomes the **source of truth** for systems built with AI.

## Why

AI coding tools generate code well but fail at **capturing requirements accurately**. Requirements are incomplete, edge cases are unspecified, business rules are implicit, and state transitions are undefined. Prompt-based development hides these problems.

SpecOS solves this with a formal intermediate representation that extracts constraints, formalizes state transitions, defines invariants, and generates tests automatically.

## Example

```specos
system SubscriptionAPI {
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

## Usage (planned)

```
specos init                       # Initialize a SpecOS project
specos validate spec.specos       # Validate a specification file
specos compile spec.specos        # Compile to output artifacts (OpenAPI, tests, agent contracts)
specos test spec.specos           # Generate and run tests
specos diff v1.specos v2.specos   # Compare spec versions for breaking changes
```

## Key Features

- **Specification DSL** — structured, human-readable language for defining types, state machines, constraints, operations, relationships, and error handling
- **Constraint discovery** — detects missing rules, undefined states, and incomplete specifications
- **Agent contracts** — compiles specs into structured JSON contracts that AI agents consume (not prompts)
- **Test generation** — automatic edge-case, property, and fuzz tests from specifications
- **Spec versioning** — breaking vs. compatible change detection with migration hints
- **Ontology layer** — reusable domain concepts (Identity, Resources, State, APIs, Reliability) that drive requirement discovery
- **Compilation targets** — OpenAPI, JSON Schema, database models, agent contracts, test suites

## SpecKit Integration

SpecOS is designed to work with [SpecKit (SDD)](https://github.com/github/spec-kit). SpecKit owns the workflow — intent capture, planning, task breakdown, and implementation. SpecOS provides the **formal specification layer** that sits between SpecKit's natural language specs and code generation, adding machine validation, artifact compilation, and spec versioning.

See [SpecOS + SpecKit Integration](docs/speckit-integration.md) for the full integration design.

## Documentation

- [Product Requirements Document](docs/PRD.md) — full architecture, DSL reference, ontology, and design rationale
- [SpecKit Integration](docs/speckit-integration.md) — how SpecOS and SpecKit work together

## Status

**v0.1 — Concept phase.** The initial scope focuses on API-driven backend systems.

## License

See [LICENSE](LICENSE).
