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
