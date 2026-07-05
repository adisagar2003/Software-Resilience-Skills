# /resilience-audit Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the SoftwareResilienceSkills repo: one Claude Code skill (`/resilience-audit`) plus reference library that audits a target repo and generates a 4R cyber-resilience plan (`RESILIENCE.md`, optionally GitHub issues).

**Architecture:** Pure-markdown deliverable. `skills/resilience-audit/SKILL.md` is the staged workflow prompt; `references/` holds stack-specific threat catalogs and runbook templates loaded on demand; `examples/` (local-only, gitignored) shows real output. No executable code.

**Tech Stack:** Claude Code skill format (SKILL.md with YAML frontmatter), GitHub-flavored markdown, `gh` CLI (optional path only).

## Global Constraints

- Spec: `docs/superpowers/specs/2026-07-05-resilience-audit-skill-design.md` — read it before any task.
- Output is a plan + open questions, NEVER a pass/fail score or certification (spec Non-goals).
- Every generated plan must cite actual file paths in the target repo; generic output = failed run.
- Sommerville citation — `Sommerville, I. (2016). Software Engineering (10th ed.). Pearson. Ch. 14` — must appear in README.md and in the footer template of generated RESILIENCE.md.
- `examples/` is gitignored (contains dogfood output for a private product); the committed example must come from a run against a public/dummy repo.
- No network access required for core flow; all `gh` failures degrade silently to file-only.
- "Verification" for markdown deliverables = structural grep checks + a real dogfood run (Task 5).

---

### Task 1: SKILL.md — the staged workflow

**Files:**
- Create: `skills/resilience-audit/SKILL.md`

**Interfaces:**
- Produces: references to five files under `skills/resilience-audit/references/` with exact names: `threat-catalog-node.md`, `threat-catalog-python.md`, `threat-catalog-generic.md`, `runbook-templates.md`, `limitations.md`. Tasks 2–3 must create files with exactly these names.

- [ ] **Step 1: Write SKILL.md with this exact content**

````markdown
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
`references/threat-catalog-*.md`. For each critical/important asset, list
threats by CIA category: Confidentiality (exposed), Integrity (tampered),
Availability (denied). Only include threats that plausibly apply to THIS
repo — tie each one to a real file, dependency, or config you found.

## Stage 3 — 4R plan

For each asset/threat pair fill all four Rs:

- **Recognition** — how the developer would detect it early (logs, alerts, monitors). Note existing recognition measures you found in the code.
- **Resistance** — controls reducing failure probability (auth, rate limits, isolation). Credit what already exists, with file:line.
- **Recovery** — how critical service is restored quickly (backups, failover, degraded mode).
- **Reinstatement** — how full normal operation resumes (restore procedure, data validation, post-incident review).

Where you cannot verify something (e.g., whether a webhook checks
signatures), record it as an **open question** in the plan — do not assert
safety you haven't confirmed.

## Stage 4 — Output

1. Write `RESILIENCE.md` at the target repo root using the structure in
   `references/runbook-templates.md`: header note, asset table, 4R
   matrices, prioritized actions (most critical Recovery/Recognition gaps
   first), runbooks, and the citation footer.
   - If `RESILIENCE.md` already exists: regenerate sections but preserve
     any region wrapped in `<!-- resilience:manual -->` ...
     `<!-- /resilience:manual -->` comments verbatim.
2. If `gh auth status` succeeds AND `git remote get-url origin` is a
   GitHub URL: create one issue per prioritized action
   (`gh issue create --title "<action>" --label resilience --body "<4R context>"`).
   On ANY failure, skip silently and add a footer line noting issues were
   not created.
3. End by telling the user: the top 3 actions, and that runbooks must be
   rehearsed by a human before they count as Recovery.

## Failure mode to avoid

A plan that could apply to any repo is a failed run. Every threat,
control, and action must name a real file, model, env var name, or config
from this repository.
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "references/threat-catalog\|references/runbook-templates.md\|references/limitations.md" skills/resilience-audit/SKILL.md`
Expected: ≥ 4 (all reference files mentioned); also `head -5 skills/resilience-audit/SKILL.md` shows YAML frontmatter with `name:` and `description:`.

- [ ] **Step 3: Commit**

```bash
git add skills/resilience-audit/SKILL.md
git commit -m "feat: add resilience-audit SKILL.md staged workflow"
```

---

### Task 2: Threat catalogs (node, python, generic)

**Files:**
- Create: `skills/resilience-audit/references/threat-catalog-node.md`
- Create: `skills/resilience-audit/references/threat-catalog-python.md`
- Create: `skills/resilience-audit/references/threat-catalog-generic.md`

**Interfaces:**
- Consumes: file names as referenced by Task 1's SKILL.md (exact names above).
- Produces: each catalog is a table the auditing agent maps assets against.

- [ ] **Step 1: Write the generic catalog** — `threat-catalog-generic.md`. Required structure (use exactly these column headers so all catalogs are uniform):

```markdown
# Threat catalog — generic web project

Applies when no specific stack is detected. Each row: only include it in
the plan if the trigger is present in the repo.

| Threat | CIA | Trigger (what to look for) | Typical Recognition | Typical Resistance |
|---|---|---|---|---|
| Secrets committed to git | C | .env files tracked, keys in source | secret-scanning alerts, pre-commit hooks | gitignore env files, rotate on leak |
| Credential stuffing / brute force | C | any login form or auth endpoint | failed-login spike alerts | rate limiting, lockout, strong hashing |
| Data loss (no backups) | I/A | any database or user-data store, no backup config found | restore-drill calendar | automated backups + verified retention |
| Injection (SQL/command) | I | raw query strings, shell-outs with user input | WAF/error-rate alerts | parameterized queries, input validation |
| Dependency compromise | I | lockfile present | audit tooling in CI | pinned versions, review major bumps |
| DoS / resource exhaustion | A | public endpoints, no rate limiting found | latency/error monitors, uptime checks | rate limits, timeouts, quotas |
| Deploy-time breakage | A | CI/CD or migrations run on deploy | post-deploy health check | staged rollout, reviewed migrations, rollback path |
| Single-provider outage | A | one cloud/db provider | uptime monitor | documented degraded mode; accept at indie scale |
```

- [ ] **Step 2: Write the Node catalog** — `threat-catalog-node.md`: same table format, Node/Next.js-specific rows. Must include at least: npm lockfile/postinstall supply-chain risk (trigger: `package.json` scripts), `JWT_SECRET`/session secret handling (trigger: jwt/express-session imports), Next.js API routes exposed without middleware auth (trigger: `app/api/` or `pages/api/`), serverless function timeout/cold-start availability (trigger: `vercel.json`/netlify config), Prisma/ORM migration-on-deploy risk (trigger: `prisma migrate deploy` in build), env leakage via `NEXT_PUBLIC_`/`VITE_` prefixes (trigger: those prefixes on sensitive-looking names), webhook signature verification (trigger: stripe/polar/webhook routes).

- [ ] **Step 3: Write the Python catalog** — `threat-catalog-python.md`: same format. Must include at least: Django `DEBUG=True`/`SECRET_KEY` in settings (trigger: settings.py), FastAPI docs endpoints exposed in prod (trigger: FastAPI app), pickle/yaml unsafe deserialization (trigger: `pickle.load`/`yaml.load`), SQL via f-strings (trigger: f-string containing SELECT/INSERT), requirements not pinned (trigger: requirements.txt without `==`), Celery/queue poisoning (trigger: celery imports), migration-on-deploy risk (trigger: `manage.py migrate`/alembic in deploy config).

- [ ] **Step 4: Verify uniform structure**

Run: `grep -l "| Threat | CIA | Trigger" skills/resilience-audit/references/threat-catalog-*.md | wc -l`
Expected: 3

- [ ] **Step 5: Commit**

```bash
git add skills/resilience-audit/references/threat-catalog-*.md
git commit -m "feat: add stack threat catalogs (node, python, generic)"
```

---

### Task 3: Runbook templates and limitations

**Files:**
- Create: `skills/resilience-audit/references/runbook-templates.md`
- Create: `skills/resilience-audit/references/limitations.md`

**Interfaces:**
- Consumes: names referenced by Task 1.
- Produces: the canonical RESILIENCE.md output skeleton (used by Stage 4) including the manual-section sentinel and citation footer that the Global Constraints require.

- [ ] **Step 1: Write `runbook-templates.md`** containing the full RESILIENCE.md skeleton the skill must instantiate:

```markdown
# RESILIENCE.md output skeleton

Instantiate every {placeholder}. Keep section order. Wrap nothing else in
sentinels — only user-added regions use them.

    # RESILIENCE.md — {project name}

    > Generated by /resilience-audit on {date}. This is a resilience plan,
    > not a certification. Resilience is a judgment (Sommerville, Ch. 14) —
    > runbooks below are only real once a human has rehearsed them.

    ## 1. Asset classification
    {table: Asset | Where (file/path) | Rank | Why}

    ## 2. Threats and the 4R plan
    {per critical asset: table Threat (CIA) | Recognition | Resistance | Recovery | Reinstatement}

    ## 3. Open questions
    {bullets: things the audit could not verify — never assert unverified safety}

    ## 4. Prioritized actions
    {numbered list, most critical Recovery/Recognition gaps first}

    ## 5. Runbooks

    ### Backup & restore
    {numbered steps specific to the detected database/storage; must end with
    a verification step comparing restored data to expectations}

    ### Incident response
    {numbered steps: rotation order for this repo's actual secrets → snapshot
    logs → assess blast radius → user communication → post-incident review}

    <!-- resilience:manual -->
    (User notes below this line are preserved across regenerations.)
    <!-- /resilience:manual -->

    ---
    *{gh issues note}. Reference: Sommerville, I. (2016). Software
    Engineering (10th ed.). Pearson. Ch. 14, Resilience Engineering.*
```

- [ ] **Step 2: Write `limitations.md`**:

```markdown
# Limitations — read before auditing

- Resilience cannot be measured or certified; it is an expert judgment
  (Sommerville, Ch. 14). This skill produces a plan and open questions,
  never a score, grade, or pass/fail.
- Resilience is sociotechnical: runbooks, procedures, and people matter as
  much as code. A generated runbook that has never been rehearsed is not
  Recovery capability — say so explicitly in the output.
- The audit sees the repo, not production: it cannot verify monitoring,
  backup retention, or infra config that lives outside the codebase. Record
  such items as open questions or actions, not findings.
- This skill does not replace vulnerability scanners (npm audit,
  Dependabot, Semgrep, /security-review). Its differentiated value is
  Recognition, Recovery, and Reinstatement planning.
- Never read or print secret VALUES during the audit — env var names only.
```

- [ ] **Step 3: Verify**

Run: `grep -c "resilience:manual" skills/resilience-audit/references/runbook-templates.md && grep -c "Sommerville" skills/resilience-audit/references/*.md`
Expected: sentinel count 2; Sommerville appears in both files.

- [ ] **Step 4: Commit**

```bash
git add skills/resilience-audit/references/runbook-templates.md skills/resilience-audit/references/limitations.md
git commit -m "feat: add output skeleton and limitations reference"
```

---

### Task 4: README

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: repo layout from Tasks 1–3.

- [ ] **Step 1: Write README.md** with these sections (write real prose, not placeholders):
  1. Title + one-line pitch: 4R resilience planning for indie projects.
  2. "Why": scanners cover Resistance; nothing covers Recovery/Reinstatement for indie devs; this skill covers all four Rs grounded in your actual code.
  3. "The 4Rs in 30 seconds": one line each for Recognition, Resistance, Recovery, Reinstatement (paraphrase Sommerville's definitions).
  4. Install: clone repo, then either copy `skills/resilience-audit/` into `~/.claude/skills/` or add the repo as a plugin/skills dir; run `/resilience-audit` inside any project.
  5. "What you get": RESILIENCE.md description + optional GitHub issues; note the `<!-- resilience:manual -->` preservation behavior.
  6. "What this is NOT": condensed from `limitations.md` (no certification, rehearse runbooks, not a scanner).
  7. References: `Sommerville, I. (2016). Software Engineering (10th ed.). Pearson. Chapter 14, "Resilience Engineering."`

- [ ] **Step 2: Verify**

Run: `grep -c "Sommerville, I. (2016)" README.md`
Expected: ≥ 1

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README with 4R primer, install steps, citation"
```

---

### Task 5: Dogfood verification run + committed example

**Files:**
- Create: `examples/RESILIENCE.example.md` (regenerate; stays gitignored/local)
- Create: `examples-public/RESILIENCE.example.md` (committed; generated from a dummy repo)
- Modify: `README.md` (link the committed example)

**Interfaces:**
- Consumes: the complete skill from Tasks 1–4.

- [ ] **Step 1: Build a dummy target repo** at scratchpad path: minimal Node/Express app with `package.json` (express, jsonwebtoken, prisma deps), `src/auth.ts` (bcrypt + JWT login), `prisma/schema.prisma` (User + Note models), `.env.example` (DATABASE_URL, JWT_SECRET names), `vercel.json`. Real files, ~10 lines each — enough for asset detection.

- [ ] **Step 2: Execute the skill workflow manually against the dummy repo** (follow SKILL.md stages 1–4 as written, as if invoked). Write output to `examples-public/RESILIENCE.example.md`.

- [ ] **Step 3: Verify output quality against spec checks**

Checks (all must pass):
- `grep -c "prisma/schema.prisma\|src/auth.ts\|JWT_SECRET" examples-public/RESILIENCE.example.md` ≥ 3 (real paths cited)
- All four R column headers present; "Open questions" section present.
- `grep -c "rehearsed" examples-public/RESILIENCE.example.md` ≥ 1 (sociotechnical honesty)
- `grep -c "Sommerville" examples-public/RESILIENCE.example.md` ≥ 1 (citation footer)
- No secret values anywhere (env names only).

- [ ] **Step 4: Also re-run stage 1–2 against an EMPTY directory** to confirm the generic-catalog fallback produces a matrix without erroring or emitting boilerplate untied to detectable facts. Record the result as a note in the plan doc; do not commit that output.

- [ ] **Step 5: Link example from README** — add under "What you get": `See [a sample RESILIENCE.md](examples-public/RESILIENCE.example.md) generated from a small Express + Prisma app.`

- [ ] **Step 6: Commit**

```bash
git add examples-public/ README.md
git commit -m "feat: add committed sample output from dummy repo dogfood run"
```

---

### Task 6: Final review pass

**Files:**
- Modify: any file failing the checks below.

- [ ] **Step 1: Spec-coverage sweep** — reread `docs/superpowers/specs/2026-07-05-resilience-audit-skill-design.md`; confirm each spec section maps to shipped files (Purpose→SKILL.md, layout→tree, error handling→SKILL.md stage 4 + catalogs, testing→Task 5, citation→README+skeleton). Fix gaps.

- [ ] **Step 2: Run all structural checks from Tasks 1–5 in one pass**

```bash
grep -l "| Threat | CIA | Trigger" skills/resilience-audit/references/threat-catalog-*.md | wc -l   # 3
grep -c "resilience:manual" skills/resilience-audit/references/runbook-templates.md                  # 2
grep -c "Sommerville, I. (2016)" README.md                                                           # >=1
grep -rc "pass/fail\|certification" skills/resilience-audit/references/limitations.md                # >=1
```

- [ ] **Step 3: Commit any fixes and tag**

```bash
git add -A && git commit -m "chore: final spec-coverage fixes" || true
git tag v0.1.0
```
