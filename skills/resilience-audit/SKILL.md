---
name: resilience-audit
description: Use when a user wants a resilience or security-resilience audit of their project — generates a cyber-resilience plan (RESILIENCE.md) structured around Sommerville's 4Rs (Recognition, Resistance, Recovery, Reinstatement), grounded in the actual codebase. Triggers on "resilience audit", "am I resilient", "security plan", "what if my app gets hacked/goes down", "backup/recovery plan".
---

# Resilience Audit

Generate a cyber-resilience plan for the current repository, following
Sommerville's cyber-resilience planning process (Software Engineering,
10th ed., Ch. 14). Read `references/limitations.md` first and keep its
framing: this is a judgment aid, not a certification.

## Stage 1 — Asset classification

Scan the repo for assets. Look at: database schemas / ORM models, env var
names (NEVER read or echo secret values — names only), auth code, payment
or billing integrations, user-data stores, external API clients, deploy
config (Vercel/Docker/CI), and backup tooling. Rank each asset:

- **Critical** — loss/leak causes unrecoverable damage (user data, auth secrets)
- **Important** — serious but recoverable (API keys, billing hooks, deploy pipeline)
- **Normal** — rebuildable from source

## Stage 2 — Threat identification

Detect the stack (package.json → Node catalog; pyproject/requirements →
Python catalog; otherwise generic) and read the matching
`references/threat-catalog-node.md`, `references/threat-catalog-python.md`,
or `references/threat-catalog-generic.md`. For each critical/important
asset, list threats by CIA category: Confidentiality (exposed), Integrity
(tampered), Availability (denied). Only include threats that plausibly
apply to THIS repo — tie each one to a real file, dependency, or config
you found.

## Stage 3 — 4R findings

For each asset/threat pair, evaluate all four Rs and emit one **finding
per gap**, in the candid-review format defined in
`references/runbook-templates.md`: severity icon + title, then
**Asset/R**, **File**, **Confidence** (Safe ✓ | Verify ⚡ | Careful ⚠️),
**Problem**, **Impact**, **Fix** with concrete code/config/steps.

- **Recognition** gaps (no alerts, monitors, logging) are usually ⚠️ Major.
- **Resistance** gaps that admit attackers or lose data are 🔥 Critical.
- **Recovery/Reinstatement** gaps: missing/unrehearsed backups are 🔥; missing degraded modes are 💭.
- Controls that already exist go in "Existing resistance" with file:line — credit them, don't re-report them.
- Anything you cannot verify from the repo (backup retention, platform
  rate limits, webhook signatures) becomes a **? Clarification Needed**
  finding with a **Question** instead of a Fix — never assert safety you
  haven't confirmed.

## Stage 4 — Output

1. Write `RESILIENCE.md` at the target repo root using the skeleton in
   `references/runbook-templates.md`: header note, asset table, findings
   ordered by severity with a summary count line, existing-resistance
   credits, runbooks, and the citation footer.
   - If `RESILIENCE.md` already exists: regenerate sections but preserve
     any region wrapped in `<!-- resilience:manual -->` ...
     `<!-- /resilience:manual -->` comments verbatim.
2. If `gh auth status` succeeds AND `git remote get-url origin` is a
   GitHub URL: create one issue per 🔥 Critical and ⚠️ Major finding
   (`gh issue create --title "<finding title>" --label resilience --body "<the full finding block>"`).
   On ANY failure, skip silently and add a footer line noting issues were
   not created.
3. End by telling the user: the finding-count summary, the top 3 findings, and that runbooks must be
   rehearsed by a human before they count as Recovery.

## Failure mode to avoid

A plan that could apply to any repo is a failed run. Every threat,
control, and action must name a real file, model, env var name, or config
from this repository.
