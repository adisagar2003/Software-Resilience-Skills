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
