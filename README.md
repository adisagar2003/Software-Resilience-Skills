# Software Resilience Skills

A Claude Code skill — `/resilience-audit` — that audits your project and generates a cyber-resilience plan structured around the **4Rs**: Recognition, Resistance, Recovery, Reinstatement.

## Why

Vulnerability scanners (npm audit, Dependabot, Semgrep) cover one thing well: stopping attacks (Resistance). Almost nothing built for indie developers covers the other three — noticing an incident early, restoring service after one, and getting back to fully normal operation. Those are exactly the things that kill small products, because a solo dev has no SRE team (site reliability engineers — the people who own uptime and recovery at big companies).

This skill audits your actual codebase and produces a `RESILIENCE.md`: an asset/threat matrix, a 4R plan per critical asset, prioritized actions, and incident runbooks (step-by-step procedures for handling a failure) — all grounded in your real files, never generic advice.

## The 4Rs in 30 seconds

- **Recognition** — detecting early signs of failure or attack (logging, alerts, uptime monitors).
- **Resistance** — reducing the chance a problem becomes a failure (auth, rate limiting, isolating critical parts).
- **Recovery** — restoring critical service quickly after a failure (backups, failover, degraded mode).
- **Reinstatement** — returning to full normal operation (restore procedures, data validation, post-incident review).

## What our dogfood run found

We ran the audit flow against a real, soon-to-ship indie SaaS (details redacted). The pattern is likely yours too:

| R | Finding |
|---|---|
| Resistance | **Solid** — password hashing, rate limiting, secrets guarded at boot. Scanners had this half covered. |
| Recognition | **Nearly absent** — no uptime monitor, no spend alerts, no failed-login alerting despite the data existing. |
| Recovery | **Unverified** — database restore relied on a managed provider's default retention: never confirmed, never rehearsed. |
| Reinstatement | **Partial** — architecture decision records existed, so post-incident review had a home. |

Top prioritized action from that run: *"Rehearse a restore once. Until then you have hope, not Recovery."*

## Install & use

1. Clone this repo:
   ```bash
   git clone git@github.com:adisagar2003/Software-Resilience-Skills.git
   ```
2. Copy the skill into your Claude Code skills directory:
   ```bash
   cp -r Software-Resilience-Skills/skills/resilience-audit ~/.claude/skills/
   ```
3. Open Claude Code inside any project and run:
   ```
   /resilience-audit
   ```

You get a `RESILIENCE.md` at your repo root. If the `gh` CLI is authenticated and your repo is on GitHub, each prioritized action also becomes a GitHub issue; otherwise the file alone.

See [a sample RESILIENCE.md](examples-public/RESILIENCE.example.md) generated from a small Express + Prisma app.

Re-running the skill regenerates the plan but preserves anything you wrote between `<!-- resilience:manual -->` and `<!-- /resilience:manual -->` markers.

## What this is NOT

- **Not a certification.** Resilience is a judgment, not a metric (Sommerville). The output is a plan plus open questions — never a score or pass/fail.
- **Not a vulnerability scanner.** Keep using npm audit / Dependabot / Semgrep; this covers what they don't.
- **Not done when generated.** A runbook you've never rehearsed is not Recovery capability. The plan will tell you this too.

## References

Sommerville, I. (2016). *Software Engineering* (10th ed.). Pearson. Chapter 14, "Resilience Engineering" — source of the 4R model and the cyber-resilience planning process this skill follows.
