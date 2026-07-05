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
