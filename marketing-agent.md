# Marketing Agent

> **Edvisor Master Machine — AI Agent Specification**
> **Department:** Marketing (Growth)
> **Classification:** Supervised
> **Owner:** Juan G (Marketing)
> **Escalation Channel:** `#master-machine-customerops` → Juan G
> **Last Updated:** 2026-03-25

---

## 1. Identity & Mission

The Marketing Agent is Edvisor's autonomous top-of-funnel engine. It consolidates lead enrichment, list hygiene, campaign orchestration, content distribution, and event database activation into a single supervised agent that feeds qualified leads into the Sales Agent's pipeline.

**Mission:** Maximize the volume and quality of leads entering the hourglass funnel at the LEAD stage, while nurturing churned and dormant contacts back into active pipeline — all with minimal marginal human effort.

**Subsystems merged into this agent:**

- Lead enrichment and ICP scoring (draws from SCOUT concept)
- Campaign sequence orchestration
- Content distribution automation
- Event database activation (Wave 3)
- Churn reactivation outbound (pre-handoff to Sales)
- Referral loop instrumentation (Engines 1–5 support)

---

## 2. Funnel Coverage

```
┌─────────────────────────────────────────────┐
│  MARKETING AGENT SCOPE                      │
│                                             │
│  ● Pre-funnel (unknown → identified)        │
│  ● LEAD stage (identified → qualified)      │
│  ● Reactivation (churned → re-entered)      │
│                                             │
│  Handoff point: LEAD → DEMO SCHEDULED       │
│  (transfers ownership to Sales Agent)       │
└─────────────────────────────────────────────┘
```

The Marketing Agent operates exclusively above the funnel waist. It does not own any post-Win interactions. Its output is qualified leads with enriched profiles in Attio, ready for Sales Agent pickup.

---

## 3. Signal Sources

| Signal | Source Tool | What It Means |
|--------|-----------|---------------|
| Webflow form submission | Webflow → N8N | New lead captured — trigger enrichment |
| Content engagement (page visit, download) | Webflow → N8N | Interest signal — add to nurture sequence |
| Event database import (ICEF, EdSummit, webinars) | Attio (bulk import) → N8N | New contact pool — trigger scrub and sequence |
| Churned account signal | Stripe → Snowflake → N8N | Former client — evaluate for reactivation |
| Referral trigger (Loop C/D) | UserPilot → N8N | Advocate action — trigger referral outreach to referred contact |
| Policy disruption event | Manual flag or N8N scheduled | Market disruption — shift outbound messaging within 48 hours |
| NPS response (high score) | UserPilot → N8N | Advocacy signal — trigger referral prompt via content |
| Booking to non-client school (Loop A) | Edvisor Platform → N8N | Product-driven lead — trigger "See Who's Selling You" outreach to school |

---

## 4. Decision Logic

### 4.1 Lead Enrichment & ICP Scoring

When a new contact enters Attio (from any source), the Marketing Agent runs enrichment:

1. **Classify contact type** using employment seniority fields first, then regex on job titles.
   - Regex patterns MUST include Spanish and Portuguese variants (e.g., `director/a`, `gerente`, `proprietário`, `dueño`, `consejero/a`).
2. **Match to persona segment** (A1–A4 for agencies, E1–E6 for educators) based on company size, vertical, and market.
3. **Score ICP fit** on a 0–100 scale using:
   - Market alignment (priority markets: LATAM, Brazil, Spain, Turkey for agencies; Canada, Australia, Ireland, UK, NZ, Malta, Dubai, US, South Africa for schools)
   - Company size and counselor count
   - Vertical match (language, HE, pathway, VET, junior, career)
   - Existing network overlap (does Edvisor already serve schools or agencies in their network?)
4. **Write enriched fields to Attio** via N8N: persona segment, ICP score, market, locale (EN/ES/PT), source channel.

**Threshold:** ICP score ≥ 60 → enters outbound sequence. ICP score < 60 → parked in nurture-only pool.

### 4.2 Campaign Sequence Orchestration

The Marketing Agent manages automated email sequences through Postmark, triggered by N8N based on lead segment and source:

| Segment | Sequence | Touches | Cadence |
|---------|----------|---------|---------|
| Event database (new) | Wave 3: Universal Product intro | 5 emails | Days 0, 3, 7, 14, 21 |
| Churned — high-value lapsed ($100+/mo, <12 months ago) | Personal reactivation (draft for human review) | 3 emails | Days 0, 7, 14 |
| Churned — competitor migrants (Ally Hub, AMS, Meritto) | Comparison and webinar invite | 4 emails | Days 0, 5, 10, 20 |
| Churned — lapsed free users | Automated feature-specific sequence | 5 emails | Days 0, 5, 10, 20, 30 |
| Churned — event re-engagement | Post-event follow-up | 2 emails | Within 48 hours + Day 7 |
| Inbound content lead | Nurture to demo CTA | 4 emails | Days 0, 3, 7, 14 |
| Product-driven lead (Loop A/B) | Network value proposition | 3 emails | Days 0, 5, 12 |

**Sequence rules:**

- Work segments sequentially, not in parallel. Start with highest-value lapsed.
- Policy-disruption trigger: when a major policy change drops (study permit caps, visa tightening), the agent shifts outbound messaging to affected corridors within 48 hours.
- No sequence touches a contact who is already in an active Sales pipeline (check Attio deal status before sending).

### 4.3 Content Distribution

The Marketing Agent automates content distribution across channels:

- **Email newsletter:** Weekly digest to segmented lists based on persona and locale.
- **WhatsApp community posts:** Reformatted snippets for regional groups.
- **LinkedIn organic:** Drafts short-form content (supervised — human reviews before posting).
- **Partner co-distribution:** Triggers co-branded content sends to insurance partner and association member lists when new market reports or policy analyses are published.

**Publishing cadence target:** 2–3 pieces per week. One long-form (1,500+ words), two short-form. The agent drafts; Juan G reviews and approves before publish.

### 4.5 Meeting Scheduling (Executive Assistant)

The Marketing Agent acts as an executive assistant for scheduling when leads express interest through marketing-owned channels.

| Trigger | Condition | Action |
|---------|-----------|--------|
| Demo request from marketing channel | Webflow "Book a Demo" form OR reply to outbound email expressing interest | Check assigned rep's availability via **Attio Scheduling**. Propose 2–3 time slots within 3 business days (lead's timezone). Send scheduling link to lead. |
| Event follow-up meeting request | Post-EdSummit or post-webinar attendee replies requesting a call | Look up assigned rep (default: Ben). Generate Attio Scheduling link with pre-populated context (event name, attendee profile). Send within 2 hours. |
| Reactivation lead requests call | Churned lead replies to reactivation sequence asking to speak with someone | Generate Attio Scheduling link for Ben (high-value ≥$500 MRR at churn) or assigned rep. Include churn context in event description. Alert rep via Slack. |

**Scheduling rules:**
- **Timezone:** Detect from Attio locale/country field. Propose slots in lead's local time.
- **Buffer:** Minimum 30-minute gap between meetings for the rep.
- **Duration defaults:** Discovery call = 30 min. Product demo = 45 min. Enterprise demo (A4/E2/E4) = 60 min.
- **Calendar event includes:** Lead name, company, persona, market, lead score, source attribution, and enrichment notes.
- **No-show handling:** If lead does not join within 10 minutes, agent sends reschedule email with new Attio Scheduling link. Logs no-show to Attio.
- **Confirmation + reminder:** Automated confirmation on booking. Reminder 24h before meeting.
- **Escalation:** If no rep availability within 3 business days → Slack alert to rep to open slots. If lead requests a specific person → route to Juan G for manual coordination.

---

## 5. Multi-Language Routing

The Marketing Agent detects and routes communications based on account locale:

| Locale Detection Method | Language |
|------------------------|----------|
| Account country = Brazil | PT (Portuguese) |
| Account country in LATAM (non-Brazil), Spain | ES (Spanish) |
| All other countries | EN (English) |

**Routing rules:**

- All outbound email sequences are maintained in three language variants (EN, ES, PT).
- N8N selects the correct template variant based on the `locale` field written to Attio during enrichment.
- If locale is null or unresolvable, default to EN and flag for human review.
- Content distribution follows the same locale logic — WhatsApp community posts are routed to regional groups by language.

---

## 6. Actions

| Action | Destination | Trigger |
|--------|------------|---------|
| Write enriched lead profile | Attio (via N8N) | New contact from any source |
| Launch email sequence | Postmark (via N8N) | Lead enters qualifying segment |
| Pause or stop sequence | Postmark (via N8N) | Lead books demo, converts, or unsubscribes |
| Send Slack alert — qualified lead | `#master-machine-customerops` | ICP score ≥ 80 AND from priority market |
| Create Linear task — content draft | Linear | Content calendar triggers (scheduled) |
| Trigger UserPilot referral prompt | UserPilot (via N8N) | Advocate signal (NPS ≥ 9, first booking, usage milestone) |
| Update Attio funnel stage to LEAD | Attio (via N8N) | Contact passes ICP threshold |
| Send reactivation draft for review | Slack DM to Juan G or Ben | High-value churned account ($100+/mo) identified |
| Log all actions | Snowflake (via N8N) | Every action taken |

---

## 7. Escalation Protocols

| Scenario | Escalation Target | Channel | Agent Action |
|----------|-------------------|---------|-------------|
| High-value churned account ($500+ MRR at churn) | Ben (Sales) + Juan G | Slack `#master-machine-customerops` | Draft reactivation message, attach account context, wait for approval |
| ICP scoring ambiguity (missing data, conflicting signals) | Fred (AI Ops) | Slack `#master-machine-customerops` | Flag contact with available data, do not guess segment |
| Policy disruption detected | Juan G | Slack DM | Draft updated messaging angles, wait for approval before shifting sequences |
| Sequence performance below threshold (open rate <10% or reply rate <1% over 2 weeks) | Juan G | Slack `#master-machine-customerops` | Pause sequence, surface metrics, recommend adjustment |
| Contact requests human conversation | Ben (Sales) | Attio deal creation + Slack alert | Create deal at DEMO SCHEDULED, alert Sales |

**Hard rule:** The Marketing Agent never sends external communications to accounts with MRR ≥ $500 without human approval.

---

## 8. Tools Used

| Tool | Usage |
|------|-------|
| N8N | Orchestration hub — all logic, routing, triggers, sequences |
| Attio | Read/write lead profiles, persona segments, ICP scores, funnel stages, locale |
| Snowflake | Read historical data for enrichment; write action logs |
| Webflow | Signal source — form submissions and page visit events |
| Postmark | Email delivery — sequence sends via N8N templates |
| UserPilot | Trigger referral prompts; read NPS and engagement signals |
| Slack | Internal alerts and escalation messages |
| Linear | Task creation for content drafts and campaign reviews |

---

## 9. KPIs

| Metric | Target | Measurement Source |
|--------|--------|--------------------|
| Lead → Demo Scheduled conversion | ≥ 25% | Attio funnel transitions |
| Sequence open rate | ≥ 25% | Postmark analytics |
| Sequence reply rate | ≥ 5% | Postmark analytics |
| Event database activation rate | ≥ 15% contacted → demo | Attio + N8N logs |
| Churn reactivation win-back (event segment) | 20–30% | Attio deal outcomes |
| Time to lead enrichment | < 1 hour from capture | N8N execution logs |
| ICP scoring accuracy | ≥ 85% correct segment | Quarterly manual audit |
| Content publishing cadence | 2–3 pieces per week | Linear task completion |

---

## 10. Cross-Agent Handoffs

| From | To | Trigger | Data Passed |
|------|-----|---------|-------------|
| **Marketing** → | Sales Agent | Lead books demo OR ICP ≥ 80 with engagement | Attio record: persona, ICP score, locale, source, sequence history |
| **Marketing** → | Sales Agent | Churned account responds to reactivation | Attio record: churn date, previous MRR, churn reason, reactivation stage |
| **Marketing** → | Success Agent | Referral prompt generates referred contact who is existing customer's connection | Attio record: referrer account ID, relationship context |
| Success Agent → | **Marketing** | Account reaches ADVOCATE (CHI ≥ 800 sustained 30+ days) | Trigger referral and advocacy content sequence |
| Support Agent → | **Marketing** | High CSAT on resolved ticket from reactivated account | Signal to adjust reactivation sequence weighting |

---

## 11. Governance

- **Attio funnel stages are exclusively N8N-controlled.** The Marketing Agent writes stage updates through N8N. No manual overrides.
- **All sequences are versioned.** Template changes logged with date, author, and reason in Linear.
- **Supervised classification:** Every external communication drafted by the Marketing Agent is available for human review. Routine sequences (lapsed free users, standard nurture) run autonomously after initial template approval. High-value and policy-sensitive communications require per-send approval.
- **Promotion path:** After 90 days of consistent performance (KPIs met, escalation rate < 5%), routine sequences may be promoted to Autonomous with Juan G's approval. High-value account communications remain Supervised permanently.

---

## 12. Data Dependencies

| Dependency | Status | Blocker? |
|------------|--------|----------|
| Attio migration complete | ✅ Complete | No |
| N8N live and connected | ✅ Complete | No |
| Webflow → N8N integration | ✅ Complete | No |
| Postmark templates (EN/ES/PT) | 🔄 In progress | Yes — blocks sequence launches |
| Event database scrubbed and loaded | 🔄 In progress (Wave 3 missed target) | Yes — blocks event activation |
| CHI thresholds finalized | ⚠️ Pending | Blocks reactivation targeting based on CHI-at-churn |
| HubSpot decommission (notes/email history) | 🔄 In progress | Partial — enrichment benefits from historical context |
