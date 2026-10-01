# REPO_STRUCTURE

## CORE REPOSITORY STRUCTURE

```
/apps
  /web              React frontend
  /extension        Chrome extension
/services
  /api              FastAPI core API
  /audio-gateway    WebSocket audio service
  /workers          ASR, pipeline, export, learning
/packages
  /schemas          Pydantic + TypeScript shared types
  /llm-gateway      routing, redaction, caching, fallback
  /terminology      glossary, code lookup, RAG
/eval               golden set, metrics scripts
/infra              Terraform, Kubernetes, Docker
/docs               architecture, decisions, runbooks
```

