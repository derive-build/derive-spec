# Derive Spec Format

An open format for capturing software architecture intent — constraints, rationale, rejected alternatives, and confidence-scored structure — so that codebases can be rebuilt, migrated, or handed off without losing the decisions that shaped them.

## The Problem

Code tells you **what exists**. It doesn't tell you **why** it exists, what was considered and rejected, or which constraints are load-bearing vs incidental. When teams migrate, rewrite, or hand off systems, the code transfers but the intent is lost.

For brownfield systems especially, code is a liability — it's the thing you're trying to replace. The specifications, constraints, and architectural decisions are the asset. But they live in people's heads, scattered Confluence pages, and Slack threads that nobody can find.

## What This Format Captures

The Derive Spec Format defines what code analysis tools miss:

- **Entry points** — the public API surface, with type signatures and confidence levels
- **Constraints** — security rules, API contracts, data schema rules, behavioral guarantees, compliance requirements
- **Rationale** — *why* each constraint exists, not just that it does
- **Rejected alternatives** — what was considered and discarded, and the reason
- **Confidence tiers** — how much to trust each element (`confirmed` / `inferred` / `uncertain`)

## Two-Layer Architecture

**Layer 1 (Human Spec)** — Markdown with YAML frontmatter. Readable in GitHub, VS Code, Obsidian, or any text editor. Designed for humans to read, review, and edit.

```markdown
---
schema_version: "0.1.0"
module_id: auth
module_path: src/auth
language: typescript
extracted_at: "2026-04-11T00:00:00Z"
confidence_summary:
  confirmed: 3
  inferred: 1
  uncertain: 0
---

## Constraints

### Passwords must be hashed before storage
- **Category:** security
- **Confidence:** inferred
- **Rationale:** PCI compliance requires bcrypt or equivalent
- **Rejected Alternatives:**
  - Store plaintext passwords — *Violates PCI DSS requirement 8.2.1*
  - Use MD5 hashing — *Cryptographically broken since 2004*
```

**Layer 2 (Agent IR)** — TOON (Token-Oriented Object Notation). Compiled automatically from Layer 1. Optimized for LLM context windows — 30-60% fewer tokens than JSON. Served to AI agents via MCP.

```
@toon 0.1.0
module: auth
path: src/auth
lang: typescript

--- constraints ---

constraint passwords-must-be-hashed-befor
  desc: Passwords must be hashed before storage
  category: security
  confidence: inferred
  rationale: |
    PCI compliance requires bcrypt or equivalent
  alternatives:
    - desc: Store plaintext passwords
      reason: Violates PCI DSS requirement 8.2.1
```

## Quick Start

- **[Format Specification](SPECIFICATION.md)** — complete field definitions, structure, and rules
- **[Layer 1 Example](examples/auth.md)** — a Human Spec for an auth module
- **[Layer 2 Example](examples/auth.ir.toon)** — the same module compiled to TOON IR
- **[Layer 1 JSON Schema](schemas/spec-v1.json)** — validate frontmatter programmatically
- **[Layer 2 JSON Schema](schemas/toon-ir-v1.json)** — validate IR structure programmatically

## Design Principles

**Specs are the asset, code is replaceable.** A complete spec with rationale and rejected alternatives contains enough information to rebuild the system on any stack. The code is an implementation detail.

**Human-editable, agent-optimized.** Layer 1 is plain Markdown that anyone can read and edit. Layer 2 is compiled automatically for efficient agent consumption. Humans never need to touch TOON.

**Confidence is graded, not binary.** Every element carries a confidence tier. `confirmed` means verifiable from code structure. `inferred` means high-confidence analysis. `uncertain` means review recommended. Tiers can only be promoted by humans.

**No source code in specs.** Specs contain structural metadata — signatures, names, constraints, rationale. No function bodies, secrets, or implementation details. A spec is safe to share even if the code isn't.

**Open format, no lock-in.** Layer 1 is standard Markdown + YAML frontmatter. You can read it without any tooling. The JSON Schemas let you validate specs with any validator. No vendor dependency.

## Constraint Categories

| Category | What It Captures |
|----------|-----------------|
| `security` | Authentication, encryption, sanitization, access control |
| `api_contract` | API signatures, endpoint contracts, interface guarantees |
| `data_schema` | Database schemas, type constraints, serialization rules |
| `behavioral` | Runtime behavior, idempotency, retry semantics, error handling |
| `compliance` | Regulatory (GDPR, PCI, HIPAA), legal, audit requirements |

## Status

**v0.1.0 — Preview.** The format is functional and tested but expect changes before v1.0. We're actively dogfooding on real brownfield projects and iterating based on what we learn.

Changes will follow semver. Breaking changes (if any) will include migration guides.

## License

Apache 2.0 — use it, extend it, build tools around it.
