# Claim Verification Plugin

A claim verification and source validation plugin primarily designed for [Cowork](https://claude.com/product/cowork), Anthropic's agentic desktop application — though it also works in Claude Code. Supports claim ingestion, source verification, deviation detection, claims dashboard review, source inspection, and user-guided resolution.

> **Important**: This plugin assists with verifying sourced claims against cited URLs but does not guarantee factual accuracy. Deviation detection is LLM-based and should not be treated as infallible. All verification results should be reviewed by the user before acting on them.

## Installation

```bash
claude plugins add insight-wave-marketplace/cogni-claims
```

## Commands

| Command | Description |
|---------|-------------|
| `/claims submit` | Claim ingestion — submit individual claims or batch-import from context, each with a source URL and title for tracking |
| `/claims verify` | Source verification — fetch cited URLs and detect deviations (misquotation, unsupported conclusion, selective omission, data staleness, source contradiction) |
| `/claims dashboard` | Claims dashboard — review all claims grouped by status with severity indicators and resolution options |
| `/claims inspect` | Source inspection — open a source page with the relevant passage highlighted for in-context review |
| `/claims resolve` | Claim resolution — correct, flag, find alternative sources, discard, or accept claims with user-guided decisions |

## Skills

| Skill | Description |
|-------|-------------|
| `claims` | Main orchestrator for claim lifecycle: ingestion, parallel verification against cited sources, dashboard presentation, and resolution workflows |
| `claim-entity` | Cross-plugin contract defining the ClaimEntity schema (claim records, deviation records, resolution records) and workspace conventions |

## Example Workflows

### Verify Claims from a Research Report

1. Run `/claims submit --batch` with a markdown file containing sourced claims
2. Run `/claims verify` to check all unverified claims against their cited sources
3. Run `/claims dashboard` to review findings grouped by status and severity
4. Run `/claims inspect claim-abc123` to see a flagged source passage in context
5. Run `/claims resolve claim-abc123` to correct or accept the claim

### Single Claim Verification

1. Run `/claims submit "The AI market will reach $1.8T by 2030" --source "https://example.com/report" --title "AI Forecast"`
2. Run `/claims verify` to fetch the source and check for deviations
3. Run `/claims dashboard` to review the result

### Cross-Plugin Claim Handoff

1. Another plugin (e.g., `cogni-research`) submits claims via the `cogni-claims:claims` skill with mode `submit`
2. Run `/claims verify` to verify the submitted claims
3. Run `/claims dashboard` to review results and resolve any deviations

## Agents

| Agent | Description |
|-------|-------------|
| `claim-verifier` | Verification worker — fetches one source URL and verifies all claims referencing it, returning deviation analysis as compact JSON |
| `source-inspector` | Browser inspector — opens a source URL and highlights the relevant passage for user review before resolution decisions |

## Configuration

Claim state is stored within the calling project's workspace under `.claims/`. No additional MCP servers or external configuration are required — the plugin uses web fetching to retrieve cited sources directly.

> **Note:** For JavaScript-rendered or paywalled pages, the plugin falls back to browser automation via the `source-inspector` agent.
