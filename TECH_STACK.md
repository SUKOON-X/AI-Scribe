# TECH STACK


## FRONTEND
React + TypeScript, Vite, TanStack Query, Zustand, Tailwind + shadcn/ui, Web Audio API + AudioWorklet (16 kHz PCM), WebSocket client, Tiptap (note editor), wavesurfer.js (audio-linked evidence), react-i18next, PWA with IndexedDB audio buffer for flaky networks

## EMR bridge
Chrome extension (Manifest V3) for paste-in, plus a REST and webhook API

## API 
Python 3.12, FastAPI, Uvicorn, Pydantic v2, WebSockets

## ORCHASTRATION
LangGraph, with the Postgres checkpointer for durable state and human-in-the-loop interrupts

## LLM ACCESS 
LiteLLM (or a thin in-house adapter), JSON-schema structured outputs

## ASYNC JOBS 
Redis + Celery (or Arq)

## DATABASE
PostgreSQL 16 with pgvector (terminology embeddings), pg_trgm (fuzzy brand-name matching), JSONB (notes), row-level security (multi-tenant), pgBouncer

## OBJECT STORAGE
S3-compatible in an Indian region (or MinIO), SSE-KMS encryption, lifecycle rules to auto-delete audio

## TERMINOLOGY
SNOMED CT, ICD-10 (ICD-11 later), LOINC, and a licensed Indian drug database (brand, generic, strengths). Verify licensing.

## INTEROP
HL7 FHIR R4, ABDM FHIR profiles, fhir.resources library

## PII HANDLING

Microsoft Presidio plus custom Indian recognizers (Aadhaar, ABHA, phone)

## AUTH
Keycloak or Auth0 (OIDC), RBAC, MFA

## OBSERVABILITY
Langfuse (self-hosted LLM tracing), OpenTelemetry, Prometheus + Grafana, Sentry

## EVAL / ML
pytest, jiwer (WER), custom medical-entity F1, Label Studio (annotation), Hugging Face + PEFT/LoRA, faster-whisper, vLLM

## INFRA DEVOPS 
Docker, Kubernetes or ECS in an Indian region, Terraform, GitHub Actions, Vault or KMS, WAF