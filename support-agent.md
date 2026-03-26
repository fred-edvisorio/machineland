# Support Agent

> **Codename:** SENTINEL  
> **Department:** Support  
> **Classification:** Supervised  
> **Owner:** Moreno (Support / Pylon)  
> **Escalation Channel:** `#master-machine-customerops`  
> **Registry:** Linear — Agent Directory Project  
> **Last Updated:** 2026-03-25

---

## 1. Purpose

The Support Agent is the front line of Edvisor's customer interaction system. It triages incoming support tickets from Pylon, classifies them by urgency and category, routes them to the correct handler (automated resolution, support team, or cross-department escalation), detects churn signals embedded in support interactions, and ensures every ticket feeds signal data back into the Master Machine for downstream action by the Success and Sales agents.

### 1.1 What This Agent Does (Agent Work)

- Classifies incoming Pylon tickets by category, urgency, and persona.
- Routes tickets to the correct handler based on classification rules.
- Drafts initial responses for common ticket types (how-to, billing, feature requests).
- Detects churn-risk language (cancellation, competitor mentions, frustration patterns) and triggers immediate escalation.
- Monitors CSAT scores post-resolution and flags low scores for follow-up.
- Generates weekly support health reports (volume, resolution time, CSAT by segment).
- Logs all ticket data to Snowflake for CHI calculation and trend analysis.

### 1.2 What Humans Do (Human Work)

- Resolving complex, ambiguous, or emotionally charged tickets.
- Handling billing disputes and payment exceptions.
- Making judgment calls on edge cases the agent cannot classify.
- Building and maintaining the knowledge base that powers agent responses.
- Approving all externally-sent responses (Supervised classification).
- Direct customer conversations requiring empathy and trust.

---

## 2. Scope & Boundaries

The Support Agent operates **across all funnel stages** — any paying customer or active trial user can submit a ticket. Unlike other agents with clear funnel-stage ownership, the Support Agent is a cross-cutting function.

| Scope | In | Out |
|-------|-----|-----|
| Pylon ticket triage and classification | ✅ | |
| First-response drafting | ✅ | |
| Churn signal detection and escalation | ✅ | |
| CSAT monitoring | ✅ | |
| Ticket routing (internal and cross-department) | ✅ | |
| Direct ticket resolution (complex/emotional) | | ❌ Human only |
| Billing dispute resolution | | ❌ Human + Finance |
| Product bug investigation | | ❌ Routes to Product/Engineering via Linear |
| Account-level health interventions | | ❌ Customer Success Agent |

---

## 3. Signal Sources

| Source | Signals Consumed | Tool |
|--------|-----------------|------|
| Pylon | New tickets, ticket updates, resolution events, CSAT responses | Pylon → N8N |
| Attio | Account context (persona, MRR, CHI, funnel stage, account owner) | Attio → N8N |
| Snowflake | Historical ticket patterns by account, CSAT trends, resolution benchmarks | Snowflake → N8N |
| UserPilot | Current user context (feature adoption, session recency) — used for contextual responses | UserPilot → N8N |
| Stripe | Billing status (active, past due, trial) — used for billing ticket context | Stripe → N8N |

---

## 4. Trigger Conditions & Actions

### 4.1 Ticket Classification & Routing

| Trigger | Action |
|---------|--------|
| New Pylon ticket received | **Step 1:** Enrich with Attio context (persona, MRR, CHI, funnel stage, account owner). **Step 2:** Classify by category and urgency. **Step 3:** Route per classification rules below. |

**Classification Categories:**

| Category | Description | Examples |
|----------|-------------|---------|
| **How-To** | User needs help using a feature | "How do I generate a quote?", "Where is the insurance tab?" |
| **Bug Report** | Something is broken or behaving unexpectedly | "Quote prices are wrong", "I can't log in", "Page won't load" |
| **Billing** | Payment, invoice, subscription questions | "I was charged twice", "How do I upgrade?", "Cancel my subscription" |
| **Feature Request** | User wants something that doesn't exist | "Can you add multi-currency invoicing?", "I need bulk quote export" |
| **Data Issue** | Incorrect data, missing schools, pricing discrepancies | "School X pricing is outdated", "My agency connections are wrong" |
| **Churn Signal** | Language indicating dissatisfaction, intent to leave, or competitor interest | "I'm considering switching to AMS", "This isn't working for us", "We want to cancel" |
| **Escalation Required** | Cannot classify or high-complexity issue | Ambiguous tickets, multi-issue threads, VIP accounts |

**Urgency Matrix:**

| Urgency | Criteria | Response SLA |
|---------|----------|-------------|
| **Critical** | Platform down, payment processing broken, data loss, churn language + MRR ≥$500/mo | 1 hour |
| **High** | Bug affecting workflow, billing error, churn language (<$500/mo) | 4 hours |
| **Medium** | How-to (paid user), data issue, feature request from high-CHI account | 8 hours |
| **Low** | How-to (trial user), general inquiry, feature request from low-CHI account | 24 hours |

**Routing Rules:**

| Classification | Route To | Action |
|---------------|----------|--------|
| How-To (simple, KB article exists) | Auto-draft response with KB link | Draft response for Moreno's team review |
| How-To (complex, no KB article) | Support team | Create draft response outline. Flag for KB article creation (Linear task). |
| Bug Report | Support team + Linear task for Product | Draft acknowledgment. Create Linear bug ticket with reproduction steps. |
| Billing (standard) | Support team + Phil (Finance) | Draft response with billing context from Stripe. |
| Billing (dispute/refund) | Moreno + Phil | **Human only.** Provide context brief. |
| Feature Request | Support team | Draft acknowledgment. Log to Linear feature request backlog. |
| Data Issue | Support team + Supply Operations | Draft acknowledgment. Create Linear task for data correction. |
| Churn Signal | **Immediate escalation** | Alert Customer Success Agent + Ben + Community Manager. See §4.2. |
| Escalation Required | Moreno | Full context brief. Agent does not attempt response. |

### 4.2 Churn Signal Detection

The Support Agent performs real-time sentiment analysis on every incoming ticket. This is the highest-priority detection function.

**Churn signal keywords (multi-language):**

```
EN: cancel, cancellation, switch, switching, leaving, not working for us,
    considering alternatives, too expensive, competitor name (Ally Hub, AMS,
    Meritto, Salesforce), downgrade, not using, waste of money, disappointed

ES: cancelar, cancelación, cambiar, dejando, no funciona, alternativas,
    demasiado caro, competidor, bajar de plan, no estamos usando, decepcionado

PT: cancelar, cancelamento, mudar, saindo, não funciona, alternativas,
    muito caro, concorrente, rebaixar, não estamos usando, decepcionado
```

| Churn Signal Detected | Account MRR | Action |
|----------------------|-------------|--------|
| Any churn keyword | ≥$500/mo | **CRITICAL.** Immediate Slack alert to Ben + Community Manager. No automated response. Generate full context brief (CHI trend, ticket history, MRR, tenure, recent activity). |
| Any churn keyword | <$500/mo | **HIGH.** Alert Community Manager via Slack. Draft empathetic acknowledgment for human review. Generate context brief. |
| Competitor name mentioned | Any MRR | Alert Ben + Community Manager. Include competitor-specific context (known differentiators, migration risks). |
| CSAT score <3 post-resolution | Any MRR | Alert Community Manager. Flag for follow-up. If CSAT <3 on 2+ tickets from same account in 30 days → escalate to Customer Success Agent as churn risk. |

### 4.3 First-Response Drafting

For non-churn, non-escalation tickets, the agent drafts initial responses:

**Draft quality requirements:**
- Address the specific issue raised (not generic).
- Include relevant context from the user's account (persona, plan, feature adoption).
- Reference specific KB articles or help documentation where applicable.
- Use the correct language (see §5 Language Routing).
- Maintain Edvisor brand voice: warm, direct, knowledgeable.
- Never promise features, timelines, or refunds — only state facts and next steps.

**Draft is always queued for human review before sending (Supervised classification).**

### 4.4 Post-Resolution Monitoring

| Trigger | Condition | Action |
|---------|-----------|--------|
| Ticket resolved | Pylon: ticket status = "resolved" | Monitor for CSAT response (24h window). Log resolution data to Snowflake. |
| CSAT received | Pylon: CSAT submitted | If CSAT ≥4 → log and close. If CSAT <3 → escalate per §4.2. If CSAT = 3 → log and flag for weekly review. |
| Repeat ticket | Same account, same category, within 14 days | Flag as recurring issue. Alert support team. If 3+ repeat tickets → escalate to Customer Success Agent. |

---

## 5. Language Routing Logic

The Support Agent determines response language based on the incoming ticket language and account locale:

```
1. Detect incoming ticket language
   → If clearly Portuguese → respond in Portuguese
   → If clearly Spanish → respond in Spanish
   → If clearly English → respond in English
   → If ambiguous → check Attio account field: `locale` (same logic as Marketing Agent §5)

2. Fallback (if ticket language unclear and locale null):
   → Check contact email domain TLD
     → .br → Portuguese
     → .mx, .co, .es, .ar, .cl, .pe, .ec → Spanish
     → All other → English
```

**Response template variants:** All canned responses and KB article references exist in EN/ES/PT. The agent selects the correct variant automatically.

**Tone calibration by language:**
- **English:** Warm, helpful, direct. No jargon. 
- **Spanish:** Same warmth. "Usted" form for LATAM. Professional but approachable.
- **Portuguese (BR):** Same warmth. "Você" form. Natural Brazilian Portuguese — avoid overly formal constructions.

---

## 6. Tools & Integrations

| Tool | Usage |
|------|-------|
| **N8N** | Orchestration layer. Ticket intake, classification, routing, escalation triggers. |
| **Pylon** | Primary support platform. Ticket creation, updates, resolution, CSAT collection. Agent reads and enriches tickets; human team resolves. |
| **Attio** | Account context enrichment (persona, MRR, CHI, funnel stage, owner). Read-only. |
| **Snowflake** | Historical ticket data, CSAT trends, resolution benchmarks. Read-only for agent; writes ticket logs. |
| **Stripe** | Billing context for billing-category tickets. Read-only. |
| **UserPilot** | User context (feature adoption, session recency) for contextual responses. Read-only. |
| **Slack** | Escalation alerts, churn signal notifications, weekly reports. Channel: `#master-machine-customerops`. |
| **Linear** | Bug ticket creation, feature request logging, KB article creation tasks. |

---

## 7. Escalation & Handoff Protocols

### 7.1 Agent → Human

| Scenario | Escalation Target | Channel | Response SLA |
|----------|------------------|---------|-------------|
| Churn signal (any) | Ben + Community Manager | `#master-machine-customerops` | 1 hour (critical) / 4 hours (high) |
| CSAT <3 | Community Manager | Slack DM | Same business day |
| Billing dispute/refund | Moreno + Phil | `#master-machine-customerops` | 4 hours |
| Unclassifiable ticket | Moreno | Slack DM | 4 hours |
| Critical severity (platform down, data loss) | Moreno + Product/Engineering | `#master-machine-customerops` + Linear | 1 hour |
| Any drafted response | Support team member | Pylon (queued as draft) | Per SLA by urgency |

### 7.2 Agent → Agent Handoffs

| Direction | Trigger | Data Passed |
|-----------|---------|------------|
| **Support → Success** | Churn signal detected in ticket | `ticket_id`, `churn_signal_type`, `account_CHI`, `MRR`, `ticket_summary`, `sentiment_score` |
| **Support → Success** | CSAT <3 on 2+ tickets in 30 days | `account_id`, `CSAT_history`, `ticket_summaries`, `CHI_trend` |
| **Support → Success** | Ticket reveals expansion opportunity (user asks about features on higher plan) | `ticket_summary`, `feature_requested`, `current_plan`, `account_id` |
| **Support → Sales** | Trial user ticket reveals high buying intent | `ticket_summary`, `trial_status`, `engagement_level` |
| **Support → Marketing** | Negative CSAT trend across segment | `segment`, `CSAT_trend`, `common_complaint_themes` |

---

## 8. Governance Rules

1. **Supervised classification.** Every outbound response is drafted by the agent and reviewed by a human before sending. No exceptions.
2. **Churn = human-only.** Any ticket containing churn signals gets escalated immediately. The agent provides context but never responds directly to churn-related tickets.
3. **No promises.** The agent never commits to feature timelines, refund amounts, or service guarantees in drafts. These require human approval.
4. **Every ticket reaches Snowflake.** Classification, resolution time, CSAT, and account context are logged for every ticket — feeding CHI calculations and trend analysis.
5. **N8N is the integration hub.** No direct Pylon-to-Attio or Pylon-to-Slack connections that bypass N8N.
6. **Billing data is read-only.** The agent reads Stripe context for billing tickets but never modifies billing records. All billing actions are human-executed.

---

## 9. KPIs & Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Ticket classification accuracy | ≥90% (validated by human corrections) | Monthly audit of agent classifications vs. final human classifications |
| First-response draft approval rate | ≥85% sent without major edits | Pylon draft vs. sent comparison |
| Churn signal detection recall | ≥95% (no missed churn signals) | Monthly audit: compare churned accounts with ticket history |
| Average first-response time | Within SLA by urgency tier | Pylon response time analytics |
| CSAT (overall) | ≥4.0 average | Pylon CSAT reports |
| Repeat ticket rate | <15% of total volume | Pylon + Snowflake |
| Escalation accuracy | <10% false positive rate on churn escalations | Community Manager feedback |
| Weekly report delivery | 100% on-time (Friday by 3pm UTC) | N8N execution logs |

---

## 10. Weekly Support Health Report

Generated every Friday and posted to `#master-machine-customerops`:

**Report Contents:**
- Total ticket volume (week-over-week change)
- Volume breakdown by category and urgency
- Average resolution time by category
- CSAT distribution (score breakdown)
- Churn signals detected (count + accounts affected)
- Top 3 recurring issues (with Linear task links if created)
- Accounts with 3+ tickets this week (flag for Success Agent review)
- Language distribution (EN/ES/PT breakdown)

---

## 11. Promotion Path

| Sub-workflow | Promotion Criteria |
|-------------|-------------------|
| How-To ticket drafting (simple, KB article exists) | ≥95% approval rate, <5% major edits over 30 days |
| Ticket classification | ≥95% accuracy over 30 days (audited) |
| Weekly report generation | 0 errors in 30 days |

**Permanently Supervised (never promoted):** Churn-signal responses, billing tickets, bug report responses, any customer-facing communication on accounts ≥$500/mo MRR.

---

## 12. Dependencies & Blockers

| Dependency | Status | Impact |
|------------|--------|--------|
| **Pylon migration** | 🔄 In progress | **CRITICAL BLOCKER** — no ticket intake without Pylon fully operational |
| Pylon → N8N integration | 🔄 In progress | Blocks automated ticket routing and classification |
| Knowledge base (EN/ES/PT) | ⬜ Not started | Blocks How-To draft quality (agent needs KB articles to reference) |
| CHI threshold finalization | 🔄 In progress | Blocks urgency scoring (CHI context needed for prioritization) |
| Attio → N8N connection | ✅ Complete | — |
| Stripe → N8N connection | ✅ Complete | — |
| Snowflake pipeline | ✅ Complete | — |

---

## 13. Relationship to Legacy Agent Codenames

| Legacy Codename | Capabilities Absorbed |
|----------------|----------------------|
| SENTINEL 🛡️ | Full scope — support triage, classification, routing, churn detection, CSAT monitoring, first-response drafting |

The Support Agent (SENTINEL) is the only agent in the fleet that maps 1:1 to a single legacy codename, as the original design was already scoped as a unified support function.
