# Customer Success Agent

> **Codename:** MERIDIAN  
> **Department:** Customer Success / Community  
> **Classification:** Supervised  
> **Owner:** Ben (Success & Sales) — operational day-to-day; Community Managers by market  
> **Escalation Channel:** `#master-machine-customerops`  
> **Registry:** Linear — Agent Directory Project  
> **Last Updated:** 2026-03-25

---

## 1. Purpose

The Customer Success Agent owns everything below the funnel waist — from "Win" through "Advocate" — across both the User Activation track and the Campus Activation track. It orchestrates onboarding, monitors Customer Health Index (CHI) continuously, intervenes when health declines, drives expansion and renewal, and cultivates advocates who feed the referral flywheel. This is the largest-scope agent in the fleet, consolidating five previously-designed agent roles into a single coordinated system.

### 1.1 What This Agent Does (Agent Work)

- Orchestrates onboarding sequences by persona tier, tracking milestone completion.
- Calculates and monitors User CHI and Campus CHI weekly from Snowflake data, updating Attio.
- Detects declining health signals and triggers graduated re-engagement (automated → human escalation).
- Monitors renewal timelines and generates renewal preparation briefs.
- Identifies expansion opportunities (plan upsell, multi-campus, transactional adoption).
- Tracks advocate behavior and triggers referral prompts at peak satisfaction moments.
- Generates weekly CHI digests for Community Managers and leadership.
- Drafts proactive outreach for Community Managers to review and send.

### 1.2 What Humans Do (Human Work)

- Community Managers: Proactive relationship work, group trainings, local gatherings, EdSummit events.
- Judgment calls on churn-risk accounts (the agent flags; humans decide the intervention).
- Executive-level retention conversations for high-ACV accounts.
- Market intelligence gathering (competitive moves, policy shifts, sentiment).
- Strategic decisions on which accounts get premium treatment vs. automated handling.
- Approving all externally-sent communications (Supervised classification).

---

## 2. Funnel Stages Owned

### 2.1 User Activation Track

| Stage | CHI Range | Agent Responsibility |
|-------|-----------|---------------------|
| **Onboarding** | Pre-CHI | Milestone tracking, guided setup sequences, time-to-value monitoring |
| **Low CHI** | 0–2 | Automated re-engagement nudges, feature adoption prompts, human escalation if no improvement in 14 days |
| **Medium CHI** | 2–8 | Stability monitoring, feature breadth expansion nudges, expansion opportunity detection |
| **High CHI** | 8–20 | Expansion targeting, advocate identification, case study candidacy |
| **Advocate** | 20+ | Referral program activation, advocate recognition, community engagement |

### 2.2 Campus Activation Track

| Stage | CHI Range | Agent Responsibility |
|-------|-----------|---------------------|
| **Onboarding** | Pre-CHI | Inventory publish tracking, agency connection mapping |
| **Low CHI** | 0–2 | Agency activity monitoring, outreach to drive quoting volume |
| **Medium CHI** | 2–10 | Booking volume growth, pricing freshness monitoring |
| **High CHI** | 10–30 | Multi-campus expansion targeting, network growth acceleration |
| **Advocate** | 30+ | School-to-school referrals, agency sponsorship opportunities |

> **Note:** CHI thresholds shown above are the working values. Final bucket thresholds are pending finalization — this is an active blocker. When thresholds are confirmed, update this document and the N8N workflow configurations simultaneously.

### 2.3 Churn Risk (Cross-Cutting)

| Signal | Action |
|--------|--------|
| CHI drops below Low threshold | Automated re-engagement sequence. Alert Community Manager. |
| CHI declining for 2+ consecutive weeks | Escalate to Community Manager with trend data. |
| Cancellation language detected (Pylon ticket) | Immediate escalation to Ben + Community Manager. No automated response. |
| Competitor mention (Pylon ticket or UserPilot event) | Alert Ben + Community Manager with full context. |
| Stripe payment failure | Alert Community Manager + Phil (Finance). Trigger payment recovery sequence. |

---

## 3. Signal Sources

| Source | Signals Consumed | Tool |
|--------|-----------------|------|
| Snowflake | Weekly CHI scores (User CHI + Campus CHI), activity metrics (sessions, quotes, bookings, payments), MRR, churn cohort data | Snowflake → N8N |
| UserPilot | Onboarding flow completion, feature adoption events, NPS responses, session frequency, referral triggers | UserPilot → N8N |
| Stripe | Subscription status, payment success/failure, plan upgrades/downgrades, renewal dates | Stripe → N8N |
| Pylon | Support ticket volume, resolution times, CSAT scores, ticket sentiment (cancellation/competitor keywords) | Pylon → N8N |
| Attio | Account owner, MRR, persona, market, deal history, contact records | Attio → N8N |
| Edvisor Platform | Booking volumes, quote volumes, agency connections, campus activity | Platform → Snowflake → N8N |

---

## 4. Trigger Conditions & Actions

### 4.1 Onboarding Orchestration (COMPASS)

| Trigger | Condition | Action |
|---------|-----------|--------|
| Win detected | Stripe subscription created, Attio deal = "Win" | Determine onboarding tier by persona. Launch onboarding sequence. Create Linear task for CS. Set onboarding target date. |
| Onboarding milestone missed | Target date passed without milestone completion | Send nudge (in-app via UserPilot for agencies, email draft for educators). If 2+ milestones missed → escalate to Community Manager. |
| Onboarding complete | All milestones checked | Update Attio to "Onboarded." Begin CHI monitoring. Trigger "welcome to the platform" celebration message. |

**Onboarding Tiers & Timelines:**

| Persona | Target Time | Milestones | Handling |
|---------|-------------|------------|---------|
| A3 (Micro) / E1 (Boutique) | 7 days | Account setup, first quote (A3) or inventory published (E1) | Self-serve with in-app guidance (UserPilot) |
| A1 (Language) / E5 (Junior) / E6 (Career) | 14 days | Account setup, team invited, 10+ quotes (A1) or inventory + 1 agency connection (E5/E6) | Guided with automated check-ins + Community Manager available |
| A2 (HE) / A4 (Enterprise) / E2 (Chain) / E4 (University) | 30 days | Account setup, team trained, workflows running, data migrated | Dedicated Community Manager. Agent generates onboarding plan; human executes. |

### 4.2 CHI Monitoring & Re-Engagement (PULSE)

| Trigger | Condition | Action |
|---------|-----------|--------|
| Weekly CHI calculation | Every Monday (Snowflake → N8N) | Update User CHI and Campus CHI in Attio. Compare to previous week. Flag any account with >20% CHI decline. |
| CHI enters "Low" range | User CHI <2 or Campus CHI <2 | **Tier 1 response (automated):** In-app nudge (UserPilot) highlighting underused features. Email draft with "getting more value" angle. |
| CHI stays "Low" for 14+ days | Two consecutive weekly calculations in Low range | **Tier 2 response (human):** Alert Community Manager via Slack with full CHI breakdown, activity history, and recommended talking points. Create Linear task. |
| CHI drops below threshold AND MRR ≥$500/mo | High-value account enters churn risk | **Tier 3 response (immediate human):** Alert Ben + Community Manager. No automated outreach. Generate intervention brief with CHI trend, support history, and recommended approach. |
| CSAT <3 on any ticket | Pylon ticket with CSAT 1 or 2 | Alert Community Manager + Support lead (Moreno). Generate context brief. Flag for priority follow-up. |
| Negative sentiment detected | Pylon ticket contains cancellation/competitor keywords | Immediate alert to Ben + Community Manager. No automated response. Agent provides context only. |

**CHI Composition (for context — calculated in Snowflake, not by the agent):**

User CHI measures engagement with user-side features (quoting, applying, booking, payments), normalized by the number of paid users. Campus CHI measures engagement with campus-side features (being quoted, receiving applications, receiving sales), normalized by the number of campuses. Each has its own Success Score. If Paid Users = 0, User CHI = 0. Free user activity is excluded from User CHI calculations.

### 4.3 Retention & Renewal (ANCHOR)

| Trigger | Condition | Action |
|---------|-----------|--------|
| Renewal approaching (T-60 days) | Stripe: annual subscription renewal in 60 days | Generate renewal preparation brief for Community Manager: CHI trend, MRR, feature adoption, support history, expansion opportunities. Alert via Slack. |
| Renewal approaching (T-30 days) | Stripe: annual renewal in 30 days | Draft renewal outreach email. If CHI is High/Advocate → include expansion offer. If CHI is Medium → standard renewal. If CHI is Low → escalate to Community Manager for personal outreach. |
| Payment failure | Stripe: payment_failed event | **Immediate:** Alert Community Manager + Phil (Finance) via Slack. Draft payment recovery email. Retry logic managed by Stripe; agent monitors and escalates if unresolved after 3 retries. |
| Plan downgrade detected | Stripe: subscription changed to lower tier | Alert Community Manager. Generate context: what changed, CHI trend, recent support interactions. Flag as potential churn precursor. |

### 4.4 Expansion (ANCHOR)

| Trigger | Condition | Action |
|---------|-----------|--------|
| CHI ≥ High threshold for 30+ days | Sustained High CHI (User CHI ≥8 for users, Campus CHI ≥10 for campuses) | Flag as expansion opportunity. Generate expansion brief: current plan, usage vs. plan limits, recommended upsell. Alert Community Manager. |
| Tier ceiling approaching | UserPilot: usage approaching plan limits (user seats, campus count) | Trigger in-app upsell prompt (UserPilot). Draft upsell email for Community Manager review. |
| Single campus live, multi-campus potential | Attio: account has 1 campus but company profile indicates multiple locations | Generate multi-campus outreach draft. Alert Community Manager. |
| Transactional adoption gap | Account quotes/books but doesn't use insurance or EdWallet payments | Generate transactional adoption campaign: highlight insurance integration, payment plan features. In-app prompt (UserPilot) + email draft. |

### 4.5 Advocacy & Referral (CHAMPION)

| Trigger | Condition | Action |
|---------|-----------|--------|
| CHI enters Advocate range | User CHI ≥20 or Campus CHI ≥30, sustained 30+ days | Add to Advocate segment in Attio. Trigger referral prompt at next peak satisfaction moment. Notify Community Manager. |
| NPS score 9–10 | UserPilot NPS response | Trigger immediate referral ask (in-app via UserPilot). Log NPS in Attio. Notify Marketing Agent for case study candidacy. |
| Referral submitted | Advocate uses referral link | Track in Attio. Notify Sales Agent of incoming referred lead. Queue reward fulfillment (EdSummit ticket, featured placement, or co-marketing spotlight). |
| Case study candidate identified | CHI ≥ High for 60+ days AND NPS 9–10 | Alert Marketing Agent. Generate case study brief with before/after metrics and usage data. |

---

## 5. Tools & Integrations

| Tool | Usage |
|------|-------|
| **N8N** | Orchestration layer. All triggers, monitoring cycles, and action sequencing. |
| **Attio** | Account health data, CHI fields (written by N8N from Snowflake), persona, MRR, contact records. Stage transitions are N8N-controlled. |
| **Snowflake** | CHI calculation source, activity metrics, cohort analysis, renewal data. Read-only for agent. |
| **UserPilot** | In-app guidance (onboarding flows, feature nudges, NPS, referral prompts, upsell triggers). Write access for in-app actions. |
| **Stripe** | Subscription status, renewal dates, payment events. Read-only. |
| **Pylon** | Support ticket monitoring, CSAT, sentiment detection. Read-only. |
| **Postmark** | Email delivery for renewal, re-engagement, expansion outreach. Sender: `support@edvisor.io`. |
| **Slack** | Community Manager alerts, escalations, weekly CHI digests. Channel: `#master-machine-customerops`. |
| **Linear** | Task creation for onboarding, retention interventions, expansion follow-ups. |

---

## 6. Escalation & Handoff Protocols

### 6.1 Agent → Human

| Scenario | Escalation Target | Channel | Response SLA |
|----------|------------------|---------|-------------|
| CHI <Low threshold + MRR ≥$500/mo | Ben + Community Manager | `#master-machine-customerops` | Same business day |
| Cancellation language in Pylon ticket | Ben + Community Manager | `#master-machine-customerops` | 2 hours |
| Payment failure unresolved after 3 Stripe retries | Community Manager + Phil | `#master-machine-customerops` | Same business day |
| Onboarding stalled (enterprise tier, 2+ milestones missed) | Community Manager | Slack DM | Next business day |
| Competitor mention | Ben + Community Manager | `#master-machine-customerops` | Same business day |
| Any external communication draft | Community Manager (or Ben for ≥$500 MRR) | Slack DM | Before next business day |

### 6.2 Agent → Agent Handoffs

| Direction | Trigger | Data Passed |
|-----------|---------|------------|
| **Sales → Success** | Win (subscription created) | `deal_value`, `persona`, `onboarding_tier`, `trial_activity_summary`, `key_contacts`, `demo_notes` |
| **Success → Sales** | Expansion opportunity requiring new deal | `current_plan`, `recommended_upsell`, `CHI_trend`, `usage_data` |
| **Success → Marketing** | Advocate identified or case study candidate | `account_id`, `CHI_score`, `NPS`, `tenure`, `usage_highlights` |
| **Success → Marketing** | Account lost (churned) | `churn_reason`, `mrr_at_churn`, `CHI_at_churn`, `tenure`, `last_interaction` |
| **Support → Success** | Resolved ticket reveals health concern | `ticket_summary`, `CSAT`, `account_CHI` |

---

## 7. Weekly Operational Rhythm

| Day | Agent Action |
|-----|-------------|
| **Monday** | CHI recalculation (Snowflake → N8N → Attio). Flag accounts with significant changes. Generate weekly CHI digest for each Community Manager. |
| **Tuesday** | Onboarding progress check. Flag stalled onboarding accounts. Generate nudge sequences. |
| **Wednesday** | Expansion scan. Identify accounts hitting tier ceilings or sustained High CHI. Generate expansion briefs. |
| **Thursday** | Renewal scan (T-60 and T-30 accounts). Generate renewal briefs. Payment failure follow-up check. |
| **Friday** | Weekly summary: new advocates identified, churn risks flagged, expansion opportunities, onboarding completion rate. Post to `#master-machine-customerops` and individual Community Manager DMs. |

---

## 8. Governance Rules

1. **N8N controls all Attio stage assignments.** The Success Agent does not write stage fields. CHI values are written to Attio by N8N from Snowflake calculations.
2. **Supervised classification.** All externally-sent communications (email, WhatsApp) require Community Manager or Ben approval.
3. **Churn-risk = human-only.** When an account is flagged as churn risk (CHI <Low + declining trend OR cancellation language), the agent provides context and recommendations but does **not** send automated outreach. Humans handle all churn-risk conversations.
4. **High-ACV threshold: ≥$500/mo MRR.** Any automated action for accounts at or above this threshold requires Ben's explicit approval. This is a configurable threshold — adjust as the customer base evolves.
5. **No data silos.** All CHI changes, escalations, onboarding events, and renewal outcomes flow to Snowflake.
6. **Community Manager assignment.** Accounts are assigned to Community Managers by market (Mexico, Colombia, Brazil, Europe, Australia, Turkey). The agent respects these assignments for all routing.

---

## 9. KPIs & Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to onboard (A3/E1) | ≤7 days | Attio milestone tracking |
| Time to onboard (A1/E5/E6) | ≤14 days | Attio milestone tracking |
| Time to onboard (A2/A4/E2/E4) | ≤30 days | Attio milestone tracking |
| Onboarded → Activated rate | TBD (pending CHI finalization) | Attio stage transitions |
| CHI decline detection → human alert | <4 hours | N8N execution logs |
| Churn rate (monthly) | Reduce by 1pt/quarter | Stripe + Snowflake |
| Expansion revenue (upsell MRR) | Grow quarter-over-quarter | Stripe |
| NPS response rate | ≥30% of prompted users | UserPilot |
| Advocate conversion rate | TBD (pending advocacy program) | Attio segment tracking |
| Weekly digest delivery | 100% on-time (Monday by 10am UTC) | N8N execution logs |

---

## 10. Promotion Path

| Sub-workflow | Promotion Criteria |
|-------------|-------------------|
| Onboarding nudges (A1/A3 self-serve tier) | 0 approval rejections in 30 days + onboarding completion rate ≥80% |
| In-app feature adoption prompts (UserPilot) | <5% Community Manager override rate over 30 days |
| Weekly CHI digest generation | 0 errors in 30 days |
| Renewal reminder emails (Medium/High CHI only) | <10% edit rate on drafts over 30 days |

**Permanently Supervised (never promoted):** Churn-risk communications, high-ACV account outreach, cancellation intervention.

---

## 11. Dependencies & Blockers

| Dependency | Status | Impact |
|------------|--------|--------|
| **CHI threshold finalization** | 🔄 In progress — **CRITICAL BLOCKER** | Blocks dashboard build, stage assignment logic, all CHI-based triggers |
| HubSpot decommission (notes/email history) | 🔄 In progress | Historical interaction data needed for retention context |
| Pylon migration | 🔄 In progress | Blocks support signal integration (CSAT, sentiment detection) |
| CS team Attio transition | 🔄 In progress | Community Managers need Attio proficiency for agent output consumption |
| UserPilot onboarding flows | ✅ Complete | — |
| Snowflake → N8N → Attio pipeline | ✅ Complete | — |
| Stripe → N8N connection | ✅ Complete | — |

---

## 12. Relationship to Legacy Agent Codenames

| Legacy Codename | Capabilities Absorbed |
|----------------|----------------------|
| COMPASS 🧭 | Onboarding orchestration, milestone tracking, time-to-value monitoring |
| PULSE 📡 | CHI monitoring, weekly health calculations, re-engagement triggers |
| ANCHOR ⚓ | Retention workflows, renewal management, expansion detection |
| CHAMPION 🏆 | Advocacy identification, referral program activation, case study candidacy |
| PHOENIX 🔥 (partial) | Re-engagement of declining-but-active accounts (pre-churn intervention) |

**PHOENIX boundary:** The Customer Success Agent handles re-engagement of **active** accounts whose CHI is declining (still paying, still on platform, but usage dropping). The Marketing Agent handles outreach to **churned** accounts (no active subscription). The Sales Agent handles **conversion** of warmed churned accounts back through the funnel.
