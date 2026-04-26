## Sprache und Zielsystem

- Das gesamte Projekt ist für ein deutsches Krankenhausinformationssystem bestimmt.
- Alle sichtbaren UI-Texte müssen auf Deutsch sein.
- Alle synthetischen Beispieldaten müssen auf Deutsch sein.
- Alle klinischen Dokumente, Arztbriefe, OP-Berichte, Diagnosen, Prozeduren und Suchbeispiele müssen auf Deutsch sein.
- Die lokale LLM-Schicht muss standardmäßig auf Deutsch antworten.
- Medizinische Begriffe sollen dem deutschen klinischen Sprachgebrauch entsprechen.
- Deutsche Begriffe wie Arztbrief, OP-Bericht, Aufnahme, Entlassung, Diagnose, Prozedur, Befund, Anamnese, Verlauf, Therapie, Empfehlung und Wiedervorstellung sind bevorzugt zu verwenden.
- ICD-, OPS-, DRG- und Abrechnungslogik sollen perspektivisch auf den deutschen Krankenhauskontext ausgerichtet sein.
- Keine echten Patientendaten verwenden.
-
- # AGENTS.md

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
