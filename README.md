# cogni-claims

A Claude Code plugin that verifies whether sourced claims actually match what their cited sources say.

LLM-generated content often cites sources but subtly misrepresents them — a number rounded too aggressively, a conclusion that goes beyond what the source says, context that changes the meaning. This plugin catches those gaps systematically by fetching cited URLs, comparing the claims against them, and guiding you through resolving any discrepancies.

> **Note**: Deviation detection is LLM-based. Findings are assessments for you to review, not verdicts. You always have final say.

## What it does

1. **Submit** claims with their source URLs — individually or batch-imported from markdown
2. **Verify** them by fetching each source and detecting deviations (misquotation, unsupported conclusions, selective omission, data staleness, source contradiction)
3. **Review** a dashboard showing all claims grouped by status, with inline deviation summaries and severity indicators
4. **Inspect** flagged claims by opening the source in your browser with the relevant passage highlighted for side-by-side comparison
5. **Resolve** each deviation — correct the claim, dispute the finding, find an alternative source, discard, or accept as-is

## Installation

```bash
claude plugins add cogni-work/cogni-claims
```

## Quick start

```
/claims submit --batch        # submit claims from a markdown file with citations
/claims verify                # verify all unverified claims against their sources
/claims dashboard             # see what needs attention
/claims inspect <claim-id>    # open the source in your browser to compare
/claims resolve <claim-id>    # decide what to do about a deviation
```

Or just describe what you want in natural language — the plugin figures out the right mode:

- "verify the claims in my research report"
- "what's the status of my claims?"
- "show me what the source actually says for that quantum computing claim"
- "let's fix the deviated claims one by one"

## How it works

Claims are stored in your project's `.claims/` directory as JSON. When you verify, the plugin dispatches a **claim-verifier** agent per unique source URL — each agent fetches the page once and checks all claims referencing it. For deviated claims, the **source-inspector** agent can open the source in Chrome and highlight the relevant passage so you can see the discrepancy in context.

The plugin is designed for cross-plugin use. Other plugins (like `cogni-research` or `cogni-portfolio`) can submit claims programmatically, and you verify and resolve them here.

## Components

| Component | Type | What it does |
|-----------|------|--------------|
| `claims` | skill | Main orchestrator — handles all five modes (submit, verify, dashboard, inspect, resolve) |
| `claim-entity` | skill | Cross-plugin data contract — defines ClaimRecord, DeviationRecord, and ResolutionRecord schemas |
| `claim-verifier` | agent | Fetches a source URL and verifies all claims referencing it |
| `source-inspector` | agent | Opens a source in the browser and highlights the relevant passage |
| `/claims` | command | Slash command entry point for all modes |

## License

[MIT](LICENSE)
