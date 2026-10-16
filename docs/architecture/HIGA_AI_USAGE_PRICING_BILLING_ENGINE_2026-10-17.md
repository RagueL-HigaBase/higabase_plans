# HIGA AI Usage, Dynamic Pricing & Billing Engine

**Status:** OWNER-AGREED BUSINESS DIRECTION / PROPOSED TECHNICAL DESIGN / DEFERRED IMPLEMENTATION  
**Recorded:** 2026-10-17  
**Priority:** Finish Candidate architecture and verification first. Implement this engine only afterward, in independently approved blocks.  
**Scope:** AI Orchestrator, usage metering, client/task-specific pricing, wallets or invoices, system administration, reports and multimodal services.  
**Canonical repository:** higabase_plans. No implementation, API route, migration or financial launch is approved by this MD.

**Related specifications:** [Financial Policy](../../FINANCIAL_POLICY.md), [AI Core](../../AI_CORE.md), [AI Orchestrator](AI_ORCHESTRATOR_PROPOSAL_2026-10-17.md), [System Architecture](../../SYSTEM_ARCHITECTURE.md), [Candidate Architecture](../../CANDIDATE_ARCHITECTURE.md), [Agent Rules](../../AGENT_RULES.md).

## 1. Explicit owner decisions

1. AI must be **metered by real use**; avoid unlimited, unaccounted AI consumption in a flat platform fee. Ordinary non-AI platform features remain separate.
2. Every call is attributable to the **initiating actor/user**, the **task/job** and the **billable customer/account**. One employee may use AI while their organization pays.
3. HIGA's **commercial coefficient x is configurable for each customer and AI task**, with fallback defaults by task category or client. Multipliers x5 and x10 are discussed examples, not global mandatory values.
4. Track **provider/compute cost separately from HIGA retail charges**. Allow platform admins to inspect consumption, actual/estimated cost, revenue and profitability by user, client, task, provider/model, modality and time.
5. Plan for text, CV/PDF/DOCX, photos and scans, generated images, audio, video and structured data; implement modalities in approved phases.
6. **Do not implement before Candidate is finished.** Follow Modular Monolith responsibilities and future independently scalable workers.

**Important distinction:** These are agreed commercial/product intentions. Precise schema, routes, tax, minimum spend, balance denomination, local GPU costing, role access and refund policies remain PROPOSED until explicit owner decisions.

**Supersession:** Older Financial Policy ideas of a **fixed per-CV price** and automatically included AI allowance no longer define the default billable AI model. Billing will be based on measured eligible consumption and effective client/task pricing. HIGA Credits may be a payment unit, not a substitute for detailed metering.

## 2. Boundaries and invariants

- **AI Core** interprets input and returns contract-validated output; it never owns Candidate or Organization canonical data and never receives unrestricted domain write permissions.
- **Orchestrator** owns execution/admission/queues/workers. **Usage Ledger** owns metered technical events. **Pricing Engine** selects immutable retail price rules. **Billing Ledger** settles charges and corrections. **System Admin** displays authorized aggregated telemetry.
- Technical usage and commercial payment are **different records**. Tokens are not euros, and not all photos/audio/video are billed by tokens.
- Actor, payer, source-domain ownership and data access are **different identities**. A global Candidate may be accessible to multiple agencies under separate permission. Paying to process a Candidate never grants Candidate access.
- Store media originals in immutable object storage (MinIO/S3 adapter); technical job/ledger stores only stable artifact references and metadata, not raw CVs/photos/prompts.
- Use provider-neutral named, versioned tasks and strict Zod input/output validation. Reject unauthorized, unsupported or malformed operations.
- Immutable/versioned tariffs, quote and financial entries; historic amounts are never silently recalculated after changing x.
- Provider APIs may report actual tokens after completion; while running they are estimates. A timeout may leave provider billing unknown. Never display invented zero.
- Internal retries, provider fallbacks and error recovery can cost HIGA; a customer must not silently pay twice.
- Strict account-based access and GDPR/financial record retention apply.

## 3. Reference architecture

~~~text
React / Native / API
       |
Express Auth + domain permissions + billable account resolver
       |
AI Task request -> quote / max-cost authorization / spend reservation
       |
Durable AiJob -> outbox -> Redis/BullMQ -> independent workers
       |
Versioned AiRuntime -> OpenAI / local Ollama / future adapters
       |                              |
       +------------------ Provider Usage / Job Events
                                      |
                            HIGA AI Usage Ledger
                                      |
                       Wholesale Cost / FX / rate version
                                      |
                  Client + Task Dynamic Pricing Rule Resolver
                                      |
                    Billing Settlement / Wallet or Invoice
                                      |
                   System Admin & Client Usage Analytics
~~~

Initial deployment strategy: **modular monolith** in existing Express and PostgreSQL, using existing provider-neutral AI runtime and object storage. Redis queue/workers only after approved implementation gate; no need for separate DBs per agent or client.

## 4. Accounting terms and ownership

- **Actor:** who initiated the operation (user/principal).
- **Payer:** authorized billing account (business Organization or future individual account), independent from actor identity.
- **Task:** named, versioned functional contract such as candidate.resume.parse@1.
- **Job:** one requested workflow; may include many processing steps and AI provider calls.
- **Provider call:** one external/local model request with its own usage, retries, model and timing.
- **Usage event:** normalized observed/estimated/unknown unit consumption.
- **Base cost:** measured upstream model/compute expense according to a documented rate version and cost basis.
- **Coefficient (x):** negotiated multiplier selected by payer and task.
- **Retail charge:** customer amount frozen at authorization/settlement policy.
- **Reservation:** temporary budget hold, not final spending.
- **Settlement:** actual debit after an approved outcome; **refund/credit** is a new correcting entry, never deletion.
- **Traffic:** distinguish network bytes and HTTP RPS, task/job throughput, provider tokens/media units, and retail billing.

## 5. Commercial pricing contract

**Illustrative formula (final cost basis, tax and rounding require review):**

    BaseCostEUR(job) = SUM(eligible provider/compute unit costs x frozen unit rates x FX)
    RetailNetEUR(job) = BaseCostEUR(job) x EffectiveMultiplier(payer, task, date)
    RetailGrossEUR = RetailNetEUR + applicable VAT/tax
    PayableEUR = eligible gross amount after disclosed discounts/credits, rounded at settlement

**Client tariff rule resolution, first applicable active rule wins:**
1. Exact client + exact task/version.
2. Client + task name.
3. Client + task category (CV, images, audio/video, chat, etc.).
4. Client-wide default.
5. Global exact task.
6. Global category.
7. Global default.

Use effective dates, rule ID/version, approval actor, change reason, currency, multiplier as DECIMAL, non-overlapping rules and an immutable snapshot on accepted jobs. Neither x5 nor x10 is a technical hard limit. No arbitrary per-user rule override without a separate commercial decision.

Example **only** (not observed model pricing): CV parsing cost EUR 0.01000, Company A x5 -> EUR 0.05000 net; Company B x8 -> EUR 0.08000 net. Video analysis cost EUR 0.02000, Company A x10 -> EUR 0.20000 net. VAT/fees separate. x10 implies a 90% margin **against the chosen base cost only**, not necessarily total company profit.

The customer must see an estimate or maximum spending authorization **before** an unbounded operation. Model changes or multi-call agents cannot secretly exceed the accepted spending cap. A new coefficient never changes a previous job's accepted pricing.

