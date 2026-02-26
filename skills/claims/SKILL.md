---
name: claims
description: |
  This skill should be used when the user asks to "verify claims", "check claims against sources",
  "claims dashboard", "show claims", "resolve claim", "submit claim", "claim verification",
  "fact-check claims", "re-verify claims", "review deviations", "inspect claim",
  "what claims are pending", or when any plugin needs to submit claims for source verification.
  Orchestrates the full claim verification lifecycle: ingestion, source-based verification,
  deviation detection, dashboard presentation, and user-guided resolution.
---

# Claims Verification Orchestrator

## Purpose

Manage the full lifecycle of sourced claims: accept claims (individually or in batch), verify them against their cited source URLs, detect deviations, present findings to the user, and guide resolution. This skill orchestrates the pipeline; verification work is delegated to the `claim-verifier` agent.

**Not for:**
- Generating claims from scratch (that is the submitting plugin's responsibility)
- Making editorial decisions (the user is the authority)
- Presenting deviation findings as definitive facts (they are LLM-based assessments)

## Operating Modes

Support five operating modes, determined by the `mode` parameter or inferred from user intent:

| Mode | Trigger | Description |
|------|---------|-------------|
| `submit` | "submit claim", batch from plugin | Ingest new claims into the registry |
| `verify` | "verify claims", "check claims" | Run verification pipeline on unverified claims |
| `dashboard` | "claims dashboard", "show claims" | Display status overview |
| `inspect` | "inspect claim <id>" | Show detailed deviation evidence for one claim |
| `resolve` | "resolve claim <id>" | Present resolution options for a deviated claim |

## Workspace Initialization

Before any operation, ensure the workspace exists:

1. Determine `working_dir` — passed as parameter, or use the current working directory
2. Check for `{working_dir}/.claims/claims.json`
3. If missing, initialize:
   - Create `.claims/`, `.claims/sources/`, `.claims/history/`
   - Write initial `claims.json` with empty claims array
4. Read `claims.json` into working context

## Mode: Submit

Accept one or more claims and add to the registry.

**Required input:**
- `statement` — the claim text
- `source_url` — URL of cited source
- `source_title` — human-readable source title
- `submitted_by` — plugin name or `"user"` for direct submission

**Process:**
1. For each claim, generate a unique ID: `claim-<uuid>`
2. Create a ClaimRecord with status `unverified`
3. Append to `claims.json`
4. Write submission event to `history/{claim-id}.json`
5. Report count of submitted claims to user

**Batch submission:** Accept an array of `{statement, source_url, source_title}` objects. Process all in a single registry update.

## Mode: Verify

Run the verification pipeline on unverified claims (or a specific claim by ID).

**Process:**

### Step 1: Select Claims

- If `--id <id>` specified, verify that single claim (even if previously verified — this is re-verification)
- Otherwise, select all claims with status `unverified`
- If no claims to verify, inform user and exit

### Step 2: Group by Source URL

Group selected claims by `source_url`. Count unique URLs (K) and total claims (N).
Report: "Verifying {N} claims against {K} unique sources."

### Step 3: Dispatch Verification

For each unique URL group, launch a `cogni-claims:claim-verifier` agent via the Task tool:

```
Task parameters:
  subagent_type: "cogni-claims:claim-verifier"
  prompt: Include working_dir, source URL, claim IDs, claim statements
```

**Launch agents in parallel** when multiple URLs exist. Each agent:
1. Fetches the source URL (WebFetch, browser fallback)
2. Caches source content in `.claims/sources/{hash}.json`
3. Compares each claim against source content
4. Returns verification results as JSON

### Step 4: Collect Results

Read verification results from each agent. For each claim:
- Update status in `claims.json` (`verified`, `deviated`, or `source_unavailable`)
- Attach DeviationRecords if deviations detected
- Record source excerpt
- Write verification event to `history/{claim-id}.json`

### Step 5: Present Summary

Show a brief verification summary:
```
Verification complete:
- {n} verified (no deviations)
- {n} deviations detected ({n} critical, {n} high, {n} medium, {n} low)
- {n} sources unavailable
```

If deviations with severity `medium` or higher exist, suggest: "Run `/claims dashboard` to review findings."

## Mode: Dashboard

Display the claims dashboard grouped by status. Consult `references/dashboard-format.md` for the complete layout specification.

**Process:**
1. Read `claims.json`
2. Group claims by status
3. Sort within groups (see dashboard-format.md for sort rules)
4. Render markdown tables per section
5. Include action hints for deviated claims

## Mode: Inspect

Show detailed evidence for a specific claim.

**Process:**
1. Look up claim by ID in `claims.json`
2. If claim has deviations, display each with:
   - Deviation type and severity
   - Source excerpt (verbatim)
   - Explanation
3. If claim is verified, display supporting excerpt
4. Offer to open source in browser for in-context inspection:
   - Launch `cogni-claims:source-inspector` agent with the source URL and relevant excerpt
   - The agent navigates to the page and highlights the passage

## Mode: Resolve

Guide the user through resolving a deviated claim.

**Process:**
1. Look up claim by ID — must have status `deviated`
2. Display the claim, deviation details, and source excerpt
3. Present resolution options using AskUserQuestion:
   - **Correct** — prompt for corrected statement text
   - **Dispute** — prompt for rationale
   - **Alternative source** — prompt for new URL and title
   - **Discard** — prompt for rationale
   - **Accept as-is** — prompt for rationale
4. Record ResolutionRecord with user's choice
5. Update claim status to `resolved`
6. Write resolution event to `history/{claim-id}.json`
7. If action is `alternative_source`, optionally trigger re-verification against new source

## Quality Constraints

- **Conservative detection**: Prefer "uncertain" findings over false positives
- **Evidence-first**: Always show the source excerpt — never present a deviation without evidence
- **Explicit ambiguity**: When the comparison is genuinely unclear, say so
- **User authority**: The system provides assessments; the user makes decisions
- **No silent failures**: Every unverifiable source is explicitly reported

## Additional Resources

### Reference Files

- **`references/verification-protocol.md`** — Detailed verification methodology, deviation type definitions, severity criteria, epistemic humility guidelines
- **`references/dashboard-format.md`** — Complete dashboard layout, resolution prompt format, truncation and sorting rules

### Scripts

- **`scripts/claims-store.sh`** — JSON state management utility for reading/writing claims registry, generating IDs and URL hashes. Invoke via: `bash "${CLAUDE_PLUGIN_ROOT}/skills/claims/scripts/claims-store.sh" <command> [args...]`

### Examples

- **`examples/claims-sample.json`** — Sample `claims.json` with claims in all statuses (unverified, verified, deviated, resolved) showing complete field structures

### Cross-Plugin Contract

For the ClaimEntity data model, see the `cogni-claims:claim-entity` skill and its reference files.

### Agents (via Task tool)

- **`cogni-claims:claim-verifier`** — Worker agent: fetches one source URL, verifies all claims against it, returns results as JSON. Launch in parallel for multiple URLs.
- **`cogni-claims:source-inspector`** — Browser agent: navigates to source URL, highlights relevant passage for user inspection.
