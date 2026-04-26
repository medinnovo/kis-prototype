# AGENTS.md

## Project rules

- Build a medical software prototype, not production clinical software.
- Never use real patient data.
- Use synthetic seed data only.
- Keep backend and frontend simple.
- Prefer readable code over clever abstractions.
- Every search result must link back to source document and patient.
- MySQL-style thinking is acceptable, but this prototype uses PostgreSQL because pgvector is required.
- Do not add external API dependencies.
- Do not add paid services.
- Keep all setup runnable with Docker Compose.

## Validation

Before finishing a task:

1. Run backend tests.
2. Run frontend build.
3. Confirm docker compose starts.
4. Update README if setup changed.
