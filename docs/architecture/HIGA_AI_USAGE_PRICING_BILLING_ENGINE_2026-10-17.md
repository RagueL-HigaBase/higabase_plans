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


## 6. Multimodal usage units and measurement accuracy

| Modality | Observed technical units | Notes |
|---|---|---|
| Text and chat | input tokens, output tokens, optional cached and reasoning subcounts | cached is included within input; reasoning is included within output; never add twice |
| CV/PDF/DOCX | file bytes/pages + tokens/usage of each AI call | uploading a PDF is not itself a known token count |
| Photo/image inspection | dimensions, detail settings, image tokens or provider-native units | rate and usage depend on model and resolution |
| Image generation/edit | generated count, size, quality, model-specific charged units | may not be token-denominated |
| Audio transcription/analysis | input duration, transcript tokens and provider-native audio units | distinguish transcription from subsequent reasoning |
| Audio generation | generated duration or provider-native audio units | separate from input audio |
| Video | video duration, extracted frames, audio tracks, model-native units | frame sampling, multiple models and preprocessing vary widely |
| Structured input | data size and model input/output usage | model usage measured separately from traffic |
| Local inference | available token counts + audited CPU/GPU time/utilization | requires an internal transfer-price rate; no fictional provider invoice |

Capture each provider request/step's actual usage where returned, along with model/provider, rate version, relevant source artifact ID, tenant/user attribution, duration, retry index, exact/estimated/unknown status. Avoid storing full sensitive payloads in the ledger.

**Real-time minute metric:** distinguish (a) tokens attributable to provider calls **completed in that minute**, (b) estimated tokens of active calls and quota reservations, and (c) actual provider RPM/TPM caps. Aggregated completed-minute token totals are exact *for captured calls with provider-reported usage*, but not literal per-second emission and not automatically equal to upstream billing. Track missing responses and a completeness percentage.

**Per-task cost:** sum all observed underlying calls and other approved cost units. A job's one user action can produce multiple model requests and technical steps. Network bandwidth, storage and provider computation are different expenses; adding byte/egress fees to customer billing requires a separately published client tariff.

## 7. Rate catalog, cost accounting and margin

Proposed cost/rate record includes provider, model, usage unit, unit rate DECIMAL, currency, plan/quality/cache/batch variant, validFrom/To, reviewedBy, source provenance and provider pricing version. Freeze a rate snapshot per event and store FX quote/source/time where applicable. Never backdate changes to settled charges.

Keep three figures distinct:
1. **Metered wholesale cost**: price of observed external provider usage or audited internal compute units.
2. **Retail customer revenue**: retail job price under frozen client + task x, separate net/tax/gross.
3. **Actual contribution margin**: revenue minus provider spend and other tracked variable costs, with separately disclosed infrastructure overhead.

Recommended initial cost basis: multiply **provider measured variable charges only** for cloud workloads, but expose extra infra/media expenses in admin margin. Whether an expanded landed cost is multiplied is **not decided**. For self-hosted models, adopt a reviewed compute-cost basis before selling usage; zero cloud spend does not mean zero AI cost.

Maintain wholesale cost for failed calls and HIGA retries independently from customer-billable usage, and reconcile periodic platform totals with provider billing. Difference may be due to unknown responses, discounts, provider fees, billing periods, or FX.

## 8. Financial ledger, balances and settlement

**Two separately audited records:**
- **AI Usage/Cost Ledger:** call/step, user, authorized payer, task, reported units, provider/model, cost rate version, estimation/reconciliation status and infrastructure attribution.
- **Customer Billing Ledger:** accepted quote, active client pricing rule, reserve, release, charge, tax, credits/refunds, payment and invoice links.

Proposed prepaid workflow:
1. Validate actor, domain access, billable account authorization, named task and enabled rule.
2. Calculate conservative quote/max spend (output/media bounds). Show to client as required by agreement.
3. Atomically reserve the authorized amount on that payer's wallet/budget and enforce concurrency/budget limits.
4. Execute bounded job; collect provider-native usage and validate the resulting domain draft.
5. Settle once using immutable retail price snapshot; release unused hold. No unauthorized negative balance.
6. If user cancels or HIGA/provider fails, follow published failure/partial-result policy; issue correcting ledger entries rather than deleting transactions.
7. Export invoice/statement as required, with separate VAT/tax and source operation references.

Alternative enterprise postpaid invoices/credit lines are possible, but need approved credit limits and collection/invoicing rules. Client tokens should not be bank transactions: aggregate micro-consumption in internal precise EUR balances (or optionally explicit HIGA Credits). Final money/tax amount rounding must be deterministic; do not round every sub-cent model call up to €0.01. Use high-precision decimals or integer subunits; NEVER binary floating-point as accounting authority.

Use append-only journal transactions or balanced accounting postings with unique charge IDs. Strong idempotency on reserve, settle, release, payment, refund and manual adjustments. Never modify settled history when tariffs change. Any manual correction needs reason, authorized actor and immutable audit trail.

**Not yet resolved:** EUR wallet versus HIGA Credits; payment provider; tax jurisdiction; refunds and partial-results charge policy; thresholds for user confirmation; prepaid-only vs enterprise credit; treatment of explicitly funded promotional allowances.

## 9. Failures, retries and customer fairness

| Event | HIGA technical ledger | Customer billing proposal |
|---|---|---|
| Successful completed task | Record every eligible measured provider call | Settle usage at accepted price, under approved max |
| Invalid/unauthorized task before provider | Record rejected attempt; no provider usage | No charge, release hold |
| Internal HIGA retry/fallback | Record real extra technical cost | Do not double-charge invisible retries |
| Provider rate limit 429 | Record error/retry | No invented usage; wait/backoff as allowed |
| Provider timeout, unknown upstream usage | UNKNOWN or pending reconciliation | No guessed final charge |
| User cancels before dispatch | No provider call | Release reservation |
| User cancels during processing | Preserve partial measured calls | Follow separately agreed disclosed policy |
| Failed strict output validation/unusable result | Technical cost remains visible | Default recommendation: do not charge for unusable result |
| User voluntarily submits a new paid task | New independent quote and job | Separately bill if approved |
| Duplicate HTTP submit with same idempotency key | Reuse existing job | Never debit twice |

External provider may charge for a timed-out call, so **exactly-once API provider billing is not guaranteed**. HIGA financial settlement can and must be idempotent. Unknown quantities are neither zero nor precise billing evidence. Customer disputes/credits require scoped review and reason.

## 10. Proposed technical schema, NOT Prisma migrations

Integrate with Orchestrator's AiJob, AiJobStep, AiProviderCall and AiOutbox; do not create duplicate business Candidate or Organization tables.

| Candidate model | Key attributes / responsibility |
|---|---|
| AiUsageMeterEvent | jobId, callId/stepId, actorId, payerId, task, time, modality, unit, quantity DECIMAL, source accuracy, dedupe key |
| AiWholesaleRateVersion | provider/model/unit/currency/rate, applicability, effective dates, pricing provenance |
| AiCostAllocation | meterEvent, rate snapshot, FX, baseCost, known/estimated/unknown, reconciliation |
| AiBillingAccount | payer type/id, currency, billing mode, status, budget policy |
| AiClientPricingRule | payer/category/task/version selector, multiplier DECIMAL, effective period, author/approver, state |
| AiPriceQuoteSnapshot | accepted job/payer/task, resolved rule/version/x, wholesale cost basis, approved cap, currency/tax/FX |
| AiSpendReservation | payer/job/quote, amount, status, expiry, hold/release reference |
| AiCustomerCharge | unique job settlement, net/tax/gross, job/call attribution, price snapshot, status |
| AiFinancialEntry | append-only balanced debit/credit/refund/payment/correction event, reason and actor |
| AiBillingAuditEvent | old/new rule versions, who/why/when, permission scope |
| AiUsageAggregate | rebuildable minute/hour/day scoped rollups, provider/job/user/client/modality/task, coverage flags |

Index payer/time, actor/time, task/time, jobId and external provider call keys. Unique idempotency constraints for single job settlement and meter event ingestion. Use PostgreSQL transactions to protect wallet/ledger invariants; coordinate with Redis/outbox for distributed execution but do not treat queue insertion as a DB transaction.

## 11. Client and actor authorization boundaries

- Each authenticated AI submission must resolve a **payer** with spend rights. User account is not necessarily a wallet. Organization employees can share the corporate payer while retaining individual usage attribution.
- A candidate/mobile identity is not the same entity as SystemUser. Any future candidate billing identity needs separate approved mapping; never merge accounts implicitly.
- Existing SystemUser organization membership limits remain authoritative; AI billing never grants or changes Organization membership.
- Global Candidate may be accessible to no organization or multiple authorized agencies. A technical payer association **does not grant or imply candidate ownership, consent, read, modify or share permission**.
- System/finance owner can see platform commercial aggregates and scoped details; organization admin sees only their company; actor sees only their own consumption as allowed. Server enforces scope on each query/filter, not React alone.
- Financial event keys and telemetry must not contain raw prompts, CV contents, media, credentials, health details or unnecessary personal data.

