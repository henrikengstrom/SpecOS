# SpecOS + SpecKit Integration

## How SpecOS and SpecKit Work Together

[SpecKit](https://github.com/github/spec-kit) defines **Spec-Driven Development (SDD)** — the methodology, workflow, and tooling for building software from specifications using AI agents. SpecOS provides a **formal specification language (DSL)** that makes SpecKit's specifications machine-verifiable and compilable.

SpecKit is the master. SpecOS does not duplicate SpecKit's workflow, orchestration, constitutional framework, or agent integration. Instead, SpecOS fills a specific gap in the SDD pipeline: the lack of a machine-readable, validatable specification format between natural language intent and code generation.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SpecKit (SDD Workflow)                       │
│                                                                     │
│   /speckit.constitution → principles & guidelines                   │
│   /speckit.specify      → natural language PRD (spec.md)            │
│   /speckit.clarify      → resolve ambiguities                       │
│                                                                     │
│        ┌──────────────────────────────────────────┐                 │
│        │            SpecOS (NEW LAYER)            │                 │
│        │                                          │                 │
│        │  spec.md ──→ spec.specos ──→ validated   │                 │
│        │  (NL)        (formal DSL)    spec graph  │                 │
│        │                    │                     │                 │
│        │              ┌─────┴──────┐              │                 │
│        │              │ Compile to │              │                 │
│        │              │ artifacts  │              │                 │
│        │              └─────┬──────┘              │                 │
│        │                    │                     │                 │
│        │    ┌───────────┬───┴───┬───--──┐         │                 │
│        │    │           │       │       │         │                 │
│        │  agent      tests  OpenAPI    data       │                 │
│        │  contracts         schema     models     │                 │
│        └──────────────────────────────────────────┘                 │
│                                                                     │
│   /speckit.plan         → implementation plan (uses SpecOS output)  │
│   /speckit.tasks        → task breakdown                            │
│   /speckit.implement    → code generation                           │
│   /speckit.analyze      → cross-artifact consistency                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## What SpecKit Already Does (SpecOS Defers)

These capabilities belong to SpecKit. SpecOS does not replicate them.

| Capability | SpecKit Mechanism | SpecOS Role |
|---|---|---|
| **Workflow orchestration** | `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement` | None — SpecOS is called within this workflow, not alongside it |
| **Intent capture** | AI-driven dialogue via `/speckit.specify` and `/speckit.clarify` | None — SpecOS consumes the output of intent capture, it does not perform it |
| **Constitutional principles** | `memory/constitution.md` with nine articles | None — SpecOS respects the constitution, does not define one |
| **Uncertainty markers** | `[NEEDS CLARIFICATION]` markers in templates | None — SpecOS expects clarified specs as input |
| **Agent integration** | Multi-agent support (Copilot, Claude, Gemini, Cursor, etc.) | None — SpecOS outputs artifacts that any agent can consume |
| **Template-driven quality** | Checklists, phase gates, detail hierarchy | None — SpecOS validates at the formal level, not the template level |
| **Branch/feature management** | `specs/[branch-name]/` directory structure | None — `.specos` files live within SpecKit's feature directories |

## What SpecOS Adds (The Gap It Fills)

SpecKit's specifications today are **structured natural language** — well-organized Markdown with templates and checklists. This is effective for guiding AI agents but has limitations:

1. **No machine validation.** A `spec.md` can contain contradictory requirements, undefined states, or impossible transitions. These are caught (if at all) by AI analysis during `/speckit.analyze`, not by deterministic validation.

2. **No formal compilation.** SpecKit's specs generate code through AI interpretation. There is no intermediate step that produces deterministic artifacts like API schemas, test cases, or agent contracts from the spec itself.

3. **No specification diffing.** When specs change, there is no systematic way to classify changes as breaking vs. compatible.

4. **No relationship or state machine enforcement.** Entity relationships, valid state transitions, and cross-entity constraints exist only as prose.

SpecOS addresses each of these:

| Gap | SpecOS Solution |
|---|---|
| No machine validation | DSL compiler detects contradictions, undefined states, unreachable transitions, missing error cases |
| No formal compilation | `.specos` files compile to OpenAPI specs, JSON schemas, agent contracts, test suites |
| No specification diffing | `specos diff` classifies changes as breaking or compatible with migration hints |
| No formal state/relationship modeling | DSL has `state_machine`, `relationships`, `constraints`, and `errors` as first-class constructs |

## Integration Points in the SDD Workflow

### 1. After `/speckit.specify` and `/speckit.clarify` — Generate `.specos`

Once the natural language spec (`spec.md`) is clarified and stable, SpecOS generates a formal specification from it:

```
spec.md (SpecKit output) → spec.specos (SpecOS formal DSL)
```

This can happen:
- **Automatically** — as part of the `/speckit.plan` step, the AI agent translates `spec.md` into `.specos` and validates it
- **Manually** — the developer writes or edits `.specos` files directly

The `.specos` file lives in the SpecKit feature directory:

```
specs/003-subscription-billing/
  spec.md           ← SpecKit (natural language requirements)
  spec.specos       ← SpecOS (formal, machine-readable specification)
  plan.md           ← SpecKit (implementation plan)
  data-model.md     ← SpecKit (can be generated from .specos)
  contracts/        ← SpecKit (can be generated from .specos)
  tasks.md          ← SpecKit (task breakdown)
```

### 2. During `/speckit.plan` — Compile Artifacts

The implementation plan can reference SpecOS-compiled artifacts instead of manually defining them:

- **Data models** — generated from `entity` definitions in `.specos`
- **API contracts** — generated from `operation` definitions in `.specos`
- **Test scenarios** — generated from `constraints` and `properties` in `.specos`
- **Agent contracts** — structured JSON that agents consume for deterministic execution

This means `data-model.md` and `contracts/` can be partially or fully generated from `.specos`, reducing manual work and ensuring consistency with the spec.

### 3. During `/speckit.analyze` — Formal Consistency Checking

SpecKit's `/speckit.analyze` command performs cross-artifact consistency analysis using AI. SpecOS can augment this with **deterministic validation**:

- Are all state transitions reachable?
- Do constraints contradict each other?
- Do operations reference entities/fields that exist?
- Are error cases defined for state-mutating operations?
- Do relationships reference valid entities?

These checks don't require AI — they're compiler-level validations that catch issues AI analysis might miss.

### 4. During `/speckit.implement` — Agent Contracts

Instead of agents interpreting `plan.md` and `spec.md` to understand what to build, they can also receive **SpecOS agent contracts** — structured JSON documents with:

- Input/output schemas
- Preconditions and postconditions
- State transition effects
- Error handling rules
- Constraint invariants
- Expected test cases

This gives agents unambiguous implementation instructions alongside the natural language context from SpecKit.

### 5. Across Feature Iterations — Spec Versioning

When a feature spec evolves (e.g., adding a `paused` state to subscriptions), SpecOS provides:

```
specos diff spec-v1.specos spec-v2.specos

BREAKING:
  - Removed state 'trial' from Subscription.state_machine

COMPATIBLE:
  - Added state 'paused' to Subscription.state_machine
  - Added operation 'pause_subscription'
```

This integrates with SpecKit's branch-based workflow — compare `.specos` files across branches to assess the impact of spec changes before merging.

## What SpecOS Does NOT Do

To keep boundaries clear and avoid reinventing SpecKit:

- **SpecOS does not orchestrate workflows.** No `/specos.plan` or `/specos.implement` commands. The workflow is SpecKit's domain.
- **SpecOS does not manage agents.** It produces artifacts agents consume. Agent selection, invocation, and management are SpecKit's responsibility.
- **SpecOS does not capture intent.** It does not have a conversational interface for gathering requirements. That's `/speckit.specify` and `/speckit.clarify`.
- **SpecOS does not define project principles.** No constitution. Architectural discipline comes from SpecKit's constitutional framework.
- **SpecOS does not manage branches or features.** File organization follows SpecKit's conventions.

## Where the Ontology Fits

SpecOS includes a **Spec Ontology Layer** — a knowledge graph of reusable domain concepts (Identity, Resources, State, APIs, Reliability). This is complementary to SpecKit, not overlapping:

- **SpecKit templates** guide what questions to ask and how to structure answers
- **SpecOS ontology** provides domain-specific knowledge about what concepts a system likely needs

Example: when `/speckit.specify` receives "Build a subscription billing system," the SpecOS ontology can suggest that the system likely involves `Subscription`, `Plan`, `Invoice`, `Payment`, `CancellationPolicy`, and `RetryPolicy` — feeding richer questions into SpecKit's clarification process.

The ontology could be exposed as a SpecKit extension or skill, providing domain context during the `/speckit.specify` and `/speckit.clarify` steps.

## Proposed CLI Integration

SpecOS ships as a standalone CLI that SpecKit can invoke:

```bash
# Validate a .specos file
specos validate specs/003-billing/spec.specos

# Compile to artifacts
specos compile specs/003-billing/spec.specos --output specs/003-billing/

# Generate agent contract
specos contract specs/003-billing/spec.specos --operation cancel_subscription

# Diff two spec versions
specos diff specs/003-billing/spec-v1.specos specs/003-billing/spec-v2.specos

# Export to OpenAPI
specos export openapi specs/003-billing/spec.specos
```

These commands can be called from SpecKit's workflow scripts, or directly by developers working with `.specos` files.

## Summary

| Concern | Owner | Notes |
|---|---|---|
| Methodology & workflow | SpecKit | SDD lifecycle, commands, templates |
| Intent capture & clarification | SpecKit | `/speckit.specify`, `/speckit.clarify` |
| Constitutional principles | SpecKit | `memory/constitution.md` |
| Agent orchestration | SpecKit | Multi-agent support, slash commands |
| Template-driven quality | SpecKit | Checklists, uncertainty markers, phase gates |
| **Formal specification language** | **SpecOS** | `.specos` DSL with types, state machines, constraints, relationships, errors |
| **Machine validation** | **SpecOS** | Compiler-level checks for consistency, completeness, reachability |
| **Artifact compilation** | **SpecOS** | OpenAPI, JSON Schema, agent contracts, test suites |
| **Spec versioning & diffing** | **SpecOS** | Breaking vs. compatible change classification |
| **Domain ontology** | **SpecOS** | Reusable concept library for requirement discovery |
| Cross-artifact consistency | Both | SpecKit via AI analysis, SpecOS via deterministic validation |
| Test generation | Both | SpecKit via templates and AI, SpecOS via formal property/constraint compilation |
