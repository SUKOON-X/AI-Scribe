
# AI SCRIBE (working title)

**A multilingual AI medical scribe for Indian clinics.**
Doctors speak in **Hindi, English or Hinglish**. The system produces a **structured clinical note and prescription in standard medical English**, which the doctor reviews, edits and signs.

> **The AI drafts, the doctor decides.** No note leaves the system unsigned.

> **Status:** Pre-development. This README describes the intended design. Commands, paths and configuration below are the target conventions and should be updated as each milestone is implemented.

---

## Table of contents

1. [Why this project](#why-this-project)
2. [Features](#features)
3. [How it works](#how-it-works)
4. [Language behavior](#language-behavior)
5. [Architecture](#architecture)
6. [Tech stack](#tech-stack)
7. [Repository structure](#repository-structure)
8. [Getting started](#getting-started)
9. [Configuration](#configuration)
10. [Evaluation and quality gates](#evaluation-and-quality-gates)
11. [Safety rules](#safety-rules)
12. [Privacy and compliance](#privacy-and-compliance)
13. [Roadmap](#roadmap)
14. [Contributing](#contributing)
15. [Disclaimer](#disclaimer)
16. [License](#license)

---

## Why this project

- Indian doctors see high patient volumes and lose a large share of each consult to typing or writing notes.
- Real consultations mix Hindi and English, with regional accents, in noisy rooms. Many global scribes handle this poorly.
- Everyday Hindi symptom words (for example "chakkar", "seene mein dard") must become precise clinical terms.
- Indian products such as EkaScribe and Augnito already exist, so this project needs a clear wedge: **[specialty / clinic type / EMR integration to be decided]**.

## Features

**MVP**
- Doctor login with MFA
- Patient and encounter creation with a consent step
- Dictation mode with browser microphone capture and live transcript
- Hindi, English and Hinglish input
- Normalization to standard medical English
- SOAP note generation from templates
- Note editor with full edit history
- Doctor sign-off (signed notes are immutable)
- PDF export and copy to clipboard
- Automatic audio deletion

**Phase 2**
- Ambient mode (doctor-patient conversation with speaker separation)
- Prescription output: brand name, dose, frequency, duration, follow-up
- Evidence links: click any sentence to hear the source audio
- Specialty templates that learn from doctor edits
- Chrome extension to insert notes into web-based EMRs
- Offline recording with later sync

**Phase 3**
- FHIR R4 and ABDM export
- ICD-10 and SNOMED CT code suggestions
- More Indian languages, on-prem or offline deployment
- Patient summaries over WhatsApp or SMS (with consent)

## How it works

```
Doctor speaks
  -> Browser captures audio (16 kHz chunks, buffered locally)
  -> WebSocket audio gateway (ack, dedupe, resume)
  -> Streaming ASR (Hinglish / codemix) -> live transcript on screen
  -> Doctor taps Stop
  -> LangGraph pipeline
       clean -> normalize to medical English -> extract entities
       -> ground to medical codes / drug DB -> detect conflicts
       -> draft note from template -> link evidence -> verify
  -> Doctor review (pipeline pauses until the doctor acts)
  -> Sign -> export (PDF / FHIR / EMR) -> store edits for learning
```

## Language behavior

- **Input:** Hindi, English, or mixed Hinglish, sometimes switching mid-sentence.
- **Transcript:** original words are preserved (Devanagari for Hindi, Latin for English) with timestamps. The transcript is the source of truth.
- **Output:** standard medical English. Translations are derived data.

**Worked example (also a permanent automated test):**

Input:

> "Patient ko teen din se bukhar hai aur sar mein dard, Paracetamol 650 din mein do baar teen din tak dena hai."

Expected output:

- Chief complaint: Fever for 3 days with headache
- Prescription: Paracetamol 650, twice daily, for 3 days
- **Flag:** unit not spoken. Confirm "650 mg"

The system flags missing information. It never guesses.

## Architecture

```
CLIENTS       Web app (React) | Mobile PWA | Chrome extension | Admin console
                 |  HTTPS (REST) + WSS (audio up, live transcript down)
EDGE          CDN -> WAF -> Load balancer (TLS)
                 |
SERVICES      Auth | Core API | Realtime Audio Gateway | Integration
              Consent & Policy | Template & Glossary | Notification | Billing
                 |
EVENTS        Redis Streams (audio.chunk, asr.segment, encounter.ended,
              note.requested, note.ready, note.signed, export.requested)
              retries + dead-letter queue + idempotency keys
                 |
WORKERS       ASR workers | LangGraph scribe pipeline | Export | Learning
                 |
AI LAYER      ASR providers      LLM gateway            Terminology / RAG
              (Sarvam, Parrotlet, (redact, route, cache,  (SNOMED, ICD, LOINC,
               Whisper)            budget, fallback)       drug DB, glossary)
                                        |
                                  LLM providers (Claude / Gemini / GPT /
                                  self-hosted open model)
                 |
DATA          PostgreSQL (RLS, pgvector, checkpoints) | Redis | S3 (KMS)
              De-identified analytics store

CROSS-CUTTING Security (KMS, mTLS, audit) | Observability (OTel, Langfuse)
              Quality (eval harness) | DevOps (Terraform, CI/CD)
```

### LangGraph pipeline

```
load_context -> finalize_transcript -> route_by_mode
  -> [ambient only: diarize_roles]
  -> clean_and_tag -> normalize_medical_english -> extract_entities
  -> ground_codes -> detect_conflicts -> draft_note -> build_evidence_links
  -> verify --fail (retry < 2)--> repair --> verify
           --pass or flagged----> human_review (INTERRUPT)
  -> finalize_sign -> fan_out (export | feedback | billing) -> END
```

If any step fails, the system retries, falls back to another model, and as a last resort delivers the raw transcript with `needs_manual` status. **Audio is never lost.**

### Encounter states

```
created -> recording -> processing -> draft_ready -> in_review -> signed -> exported -> archived
                            |                            |
                            v                            v
                   processing_failed ------------> needs_manual
```

## Tech stack

@TECH_STACK.md

> The final ASR and LLM choices are made by a **bake-off on our own consented, de-identified clinic audio**, measured on medical-entity accuracy and hallucination rate rather than general benchmarks.

## Repository structure

@REPO_STRUCTURE.MD

## Getting started

> These are the **target** conventions. Update this section as the tooling is built.

### Prerequisites

- Docker and Docker Compose
- Python 3.12 and a package manager such as `uv` or `pip`
- Node.js 20+ and `pnpm`
- API keys for the ASR and LLM providers you want to use (see [Configuration](#configuration))

### Run locally

```bash
# 1. Clone
git clone <repo-url> AI_SCRIBE
cd AI_SCRIBE

# 2. Configure
cp .env.example .env        # then fill in values

# 3. Start infrastructure (Postgres, Redis, MinIO)
docker compose up -d

# 4. Backend: install, migrate, run
make api-install
make migrate
make api-dev                # FastAPI on http://localhost:8000

# 5. Workers
make workers-dev

# 6. Frontend
make web-install
make web-dev                # React on http://localhost:5173
```

### Tests and evaluation

```bash
make test                   # unit + integration tests
make eval                   # replay the golden set, print WER / entity F1 / hallucination rate
```

**Never use real patient data in development.** Use synthetic or consented, de-identified samples only.

## Configuration
@env.example


## Evaluation and quality gates

Quality is measured continuously, not assumed.

| Area | Metric | Target |
|---|---|---|
| ASR | Word error rate and medical-keyword accuracy on the gold set | Beat baseline engine |
| Extraction | Drug, dose and allergy F1 | [e.g. >= 0.95, set with clinical advisor] |
| Safety | Major hallucinations per sampled note | Near zero |
| Speed | Median time from Stop to draft ready | [e.g. < 30 s] |
| Adoption | Share of notes signed with few edits | Rising week over week |

CI replays the golden set on every release candidate. **A release is blocked if WER, entity F1 or hallucination rate regress.**

For model selection, test candidate LLMs with reasoning both on and off. Stronger reasoning does not automatically improve note fidelity.

## Safety rules

@SEFETY_RULES.MD


## Privacy and compliance

- **Consent** is recorded (notice version, timestamp, purpose) before any recording. Withdrawal must be as easy as giving consent.
- **DPDP Act and Rules (India):** most obligations for health apps apply from **13 May 2027**, and the date could move earlier. The project builds to that standard from the start.
- **Data residency:** data processed within ABDM must be stored in India, so production hosts in an Indian region.
- **Retention:** audio is deleted automatically after the configured period. Erasure requests remove audio, transcripts and derived data, except records that clinical-record rules require.
- **Breach response:** notify the Data Protection Board and affected individuals, and follow up with a detailed report within 72 hours.
- **Vendors:** ASR and LLM providers act as data processors under signed agreements. Responsibility stays with us as data fiduciary.
- **Regulatory classification (CDSCO) and retention defaults** must be confirmed with legal counsel.

| Data | Store | Protection |
|---|---|---|
| Raw audio | S3 | KMS encryption, per-tenant key, auto-expiry |
| Transcripts, entities, notes | PostgreSQL | Row-level security, immutable once signed |
| Consents, audit log | PostgreSQL | Append-only |
| LLM traces | Langfuse | PII-redacted only, short retention |
| Eval / analytics | Analytics store | De-identified only |

## Roadmap

1. **Foundation:** repo, Docker Compose, migrations, auth, CI
2. **Evaluation harness:** WER, entity F1, hallucination metrics
3. **Audio path:** browser recorder, WebSocket gateway with sequence and ack, S3 storage
4. **Transcription:** ASR adapter with fallback, live transcript
5. **LangGraph pipeline v1:** clean, normalize, extract, draft SOAP note
6. **Review and sign:** editor, version history, sign-off, PDF export
7. **Verification and evidence links**
8. **Governance:** consent ledger, audit log, retention jobs, row-level security
9. **Ambient mode, prescription, Chrome extension, FHIR export**
10. **Hardening:** load test, failover drills, security review, pilot readiness

Each milestone has a "done" criterion. Do not start the next milestone until the previous one passes. Target: pilot with 20-50 doctors after roughly six months.

## Contributing

- Work one milestone at a time and write tests before moving on.
- Write a short design note before each milestone and get it reviewed before writing code.
- Keep the worked example in [Language behavior](#language-behavior) as a permanent test and add more of your own.
- Run `make test` and `make eval` before opening a pull request.
- Do not commit patient data, audio, or secrets.

## Disclaimer

This software assists with clinical documentation. It is not a diagnostic tool and does not replace clinical judgement. **A licensed clinician must review, correct and sign every note.** Accuracy varies by language, accent and audio quality, and Hindi medical speech recognition in particular is not perfect.

## License

**Apache-2.0 **