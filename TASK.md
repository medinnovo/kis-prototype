# Task: Build first prototype of a minimal KIS system

## Goal

Build a working prototype of a minimal hospital information system with:

1. Minimalistic clinical frontend
2. Patient and encounter backend
3. Clinical document storage
4. Hybrid search foundation
5. Statistics and accounting dashboard foundation

This is a prototype, not a certified medical product.

## Tech stack

Use:

- Frontend: React with Vite
- Backend: FastAPI
- Database: PostgreSQL
- Vector search: pgvector
- ORM: SQLAlchemy
- Auth: simple role-based prototype auth, no production auth yet
- Docker Compose for local development

## Main entities

Create database models for:

- Patient
- Encounter
- ClinicalDocument
- Diagnosis
- Procedure
- BillingCase
- AccountingEvent
- AuditLog

Use stable IDs:

- patient_id
- encounter_id
- document_id
- chunk_id

## Features

### 1. Patient dashboard

Create a clean frontend with:

- Patient search
- Patient list
- Patient profile header
- Timeline of encounters
- Clinical documents
- Diagnoses
- Procedures
- Accounting summary

### 2. Clinical document editor

Allow creation of a clinical note with:

- document type
- title
- body text
- patient_id
- encounter_id

After saving, store it in PostgreSQL.

### 3. Hybrid search prototype

Implement:

- structured SQL search by patient name, date of birth, diagnosis, procedure
- full-text search over clinical documents
- placeholder embedding pipeline
- pgvector column for document chunks
- vector search function stub that can later be connected to a real embedding model

For now, create deterministic mock embeddings so the prototype runs locally without external API keys.

### 4. Research cohort builder

Create a simple query page where the user can filter by:

- age range
- diagnosis text or code
- procedure text or code
- document text search

Return matching patients and source documents.

### 5. Accounting and statistics

Create dashboard cards for:

- number of patients
- number of encounters
- number of documents
- number of procedures
- number of billing cases
- estimated total revenue
- estimated total costs
- estimated margin

Create simple charts or tables if feasible.

### 6. Audit log

Log important events:

- patient opened
- document created
- search performed
- billing case viewed

## UI style

Minimal, clinical, calm.

Use:

- white or very light background
- clean cards
- clear typography
- no unnecessary decoration
- fast navigation
- search bar always visible

## Required pages

- `/`
- `/patients`
- `/patients/:id`
- `/search`
- `/research`
- `/accounting`
- `/documents/new`

## Deliverables

1. Working Docker Compose setup
2. Backend API
3. Frontend UI
4. Seed data with at least:
   - 10 patients
   - 20 encounters
   - 30 clinical documents
   - 20 diagnoses
   - 20 procedures
   - 10 billing cases
5. README with setup instructions
6. Basic tests for backend endpoints

## Acceptance criteria

The app must run locally with:

```bash
docker compose up

# Add Local LLM Layer

## Goal

Add a local LLM layer to the KIS prototype. The LLM must run locally via Ollama or another local inference server. No external API calls are allowed.

The LLM layer should help with:

1. Finding previous relevant findings
2. Drafting clinical letters from current and previous encounters
3. Detecting healing or worsening patterns
4. Suggesting possible coding and DRG-relevant documentation gaps
5. Showing therapy context from prior records and local SOPs

This is clinical assistant functionality only. All outputs require human review.

## Architecture

Add services:

- `llm_service`
- `ollama`

The backend should call `llm_service`, not Ollama directly from the frontend.

## Required endpoints

Create these backend endpoints:

- `POST /llm/find-prior-findings`
- `POST /llm/draft-letter`
- `POST /llm/pattern-analysis`
- `POST /llm/coding-suggestions`
- `POST /llm/therapy-context`

## Retrieval rules

The LLM must never access the database directly.

The backend must retrieve allowed source data first, then send only that context to the LLM.

Every LLM output must include:

- source document IDs
- source encounter IDs where available
- confidence level
- missing information
- `requires_human_review: true`

## Structured output

Use JSON schemas for all LLM outputs.

The LLM must return valid JSON for:

- prior findings
- letter draft
- pattern analysis
- coding suggestions
- therapy context

## Local model

Use Ollama as the default local LLM server.

Add configuration variables:

- `OLLAMA_BASE_URL`
- `OLLAMA_GENERATION_MODEL`
- `OLLAMA_EMBEDDING_MODEL`

Use mock mode if Ollama is not available.

## Embeddings

Use the existing document chunks.

Add an embedding interface:

- `embed_text(text: str) -> list[float]`

Support two implementations:

1. Mock embeddings for tests
2. Ollama embeddings for local use

## UI features

Add buttons on the patient page:

- "Find prior findings"
- "Draft letter"
- "Analyze pattern"
- "Coding suggestions"
- "Therapy context"

Show results in a review panel.

For every generated statement, show source document links.

## Safety constraints

- Do not use real patient data.
- Do not call external APIs.
- Do not make autonomous diagnoses.
- Do not make autonomous treatment decisions.
- Do not finalize coding automatically.
- Do not finalize DRG automatically.
- All outputs must be drafts.
- All outputs must be auditable.
- Every generated clinical claim must be tied to a source document.

## Acceptance criteria

The user can:

1. Open a synthetic patient
2. Click "Find prior findings"
3. See source-linked prior findings
4. Click "Draft letter"
5. See a structured draft letter
6. Click "Analyze pattern"
7. See a source-linked healing or worsening pattern
8. Click "Coding suggestions"
9. See possible codes and documentation gaps
10. Run the whole system locally with Docker Compose
