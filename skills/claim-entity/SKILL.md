---
name: claim-entity
version: 1.0.0
description: |
  This skill should be used when any plugin or agent needs to understand the ClaimEntity schema,
  "claim data model", "claim schema", "ClaimRecord fields", "DeviationRecord structure",
  "claim contract", "deviation types", "deviation severity levels", "resolution actions",
  "claim status transitions", "claim lifecycle states", or "workspace .claims directory layout".
  Internal contract definition — provides the cross-plugin data model and record type specifications
  for claim lifecycle management in the cogni-claims verification system.
---

# ClaimEntity Contract

## Purpose

Define the cross-plugin data model for claim verification and lifecycle management. Any plugin that submits claims for verification or consumes verification results MUST follow this contract.

## Core Data Model

Three record types compose the claim lifecycle. For complete field definitions and examples, see `references/schema.md`.

### ClaimRecord

A single verifiable claim with its current state. Key fields: `id`, `statement`, `source_url`, `source_title`, `submitted_by`, `status`, `deviations[]`, `resolution`.

### DeviationRecord

A specific discrepancy between claim and source. Key fields: `type`, `severity`, `source_excerpt`, `explanation`.

**Deviation types:** `misquotation`, `unsupported_conclusion`, `selective_omission`, `data_staleness`, `source_contradiction`

**Severity levels:** `low`, `medium`, `high`, `critical`

### ResolutionRecord

The user's decision on a deviated claim. Key fields: `action`, `corrected_statement`, `rationale`.

**Resolution actions:** `corrected`, `disputed`, `alternative_source`, `discarded`, `accepted_override`

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

## Cross-Plugin Integration

This skill defines the data structures; the `cogni-claims:claims` skill handles submission, verification, and query execution. To submit or query claims from another plugin, invoke `cogni-claims:claims` skill. See `references/schema.md` for batch submission format and query interfaces.

## Hard Rules

- All resolutions require explicit user confirmation — never auto-resolve
- Deviation detection is LLM-based — communicate findings as assessments, not facts
- Prefer cautious "uncertain" findings over false-positive deviations
- Always include the source excerpt as evidence
- Never mark an unverifiable claim as verified

## Additional Resources

- **`references/schema.md`** — Full JSON schema, field tables, deviation type definitions, severity criteria, batch submission format, query interfaces
- **`references/workspace-conventions.md`** — Directory structure, file formats, initialization, caching rules
- **`examples/claim-lifecycle.json`** — End-to-end example showing a claim progressing through unverified, deviated, and resolved states with all three record types populated
