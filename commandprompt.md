## PROJECT CONTEXT: Master Machine Implementation — Edvisor

### WHO YOU'RE WORKING WITH
Frederick is an AI Harness Engineer (https://openai.com/index/harness-engineering/) at Edvisor, an international education technology company (EdTech SaaS) serving two customer personas:
- **Agencies** (recruiting agencies placing students in schools) — B2C model, MRR based on user seats
- **Campuses** (schools/institutions listing programs) — B2B model, MRR based on campus inventory listings

Markets: LATAM, Brazil, Spain, Turkey, and other international regions. Multi-language operations (English, Spanish, Portuguese).

---

### THE PROJECT: "The Master Machine"
A full transformation from **people-driven to system-driven operations**. The goal is a self-scaling growth engine that automates the entire customer journey — from lead capture through advocacy — with minimal human intervention.

**North Star:** Build a funnel-driven, self-scaling business with automated communication, health scoring, churn prevention, and referral loops operating 24/7.

---

### CORE TECH STACK
| Tool | Role |
|------|------|
| **Attio** | Primary CRM — pipeline, contacts, account data. All signal data surfaces here for Sales/CS |
| **N8N** | Central brain — processes all signals, orchestrates automations, alerts, tasks |
| **Snowflake** | Data warehouse — sessions, quotes, bookings, sales, payments, logins; source for CHI scores |
| **UserPilot** | Product analytics + in-app messaging; tracks activation signals |
| **Pylon** | Customer support ticketing (replaced Intercom); AI-assisted ticket triage |
| **Postmark** | Transactional email delivery |
| **Webflow** | Marketing site + lead capture forms |
| **Stripe** | Payments, subscriptions, MRR |
| **Linear** | Task management for human + AI workflows |
| **Watchtower** | BI dashboards built in-house at Edvisor |
| **Slack** | Internal alerts when human intervention is required |

**Being decommissioned:** HubSpot (CRM, marketing, support, finance modules), Metabase, Jira.

---

### TEAM & OWNERSHIP
| Person | Role | Domain |
|--------|------|--------|
| **Danilo** | PM + Automation Lead | N8N architecture, signal capture, campaign automation |
| **Fred** | CRM Migration Lead | Attio setup, data clean-up, HubSpot decommission |
| **Juan G** | Success & Marketing Director | CS team adoption, Webflow, lead gen |
| **Ben** | Sales and Success Director | Sales and Success team adoption of Attio, value prop |
| **Moreno** | Head of Support | Pylon migration and adoption |
| **Phil** | Finance Director | Finance automation verification |

---

### THE UNIVERSAL GROWTH FUNNEL
The customer journey is defined by these stages, all controlled by N8N (never manually):

| Stage | Definition | CHI Score | Owner |
|-------|-----------|-----------|-------|
| Lead | Initial contact captured | — | System |
| Demoed | Received demo | — | Sales |
| On Trial | Active trial | — | System + Sales |
| Win | Converted to paying | — | Customer Success |
| Onboarded | Setup complete | 0–99 | CS |
| Low Activity | — | 100–299 | System (churn prevention) |
| Medium Activity | — | 300–499 | System (growth nurture) |
| High Activity | — | 500–799 | System (upsell) |
| Advocate | High engagement + referrals | 800+ | CS |
| Churn Risk | CHI drops critically | — | CS (urgent) |
| Lost | Churned | — | CS (post-mortem) |

**🚨 GOVERNANCE RULE:** Funnel stages are EXCLUSIVELY controlled by N8N. No human can manually change a funnel stage in Attio. This is non-negotiable for system integrity.

---

### CUSTOMER HEALTH INDEX (CHI)
- **What it is:** A composite score measuring product engagement, feature adoption, and usage
- **Two score types:**
  - **Agency CHI** — based on user seat MRR (B2C)
  - **Campus CHI** — based on campus listing MRR (B2B)
- **Calculated from:** Sessions, Quotes, Bookings, Sales, Payments, Insurance activity, Logins (all from Snowflake)
- **Score flow:** Snowflake → N8N → Attio (as account field)
- **Visual tiers in Attio:** Red (low) → Orange (med) → Green (high) → Purple (advocate)
- **⚠️ PENDING:** CHI bucket threshold ranges (exact point ranges for each tier) are not yet finalized. This is a blocker for dashboard development.

---

### SIGNAL ARCHITECTURE (N8N)
N8N ingests signals from:
- **Webflow** — form submissions, page visits (HIGH priority)
- **Attio** — deal events, notes, tasks (HIGH)
- **Stripe** — payments, churn, upgrades (HIGH)
- **UserPilot** — sessions, quotes, sales, bookings (HIGH)
- **Pylon** — support tickets, CSAT scores (MEDIUM)
- **Snowflake** — CHI scores, product usage (HIGH)

N8N outputs:
- Funnel stage updates → Attio
- Slack alerts → team (when human action required)
- Task creation → Linear
- Automated messages → UserPilot (in-app) or Postmark (email)
- Data push → Attio account fields (CHI, MRR, activity metrics)

**Automated Slack alerts fire when:**
- Lead submits demo form → Sales (regional owner)
- No trial activity for 5 days → Sales (account owner)
- CHI drops below 25 → CS (account owner) — churn risk
- Stripe payment failure → CS + Finance
- CHI at 85+ for 30 consecutive days → CS — expansion opportunity
- CSAT below 3 or negative ticket language → CS + Support Manager

---

### CURRENT STATUS (as of project handoff)
✅ **Completed:**
- Attio CRM migration — customer data successfully transferred
- N8N automation workflows live and routing signals
- UserPilot ↔ Attio connection established
- Universal Growth Funnel SOP documented and published
- CS team transition guide from HubSpot to Attio published
- Team roles, escalation paths, and Attio governance rules defined

🔄 **In Progress:**
- HubSpot decommission — final phase, not yet complete
- CHI score calculation methodology — threshold/bucket ranges pending decision
- CS team operational procedures within Attio — active transition
- Pylon support migration (Moreno)

⏳ **Next Priorities (unblock in this order):**
1. Finalize CHI score bucket thresholds → unblocks dashboard development
2. Complete HubSpot decommission
3. Build email sequence automation (multi-language: EN, ES, PT)
4. Expand N8N signal coverage to full customer journey
5. Activate churn prevention sequences and referral/advocate loops
6. Hex dashboard buildout for leadership KPIs

---

### ATTIO GOVERNANCE (CRITICAL)
**NEVER ALLOWED:**
- Manually changing funnel stages
- Creating custom fields or modifying pipeline config
- Creating workflow automations
- Editing system-generated fields (CHI, MRR, Activity)
- Changing account owners directly

**ALWAYS ALLOWED:**
- Adding notes to account records
- Updating contact information
- Viewing metrics
- Completing assigned tasks
- Creating reminders
- Searching and filtering

---

### KEY KPI TARGETS
- Lead → Demo conversion: ≥25%
- Demo → Trial: ≥60%
- Trial → Win: ≥35%
- Time to First Value: ≤3 days
- N8N Workflow Success Rate: ≥99.5% daily
- Signal Accuracy Rate: ≥98%
- Attio Data Quality Score: ≥97%

---

### GUIDING PRINCIPLES
1. **System over people** — automation drives the funnel; humans execute assigned tasks only
2. **Speed of implementation** — agility and rapid iteration are essential; don't over-engineer
3. **Signal completeness** — more signal sources = better decisions by N8N
4. **Governance integrity** — any manual override of system data corrupts the entire model
5. **Segment separately** — Agency and Campus personas need distinct messaging paths but can share one unified pipeline structure via attribute-based filtering
6. **Phase sequentially** — infrastructure first, then automation layers, then advanced intelligence

---

### ESCALATION CHANNELS
| Issue | Contact | Channel |
|-------|---------|---------|
| Attio access/setup | Fred | #master-machine-customerops |
| Data migration | Fred | #master-machine-customerops |
| N8N automation | Danilo | #master-machine-customerops |
| Workflow questions | Danilo | #master-machine-customerops |



