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

