# AI Core

Last reviewed: 2026-10-07

Cross-project authority:
- `HigaBase_Plans/AI_CORE.md`;
- `HigaBase_Plans/SYSTEM_ARCHITECTURE.md`.

## Purpose

The backend AI Core is a provider-neutral execution boundary. Domain code defines a named, versioned task with strict input/output schemas. A provider adapter receives only validated task input and returns an untrusted response. The runtime validates that response before any domain service can use it.

## Runtime contract

`src/ai/contracts.ts` owns named/versioned task definitions.

`src/ai/provider.ts` owns the provider adapter boundary. Provider-specific SDK payloads must not escape an adapter.

`src/ai/runtime.ts` owns execution:
1. validate task input;
2. call the provider;
3. contain provider failure;
4. validate provider output;
5. return validated output plus bounded execution metadata.

Stable runtime failure codes:
- `AI_INPUT_INVALID`;
- `AI_PROVIDER_FAILED`;
- `AI_OUTPUT_INVALID`.

AI Core currently has no HTTP endpoint and no database persistence.

## Source artifacts and provenance

Original source files are owned by the future Higa file/object-storage boundary, not AI Core. AI tasks receive stable source references or extracted content appropriate to the task.

The original artifact remains available and immutable evidence. Derived text/results never replace it.

Execution metadata currently supports task name/version, provider, optional model, duration and optional token usage. Persistent provenance will be designed when the first real AI workflow requires storage.

## Security and authority

AI Core:
- does not write canonical Prisma/domain records directly;
- does not grant permissions or approve workflow transitions;
- does not invent canonical ESCO identifiers;
- does not define Candidate/Resume persistence;
- does not expose provider-specific contracts to domain code.

The owning domain validates/authorizes persistence after AI execution.

## First proving use case

The first proving task is `candidate.resume.parse@1`, producing the strict experimental `ResumeDraftV1` contract. Task input now supports text or PDF/DOCX file content; the development HTTP parse route still accepts text only. Real PDF/DOCX parsing is exercised by a private local OpenAI research CLI, not a production upload API. The draft repository currently retains only contract-versioned JSONB and timestamps, not durable source/provenance or Candidate identity.

Current provider adapters:
- `OpenAiResponsesProvider` for the cloud Responses API path;
- `OllamaProvider` for local/self-hosted inference over Ollama HTTP.

Provider selection is configuration only. ResumeDraft, the task contract and AiRuntime do not depend on OpenAI or Ollama. The local benchmark defaults to `http://localhost:11434` with `gpt-oss:20b`.

The Ollama path is an experimental development benchmark and accepts text input only, not binary PDF/DOCX. Neither the local nor OpenAI parser experiment is a production Candidate inference decision. There is no automatic cloud/local fallback or provider router yet. Production Candidate processing must establish SourceArtifact ownership, access/consent, original-source preservation and review checkpoints. See `docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md`.


## Development-only browser file preview (2026-10-08)

An experimental, authenticated browser bridge is available in non-production environments:
`POST /api/dev/resume-drafts/preview-file`.

It accepts the raw PDF/DOCX body (max 8 MiB), an encoded `X-Resume-Filename` header, and requires an unlocked Organization workspace session. Input is passed through the existing versioned parse task and OpenAI Responses file-input adapter. The result is returned as a validated ephemeral `ResumeDraftV1` preview only.

No source artifact, ResumeDraft row, Candidate identity or Organization relationship is stored by this new route. No multi-model file routing exists: Ollama currently supports text input only. The existing development text parse routes are separate research endpoints and do not gain production authorization or ownership from this feature.

This bridge is deliberately not a production upload API. Full source validation, object storage, retention/deletion, consent, idempotency and durable job handling remain future Candidate intake requirements. See `docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md`.
