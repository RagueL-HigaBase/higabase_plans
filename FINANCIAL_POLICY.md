# HigaBase Financial Policy

Status: DRAFT / planning note  
Last reviewed: 2026-10-06

This note records the current pricing direction for HigaBase. It is not a final public price list.

## Core direction

- HigaBase should compete on quality, automation and operational value, not on being the cheapest ATS.
- Base pricing should be primarily company/subscription based, not per user seat.
- Normal platform usage belongs inside the base subscription.
- Optional operational capabilities may be sold as fixed-price monthly modules.
- Variable-cost AI operations require a separate usage boundary so unlimited AI use is not bundled into a low fixed subscription.
- Customer-facing pricing must remain independent from any single AI provider or model.

## Base subscription

The base subscription should provide a genuinely usable product.

Current direction:
- no primary per-user pricing;
- a company may have many users without paying for every extra account;
- ordinary navigation, candidate/vacancy work and standard platform features belong to the subscription.

## Optional modules

Examples:
- Housing;
- Transport;
- advanced analytics;
- specialist operational modules;
- selected integrations;
- future premium automation packages.

Modules should normally use clear fixed monthly pricing instead of per-click billing.

## Integrations

Preferred default:

implementation/setup -> normally no separate development fee  
continued maintained access -> recurring monthly fee

Where possible, an integration created for one customer becomes a reusable HigaBase connector that can later be offered to other customers using the same external system.

Exceptional bespoke work may require a separate agreement.

## AI and variable-cost automation

Metered limits are appropriate where HigaBase has a real variable cost.

Primary examples:
- CV/resume parsing;
- high-volume AI document processing;
- AI generation/analysis;
- future automation whose cost scales directly with usage.

The subscription may include a reasonable allowance. Additional capacity can be purchased after that allowance is used.

## Internal credits / units

HigaBase may introduce an internal commercial unit such as Higa Credits, Base Credits, or another final name.

This is a Higa billing unit, not an OpenAI token.

Concept:

EUR payment -> Higa balance -> eligible metered operations consume fixed Higa units

The customer should see stable Higa pricing even if Higa changes AI provider/model internally.

## CV parsing

CV parsing is a primary metered case because cost scales with the number of documents processed.

Direction:
- include a reasonable allowance with the relevant subscription;
- consume a fixed Higa amount per successfully processed CV;
- allow additional capacity to be purchased;
- internal retries/model escalation are Higa's responsibility and should not double-charge the customer;
- no silent negative balance;
- exact limits and prices are decided later from real benchmark data.

## Vacancy publishing / promotion

Ordinary vacancy creation can remain inside the platform subscription.

Paid promotion/distribution or other externally billable vacancy actions may use a separate balance or credit boundary.

Third-party advertising/media spend should remain distinguishable from the Higa platform fee.

## Transparency

For metered capabilities, the customer should be able to understand:
- remaining included allowance / balance;
- operation cost where relevant;
- warnings before exhaustion;
- how to buy more capacity;
- usage history.

Default principle: no hidden overage and no surprise invoice.

## Internal cost ledger

HigaBase should internally track enough data to understand margin:
- operation type;
- provider/model where relevant;
- provider usage/cost;
- retries/fallbacks;
- material infrastructure/storage cost;
- customer-facing credits/revenue.

This internal cost data is separate from customer-facing pricing.

## Commercial structure

Base subscription
+ optional fixed-price modules
+ included AI/automation allowance
+ optional additional Higa credits/capacity
+ recurring integration subscriptions
+ clearly separated third-party variable spend

## Open decisions

Still to be decided:
- final name of the internal unit;
- EUR-to-unit conversion;
- base subscription price;
- included CV allowance;
- CV parsing unit price;
- vacancy promotion model;
- module prices;
- integration price bands;
- enterprise/high-volume rules.

These values should be based on real Higa cost benchmarks and early customer usage.


## Local / self-hosted AI direction

HigaBase should not be architecturally tied to one external AI provider.

Preferred direction:
- keep the Higa AI Core provider-agnostic;
- support cloud providers and local/self-hosted inference behind the same application-level contract;
- use local models where they provide sufficient quality, privacy, reliability or cost control;
- keep stronger cloud models available for escalation when local confidence or benchmark quality is insufficient;
- do not expose provider/model choice as part of the customer-facing billing model.

Initial development benchmark:
- use Ollama as the simplest local inference runtime;
- start with `gpt-oss-20b` as one local benchmark candidate;
- compare it against the selected cloud model on the same CV test set;
- measure extraction accuracy, hallucinations, structured-output validity, ESCO mapping quality, latency and real operating cost.

Initial local deployment may run on the development machine and does not require a custom AI microservice. Higa Express can call the local inference runtime over HTTP through an AI provider adapter.

Possible production evolution:

Higa Express -> Higa AI Core -> provider adapter -> local inference runtime / cloud provider

If local inference proves useful at scale, the runtime can later move to a dedicated GPU host and use a production-oriented inference server such as vLLM without changing the higher-level Higa contracts.

Principle: choose the minimum sufficient compute/model for the required quality level, not simply the cheapest model and not automatically the largest model.
