# Recruitment Candidates — Backend Integration Boundary

Status: **PLANNING / NOT IMPLEMENTED**
Date: 2026-10-08

Canonical cross-project authority:
- https://github.com/RagueL-HigaBase/HigaBase_Plans/blob/main/SYSTEM_ARCHITECTURE.md
- https://github.com/RagueL-HigaBase/HigaBase_Plans/blob/main/CANDIDATE_ARCHITECTURE.md

Technical backend audit and ordered gates:
- https://github.com/RagueL-HigaBase/higa_systems_express/blob/main/docs/planning/CANDIDATE_PROCESSING_AUDIT_2026_10_08.md

## Current surface

- Route: /recruitment/candidates.
- src/pages/recruitment/Candidates.tsx uses the existing Phoenix-derived AdvanceTable, search, date filter, pagination and a primary Add Candidate button.
- candidateRows is intentionally empty; there is no Candidate list/query API.
- Add Candidate now opens a development-only PDF/DOCX drag/drop + validated AI preview modal. A Candidate is **not** created; production intake and persistence are still absent.
- Candidate Prototype is intentionally independent.
- CURRENT_TASK.md is still VERIFY for the frontend table scaffold; do not close it merely because this planning note exists.

## Non-negotiable backend contract before connecting

1. Candidate is a global identity, not a business SystemUser and not a Recruitment-app-owned record.
2. Organization relationships/visibility remain separate from Candidate-owned facts. One Candidate can relate to multiple Organizations only through approved and consented relations.
3. A recruiter must not be able to infer whether a Candidate already exists through Add Candidate, list or search.
4. Read and intake endpoints must be authenticated and scope-aware. Never use /api/dev/resume-drafts endpoints for a deployed recruiter UI.
5. The original PDF/DOCX and derived ResumeDraft are separate from the canonical Candidate and Resume.
6. AI and ESCO propose facts/relations; the UI must distinguish source evidence from confirmed facts.
7. Preserve Phoenix reference structure, existing Higa styling and 28-locale i18n.

## Suggested first connected workflow (not yet approved)

Add Candidate -> Upload CV -> Processing -> Waiting for candidate -> Needs screening if required -> Ready.

The status labels are *derived frontend states*. Backend request, identity resolution, artifact management, consent and screening details must not be represented by invented frontend-only data. Keep recruiter interaction small and require only the next action.

## Integration acceptance gates

- backend owner/consent decisions resolved;
- typed API contracts, stable error codes and pagination/read models available;
- negative tests for unauthorized and cross-Organization access;
- end-to-end PDF/DOCX extraction/parser failure handling and retry behavior verified;
- Phoenix side-by-side page check and local npm run verify reported green by owner;
- zero demo Candidate records in the production list.

This note authorizes no new frontend functionality. It records the safe boundary while Candidate-domain processing is designed.

## Development-only experimental CV preview — implemented 2026-10-08

- Vite development only. File is sent as a raw binary body to authenticated backend `/api/dev/resume-drafts/preview-file` with MIME and encoded filename headers; max 8 MiB.
- Phoenix Dropzone reused with selection/removal state; no new visual system or CSS.
- OpenAI file-input AI Core returns a strict ResumeDraftV1 identity/contact preview. No file or draft storage from this route, and no Candidate row is created.
- Do not mistake this for consented Candidate-Organization intake or a production storage API.
- Browser end-to-end behavior is pending owner verification. Backend/frontend CI and source review do not establish that the local provider, cookie session and proxy work together in the user's environment.
