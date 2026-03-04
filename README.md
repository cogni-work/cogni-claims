# cogni-claims

A Claude Code plugin that verifies whether sourced claims actually match what their cited sources say.

## Why this exists

LLMs cite sources confidently — but the citations are often wrong. Numbers get rounded into different claims, conclusions overshoot what the source actually says, and URLs sometimes point to pages that don't exist. The gap between "cited" and "correct" is large enough to cause real harm, and it's well-documented:

| Problem | Finding | Source |
|---------|---------|--------|
| Fabricated citations | 14–95% of LLM citations are hallucinated depending on domain (2.2M citations analyzed) | [GhostCite, 2025](https://arxiv.org/html/2602.06718) |
| Inaccurate citations | AI search engines fail to produce accurate citations in >60% of tests (8 engines tested) | [CJR Tow Center, 2025](https://www.cjr.org/tow_center/we-compared-eight-ai-search-engines-theyre-all-bad-at-citing-news.php) |
| Bibliographic errors | 45.4% of GPT-4o citations contain bibliographic errors (most commonly invalid DOIs); 19.9% entirely fabricated | [JMIR Mental Health, 2025](https://mental.jmir.org/2025/1/e80371) |
| Real-world harm | Lawyers sanctioned after submitting AI-fabricated case citations to court | [Mata v. Avianca, 2023](https://law.justia.com/cases/federal/district-courts/new-york/nysdce/1:2022cv01461/575354/54/) |

Every claim above has been verified against its source using this plugin. This plugin exists because "cited" doesn't mean "correct."

## What it is

A systematic claim-verification workflow for Claude Code. Other plugins generate sourced content — this one checks whether the sources actually say what's claimed. It's designed for cross-plugin use: submit claims from anywhere, verify and resolve them here.

## What it does

1. **Submit** claims with their source URLs — individually or batch-imported from markdown
2. **Verify** them by fetching each source and detecting deviations (misquotation, unsupported conclusions, selective omission, data staleness, source contradiction)
3. **Review** a dashboard showing all claims grouped by status, with inline deviation summaries and severity indicators
4. **Inspect** flagged claims by opening the source in your browser with the relevant passage highlighted for side-by-side comparison
5. **Resolve** each deviation — correct the claim, dispute the finding, find an alternative source, discard, or accept as-is

## What it means for you

If you ship research, reports, or any content that leans on sourced claims, this is your safety net before publish.

- **Catch errors before they reach your audience.** Sourced claims get checked against the actual source — not taken on faith.
- **Stay in control.** Deviation detection is LLM-based. Findings are assessments for you to review, not verdicts.
- **Keep a paper trail.** Every claim, verification result, and resolution decision is stored as structured JSON you can reference later.

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

## Example

You're writing a report and cite a stat: *"The global AI market will reach $1.8 trillion by 2030."*

```
> /claims submit "The global AI market will reach $1.8T by 2030"
  --source "https://example.com/ai-forecast" --title "AI Market Report"

Submitted claim-a1b2c3d4.

> /claims verify

Verifying 1 claim against 1 source...

  claim-a1b2c3d4  DEVIATED (high)
  Claim says $1.8T, but source projects $800B–$1.5T.
  The claim exceeds the upper bound of the source's projection.

> /claims resolve claim-a1b2c3d4

  Suggested correction: "The global AI market is projected
  between $800B and $1.5T by 2030."

  [Correct] [Dispute] [Alternative source] [Discard] [Accept as-is]
```

The claim was wrong. Now you know — before your reader does.

## How it works

Claims are stored in your project's `.claims/` directory as JSON. When you verify, the plugin dispatches a **claim-verifier** agent per unique source URL — each agent fetches the page once and checks all claims referencing it. For deviated claims, the **source-inspector** agent can open the source in Chrome and highlight the relevant passage so you can see the discrepancy in context.

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
