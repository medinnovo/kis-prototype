A good KIS should not start as “one big hospital software.” It should start as a clinical operating system with three hard requirements:

1. Fast minimalistic clinical frontend
2. Search and research layer over structured and unstructured data
3. Reliable backend for statistics, controlling, coding, accounting, and audit

Core principle

The KIS should separate clinical work, data storage, semantic search, and billing/statistics.

Minimal frontend
        ↓
Clinical workflow API
        ↓
Core patient database
        ↓
Event log / document store / coding layer
        ↓
Search index + vector index + analytics warehouse
        ↓
Statistics, research, accounting, controlling

The mistake would be to build one monolithic MySQL app that tries to do everything. That becomes slow, hard to validate, and unpleasant for doctors.

1. Frontend philosophy

The frontend should be minimal, fast, and task-based.

A doctor should not “navigate the KIS.” A doctor should see:

Patient
Timeline
Problem list
Today’s task
Documents
Orders
Results
Search

For example:

Patient dashboard
├── Alerts
├── Current diagnoses
├── Timeline
├── Notes
├── OP reports
├── Lab / imaging
├── Medication
├── Coding suggestions
└── Research tags

The clinical screen should be closer to a good command center than a bureaucratic form system.

Recommended frontend structure:

Area	Function
Patient header	Name, age, MRN, allergies, location, case number
Timeline	Encounters, procedures, letters, imaging, labs
Command search	Search patient, document, diagnosis, operation, cohort
Smart note editor	Dictation, structured tags, templates
Clinical cards	Problems, plans, orders, results
Coding sidebar	ICD, OPS, DRG suggestions, documentation gaps
Research sidebar	Consent, registry eligibility, tags

The interface should be sparse by default and detailed only when needed.

2. Backend architecture

I would plan the backend in layers.

1. Identity and access layer
2. Patient master data layer
3. Clinical data layer
4. Document layer
5. Search layer
6. Research and analytics layer
7. Accounting and controlling layer
8. Interoperability layer
9. Audit and compliance layer

Each layer has a clear job.

3. Database model

Use a relational database for the authoritative record.

For example:

PostgreSQL or MySQL

I would prefer PostgreSQL for a new system because it has strong JSON support, full-text search, extensions, analytics options, and pgvector. MySQL is usable, but PostgreSQL gives more flexibility for clinical data models.

Core relational tables:

patients
encounters
cases
departments
beds
diagnoses
procedures
orders
results
documents
users
roles
permissions
billing_cases
accounting_events
audit_logs

The relational database is the source of truth.

The vector database is not the source of truth. It is an index.

4. Search architecture

Search should be hybrid.

Structured database search
+
Keyword full-text search
+
Vector semantic search
+
Permission filtering

A proper search query should be able to answer:

Find all patients under 10 years old
with anorectal malformation
who had a PSARP
and later developed constipation or soiling
despite laxative therapy.

This requires several search modes:

Query part	Best system
under 10 years old	SQL
anorectal malformation	diagnosis table, ICD, registry tag
PSARP	procedure table, OPS, OP note
constipation or soiling	keyword and vector search
despite laxative therapy	vector search over letters and notes

The search flow should look like this:

User query
        ↓
Query parser
        ↓
Structured filters from SQL
        ↓
Semantic search in vector index
        ↓
Keyword validation
        ↓
Permission check
        ↓
Results with source documents

5. How to tie SQL and vector search together

Every document and every vector chunk must keep the same identifiers as the relational database.

patient_id
case_id
encounter_id
document_id
chunk_id
department_id
author_id
created_at

Example:

{
  "chunk_id": "chunk_001922",
  "document_id": "doc_8831",
  "patient_id": "pat_441",
  "case_id": "case_2026_00421",
  "encounter_id": "enc_762",
  "text": "Persistent soiling despite macrogol therapy...",
  "embedding": [0.01, -0.33, 0.88],
  "metadata": {
    "document_type": "Arztbrief",
    "department": "Kinderchirurgie",
    "created_at": "2026-04-26"
  }
}

The app never trusts the vector result alone. The vector result only says:

This chunk may be relevant.

Then the system fetches the verified patient, case, document, and permissions from SQL.

6. Document and note system

Clinical documents should be stored as both:

original text
+
structured metadata

Example document structure:

document_id
patient_id
case_id
document_type
author
date
status
signed_at
text_body
structured_json
version

The structured JSON can contain sections:

{
  "anamnese": "...",
  "befund": "...",
  "diagnosen": [],
  "prozeduren": [],
  "plan": "...",
  "coding_relevant_findings": []
}

This allows the same document to serve:

1. Clinical communication
2. Search
3. Research
4. Coding
5. Accounting
6. Quality management

7. Research layer

Research should not query the live KIS directly.

Use a separated research layer:

Production KIS
        ↓
Validated extract pipeline
        ↓
Research data mart
        ↓
Pseudonymization / consent check
        ↓
Registry / cohort builder

Research functions:

Cohort search
Registry builder
Export to REDCap
Outcome tracking
Follow-up reminders
Data quality checks
Natural language cohort discovery

For example:

ARM registry
├── patient pseudonym
├── Krickenbeck type
├── associated anomalies
├── operative technique
├── complications
├── bowel management status
├── continence score
└── follow-up dates

Important: the research database should contain only what is needed, with consent and governance rules.

The European Health Data Space Regulation creates a framework for access, exchange, and certain secondary use of electronic health data in the EU, including scientific research and public-interest purposes. This should be considered early when designing research access and governance.  ￼

8. Statistics and controlling backend

For statistics and accounting, do not rely only on clinical notes. Build a proper accounting event model.

Clinical event
        ↓
Coding event
        ↓
Billing event
        ↓
Accounting event
        ↓
Controlling dashboard

Tables:

billing_cases
diagnosis_codes
procedure_codes
drg_grouping_results
revenue_estimates
cost_centers
resource_usage
length_of_stay
or_minutes
icu_days
implant_costs
material_costs
staff_time_estimates

The system should know:

Which case?
Which department?
Which diagnosis?
Which procedure?
Which DRG?
Which cost center?
Which physician?
Which material?
Which revenue?
Which margin?
Which documentation gaps?

This allows dashboards like:

Department margin by diagnosis group
Average length of stay by procedure
OR time versus reimbursement
Rare disease deficit analysis
Coding completeness
Unbilled services
DRG outlier cases

For pediatric surgery, this is critical because many complex congenital cases are clinically important but financially poorly represented.

9. Accounting model

The accounting module should be event-based, not just invoice-based.

Example:

Admission created
Diagnosis added
Procedure performed
Material used
OP time recorded
ICU stay recorded
Discharge letter signed
Coding completed
DRG grouped
Invoice released
Payment received

Each event becomes auditable.

{
  "event_type": "procedure_performed",
  "case_id": "case_2026_00421",
  "procedure_code": "OPS...",
  "timestamp": "2026-04-26T10:22:00",
  "source": "OP documentation",
  "user_id": "user_17"
}

This gives you a strong backend for statistics, reimbursement, and quality control.

10. Interoperability

Do not design a closed system.

Plan interfaces from the beginning:

FHIR
HL7 v2
DICOM
IHE profiles
CSV / Parquet exports
REST APIs
Message queue

FHIR is the modern standard family for exchanging healthcare information electronically. The current published HL7 FHIR specification page identifies FHIR R5 as the current published version, while many real-world implementations still depend on R4 or national profiles.  ￼

IHE profiles are useful because they define practical integration patterns for healthcare IT products and give buyers and developers a common implementation language.  ￼ IHE also describes itself as promoting coordinated use of established standards such as DICOM and HL7 for clinical interoperability.  ￼

For a German hospital context, I would not build without considering national interoperability requirements, Gematik/ISiK alignment, HL7/FHIR, and billing-related interfaces. Exact implementation details need local verification before procurement or development.

11. Suggested technical stack

For a modern but realistic stack:

Frontend:
React / Next.js
Backend:
NestJS, FastAPI, or Spring Boot
Primary database:
PostgreSQL
Search:
OpenSearch or Elasticsearch
Vector search:
pgvector, Qdrant, or Weaviate
Documents:
PostgreSQL + object storage
Analytics:
DuckDB / ClickHouse / PostgreSQL warehouse / BigQuery equivalent if cloud is allowed
Messaging:
Kafka, RabbitMQ, or NATS
Identity:
Keycloak with hospital LDAP / Active Directory
Audit:
append-only audit log
Deployment:
Docker, Kubernetes if the team can maintain it

For a smaller first version:

React
FastAPI
PostgreSQL
pgvector
OpenSearch
Keycloak
Docker

12. Minimalistic frontend, maximal backend

The frontend should not expose backend complexity.

Doctor view:

Search patient
Open timeline
Dictate note
Sign letter
See coding gaps
Find similar cases

Backend view:

Every note is indexed
Every change is versioned
Every access is audited
Every clinical event can support statistics
Every case can support accounting
Every research export is governed

That is the right separation.

13. AI layer

The AI layer should be assistant-like, not autonomous.

Good uses:

Summarize patient timeline
Suggest ICD / OPS codes
Find missing documentation
Find similar cases
Draft Arztbrief
Extract registry variables
Explain cost drivers

Unsafe uses without human verification:

Final diagnosis
Automatic billing release
Unreviewed research cohort inclusion
Autonomous treatment decision

The AI should always show source documents.

Example:

Suggested code:
Q43.1 Hirschsprung disease
Evidence:
OP report from 12.03.2026
Discharge letter from 18.03.2026
Pathology report from 15.03.2026

14. Security and governance

For a KIS, this is not optional.

Required:

Role-based access control
Patient-context access
Emergency access with break-glass logging
Audit logs
Encryption at rest
Encryption in transit
Backups
Immutable logs
Consent management
Research governance
Pseudonymization
Data retention rules
Medical device boundary analysis

[Inference] If the system influences clinical decisions, coding, or patient management, legal and regulatory classification may become relevant. This needs formal assessment by regulatory and data protection experts.

15. Development phases

Phase 1: Clinical core

Patient registry
Encounter management
Document editor
Timeline
Basic search
User roles
Audit log

Phase 2: Hybrid search

Full-text search
Vector search
Document chunking
Semantic cohort search
Source-linked results

Phase 3: Coding and accounting

ICD / OPS support
DRG grouping interface
Documentation gap detection
Case profitability dashboard
Department statistics

Phase 4: Research layer

Cohort builder
Registry exports
Consent management
Pseudonymized research database
Outcome dashboards

Phase 5: Interoperability

FHIR interfaces
HL7 v2 interfaces
DICOM links
Laboratory and radiology integration
Billing interfaces
External referral portal

16. The system in one diagram

                          ┌──────────────────────┐
                          │ Minimal clinical UI   │
                          └──────────┬───────────┘
                                     │
                          ┌──────────▼───────────┐
                          │ Application API       │
                          └──────────┬───────────┘
                                     │
        ┌────────────────────────────┼────────────────────────────┐
        │                            │                            │
┌───────▼────────┐          ┌────────▼────────┐          ┌────────▼────────┐
│ Core SQL DB     │          │ Document store  │          │ Audit log       │
│ Patient truth   │          │ Notes/letters   │          │ Legal trace     │
└───────┬────────┘          └────────┬────────┘          └─────────────────┘
        │                            │
        │                   ┌────────▼────────┐
        │                   │ Search indexes  │
        │                   │ keyword/vector  │
        │                   └────────┬────────┘
        │                            │
┌───────▼────────┐          ┌────────▼────────┐
│ Billing layer   │          │ Research layer  │
│ DRG/accounting  │          │ cohorts/registry│
└───────┬────────┘          └────────┬────────┘
        │                            │
┌───────▼────────┐          ┌────────▼────────┐
│ Controlling     │          │ Pseudonymized   │
│ dashboards      │          │ data mart       │
└────────────────┘          └─────────────────┘

Bottom line

I would design the KIS as a thin, elegant clinical frontend over a strong event-based backend.

The winning structure is:

Relational database for truth
Document system for clinical narrative
Hybrid search for meaning
Analytics warehouse for statistics
Accounting event layer for controlling
FHIR/IHE interfaces for interoperability
Strict audit and permission model for trust

The most valuable differentiator would be a doctor-first search bar that can search across patients, notes, operations, diagnoses, accounting gaps, and research cohorts, while always returning verifiable source documents.
