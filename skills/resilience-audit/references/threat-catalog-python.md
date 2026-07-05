# Threat catalog — Python (Django / FastAPI / Flask)

Use alongside the generic catalog. Each row: only include it if the
trigger is present in the repo.

| Threat | CIA | Trigger (what to look for) | Typical Recognition | Typical Resistance |
|---|---|---|---|---|
| `DEBUG=True` or `SECRET_KEY` in settings | C | `settings.py` with literal secret/debug values | error pages leaking stack traces in prod | env-driven settings, refuse to boot with debug in prod |
| API docs endpoints exposed in prod | C | FastAPI app (auto `/docs`, `/openapi.json`) | access logs on `/docs` from unknown IPs | disable or auth-gate docs in prod config |
| Unsafe deserialization | I | `pickle.load`, `yaml.load` (without SafeLoader) on external input | crash/error-rate alerts on ingest paths | `yaml.safe_load`, JSON instead of pickle, validate schemas |
| SQL injection via f-strings | I | f-strings containing SELECT/INSERT/UPDATE with interpolated variables | DB error-rate alerts, slow-query anomalies | ORM/parameterized queries only |
| Unpinned dependencies | I | `requirements.txt` without `==` pins, no lockfile | CI audit step | pin versions (pip-tools/uv/poetry lock), review bumps |
| Task-queue poisoning | I/A | celery/rq/dramatiq imports | dead-letter queue monitoring, task failure alerts | validate task payloads, don't enqueue raw user input |
| Migration-on-deploy data damage | I/A | `manage.py migrate` or alembic in deploy config | post-deploy health check counting core tables | reviewed additive migrations, restore point before deploy |
