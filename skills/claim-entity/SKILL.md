---
name: claim-entity
description: |
  This skill should be used when any plugin needs to understand the ClaimEntity schema,
  "submit claims", "query claim status", "claim data model", "claim contract",
  "deviation types", "resolution actions", or integrate with the cogni-claims verification system.
  Internal contract definition — provides the cross-plugin data model for claim lifecycle management.
version: 1.0.0
---

# ClaimEntity Contract

## Purpose

Define the cross-plugin data model for claim verification and lifecycle management. Any plugin that submits claims for verification or consumes verification results MUST follow this contract.

## Core Data Model

Three record types compose the claim lifecycle:

### ClaimRecord

Represents a single verifiable claim with its current state.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | `claim-<uuid-v4>` — auto-assigned |
| `statement` | string | The claim text |
| `source_url` | string | URL of cited source |
| `source_title` | string | Human-readable source title |
| `submitted_by` | string | Plugin name (e.g., `cogni-research`) |
| `status` | enum | `unverified` / `verified` / `deviated` / `source_unavailable` / `resolved` |
| `deviations` | array | DeviationRecord[] — empty if clean |
| `resolution` | object\|null | ResolutionRecord when resolved |
| `source_excerpt` | string\|null | Verbatim excerpt supporting or contradicting |

### DeviationRecord

Represents a specific discrepancy between claim and source.

**Deviation types:**
- `misquotation` — claim misrepresents source wording
- `unsupported_conclusion` — claim draws unsupported inference
- `selective_omission` — claim omits meaning-changing context
- `data_staleness` — claim uses outdated source data
- `source_contradiction` — source directly contradicts claim

**Severity levels:** `low`, `medium`, `high`, `critical`

### ResolutionRecord

Records the user's decision on a deviated claim.

**Resolution actions:**
- `corrected` — claim text updated to match source
- `disputed` — user disputes the deviation finding
- `alternative_source` — user provides different source
- `discarded` — claim removed from consideration
- `accepted_override` — user keeps claim despite deviation

## Status Lifecycle

```
unverified ──> verified           (no deviations)
unverified ──> deviated           (deviations detected)
unverified ──> source_unavailable (source unreachable)
deviated   ──> resolved           (user resolves all deviations)
any status ──> re-verify          (returns to verified/deviated/source_unavailable)
```

## Workspace Layout

Claim state persists in the calling project's `.claims/` directory:

```
{working_dir}/.claims/
├── claims.json          # Registry of all ClaimRecords
├── sources/{hash}.json  # Cached source content per URL
└── history/{id}.json    # Audit trail per claim
```

## Cross-Plugin Submission

To submit claims from another plugin, invoke `cogni-claims:claims` skill with mode `submit`:

```
Parameters:
  mode: "submit"
  working_dir: "/path/to/project"
  submitted_by: "plugin-name"
  claims: [{statement, source_url, source_title}, ...]
```

To query results, invoke with mode `query`:

```
Parameters:
  mode: "query"
  working_dir: "/path/to/project"
  query_type: "by_id" | "by_plugin" | "by_status"
  query_value: "<claim-id>" | "<plugin-name>" | "<status>"
```

## Hard Rules

- All resolutions require explicit user confirmation — never auto-resolve
- Deviation detection is LLM-based — communicate findings as assessments, not facts
- Prefer cautious "uncertain" findings over false-positive deviations
- Always include the source excerpt as evidence
- Never mark an unverifiable claim as verified

## Additional Resources

### Reference Files

For complete schema details with all field definitions and examples:
- **`references/schema.md`** — Full JSON schema, field tables, deviation type definitions, severity criteria
- **`references/workspace-conventions.md`** — Directory structure, file formats, initialization, caching rules
