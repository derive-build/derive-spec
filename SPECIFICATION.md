# Derive Spec Format v0.1

> **Status:** Preview — expect changes before v1.0.
> **License:** Apache 2.0 (format specification). CLI tooling licensed separately.

## Overview

The Derive Spec Format defines a two-layer system for capturing software architecture intent:

- **Layer 1 (Human Spec):** Markdown + YAML frontmatter — readable in any editor or GitHub without Derive tooling
- **Layer 2 (Agent IR):** TOON (Token-Oriented Object Notation) — optimized for LLM context windows

Layer 1 is written by extraction and edited by humans. Layer 2 is compiled automatically from Layer 1 and served to agent environments via MCP.

## Layer 1: Human Spec

### File Location

`.derive/spec/{module_id}.md`

### Format

Standard Markdown with YAML frontmatter (parseable by gray-matter, Jekyll, Hugo, GitHub).

### Frontmatter Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | string (semver) | yes | Format version, currently `"0.1.0"` |
| `module_id` | string | yes | Unique module identifier (e.g., `"auth"`) |
| `module_path` | string | yes | Source path relative to repo root (e.g., `"src/auth"`) |
| `language` | enum | yes | `"typescript"` \| `"javascript"` \| `"python"` |
| `extracted_at` | string (ISO 8601) | yes | Extraction timestamp with timezone offset |
| `last_reviewed` | string (ISO 8601) | no | Last human review timestamp |
| `confidence_summary` | object | yes | `{ confirmed, inferred, uncertain }` integer counts |
| `derive_version` | string | yes | CLI version that produced the spec |
| `source_file_count` | integer | no | Source files at extraction time (for drift detection) |

### Markdown Body Sections

```markdown
## Entry Points
### {function/class name}
- **Signature:** `{type signature}`
- **Confidence:** {confirmed|inferred|uncertain}
- **Description:** {optional description}

## Constraints
### {constraint description}
- **Category:** {security|api_contract|data_schema|behavioral|compliance}
- **Confidence:** {confirmed|inferred|uncertain}
- **Rationale:** {why this constraint exists}
- **Rejected Alternatives:**
  - {alternative} — *{reason rejected}*

## Dependencies
### External
- `{module}` ({confidence})
### Internal
- `{module}` ({confidence})
```

### Confidence Model

| Tier | Meaning |
|------|---------|
| `confirmed` | Verifiable from code structure — deterministic, high trust |
| `inferred` | High-confidence inference (score >=70) |
| `uncertain` | Low-confidence inference (score <70) — review recommended |

An element's tier can only be promoted by human review — editing the spec directly.

## Layer 2: Agent IR (TOON)

### File Location

`.derive/ir/{module_id}.ir.toon`

### TOON Format

TOON (Token-Oriented Object Notation) is an indentation-based format designed to be more token-efficient than JSON while remaining readable by both humans and LLMs.

**Design principles:**
- No brackets `{}[]`, no quoted keys, no commas
- Indentation-based nesting (2 spaces)
- Section delimiters (`--- section_name ---`)
- Multiline values via `|` block notation
- Comment lines start with `#`

### Key Abbreviation Mapping

TOON uses abbreviated keys for compactness:

| TOON Key | Full Name | Description |
|----------|-----------|-------------|
| `sig` | `signature` | Function/class type signature |
| `desc` | `description` | Human-readable description |
| `lang` | `language` | Programming language |
| `type` | `dep_type` | Dependency type (internal/external) |
| `alternatives` | `rejected_alternatives` | Rejected design alternatives |
| `tokens` | `token_count` | Estimated token count |
| `extracted` | `extracted_at` | Extraction timestamp |
| `path` | `module_path` | Source path |
| `module` | `module_id` | Module identifier |

### TOON Structure

```
@toon {schema_version}
# Format: https://derive.build/docs/toon-ir
# Keys: sig=signature desc=description lang=language type=dep_type alternatives=rejected_alternatives
module: {module_id}
path: {module_path}
lang: {language}
extracted: {extracted_at}
tokens: {token_count}

--- entry_points ---

entry_point {id}
  name: {name}
  sig: {signature}
  confidence: {confirmed|inferred|uncertain}
  desc: {description}

--- constraints ---

constraint {id}
  desc: {description}
  category: {security|api_contract|data_schema|behavioral|compliance}
  confidence: {confirmed|inferred|uncertain}
  rationale: |
    {multiline markdown rationale}
  alternatives:
    - desc: {alternative description}
      reason: {reason rejected}

--- dependencies ---

dependency {id}
  module: {module specifier}
  type: {external|internal}
  confidence: {confirmed|inferred|uncertain}
```

### Constraint Categories

| Category | Meaning |
|----------|---------|
| `security` | Authentication, encryption, sanitization, access control |
| `api_contract` | API signatures, endpoint contracts, interface guarantees |
| `data_schema` | Database schemas, type constraints, serialization rules |
| `behavioral` | Runtime behavior, idempotency, retry semantics, error handling |
| `compliance` | Regulatory (GDPR, PCI, HIPAA), legal, audit requirements |

## JSON Schemas

Machine-readable schemas for both layers:

- `schemas/spec-v1.json` — validates Layer 1 YAML frontmatter
- `schemas/toon-ir-v1.json` — validates Layer 2 IR structure

Both target JSON Schema Draft 2020-12.

## Versioning

- **Semver:** `MAJOR.MINOR.PATCH`
- **Major bump:** Breaking change (migration guide provided)
- **Minor bump:** Backward-compatible additions (forward-compatible with warning)
- **Patch bump:** Documentation/tooling only
- **Current:** `0.1.0` (preview — changes expected before 1.0)

## Privacy Guarantees

The Derive Spec Format is designed for intent capture, not source code storage:

- Specs contain **structural metadata only** — signatures, names, types, confidence tiers
- **No source code bodies**, comments, or implementation details
- **No secrets**, credentials, or environment variables
- Specs are safe to version-control and share within teams

## Reading Without Derive

Layer 1 (Human Spec) is standard Markdown + YAML frontmatter. Any tool that reads Markdown with frontmatter can parse it: GitHub, VS Code, Obsidian, Jekyll, Hugo, or `gray-matter` in Node.js.

Layer 2 (TOON IR) includes format reference comments. Agents receiving TOON via MCP can parse it using the structure described above. The JSON Schema at `schemas/toon-ir-v1.json` provides a machine-readable validation contract.
