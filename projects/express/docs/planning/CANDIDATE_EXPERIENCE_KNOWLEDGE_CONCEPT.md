# HIGA Candidate Experience Knowledge — initial concept

Status: **DESIGN DRAFT — no implementation authorized**
Recorded: 2026-10-11
Scope: candidate CV/work-experience representation and reusable HIGA knowledge concepts.
No database migrations, API changes, screening workflow, matching engine, or UX implementation in this phase.

## Primary design decisions

1. **Work Experience is the time-bounded evidence unit**, with employer, role/title, start date, end date, and source CV evidence.
2. Each Work Experience can link to shared HIGA concepts through **four visible badge groups**: **Responsibilities**, **Skills**, **Knowledge**, **Tools & Technologies**. The same concept can be reused across multiple experiences and vacancies. A concept does not inherit the entire duration automatically.
3. **Qualifications** (certificates, permits, licenses, short vocational/professional courses) are a **separate candidate-profile section**, optionally associable with an experience if useful. Qualifications are not proof of all implied skills and require specific credentials/validity where available.
4. **Education** (school, institution, formal studies, degrees) is separately modeled and displayed; do not conflate with qualifications.
5. The concept catalog must **resolve semantic equivalents before proposing a new concept**. Examples of distinct meanings: order picker (role/title) vs picking and packing (activities); welder (role) vs welding (process/skill); distinct welding certifications must not be merged simply because the names look alike. An occupation/title can remain descriptive metadata rather than the governing taxonomy.
6. Each concept has a stable ID, type, concise canonical definition, multilingual labels and descriptions, and provenance. Candidate-specific experience links point to it rather than duplicating definitions. Separate definition from evidence/source phrasing. Do not assert facts from inference.
7. Short tooltips: Responsibilities/Knowledge up to ~200 characters; Skills preferably shorter (~120–150); tools/product descriptions concise, researched from trustworthy official sources when available, with verifiable URL/source and no invented claims. Translation should preserve sense, not literal wording.
8. AI may **suggest or draft**, but must never silently invent, certify or overwrite candidate facts. Distinguish extracted evidence, semantic inference and candidate-confirmed answers. Use clear acceptance/review state where appropriate.
9. **Future, not now**: candidate screening via email link and conversational/voice AI confirmation, adding facts or correcting only with user input; CV versus vacancy knowledge extraction/reuse; manual visual matching and AI-assisted intelligent matching. No invitation, screening, voice, matching, or candidate-auth implementation now.
10. **Future, separate from graph**: calculate total experience durations per concept using explicit time ranges, overlap elimination, and evidence about actual period of use. A badge attached to a job does not prove full-period usage.

## CV-based visual example — Feel Flex BV

Source: user-shared one-page CV; **sample display only, not DB state**.
Timeframe: Feb 2021 — Nov 2025.
Role: Recruiter & Process Automation.

### Responsibilities
- Full-cycle Recruitment — sourcing, screening, interviews, offers and onboarding.
- Candidate Management — relocation, housing and document assistance.
- Client Coordination — coordinating clients, candidates and European partners.
- HR Administration — payroll/finance collaboration, contracts and compliance.
- Process Automation — implementing ATS integrations and eliminating duplicate entry.
- Reporting Automation — ETL/cron reporting and data-quality improvements.

### Skills
- Talent Sourcing
- Candidate Screening
- Interviewing
- Stakeholder Communication
- API Integration
- Data Synchronization
- Workflow Optimization

### Knowledge
- Recruitment Lifecycle
- HR Operations
- HRIS / ATS Workflows
- Reporting & Data Quality

### Tools & Technologies
- EasyFlex
- RecruitNow
- REST APIs
- Docker
- PostgreSQL / Prisma (appears in CV general skill set; **do not automatically link to Feel Flex without specific work-history evidence**)

### Profile-level qualifications (not automatically experience-linked)
- VCU certificate (claimed; unverified)
- Driving License B (claimed; unverified)
- Web Developer Courses (claimed; course details unverified)
- Security & Bookkeeping Courses (claimed; details unverified)

### Profile-level formal education
- Riga Business School — Economics (CV lists institution/subject, degree and completion not specified).

## Identity, reuse and UI rules

- Badge = reference to a typed canonical HIGA concept; user-facing translation/tooltip from shared catalog.
- Experience badge link = candidate, experience, concept ID, evidence snippet/source, status, optional verified usage interval; fields are conceptual only, **not DB schema approved**.
- Avoid tooltip clutter. Show Responsibilities first; Skills and Knowledge next; Tools optionally collapsed. Qualifications and Education appear in independent sections.
- Distinguish software/product names (EasyFlex, RecruitNow) from technologies/standards (REST API), and from professional skills (API integration).
- Matching can later compare graph-linked requirements and evidence without altering canonical definitions. Human review and AI matching are separate modes.
- CV parser currently outputs skills, tools, work history as structured draft; it **does not currently implement** concept linking, vacancy interpretation, screening, automatic translations, or matching as a complete pipeline. Existing parse results should not be represented as verified facts.
- No code, DB, or API changes performed as part of this document.

## Open design questions

- Whether Responsibilities become first-class HIGA concept kind or distinct reusable activities; current HIGA resolver kinds are OCCUPATION, EQUIPMENT and CREDENTIAL, so **implementation requires a deliberate later contract/schema decision**.
- How deep semantic equivalence versus specialization should be represented without over-merging.
- Candidate validation UX and status model.
- Scope of tooltip localization and trustworthiness of tool descriptions from web sources.
- Full-period versus explicit sub-period attribution for experience durations.
