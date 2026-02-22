# cogni-claims

A Claude Cowork plugin for claim verification and management.

## Problem

Plugins that rely on web search produce sourced claims that may deviate from what sources actually say. Users need a systematic way to verify, track, and resolve these discrepancies.

## Capabilities

### Claim Ingestion

Accept claims (statement + source URL + source title) individually or in batch, from its own web search or from other plugins. Each claim receives a unique ID and is tracked as unverified.

### Source Verification

For each unverified claim, read the source URL content and determine whether the source supports, contradicts, or is silent on the claim. Detect specific deviation types:

- **Misquotation** — claim misrepresents what the source says
- **Unsupported conclusion** — claim draws a conclusion the source does not support
- **Selective omission** — claim omits context that changes meaning
- **Data staleness** — claim uses outdated data from the source
- **Source contradiction** — source directly contradicts the claim

Each deviation is recorded with its type, severity (low / medium / high / critical), the verbatim source excerpt, and a plain-language explanation. If no deviation is found, the claim is marked as verified with the supporting excerpt.

### Retrieval Failure Handling

If a source is unreachable or in an unsupported format, the failure reason is recorded and the claim is marked as `source_unavailable`. An unverifiable claim is never silently marked as verified.

### Claims Dashboard

Findings are presented grouped by status. For medium+ severity deviations, resolution options are offered:

- Correct the claim
- Flag as disputed
- Search for alternative source
- Discard
- Accept as-is with override

The user is the authority — the system provides evidence, the user decides.

### Source Inspection

Users can navigate to the source page with the relevant passage highlighted to judge deviations in context before deciding.

### Claim Lifecycle Tracking

The full history is persisted: submission, verification, deviations, resolution. Re-verification on demand is supported. State survives across the session.

### Cross-Plugin Contract

A `ClaimEntity` schema (claim record + deviation record + resolution record) is published for consuming plugins. Interfaces are provided for:

- Submitting claims
- Querying verification results by claim ID
- Querying all claims by submitting plugin

Fetched sources are cached to avoid redundant retrieval.

## Quality Constraints

- Prefer cautious "uncertain" findings over false-positive deviations
- Always show the source excerpt as evidence
- Communicate ambiguity explicitly
- Batch verification of N claims sharing K URLs requires at most K source fetches

## Hard Rules

- All resolutions require user confirmation — never auto-correct or auto-retract
- Deviation detection is LLM-based and must not be presented as infallible
