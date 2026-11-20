# Group grocery shopping agent PRD

**Team collaboration:** [Group GitHub repository — occasions-agent](https://github.com/shivaylamba/occasions-agent). Coordinate frontend, common-memory MCP backend, shopping data, and agent execution work here.

Build a group grocery shopping expert that turns people’s ongoing requests and preferences into a coordinated shopping order. It maintains shared context, finds suitable products and eligible deals across accessible shopping sources, and helps the group purchase groceries for delivery with a clear audit trail.

The target users include families, roommates, students living together, and nearby friends who choose to consolidate a shopping run. Group membership is explicit; geographical proximity alone does not grant access to someone’s preferences or requests.

This PRD defines product behavior, engineering responsibilities, and integration research. The intended experience is real and end to end. Retailer access determines the implementation route, but a generated recommendation or simulated checkout must not be represented as a completed purchase.

# Product purpose

**Value proposition:** Everyone adds what they need, whenever they think of it. The agent remembers the group’s requirements, compares feasible grocery baskets, and coordinates an approved order.

The two core capabilities are **shared group context** and **shopping intelligence with execution**. Privacy and auditability apply across both. General household administration, childcare, healthcare, and calendars are outside the current scope.

The product addresses two kinds of work: collecting and reconciling multiple people’s needs, and researching where and how to buy those items economically. A shared list solves only part of the first problem; an individual deal finder does not understand the group’s ongoing needs.

# Users and group model

A member can add or change their own requests, record preferences, respond to substitutions, and inspect shared shopping decisions. A coordinator manages the shopping run, deadline, delivery destination, and group budget. A payer authorizes payment using their own connected account. One person can hold multiple roles, but being the group administrator does not automatically confer payment authority.

The initial market is the United States, starting in California. Each shopping run uses one shared delivery address. Members may be a household, roommates, students, or nearby friends. Support two payment modes as core functionality: one group payer covers the order, or each member pays their allocated grocery share. The exact California ZIP code is supplied during setup and determines available retailers and delivery service.

Requests distinguish **shared supplies** from **personal quantities**. Two people asking for the same shared detergent may mean one bottle; two people each requesting their own milk mean two allocations. Preserve original requests and ownership instead of deduplicating by product name alone.

## Payment modes and final budget approval

**One group payer:** one linked payer approves and funds the entire basket. Other members can review item allocations, but their payment accounts are not charged.

**Everyone pays their share:** each member links payment and approves their proposed charge after the basket is assembled. Personal groceries are allocated to the requesting member. Shared goods use a visible, editable allocation rule; propose equal shares among participating members by default. Distribute item discounts to the associated items and show how order-level discounts, taxes, fees, and tip are allocated. Rounded member totals must reconcile exactly to the order total.

This mode means actual payment by each participant, not simply displaying debts for later reimbursement. The purchase-execution owner must verify a provider-supported funding and retailer-payment mechanism; do not assume a retailer accepts multiple cards for one order. Payment linkage alone is not spending authorization.

The agent waits until every required approval and authorization succeeds. If a member declines or payment fails, pause and explain the blocker. Removing their items or changing shared fees creates a revised plan requiring approval from affected payers. Never silently charge another member for the shortfall.

Use a final proposed total or clearly disclosed maximum authorization where variable-weight goods or allowed substitutions prevent an exact total. Purchases must stay within each payer’s approved limit and the approved item/substitution rules. Return to affected users only for exceptions that exceed those rules. Record final charges and allocate cancellations, refunds, and released authorizations back to the original payers.

Acceptance includes both payment modes, exact allocation reconciliation, missing approval, failed authorization, partial payment failure, duplicate execution prevention, and refunds. Purchases across external systems are not presumed atomic: reconcile uncertain outcomes and reverse or release funds appropriately before any retry.

# Core user journey

1. Join a group, provide the shared California delivery address and shopping deadline, choose group-payer or individual-share payment, and securely link the relevant payment accounts. An initial spending preference is optional; final budget approval happens after the agent constructs the basket.

2. Add needs asynchronously through text, with optional quantity, brand, dietary constraints, maximum price, urgency, and substitution preference.

3. Review the structured request the agent extracted. Save recurring preferences explicitly; keep one-off needs separate.

4. The agent consolidates requests, retrieves permitted preferences, and asks targeted questions about conflicts or ambiguous quantities.

5. Start a shopping run. The agent queries accessible sources for the relevant location, fulfillment method, and requested quantities.

6. Compare feasible baskets using comparable products, verified discounts, known fees, availability, and delivery constraints.

7. Review a recommended plan with alternatives, source evidence, missing information, and an estimated or confirmed total.

8. After searching and constructing the basket, the agent presents where to buy, the items and quantities, delivery details, and the final proposed budget with fees and discounts. In group-payer mode the payer approves the full amount; in individual-share mode each member approves their own allocation. Approval covers the displayed plan and spending limit.

9. Once all required budget approvals and payment authorizations are complete, the agent creates the cart, purchases the groceries, and arranges delivery to the shared address. Routine retailer selection and checkout are handled by the agent. A mandatory manual retailer checkout is an integration gap against this target experience, not a completed implementation.

10. Record the order confirmation and delivery information that the provider actually returns. Update request status and preserve an auditable record of the outcome.

# Shared context requirements

| Capability | Required behavior |
| --- | --- |
| Asynchronous intake | Preserve the author, time, group, intended audience, and source text for each request. |
| Structured requests | Capture item, quantity and unit, intended recipient or shared use, deadline, budget, and substitution policy. Unknown values remain unknown. |
| Preference memory | Store subject, preference, hard or soft constraint, visibility, confirmation state, source, update time, and optional expiry. |
| Consolidation | Combine compatible requirements while retaining attribution and allocations. Ask before merging ambiguous duplicates. |
| Corrections | A corrected preference supersedes the earlier value for future planning. A cancelled request disappears from active planning. |
| Multiple sessions | New sessions retrieve the same current group state and confirmed preferences. |
| Group isolation | Authorization is enforced before reads, retrieval, writes, and execution. Another group’s records are inaccessible. |
| Concurrent updates | Version the shopping run; a late change cannot silently modify an already approved order. |
| Inspectable memory | Members can see and correct what the agent remembers about them and choose what to share. |

Use a shared requirement only with the member’s permission. A member may share “Choose fragrance-free detergent” while keeping the reason private. Sensitive personal explanations should not appear in shared recommendations, notifications, analytics, or audit views.

# Shopping intelligence requirements

## Define the best deal

Recommend the lowest eligible total among the feasible baskets and sources actually checked, subject to the group’s requirements. Say “Best among the sources checked,” with locations and timestamps, rather than claiming a global best price.

Compare the same requested quantities and compatible variants. Normalize pack sizes and unit prices, but calculate actual packages that must be purchased. A larger package can have a lower unit price while increasing cash spend or creating unwanted surplus.

The basket total includes item cost, eligible discounts, delivery and service fees, taxes where available, any applicable deposits or minimum-order charges, and the selected tip where relevant. Unknown components remain labeled; they are not treated as zero. Show estimated and checkout-confirmed totals distinctly.

A basket is feasible only when it satisfies required product attributes, requested quantities, stock and service-area constraints, delivery timing, and budget. Missing items must not make a partial basket appear cheaper than a complete one.

## Product and coupon evidence

For each offer, preserve retailer and platform, store or location, SKU or barcode when present, product and pack size, requested quantity, price, currency, stock status, fulfillment mode, source URL, and retrieval time. Distinguish an exact product match from an allowed substitute and an uncertain match.

For each coupon, preserve its source, expiry, eligible items, minimum spend, account or membership restrictions, quantity limits, and stacking rules when available. Mark it as **discovered**, **eligibility checked**, or **applied at checkout**. Do not count a discovered but unverified coupon as guaranteed savings. Do not invent a coupon when none is found.

Use confirmed hard constraints before soft preferences. Offer a substitute only within the member’s substitution policy. A cheaper incompatible item is not a better deal. Label missing attribute evidence instead of asserting compliance.

## Basket planning and execution

Compare complete single-retailer baskets first. Keep split-retailer optimization as an optional extension: savings must exceed additional fees and coordination costs, and the payer must approve the separate orders.

Approval binds to the basket version, products, quantities, substitutions, retailer, delivery details, payment authority, and total or explicitly authorized limit. Revalidate price, coupon application, stock, and destination before committing. New information that exceeds the authorization returns the run to review.

Maintain separate states for **searching**, **basket ready**, **awaiting budget approvals**, **authorizing payments**, **purchasing**, **order confirmed**, **out for delivery**, and **delivered**, plus failure and outcome-unknown states. Shopping links and carts can be internal intermediate steps; they do not satisfy purchase completion. Show delivery completion only when supported by provider evidence or explicit recipient confirmation.

Persist action attempts and external references. Use idempotency where available and reconcile an uncertain checkout result before retrying. An expired session, changed price, or rejected coupon should preserve the plan and explain the next action.

# Frontend design scope

The frontend owner designs and implements a standalone group shopping experience. The main interface should make shared work and decisions visible without requiring users to read an entire chat transcript.

| Surface | Content and interactions |
| --- | --- |
| Group setup | Group name, membership, roles, delivery location, payment mode, secure payment linkage, deadline, and connection status. |
| Shared list | Requests grouped by product or category; member attribution, shared versus personal allocation, quantities, conflicts, and quick add or edit. |
| Personal input | Conversation or composer with explicit audience; structured capture confirmation and optional “Remember this preference.” |
| Shopping plan | Recommended retailer and basket, comparison alternatives, item coverage, total breakdown, delivery constraints, and outstanding questions. |
| Item details | Product match, size, quantity, unit price, preference fit, substitution options, sources, and checked time. |
| Approval | Final proposed budget after basket construction, retailer, basket version, delivery destination, total and per-person shares, pending approvals, and approve, edit, or cancel actions. |
| Order and audit | Accurate progress, retailer reference, payment and purchase status, delivery tracking and evidence, and an expandable decision timeline. |
| Group memory | Shared and personal preferences, visibility controls, corrections, expiry, and deletion. |

The comparison view must show item coverage and unknown costs alongside the headline total. “Why this basket?” explains the meaningful tradeoffs: cost, required attributes, availability, and delivery.

Design loading, partial-data, no-matching-product, conflict, permission-denied, stale-quote, coupon-rejected, disconnected-account, and outcome-unknown states. Use readable text statuses, keyboard access, and visible focus states. Product copy should describe what is happening; MCP and provider internals belong in engineering diagnostics.

# Engineering responsibilities

| Workstream | Staffing | Ownership and concrete deliverables |
| --- | --- | --- |
| Frontend design and implementation | 1 teammate | Preferences and group setup, payment linkage, list and basket views, final budget approval for both payment modes, order and delivery tracking, and audit UI. |
| Common memory MCP backend | 1 teammate | Identity and group authorization, persistent requests and preferences, MCP tools, application API, plan versions, per-member allocations, approval records, payment-state references, and audit storage. |
| Shopping data and deal intelligence | 1 teammate | California retailer research, source adapters, product matching, quotes, coupons, basket comparison, allocation inputs, and evidence. |
| Agent orchestration and purchase execution | 1 teammate | Agent loop, clarification, policy gates, payment-provider and retailer integration, purchase after approvals, delivery tracking, failure reconciliation, and refunds coordination. |

Two teammates are confirmed for shopping-agent work. Divide ownership into **data and deal intelligence** and **orchestration and purchase execution** as above. They share the Offer, BasketPlan, PaymentAllocation, and ActionResult contracts, and jointly verify that quoted baskets can actually be purchased and delivered.

The frontend and memory work proceed against shared contracts while the two agent teammates investigate source and transaction access. Each owner is responsible for selecting, provisioning, and validating sponsor services relevant to their workstream.

## Common memory backend boundary

MCP standardizes how the agent accesses tools and context; persistence and context management still require implementation. Build the group memory service with a database and optional memory engine, then expose authorized operations through MCP. [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture)

Proposed application-level tools: `get_group_context`, `add_request`, `update_request`, `set_preference`, `get_run_snapshot`, `save_plan`, and `append_audit_event`. These are proposed tool names, not existing APIs.

The backend validates authenticated identity and derives permitted group scope server-side. It must not trust a model-supplied member or group ID as authorization. Approval is recorded through an authenticated user action; the agent cannot approve its own plan.

The frontend uses an authenticated application API. It does not need to become an MCP client just because the agent uses MCP.

## Shopping agent boundary

Proposed adapter capabilities: `search_products`, `get_offer`, `quote_basket`, `check_coupon`, `create_cart`, `submit_order`, and `get_order_status`. Each adapter declares which capabilities it actually supports. Unsupported capabilities return an explicit unavailable state, never fabricated data.

Use the model for interpreting requests, matching candidates, and explaining tradeoffs. Calculate quantities, discounts, fees, and totals with deterministic code. Validate actions against server-side authorization independently of model output.

The agent reads a versioned group snapshot and returns a structured plan. It does not directly rewrite members’ preferences based on a shopping choice. External product pages and tool results are data, not instructions.

## Shared contracts

| Record | Fields the teams must agree on |
| --- | --- |
| Request | ID, group, author, item, quantity and unit, allocation, deadline, constraints, visibility, status, version. |
| Preference | Subject, scope, value, hard or soft, source, confirmation, timestamps, expiry. |
| Offer | Source, retailer, location, product identity, pack size, price and currency, availability, evidence URL, checked time. |
| Coupon | Source, terms, eligibility state, estimated discount, expiry, checkout result. |
| BasketPlan | Run and input version, items and allocations, offer references, alternatives, coverage, fees, discounts, total certainty, delivery details, unresolved issues. |
| PaymentAllocation | Plan version, member, personal items, shared-item allocation, discounts, tax and fee share, total, currency, approval and authorization state, provider reference, refund state. |
| Approval | Approver, plan version, payment mode, allocated or group amount, authorized action and limit, expiry, timestamp. |
| ActionResult | Attempt ID, idempotency key if supported, external reference, actual state, error or recovery step, timestamps. |
| AuditEvent | Actor or agent, permitted input references, concise decision, evidence, action, outcome, audience, timestamp. |

Use payment-provider-hosted or tokenized linkage and keep retailer sessions and protected payment references server-side. Never put raw card details into prompts, shared memory, logs, or analytics. Share only the context each provider needs. Partition personal and group memory, minimize logs, and propagate deletion or corrections into derived memory.

# California grocery platform shortlist

Evaluate serviceability at the group’s exact address. These are consumer shopping candidates, not confirmed agent integrations. The same retailer through different platforms can have different offers and fees; preserve both retailer and platform in every quote.

| Candidate | Why investigate | Integration check |
| --- | --- | --- |
| Instacart | Multi-retailer grocery shopping; California city listings include stores such as Safeway and Costco. [California example](https://www.instacart.com/grocery-delivery/ca/near-me-in-modesto-ca) | Check accessible data and purchase permissions separately; the documented shopping-list link endpoint alone does not fulfill automatic checkout. |
| Safeway | Direct online grocery ordering, delivery, weekly ads, and personalized for U deals. [Official site](https://www.safeway.com/) | Validate address coverage, account-specific coupon eligibility, actual checkout access, and delivery status. |
| Ralphs | Direct grocery delivery with online coupons and substitution options. [Delivery](https://www.ralphs.com/i/ways-to-shop/delivery) | Validate California ZIP coverage and Kroger developer/account capabilities against the exact required actions. |
| Costco Same-Day | Candidate for consolidated or bulk purchases. [Same-Day](https://www.costco.com/same-day.html) | Confirm membership and platform conditions, required pack sizes, delivered total, and accessible checkout. |
| Target | Same-day delivery fulfilled by a Shipt shopper. [Delivery](https://www.target.com/c/same-day-delivery/-/N-bswkz) | Check grocery coverage, local stock, promotions, fees, and authorized purchase integration. |
| Whole Foods Market through Amazon | Grocery delivery and pickup in selected ZIP codes. [Online ordering](https://www.wholefoodsmarket.com/online-ordering) | Check the delivery address, account eligibility, current offers, purchase access, and status retrieval. |
| Walmart | Additional consumer grocery candidate for local investigation. | Verify the consumer delivery route and exact address coverage. Marketplace seller APIs do not by themselves establish consumer checkout capability. [Developer scope](https://developer.walmart.com/global-marketplace/docs/getting-started) |

Start the technical investigation with Instacart, Safeway, and Ralphs, then assess Costco, Target, Whole Foods/Amazon, and Walmart against the same capability matrix. This is a research priority, not a commitment to implement every retailer. Select sources based on proven data and transaction access as well as coverage.

# Shopping data and execution research

Research status as of November 20, 2026: the following are documented capabilities and access constraints, not tested integrations or provisioned credentials. “Open data,” “publicly documented API,” and “available to our account” are different things.

| Candidate | Useful capability | Constraint and next step |
| --- | --- | --- |
| Open Food Facts | Open product information, including ingredients, allergens, and nutrition. [Official project](https://openfoodfacts.github.io/) | Useful for product enrichment and matching. It does not establish current local stock, checkout price, or a purchase route. Validate the exact product and data completeness. |
| Open Prices | Crowdsourced price observations with receipt or price-tag evidence; API and downloadable data are available. [Price documentation](https://openfoodfacts.github.io/documentation/docs/Product-Opener/api/tutorials/product-prices/) | Assess target-area coverage and observation dates. Treat observations as historical or observed evidence, not a live deliverable quote. Dataset reuse has license conditions. [Data access](https://openfoodfacts.github.io/open-prices/guides/data/) |
| Instacart Developer Platform | The shopping-list endpoint creates an Instacart-hosted page link; users then select a store, add products, and check out. [Endpoint](https://docs.instacart.com/developer_platform_api/api/products/create_shopping_list_page/) | This endpoint does not itself prove a cart, purchase, or an API-readable cross-retailer quote. Instacart states its Public API does not expose Instacart data. [Developer platform](https://company.instacart.com/business/developers) |
| Instacart access | Development and production keys are distinct; its onboarding guide gives an average 30–40 days from access request through approval and production access. [Get started](https://docs.instacart.com/developer_platform_api/get_started/overview) | Do not make same-day production access an assumption. Check existing team credentials or sponsor-provided access immediately. |
| Kroger developer platform | Candidate for direct retailer integration. [Official reference](https://developer.kroger.com/reference/) | Its reference page did not expose readable content in this research. Verify available endpoints, regional coverage, credentials, quote fields, cart actions, and checkout support before selecting it. No capability is assumed here. |
| Retailer websites or authorized browser flows | Potential source of location-specific offers and a user-authorized shopping route. | Choose named sites and verify access, supported use, login requirements, reproducibility, and confirmation retrieval. Public visibility alone does not make a stable integration. |

**Current finding:** open product and observed-price data can support matching and research, but this research has not established an unrestricted source for live multi-platform grocery totals, universally valid coupons, and autonomous purchase together. Data comparison and transaction execution may require different integrations.

Do not silently substitute retailer-hosted product selection for your own cross-platform comparison. If the integration exposes only a shopping link, label that capability and the remaining checkout steps precisely.

## Source research deliverable for the agent team

Produce a capability matrix for named candidate platforms: geography; authentication; accessible product identifiers; current price; location-specific stock; coupon terms and validation; basket fees; delivery windows; cart creation; checkout; order confirmation; rate limits; access lead time; and a reproducible successful request where credentials are available.

Return a recommendation supported by actual accessible fields and actions. If live comparison across two sources is blocked, document the blocker and available alternatives before narrowing the product promise. Do not fabricate a second source or imply historical prices are live quotes.

# Sponsor resources and ownership

The sponsor mapping below carries forward the earlier research, updated to group shopping. These tools support the system; none is assumed to provide a universal grocery catalog, coupon feed, or payment integration.

| Sponsor | Proposed contribution | Priority and proof |
| --- | --- | --- |
| Cognee | Persistent, permissioned recall of confirmed group preferences. Its platform provides agent memory and self-hosted or hosted options. [Docs](https://docs.cognee.ai/) | Core differentiator. Demonstrate remembering a preference in a new run and correcting it. Explicitly configure authentication and compatible access-control backends. [Permissions](https://docs.cognee.ai/setup-configuration/permissions) |
| Arcee AI | Model inference for extracting requests and proposing plans. The documented API supports chat completions with open models. [Docs](https://docs.arcee.ai/) | Core model candidate. Evaluate structured extraction on the exact group examples before committing. Hosted inference remains a data-processing boundary. |
| SingleStore | Canonical application records, plan versions, approvals, and audit events. [Docs](https://docs.singlestore.com/cloud/getting-started-with-singlestore-helios/) | Core storage candidate. Demonstrate durable run state and audit retrieval. Keep Cognee storage separate unless the backend combination is explicitly supported; do not assume native compatibility. |
| Senso | Ground product decisions in a curated, non-sensitive product-information collection or shared reference context. Its docs describe shared context and source-based verification workflows. [Docs](https://docs.senso.ai/docs/overview) | Strong challenge fit if ready access exists. Show an actual retrieved source affecting a choice. It does not by itself supply live retailer price, stock, or checkout. Avoid duplicating private group memory here. |
| Firedrill | Exercise synthetic group users and custom shopping tools against failures and conflicts. [Docs](https://docs.firedrill.run/) | High-value validation after the live path works. Show a trace for duplicate execution, timeout, cancellation, or unauthorized access. Keep synthetic evaluation separate from live shopping completion. |
| Guild AI | Runtime, triggers, credential policies, and agent/tool event logs. [Docs](https://docs.guild.ai/) | Add if its setup and custom-app entry point are readily usable. Prove one run through a trigger and scoped credential. Platform governance does not replace per-member app authorization. |
| Superlinked | Task-specific extraction or reranking of candidate products; its SIE offering includes structured output, retrieval, and reranking. [Product](https://superlinked.com/serverless) | Optional focused extension. Use it for one measurable task, such as ranking compliant candidates. Avoid sending the whole group history. |
| Amplitude | Measure capture-to-approval-to-completion funnels through explicit events. [Docs](https://amplitude.com/docs) | Lightweight addition after the core flow. Send event types, latency, and status; exclude names, shopping text, private preferences, and health data. Disable broad autocapture and session replay. |
| Statsig | Control rollout of proactive reminders or alternative presentation. [Docs](https://docs.statsig.com/welcome) | Optional small integration. Demonstrate a flag; do not claim meaningful experiment results from a tiny demo sample. A flag is not an authorization mechanism. |
| Airbyte | Read authorized connected context when a supported source is useful. Its Context Layer provides live read-only access and indexed search, with Cloud availability constraints. [Docs](https://docs.airbyte.com/platform/context-layer) | Later unless there is an immediately useful connector. Verify personal-account support and permission boundaries first. It is not the shopping write adapter or a guaranteed calendar integration. |
| Persona | Verify a person or a required attribute for a specific agent action. [Docs](https://help.withpersona.com/articles/CmxigxTOZVhlr4bn7xSX9S/) | Optional only where the purchase warrants it. Use a minimal verification result, not copied identity documents. Persona does not encrypt group memory, establish group membership, or control the agent’s permissions. |

**Sponsor ownership:** each teammate evaluates and selects sponsor services for their own workstream. The table is a research shortlist, not a committed stack. Coordinate shared dependencies, credentials, cost, and data boundaries before integration; record selected services and access status in the group repository.

**Prioritization:** choose one model/runtime path, one canonical state store, and one memory approach. Add Senso for useful source-grounded reference context, Superlinked for a measurable matching or reranking need, and Firedrill for failure evaluation. Sponsor count should not create duplicate stores of private group data. Credits, event eligibility, and account-specific access remain unconfirmed.

# Integration sequence and acceptance criteria

Agree on the shared records and service boundaries first. The frontend can build against fixtures that match those contracts; the backend can persist real group data; the agent team can investigate sources independently. Fixtures are development aids and do not satisfy release completion.

Then connect the real vertical path: member preferences and payment linkage → saved requests and memory retrieval → sourced offers → computed basket and payment allocations → final budget approvals → payment authorizations → agent purchase → delivery tracking and confirmed outcome. Validate both payment modes. Expand source coverage only after the first route works.

Required acceptance checks:

- Two separately authenticated members contribute asynchronously and retain correct attribution.

- Shared supplies and individual quantities consolidate correctly.

- Confirmed preferences persist across sessions and corrections affect the next plan.

- Every offer and coupon has evidence, timestamp, and a truthful verification state.

- Comparison uses equivalent quantities and reveals missing items and unknown fees.

- The selected basket respects hard constraints, delivery requirements, and the approved budget.

- Unauthorized users cannot read private context, approve a purchase, or alter another group.

- A changed approved plan requires renewed authorization.

- A timed-out or repeated execution request does not cause an unexamined duplicate order.

- The normal flow completes agent-executed purchasing after budget approval and tracks delivery. The UI never labels a shopping link, cart, or checkout handoff as a completed purchase.

- The audit explains the inputs, sources, decision, approval, and actual result without revealing private reasons.

Measure completion of the real shopping task, preference satisfaction, clarification burden, comparison coverage, and quote-to-checkout variance. Any savings claim must compare equivalent feasible baskets with the same cost basis and checked time. A synthetic fixture cannot substantiate real savings.

# Confirmed decisions and remaining integration work

The product direction below is confirmed. Provider selection and technical feasibility remain implementation responsibilities; they do not change the intended user experience.

- **Market and delivery:** United States, California; one delivery address per run. Address and ZIP-code input determine actual retailer availability.

- **Shopping-source research:** the data and deal intelligence owner checks common California grocery retailers and platforms, then validates live quote coverage, coupons, fees, account access, and purchase compatibility.

- **User experience:** collect needs, preferences, delivery details, and payment linkage; the agent searches and builds the basket; users approve the final proposed budget; the agent purchases and arranges delivery. A routine manual checkout handoff does not meet the target flow.

- **Payment optionality:** support both one group payer and each member paying their own allocated share. All required approvals and authorizations must succeed before committing the group purchase.

- **Staffing:** one frontend teammate, one common-memory MCP backend teammate, and two shopping-agent teammates.

- **Sponsor services:** each teammate chooses and verifies services for their workstream. No sponsor stack is committed yet.

**Remaining integration work:** select the initial serviceable California ZIP code and retailers; prove current offer and coupon access; choose a payment mechanism for both modes; verify agent-executable checkout and delivery status; and implement failure, cancellation, and refund behavior. Do not mark automatic purchasing or individual-share payments complete until the real integrations support them.

