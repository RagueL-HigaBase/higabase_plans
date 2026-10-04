# HigaBase System Architecture

Status: **FOUNDATION**
Last reviewed: **2026-10-04**

This document defines cross-project architectural invariants for HigaBase.

These rules apply across the backend, web frontend and other business-facing clients unless a later explicit architecture decision changes them.

Implementation repositories may add local rules, but must not contradict these invariants.

---

## 1. Core identity model

### 1.1 SystemUser

`SystemUser` is the authenticated business-system identity.

A SystemUser represents one real business user account and is identified by one unique email address.

Core invariant:

```text
1 SystemUser
→ maximum 1 business Organization
→ zero or more Locations inside that Organization
```

A SystemUser must never simultaneously belong to two unrelated business organizations.

Example — allowed:

```text
user@example.com
→ Philips
   ├─ Eindhoven
   ├─ Amsterdam
   └─ Rotterdam
```

Example — forbidden:

```text
user@example.com
→ Philips
→ Shell
```

The system must enforce this invariant server-side.

### 1.2 Organization membership

Organization membership represents the user's business-company boundary.

A business SystemUser may have at most one active Organization membership.

The implementation may use a membership table for role/status metadata, but the domain invariant remains one business Organization per SystemUser.

Platform-level roles are separate from Organization membership.

### 1.3 Location access

A user may have access to multiple Locations only when those Locations belong to the same Organization as the user.

Location access is subordinate to Organization membership.

A Location assignment must never be used to bypass the one-Organization-per-user rule.

---

## 2. Platform scope versus Organization scope

Platform administration and Organization membership are different scopes.

Examples of platform roles:
- VIEWER;
- ADMIN;
- SYSTEM_OWNER.

Examples of Organization roles:
- USER;
- ADMIN;
- OWNER.

Platform privileges do not create business membership in arbitrary Organizations.

A SYSTEM_OWNER may administer the HigaBase platform while still having at most one ordinary business Organization membership.

Authentication, authorization and scope decisions are backend-authoritative.

---

## 3. Candidate identity is a separate domain

Candidate/mobile identity is not the same entity as business `SystemUser`.

Candidate-facing applications may use separate candidate/user entities and pass candidate-domain data through APIs.

Do not merge candidate identity into SystemUser merely because the same person or email might appear in both domains.

---

## 4. Invitation domain

An invitation is not a temporary user and must not create a SystemUser before acceptance.

An invitation represents:

```text
a specific email
+ target Organization
+ target Location
+ inviter
+ intended identity details
+ secure acceptance capability
```

The invitation may exist before a SystemUser exists.

Suggested conceptual fields:

```text
Invitation
- id
- organizationId
- locationId
- email
- firstName
- middleName?
- lastName
- invitedByUserId
- tokenHash
- status
- expiresAt
- acceptedAt?
- revokedAt?
- createdAt
- updatedAt
```

Invitation lifecycle:

```text
PENDING
→ ACCEPTED

PENDING
→ REVOKED

PENDING + expiresAt < now
→ EXPIRED presentation state
```

The raw invitation token must never be stored. Persist only a cryptographic hash.

Initial invitation TTL:

```text
72 hours
```

The TTL must be configuration-driven rather than hard-coded into business logic.

---

## 5. Invitation eligibility rules

Invitation eligibility is based on the target Organization and Location, not only on whether the email exists.

### 5.1 Email does not exist

Allowed.

The invitation remains pending until accepted.

After valid acceptance, the system creates the SystemUser and its business membership.

### 5.2 Email exists and belongs to the same Organization

Allowed only when the user is not already assigned to the target Location and there is no conflicting active invitation.

The existing SystemUser is reused.

No duplicate account is created.

### 5.3 Email exists and belongs to a different Organization

Forbidden.

A single SystemUser/email cannot simultaneously enter another unrelated Organization.

The invitation must not be created.

### 5.4 User is already active in the target Location

Forbidden.

Return a stable domain error such as:

```text
ALREADY_LOCATION_MEMBER
```

### 5.5 Pending invitation already exists for the same target

Do not create a second active invitation.

Return a stable domain condition such as:

```text
INVITATION_ALREADY_PENDING
```

Resend must be handled by the existing invitation lifecycle.

---

## 6. Invitation onboarding is a separate flow

Invitation onboarding must remain separate from:

- ordinary Sign Up / Create Company Account;
- Forgot Password / Reset Password;
- email-verification recovery.

The invitation token provides the right to join the target Organization/Location.

Password setup provides account credentials.

These are different responsibilities and must not be collapsed into one flow.

Conceptual flow for a new user:

```text
Invitation email
→ Accept Invitation token
→ token validation
→ Invitation Account Setup
→ create SystemUser
→ create Organization membership
→ create target Location assignment
→ invitation ACCEPTED
→ user ACTIVE
```

Conceptual flow for an existing SystemUser in the same Organization:

```text
Invitation email
→ Accept Invitation token
→ authenticate existing SystemUser when required
→ verify invitation email == authenticated account email
→ create target Location assignment
→ invitation ACCEPTED
→ user ACTIVE for that Location
```

A public `Connect` or second generic registration route is not required.

The invitation-specific onboarding surface exists only through a valid invitation token.

---

## 7. Email and frontend validation

The frontend may perform an availability/preflight check to provide fast UX feedback.

The frontend must never be the authority for invitation eligibility.

The create-invitation backend endpoint must repeat all checks transactionally before persistence.

The client must branch on stable error codes rather than message text.

---

## 8. Membership and presentation states

Invitation state and membership state are separate concepts.

Invitation domain states:

```text
PENDING
ACCEPTED
REVOKED
EXPIRED (computed or normalized)
```

Membership/access states:

```text
ACTIVE
BLOCKED
```

A unified Team read model may present:

```text
Pending
Active
Expired
Blocked
```

without forcing invitation records and membership records into one database entity.

---

## 9. Audit and historical identity

Historical business actions must remain understandable even if a SystemUser is later removed, anonymized or deactivated.

Audit/history records must not depend only on a live foreign-key lookup for display identity.

Where a durable business record needs actor identity, store an immutable actor snapshot appropriate to that record, for example:

```text
actorUserId?
actorFirstName
actorMiddleName?
actorLastName
actorEmail?
```

The exact snapshot fields may differ by domain.

Do not duplicate snapshots indiscriminately on every table. Use them where history must survive account lifecycle changes.

Audit/history architecture will be designed separately from ordinary entity ownership.

---

## 10. Permissions architecture boundary

Detailed permissions are intentionally not finalized yet.

Current invariant:

```text
Organization membership
→ establishes company boundary

Location assignment
→ establishes where inside that Organization the user may operate

Role / permission / scope
→ establishes what the user may do there
```

Do not encode a speculative final permission matrix into the database before the product navigation and core domain skeleton are sufficiently complete.

After the invitation flow and remaining major workspace structure are established, define the permission architecture centrally in HigaBase_Plans before implementing the final permission model.

Until then:
- backend remains authoritative for all implemented security decisions;
- existing USER / ADMIN / OWNER roles remain valid current implementation facts;
- new fine-grained permissions require an explicit architecture decision;
- frontend visibility must never be treated as authorization.

---

## 11. Cross-project documentation ownership

`HigaBase_Plans` owns cross-project conceptual architecture.

Implementation repositories own:
- framework-specific rules;
- source structure;
- API implementation state;
- UI conventions;
- migration history;
- current tasks;
- verified production records.

Do not copy this entire document into implementation repositories.

Instead, implementation repositories should reference it and document only local consequences.

---

## 12. Change discipline

Any future change to these invariants must be explicit.

Especially sensitive changes:
- allowing one SystemUser to belong to multiple Organizations;
- merging candidate identity with business SystemUser;
- changing Organization/Location ownership hierarchy;
- changing invitation authorization semantics;
- mixing invitation acceptance with password recovery;
- introducing fine-grained permission persistence;
- changing historical actor/audit identity strategy.

Such changes require updating this central architecture before implementation.

---

## 13. Organization network and data-flow architecture

HigaBase organizations are structurally autonomous and symmetric.

Each Organization uses the same internal organizational model regardless of whether another Organization sees it as a client, partner or another relationship type.

Conceptually:

```text
Organization
└─ Locations
   └─ Departments
      └─ Users / assignments
```

No Organization is physically nested inside another Organization.

Cross-company cooperation is represented by relationships between autonomous Organizations.

### 13.1 Relationship perspective is directional

A relationship is interpreted from the perspective of the current Organization.

Example:

```text
Philips → Shell = PARTNER
Shell   → Philips = CLIENT
```

This does not mean two unrelated records with independent meaning.

It describes the same business connection viewed from opposite sides.

The relationship determines the allowed direction and semantics of data flow between the Organizations.

A future relationship model must preserve this complementary perspective explicitly and must not rely on UI naming alone.

### 13.2 Relationships form a graph, not an ownership tree

Internal Organization structure is hierarchical.

External Organization relationships form a graph.

Conceptually:

```text
INTERNAL
Organization
└─ Location
   └─ Department

EXTERNAL
Organization A ↔ Organization B ↔ Organization C
```

This graph may grow recursively without changing the underlying domain model.

Adding another client or partner must create another relationship edge, not a new special-case architecture.

The same routing rules must continue to work regardless of how many Organizations participate in the network.

### 13.3 Relationship determines direction

Relationship type determines what kind of business exchange exists and in which direction data may move.

Examples include:
- receiving candidate data;
- sending candidate data;
- receiving requests;
- sending vacancies;
- returning processed candidate or request outcomes.

The architecture must not assume that every relationship is bidirectional for every domain object.

A future routing layer must be able to express allowed inbound and outbound flows per relationship.

---

## 14. Department as data scope

Departments do not define a separate external relationship.

Departments define the internal scope of data exposed through an existing Organization relationship.

Conceptually:

```text
Organization Relationship
        ↓
allowed Location scope
        ↓
allowed Department scope
        ↓
allowed domain data
```

Example:

```text
Philips ↔ Shell
relationship = CLIENT / PARTNER

Shell exposes:
- Location A
  - Recruitment Department
  - Planning Department

Only Recruitment may participate in the relationship.
```

In this case, the Organization relationship exists at Organization level, while Department assignment limits the data volume visible or transferable through that relationship.

The same concept applies to Locations.

A future routing policy may therefore scope a relationship by:
- Organization;
- one or more Locations;
- one or more Departments;
- one or more domain flows.

This scope controls data exposure and routing.

It is not the same thing as user permission.

---

## 15. Core domain flows

The initial cross-organization business flows are conceptually separate domains:

```text
Candidate
Vacancy
Request
```

These domains must remain independent business entities even when they travel through the same relationship network.

A routing layer may determine whether a specific relationship permits:

```text
Candidate → outbound
Candidate ← inbound

Vacancy → outbound
Vacancy ← inbound

Request → outbound
Request ← inbound
```

The same Organization relationship can therefore support different directions for different domain objects.

Do not collapse Candidate, Vacancy and Request into one generic persisted "business object" merely because they share routing infrastructure.

Shared routing infrastructure may operate across them, but their domain ownership and lifecycle remain separate.

---

## 16. Apps and capability modules

Apps are capability modules attached to the HigaBase platform.

An App is not an Organization membership and is not a data owner.

An App may:
- execute a defined business operation;
- display or transform existing domain data;
- integrate an external product or service through API;
- remain entirely internal to one Organization;
- exchange data with external systems when explicitly configured;
- be free or paid.

Examples may include:
- recruiting functionality;
- planning/scheduling integrations;
- workforce-management systems;
- external products such as Planbition, FullFlex or NoCore.

Each App must have a clear contract defining:
- which domain data it may read;
- which domain data it may create or modify;
- whether it may send data outside the Organization;
- which Organization/Location/Department scope it operates in.

Apps must not silently create new cross-Organization access paths.

Any external data exchange must still respect the Organization relationship and routing architecture.

---

## 17. User as actor, not owner

A SystemUser is an operational actor.

A SystemUser is not the durable owner of business data.

Conceptually:

```text
Organization owns business context
Domain entity owns its lifecycle
User performs an action
```

A user may:
- create;
- edit;
- review;
- approve;
- route;
- send;
- receive;
- assign;
- manage

a domain object when authorized.

The business object must remain valid when that user:
- leaves the company;
- is blocked;
- is deactivated;
- is deleted;
- is replaced by another employee.

Do not model core business ownership in a way that requires a live user account for the object to continue existing.

User references may identify:
- current assignee;
- current responsible actor;
- creator;
- last editor;
- approver;
- historical actor.

These relationships are operational metadata, not durable ownership.

---

## 18. Assignment and replacement principle

Operational responsibility may move from one user to another without moving or recreating the underlying business object.

Example:

```text
Vacancy / Candidate / Request / Task
        ↓
assigned actor = User A

User A leaves
        ↓
assigned actor = User B
```

The domain entity remains the same.

Its Organization, Location, Department and relationship context do not change merely because the responsible user changes.

This principle is mandatory for long-lived business data.

---

## 19. Data consistency across user lifecycle

User lifecycle changes must not destroy business continuity.

When a SystemUser is removed or deactivated:
- Organization-owned data remains;
- domain entities remain;
- relationship/routing context remains;
- historical actions remain interpretable;
- active assignments may be reassigned;
- audit records preserve actor identity snapshots where required.

Foreign keys to users must therefore be chosen by purpose.

Use:
- nullable/set-null references when the relationship is historical or optional;
- restricted references when deletion must be prevented;
- immutable actor snapshots when human-readable history must survive user removal.

Do not use cascading user deletion for durable business-domain records unless that record is genuinely user-owned and disposable.

---

## 20. Separation of architecture layers

The system must keep these concerns separate:

```text
OWNERSHIP
Who owns the data?
→ Organization / domain entity

STRUCTURE
Where does it live internally?
→ Location / Department

RELATIONSHIP
Which Organizations are connected?
→ Client / Partner / future relationship types

ROUTING
What data may move and in which direction?
→ Candidate / Vacancy / Request flows

APPLICATION CAPABILITY
Which module performs or exposes operations?
→ Apps

AUTHORIZATION
What may this specific user do?
→ Roles / permissions / scopes

ACTOR
Who performed the operation?
→ SystemUser
```

These layers may reference each other but must not be collapsed into one table or one permission concept.

This separation is a core architectural invariant.

---

## 21. Permissions dependency

Final fine-grained permissions must be designed only after the following are sufficiently stable:

1. Organization / Location / Department structure;
2. invitation and membership model;
3. Client / Partner relationship model;
4. Candidate / Vacancy / Request domain boundaries;
5. relationship routing rules;
6. App capability boundaries.

Permissions will then answer:

```text
Which user
may perform which action
on which domain object
inside which Organization / Location / Department scope
through which relationship or App context?
```

Until that architecture exists, do not prematurely encode a large permanent permission matrix into the database.

---

## 22. Create Once, Use Forever

A core HigaBase design principle is:

```text
CREATE ONCE
USE FOREVER
```

Business data should be created once at its authoritative source and reused throughout the system.

The architecture must avoid repeated manual recreation of the same semantic information at every downstream step.

Examples:
- a Vacancy description is created once and reused by Requests;
- Organization identity is created once and reused throughout its Locations and flows;
- Candidate profile data is created once and exposed through permitted views;
- relationship routing is configured once and reused by subsequent domain flows.

The system should move references, scope and visibility rules whenever possible rather than forcing users to reproduce the same business data.

---

## 23. Source-of-truth propagation

Every shared domain object has an authoritative source.

The initiating Organization remains the source of truth for the fields it owns.

Conceptually:

```text
Initiator
   ↓
Authoritative Domain Object
   ↓
Relationship / Scope / Visibility
   ↓
Downstream presentation
```

If the initiator changes an authoritative field, all downstream live representations that are allowed to expose that field must resolve the updated value automatically.

Example:

```text
Factory creates Vacancy
salary = 20.00

Factory changes salary = 21.50

Every active downstream view
that is allowed to see salary
must resolve 21.50.
```

Downstream Organizations must not fork or silently overwrite the initiator-owned source field.

This provides one semantic source of truth across the flow.

---

## 24. Visibility, not duplication

Cross-Organization flow is primarily controlled by visibility rules.

For every relevant field or block, the routing context determines whether it is:

```text
VISIBLE
or
INVISIBLE
```

A downstream participant may see only the subset permitted by:
- Organization relationship;
- Location scope;
- Department scope;
- domain-flow configuration;
- future permission policy;
- App capability contract.

Invisible data remains at the authoritative source and is not exposed downstream.

The system should not create unnecessary copied records merely to hide fields.

Where practical, downstream views should resolve the authoritative source through a controlled projection.

---

## 25. Live data versus historical snapshots

Automatic propagation applies to live business state.

Historical evidence must remain historically correct.

Therefore distinguish:

```text
LIVE VIEW
→ resolves current authoritative values

HISTORICAL SNAPSHOT / EVENT
→ preserves values that were effective at that historical moment
```

Examples:
- current Vacancy view may immediately show a newly changed salary;
- a signed agreement, submitted offer, invoice, audit entry or historical decision may retain a snapshot of the previous value.

Create Once, Use Forever does not mean rewriting history.

The domain must explicitly decide where live propagation is required and where immutable snapshots are required.

---

## 26. Flow control points

A relationship flow may contain control points.

A control point can decide whether data:
- continues;
- stops;
- becomes visible;
- becomes hidden;
- changes destination;
- requires approval;
- requires additional data;
- activates another domain process.

Conceptually:

```text
Source
  ↓
Relationship
  ↓
Control Point
  ├─ continue
  ├─ stop
  ├─ expose selected data
  └─ route to another destination
```

Control points belong to flow configuration and domain workflow.

They are not user ownership.

Users may operate a control point when authorized, but the configured flow exists independently of the individual user account.

---

## 27. Vacancy as reusable position definition

A Vacancy should represent a reusable description of a position or role.

It may contain stable position-level information such as:
- title;
- description;
- requirements;
- working conditions;
- compensation information;
- skills;
- certifications;
- Location;
- Department;
- other domain-specific attributes.

The Vacancy should be created once and reused.

A Request should not duplicate the complete Vacancy definition.

Conceptually:

```text
Vacancy
= what the position is

Request
= how much / when / under what immediate demand
```

---

## 28. Request as lightweight demand activation

A Request is a demand event connected to an existing Vacancy and organizational scope.

Typical Request-specific data may include:
- requested quantity;
- requested start date;
- urgency;
- shift or time-specific demand;
- temporary request-specific conditions;
- lifecycle/status.

Conceptually:

```text
Organization
  ↓
Location
  ↓
Department
  ↓
Vacancy
  ↓
Request
```

A user should be able to activate a Request from an existing Vacancy with minimal additional input.

Example:

```text
Vacancy:
Warehouse Operator

Request:
quantity = 15
startDate = 2026-11-01
```

The Request resolves the position description from the linked Vacancy rather than copying the full description.

If the authoritative Vacancy changes, active Requests and downstream live views resolve the updated Vacancy data unless a specific workflow requires a historical snapshot.

---

## 29. Vacancy and Request routing

Vacancies and Requests participate in relationship flows.

The routing configuration determines:
- which relationship receives the data;
- in which direction;
- which Location scope is included;
- which Department scope is included;
- which fields are visible;
- which control points apply.

A configured flow may allow one Organization to:

```text
create Vacancy
→ activate Request
→ route Request to Partner
→ Partner continues flow
→ downstream Organization receives permitted data
```

The same Vacancy may support multiple Requests over time without recreating the Vacancy.

The same relationship routing may be reused for future Requests.

---

## 30. Minimal-input principle

When the system already knows a fact from authoritative domain context, the user should not have to enter it again.

Examples:
- Request reuses Vacancy description;
- Request reuses Department and Location context;
- downstream flow reuses configured Organization relationship;
- App reuses its configured scope;
- invitation reuses Organization and Location target;
- user assignment reuses existing domain object rather than recreating it.

New input should be limited to genuinely new business information.

This principle reduces:
- duplication;
- inconsistent data;
- user error;
- maintenance cost;
- reconciliation work.

---

## 31. Flow continuation and flow initiation

A domain participant may either initiate a new flow or continue an existing one.

Conceptually:

```text
INITIATE
→ create authoritative domain object or demand

CONTINUE
→ receive permitted object/context
→ apply allowed local operation
→ route onward
```

Both operations use the same relationship/routing architecture.

The system must not require separate special-case architectures for:
- the original initiator;
- an intermediary Organization;
- a downstream partner.

The participant's position in the graph determines its role in the current flow.

---

## 32. Reusable routing architecture

Configured routing must be reusable.

Once a relationship and its scope are configured, future eligible domain objects should be able to use that route without rebuilding the flow manually.

Conceptually:

```text
Relationship configured once
+ Scope configured once
+ Visibility configured once
+ Control points configured once
= reusable domain pipeline
```

New Vacancies, Requests, Candidates or supported future domain objects can then enter that pipeline when activated.

This is a central scalability property of HigaBase.

---

## 33. ESCO as semantic source of truth

ESCO is the canonical semantic layer for occupations, skills and related professional concepts used by HigaBase.

HigaBase domain entities must not independently invent competing occupation or skill vocabularies when an applicable ESCO concept exists.

Conceptually:

```text
Free text / external data / user input
        ↓
Normalization
        ↓
ESCO concepts
        ↓
HigaBase domain usage
```

The normalized ESCO layer is therefore shared by:
- Vacancies;
- Candidate profiles;
- matching;
- AI-assisted parsing;
- skill comparison;
- occupation comparison;
- future recommendation and routing logic.

The existing Higa ESCO normalized dataset remains a separate reference domain and Higa business entities reference normalized `EscoConcept` identities rather than duplicating ESCO data.

---

## 34. Vacancy normalization through ESCO

A Vacancy may contain human-readable business text, but its professional meaning should be normalized into ESCO concepts.

Conceptually:

```text
Vacancy description
+ requirements
+ responsibilities
        ↓
AI / deterministic extraction
        ↓
ESCO occupations
ESCO skills
ESCO relations
        ↓
Structured Vacancy profile
```

The original business text remains available.

ESCO normalization does not replace the original description; it provides the structured semantic layer used by the system.

A Vacancy should therefore have both:

```text
HUMAN VIEW
→ readable job description

SEMANTIC VIEW
→ normalized ESCO profile
```

This allows the same Vacancy to be reused across Requests, matching, relationship routing and Apps without repeatedly interpreting its meaning.

---

## 35. Candidate normalization through ESCO

Candidate information may originate from:
- CV upload;
- document parsing;
- manual profile entry;
- recruiter input;
- voice input;
- external systems.

Regardless of source, professional experience and skills should be normalized against the same ESCO reference layer used by Vacancies.

Conceptually:

```text
Candidate source data
        ↓
AI-assisted interpretation
        ↓
ESCO normalization
        ↓
Structured Candidate profile
```

AI may propose mappings.

AI is not the source of truth for occupation or skill identity.

The selected normalized ESCO concepts are the semantic reference used by downstream matching.

---

## 36. Matching on a common semantic layer

Candidate-to-Vacancy matching should compare structured profiles built on the same ESCO vocabulary.

Conceptually:

```text
Candidate ESCO profile
        ↘
         Matching Engine
        ↗
Vacancy ESCO profile
```

This enables explainable comparison of:
- occupation fit;
- required skills;
- possessed skills;
- missing skills;
- transferable skills;
- related occupations;
- relevant certifications or future additional domain signals.

A match result may expose a percentage or score, but the score must be supported by explainable contributing factors.

The system should be able to answer not only:

```text
Match = 82%
```

but also:

```text
why 82%?
which required skills matched?
which skills are missing?
which related ESCO concepts contributed?
```

---

## 37. AI role in semantic processing

AI is an interpretation and assistance layer.

AI may:
- parse unstructured CVs;
- interpret recruiter/user descriptions;
- propose occupation mappings;
- propose skill mappings;
- summarize candidate or Vacancy information;
- identify likely ESCO concepts;
- generate human-readable explanations of matching results.

AI must not silently redefine canonical ESCO concepts.

Where confidence is insufficient or mappings are ambiguous, the system should expose the uncertainty to an authorized human checkpoint rather than fabricate certainty.

---

## 38. Automated flow with human checkpoints

HigaBase should automate the routine flow and reserve human attention for decision points.

Conceptually:

```text
Input
↓
Normalize
↓
Evaluate
↓
Route automatically
↓
CHECKPOINT
├─ approve / continue
├─ decline
├─ request clarification
├─ reroute
└─ stop
↓
continue automated flow
```

Checkpoints should exist only where a business decision, legal decision, quality decision or explicit responsibility requires human confirmation.

The system should prepare the decision by presenting:
- source data;
- normalized data;
- match result;
- missing or conflicting information;
- recommended next action;
- relevant provenance/context.

The responsible user should not need to manually reconstruct information already available to the system.

---

## 39. Minimal-human-effort principle

The default operating principle is:

```text
SYSTEM DOES THE WORK
HUMAN CONFIRMS EXCEPTIONS AND DECISIONS
```

Users should not spend time on:
- repeated data entry;
- manually comparing data the system can normalize;
- manually moving objects through an already configured route;
- rebuilding the same Vacancy/Request/Candidate context;
- searching for information the platform already owns.

Human effort should concentrate on:
- approval;
- rejection;
- exception handling;
- judgment where automation confidence is insufficient;
- relationship/customer decisions;
- legally or commercially sensitive checkpoints.

---

## 40. Checkpoint outcomes

A generic workflow checkpoint may conceptually support outcomes such as:

```text
CONTINUE
DECLINE
HOLD
REQUEST_INFORMATION
REROUTE
STOP
```

The exact allowed outcomes depend on the domain.

For example, a recruitment matching checkpoint might allow:

```text
SHORTLIST
DECLINE
REQUEST_REVIEW
FORWARD
```

A checkpoint is part of domain workflow configuration.

It must not be confused with ownership or user identity.

An authorized user acts on the checkpoint; the workflow remains owned by the domain/Organization.

---

## 41. Explainability and provenance

Automated decisions and recommendations must retain enough provenance to understand their origin.

For ESCO-based matching, provenance may include:
- source Candidate data;
- source Vacancy data;
- normalized ESCO concepts;
- mapping confidence;
- matching factors;
- rule/version used;
- AI-assisted interpretations;
- human overrides or confirmations.

This supports:
- quality review;
- debugging;
- auditability;
- future model improvement;
- user trust.

The system should never reduce a complex matching decision to an unexplained score when the contributing structured data is available.

---

## 42. ESCO inside Create Once, Use Forever

ESCO normalization follows the same Create Once, Use Forever principle.

A normalized skill or occupation mapping should be created once for the authoritative domain object and reused downstream.

Example:

```text
Vacancy
→ normalized once to ESCO
→ Request reuses Vacancy ESCO profile
→ Partner view reuses permitted ESCO profile
→ Matching reuses same ESCO profile
→ App integrations reuse same semantic structure
```

If the authoritative Vacancy changes materially, the ESCO normalization may be recalculated and the updated semantic profile propagates through live downstream views.

Historical decisions may retain the ESCO snapshot/version that was used at the time of that decision.

---

## 43. Data contract boundary

Every external or internal integration must cross a defined Higa data contract boundary.

External systems must not write arbitrary payload structures directly into Higa domain persistence.

Conceptually:

```text
External Source
      ↓
Adapter
      ↓
Higa Data Contract
      ↓
Validation
      ↓
Normalization
      ↓
Consistency Evaluation
      ↓
Canonical Domain Model
```

Adapters are responsible for translating source-specific formats into Higa contracts.

Domain logic must depend on canonical Higa contracts rather than on provider-specific payload shapes.

This allows integrations to change independently without contaminating the core domain model.

---

## 44. Canonical data domains

Consistency rules should be defined by semantic data domain, not repeated independently for every provider.

Examples include:
- personal identity;
- contact data;
- employment data;
- Organization / Location / Department assignment;
- financial and commercial data;
- Vacancy data;
- Request data;
- Candidate data;
- ESCO occupation and skill data;
- planning/scheduling data;
- documents and certifications.

Different APIs may provide the same semantic information.

After adaptation, that information must be evaluated against the same canonical Higa rules.

---

## 45. Source authority

For every data category or field where multiple systems can provide values, Higa must be able to define source authority.

Conceptually:

```text
Field / Domain
→ authoritative source
→ secondary sources
→ conflict policy
```

Examples:

```text
Identity data
→ Higa identity domain may be authoritative

Planning schedule
→ configured planning integration may be authoritative

Payroll identifier
→ configured payroll system may be authoritative

Client Request
→ originating client Organization may be authoritative
```

A difference does not automatically mean that incoming data should overwrite the current value.

The system must understand whether the incoming source is authoritative for that information.

---

## 46. Consistency evaluation

Incoming data should be classified before it affects canonical state.

Conceptual outcomes:

```text
SAME
→ no action

NORMALIZABLE
→ deterministic normalization
→ no human attention required

VALID_UPDATE
→ source is authoritative and update is allowed

CONFLICT
→ values differ and policy cannot safely resolve

MISSING_REQUIRED
→ required information is absent

INVALID
→ violates canonical contract

UNCERTAIN
→ semantic interpretation is insufficiently confident
```

Only outcomes requiring business judgment or unresolved ambiguity should create a human checkpoint.

---

## 47. Automatic normalization

Deterministic differences should be resolved automatically where safe.

Examples may include:
- phone formatting;
- whitespace;
- casing where semantics are unchanged;
- canonical country codes;
- normalized date formats;
- stable identifier formatting;
- known external enum mappings.

Normalization must not silently change semantic meaning.

A normalization rule belongs to the canonical data contract, not to an individual user's manual workflow.

---

## 48. Conflict detection and anomaly feed

The system should actively detect data inconsistencies and surface only unresolved anomalies.

Users should not manually inspect every successful data exchange.

Conceptually:

```text
Large data volume
      ↓
automatic validation / normalization / comparison
      ↓
most records pass silently
      ↓
small anomaly set
      ↓
human attention
```

The user-facing experience should focus on:
- conflicting values;
- missing required information;
- invalid values;
- uncertain mappings;
- incompatible domain relationships;
- failed integration assumptions.

This anomaly feed is an operational surface, not a replacement for canonical domain state.

---

## 49. AI in data consistency

AI may assist with:
- parsing;
- semantic comparison;
- anomaly detection;
- explanation;
- summarization;
- identifying likely contradictions;
- proposing ESCO mappings;
- preparing human-readable difference summaries.

AI must not:
- approve a lifecycle transition;
- grant access;
- change permissions;
- publish externally;
- accept an Organization relationship;
- override an authoritative source;
- silently resolve a business conflict when policy requires human approval.

AI detects and explains uncertainty.

Workflow enforces process.

Humans own business decisions when a checkpoint requires them.

---

## 50. Difference review contract

When human review is required, the system should present the minimum information necessary to make the decision.

A review surface should be able to show:

```text
CURRENT VALUE
INCOMING VALUE
SOURCE
AUTHORITY
DIFFERENCE
IMPACT
AI EXPLANATION (optional)
```

Possible human outcomes depend on domain policy, for example:

```text
ACCEPT
KEEP_CURRENT
IGNORE
FIX
REQUEST_INFORMATION
DECLINE
STOP_FLOW
CONTINUE
```

The UI must not force users to inspect unrelated fields when only a small subset changed.

---

## 51. Dirty-data containment

Provider-specific dirty data must be contained at the system boundary.

The canonical Higa domain must not accumulate:
- provider-specific field names;
- duplicate semantic fields;
- uncontrolled free-form enums;
- unvalidated identifiers;
- arbitrary provider status values;
- contradictory copies of authoritative facts.

External complexity belongs in adapters and mapping configuration.

Internal domain models remain canonical and predictable.

---

## 52. Observability without manual monitoring

The system, not the user, is responsible for continuously observing configured data flows.

Operational users should not need to watch integrations manually.

The platform should detect and classify:
- failed imports;
- contract violations;
- data drift;
- stale mappings;
- source conflicts;
- unexpected deletions;
- missing required relationships;
- semantic inconsistencies.

Only actionable exceptions should require user attention.

---

## 53. Workflow independence from personnel

Configured flows, consistency rules and integration contracts belong to the Organization/system, not to individual users.

A user may operate a checkpoint or resolve an anomaly when authorized.

If that user:
- leaves;
- is absent;
- is blocked;
- changes Department;
- is replaced;

the configured flow must continue unchanged.

Only responsibility/assignment changes.

This is a mandatory consistency invariant.

---

## 54. Operational architecture summary

The core operational pipeline is:

```text
SOURCE
→ ADAPTER
→ DATA CONTRACT
→ VALIDATE
→ NORMALIZE
→ COMPARE
→ AUTHORITY CHECK
→ CONSISTENCY RESULT
→ ROUTING / WORKFLOW
→ CHECKPOINT ONLY IF REQUIRED
→ CONTINUE
```

AI may assist inside parsing, normalization, comparison and explanation.

AI does not own the pipeline.

The pipeline remains deterministic, auditable and Organization-controlled.

