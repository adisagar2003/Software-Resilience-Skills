# Design: SoftwareResilienceSkills — /resilience-audit

Date: 2026-07-05
Status: Approved (pending spec review)

## Purpose

A public repository containing a Claude Code skill, `/resilience-audit`, that helps indie developers apply resilience engineering (Sommerville, Software Engineering 10th ed., Ch. 14) to their own projects. The skill audits a target repo and produces a concrete cyber-resilience plan structured around the 4Rs: Recognition, Resistance, Recovery, Reinstatement.

Differentiator: existing tools (npm audit, Dependabot, Semgrep, /security-review) cover vulnerability scanning (the Resistance half). Almost nothing for indie devs covers Recovery and Reinstatement — backups, restore drills, incident runbooks. This skill covers all four, grounded in the actual codebase rather than generic advice.

## Non-goals

- Not a vulnerability scanner; it does not replace SAST/dependency tooling.
- Not a certification. Resilience is a judgment (per Sommerville); output is a plan plus open questions, never a pass/fail score.
- No scaffolding code generation in v1 (no health-check endpoints, backup scripts, etc.). Plan-only output.

## Skill flow

Single entry point `/resilience-audit`, staged workflow mirroring Sommerville's cyber-resilience planning process:

1. **Asset classification** — scan the target repo (DB schemas, env vars, auth code, payment integrations, user data stores, external APIs) and rank assets: critical / important / normal.
2. **Threat identification** — for each critical or important asset, list threats grouped by CIA category:
   - Confidentiality: data exposed to people who shouldn't have it
   - Integrity: systems or data damaged/tampered
   - Availability: authorized users denied service
3. **4R plan** — for each asset/threat pair:
   - Recognition: how the developer would detect it early (logging, alerting, anomaly signals)
   - Resistance: controls that reduce failure probability (auth, rate limiting, isolation of critical parts)
   - Recovery: how critical service is restored quickly (backups, failover, degraded mode)
   - Reinstatement: how full normal operation resumes (restore procedure, data validation, post-incident review)
4. **Output**:
   - Write a single `RESILIENCE.md` at the target repo root: asset/threat matrix, prioritized action plan, inline runbooks (backup-restore, incident response).
   - If `gh` CLI is available and authenticated and the repo is on GitHub: create one GitHub issue per prioritized action. Any `gh` failure degrades silently to file-only.

## Repository layout

```
SoftwareResilienceSkills/
├── README.md                         # what/why, 4R primer, install instructions
├── skills/
│   └── resilience-audit/
│       ├── SKILL.md                  # entry point, staged workflow
│       └── references/               # loaded on demand, keeps SKILL.md small
│           ├── threat-catalog-node.md      # Node/Next.js-specific threats & signals
│           ├── threat-catalog-python.md    # Python/Django/FastAPI
│           ├── threat-catalog-generic.md   # fallback for any web project
│           ├── runbook-templates.md        # backup-restore, incident-response templates
│           └── limitations.md              # judgment-not-certification framing; runbooks must be rehearsed
└── examples/
    └── RESILIENCE.example.md         # sample output for a small SaaS app
```

## Error handling

- Empty or unrecognized stack → use `threat-catalog-generic.md`; still produce a matrix from whatever is detectable (env vars, configs).
- No network access required for core flow.
- `gh` missing/unauthenticated/non-GitHub remote → skip issues, note it in RESILIENCE.md footer.
- Existing RESILIENCE.md in target repo → skill updates it rather than overwriting blindly (diff-aware regeneration; preserve manually-edited sections marked with an HTML comment sentinel).

## Testing

- Run the skill against 2–3 sample repos (a Node/Next.js app, a Python API, an empty repo) and check: assets correctly classified, no generic boilerplate untied to real files, all four Rs populated per critical asset, graceful `gh` fallback.
- The `examples/RESILIENCE.example.md` is produced by an actual run, not hand-written.

## References

- Sommerville, I. (2016). *Software Engineering* (10th ed.). Pearson. Chapter 14, "Resilience Engineering" — source of the 4R model (Recognition, Resistance, Recovery, Reinstatement) and the cyber-resilience planning process this skill follows.

This citation must also appear in the published repo's README.md and in the footer of every generated RESILIENCE.md.

## Open design constraints (from critique, kept deliberate)

- Output must reference actual files/paths in the target repo; a plan that could apply to any repo is a failed run.
- Sociotechnical honesty: RESILIENCE.md must state that runbooks are only real once rehearsed by a human.
