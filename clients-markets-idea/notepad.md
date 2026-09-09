# Clients & Markets — Idea Notepad

> **Status: Exploratory. Nothing here is confirmed. Pre-meeting notes only.**
> Last updated: August 14, 2026

---

## Source Material — Verbatim (Notes Received)

> This is the original list received. Best reference available at this stage.

```
Opportunities:

Risk Management (CRA, TPRA, SANs, ERAs)(CRM/Sentinel/CEAC)
RFP agent (doc extraction from FM or portal, summarise key info and send to relevant KPMG stakeholder to qualify)
Here's what you should do today for CRM opps


POCs

2a RFP agent - doc extraction from FM email and send content to AI

2b RFP agent - doc extraction from portal and send content to AI


Actions (if we proceed)

To validate opportunities 1 and 3 we need to get a series of screenshots of the systems to see the data
to Campaigns team: What data analytics do you see? Who is it shared with?
CRM stage gate and key actions info from CRM team
```

---

## Context

Exploring a potential AI agent project for a corporate sales / Clients & Markets team (likely KPMG internal). Still reviewing packs and materials to get up to speed before an upcoming meeting.

---

## Key Terms & Understanding

### Risk Management

Pre-sale compliance and acceptance gates:

| Term | Meaning |
|------|---------|
| **CRA** | Client Risk Assessment — is this client acceptable to take on? |
| **TPRA** | Third Party Risk Assessment — vetting subcontractors/partners involved in delivery |
| **SANs** | Specific Acceptance Notes — formal documented exceptions or conditions on engagement acceptance |
| **ERAs** | Engagement Risk Assessments — risk review at the individual project level |

Internal systems that run these processes:

| System | Meaning |
|--------|---------|
| **CRM** | Client Relationship Management system (e.g. Salesforce) — tracks pipeline, opps, clients |
| **Sentinel** | KPMG's internal client/engagement acceptance tool — where risk assessments are submitted and approved |
| **CEAC** | Client and Engagement Acceptance & Continuance — the overall process; Sentinel is the tool that runs it |

---

### RFP Agent

**RFP = Request for Proposal** — a formal document issued by a client/prospect inviting consultants to bid on work.

Proposed agent purpose: automate early-stage RFP triage so the right KPMG stakeholder can quickly qualify or pass.

**Workflow (as understood so far):**
1. Receive / extract RFP document (from portal or FM email — see open questions)
2. Parse key info: client name, industry, service line, deal size, deadline, evaluation criteria
3. Look up client in CRM — existing relationship? Who owns it? Which sector/practice?
4. Route RFP summary to the correct partner/director/pursuit lead
5. That stakeholder decides: qualify (pursue) or no-qualify (pass)

**Why CRM access matters:**
- Acts as the routing table — maps incoming RFP to the right person
- Avoids conflict where multiple partners think they own the same client
- Informs response strategy (new logo vs. existing relationship)

---

### "Here's what you should do today for CRM opps"

Likely an AI-generated daily digest for sellers/pursuit managers surfacing:
- Opportunities needing a stage update or next step logged
- Stale opps at risk
- Follow-up actions due today (calls, proposals, risk submissions)

---

## Working Assumption

The **PoC being built** centers on the two RFP agent workflows described in the meeting notes:
- One agent ingesting from a **client/procurement portal**
- One agent ingesting via **FM email**

Everything else in this notepad is background context only.

---

## Open Questions

1. **Single agent vs. two separate workflows?**
   The RFP agent has been described in two configurations:
   - One that pulls RFP content from a **client/procurement portal**
   - One that receives content via an **FM (file management?) email**

   It's unclear whether this will be:
   - One agent with dual ingestion capabilities (portal + email)
   - Two entirely separate workflows/agents

   *To clarify in the upcoming meeting.*

---

## Notes / Things to Read Up On

- [ ] Review packs/materials provided
- [x] Understand what "FM" stands for — **Functional Mailbox** (confirmed in meeting)
- [x] Understand if CRM in scope — **KPMG Dynamics 365** (confirmed in meeting)

---

## Meeting Notes — August 20, 2026

> **Anonymization key** (names not written to disk):
> Process Owner · AI Lead · AI Team · Senior Stakeholder · Engagement Lead · Internal Tech Consultant

### Attendees
- AI Lead (KPMG AI Solutions, leading PoC)
- AI Team (KPMG AI Solutions, Philippines)
- Process Owner (Fed Gov team — current manual process owner)
- Senior Stakeholder (Fed Gov team)
- Engagement Lead (Fed Gov team)

---
add a 
### The Human Process Being Automated — Process Owner's Workflow

This is the workflow the agent is intended to replicate or assist with. Process Owner currently owns this end-to-end.

**Step 1 — Monitor Functional Mailbox**
- Process Owner checks the functional mailbox (shared inbox) for incoming RFP emails from federal and ACT government clients.
- RFPs arrive as emails from government stakeholders / portals.
- Example shown: email RFP from Austrade.

**Step 2 — Identify the RFP source and access the document**
- ~90% of offers come through a client portal (not directly via email attachment).
- There are approximately **22 portals** across clients.
- Core process quote from Process Owner: *"I log into the portal → get the RFQ → circulate to team."*
- Login credentials vary per portal — some external tenders provide a passcode/password directly; some use the functional mailbox as login; some have separate credentials stored elsewhere.
- **Some portals have MFA**, which complicates automation.
- Portal behaviour is inconsistent: some require download, some don't; some provide direct links, some don't.
- Some portals (e.g. buy ICT) have a structured project page per RFP with metadata, documents, and Q&A — see Step 7.

**Step 3 — Fill out a routing form**
- Process Owner fills out a predefined circulation template with key fields extracted from the RFP/RFQ document.
- Fields in the template are driven by data in the RFQ that maps to the template structure.
- If the RFP is marked **confidential** (noted in the email), Process Owner appends this to the circulation template and manually limits distribution to only the relevant people. Senior Stakeholder noted ~99% of RFQs carry a confidential marker.

**Circulation Template Format (confirmed):**

The template is a **two-column table** — left column is the field name, right column is the value filled in by Process Owner. The email body essentially is this table.

| Field (left column) | Value populated (right column) | Source |
|---------------------|-------------------------------|--------|
| **Attn** | Names of CLPs/stakeholders who need to act | CLP cheat sheet / CRM |
| **RFQ from** | Government client / agency that sent the RFQ | Email sender / portal |
| **Response Due — Date/Time** | Deadline for KPMG to submit a response | RFQ document |
| **Requirement** | Short description of what is being procured + Teams SharePoint link to the docs | RFQ document + SharePoint |
| **From Panel** | The specific engagement panel the RFQ falls under | RFQ document / portal |
| **Industry Briefing** | Quick description of the client's industry | RFQ document / portal |
| **Deadline for Questions** | Deadline for submitting questions to the client | RFQ document |

> **Agent implication:** The agent must extract all seven field values from the RFQ document and/or portal page, then render them into this table structure for the outbound email. The **Requirement** field requires a SharePoint link — docs must be saved to Teams *before* the circulation email can be sent. The **Attn** field requires resolving the correct CLPs from the CRM or cheat sheet.

**Step 4 — Circulate the RFP**
- Two circulation layers:
  - **Broad list send**: Senior Stakeholder maintains a large email list of Associate Directors and above. Everyone on this list receives all RFQ circulations.
  - **CLP-specific attention**: Specific partners/directors assigned to the client are **called out by name** in the email if they are directly involved.
- CLPs are also noted in **KPMG Dynamics 365** (internal CRM — the canonical source).
- Process Owner also maintains a **personal cheat sheet** (Word doc) mapping specific projects to their assigned CLP. This is her own reference tool, not a system. Access to this doc is TBD — it will be critical for configuring the agent's sendout logic.
- The **canonical source** for project-to-person mapping is the CRM (Dynamics 365), not the cheat sheet.

**Step 5 — Save documents to Teams**
- RFP docs are **manually saved** by Process Owner into a Teams SharePoint folder for the team.
- Confirmed manual — no automation in place here.

**Step 6 — Handle Addenda**
- Clients frequently send addenda (additional information appended to an RFP) — can happen **up to ~10 times per RFP**.
- Addenda docs are uploaded to the same Teams SharePoint folder as the original RFP.
- Addenda arrive via email with a reference number; Process Owner uses the reference to locate and link back to the original RFP.

**Step 7 — Handle Q&A (buy ICT portal — confirmed)**
- Some portals (notably **buy ICT**) have a dedicated project page per RFP listing critical project information, with documents as downloadable attachments.
- This page also contains **Q&A specific to the project**, which gets updated over time.
- Process Owner's current workflow for this portal:
  1. Manually **prints the portal page** in-browser and adds it alongside the downloaded documents.
  2. Manually **copies the Q&A** from the portal page.
  3. **Re-circulates the updated Q&A** to the team each time it changes.
- This Q&A handling is a recurring, manual, high-effort task layered on top of the main RFP workflow.

---

### Document Types Received (Expected Package)

Varies by client, but typically includes:
- The RFP document itself
- Other supporting project and client documents

**Terminology note:**
| Term | Meaning |
|------|---------|
| RFP | Request for Proposal |
| RFQ | Request for Quotation — same as RFP in this context |
| RFT | Request for Tender — same as RFP in this context |
| EOI / RFI | Expression of Interest / Request for Information — *not* a procurement decision, just market sounding. Lower priority. |

---

### Confidentiality Handling

- ~99% of RFQs carry a **confidential marker** (confirmed by Senior Stakeholder).
- When confidential, Process Owner reads only the top section before routing.
- Confidentiality is noted in the email; Process Owner appends this flag to the circulation template and manually restricts distribution to only the relevant people.
- In some cases an **NDA is signed first**, and only then is the RFP released/sent.

---

### Key Challenges

| Challenge | Detail |
|-----------|--------|
| Portal inconsistency | ~22 portals, each with different login methods, document access flows, and download behaviour |
| Q&A recirculation | buy ICT portal requires Process Owner to manually print project pages and re-circulate updated Q&A each time it changes |
| Document saving | All docs saved manually to Teams SharePoint — no automation |
| CLP cheat sheet access | Process Owner's project-to-CLP Word doc is a personal artefact — access TBD; CRM is the canonical source but may be harder to query |
| Near-universal confidentiality | ~99% of RFQs are confidential — agent must handle restricted distribution by default, not as an edge case |
| MFA on some portals | Blocks standard RPA/automation approaches |
| Credential management | Credentials vary per portal; stored location unclear |
| Addendum linkage | Addendums arrive separately and must be matched to original RFP by reference number |
| Document formats | Different clients use different RFP formats |
| Confidential RFPs | Some require NDA before access — human judgment required |

---

### Priority Portals to Crack

Per Senior Stakeholder: if these two are solved, that covers the majority of volume.

| Portal | Used for |
|--------|---------|
| **AusTender** | Consulting services tenders |
| **buy ICT** | Technology-related services tenders |

> "90% of offers go through the portal" — Senior Stakeholder

---

### Senior Stakeholder's Goal

> **How much of Process Owner's workflow can we automate?**

- **Priority 1 — Crack the two main portals:** If we can automate AusTender and buy ICT, that would be an amazing start and covers the majority of volume.
- **Priority 2 — Optimise the inbox workflow:** Even if portal scraping/login automation isn't feasible, we can leave human data retrieval in place and focus on automating everything that happens once the information is already in the inbox — extraction, template population, routing, saving, tracking.

---

### CRM / CLP Context

- CRM in use: **KPMG Dynamics 365**
- CLPs (Client Lead Partners) are recorded in Dynamics 365
- AI Lead has access to the CRM test environment to investigate what data is present at each pipeline stage
- If opportunity data is stored in Fabric, a daily pull → email alert approach may be viable for the CRM nudge scenario

---

### Automation Path — Open Decision

Two candidate approaches for portal document acquisition:

| Approach | Notes |
|----------|-------|
| **RPA** (Robotic Process Automation) | May not be ready; to be validated with Internal Tech Consultant |
| **CUA** (Computer Use Agent) | AI-driven browser/UI automation — alternative if RPA isn't viable |

---

### Next Steps / Actions

- [x] Get time with Internal Tech Consultant — **meeting scheduled for next week**
- [ ] Walk Internal Tech Consultant through Process Owner's workflow
- [ ] Show Internal Tech Consultant the two priority portals: AusTender and buy ICT
- [ ] Show download links and portal access flows
- [ ] Key question for Internal Tech Consultant: what tech do we have available to handle portal login and document scraping? (RPA? CUA? other?)
- [x] Clarify how documents are saved to Teams — **confirmed manual** by Process Owner
- [ ] Obtain the CLP routing matrix (client → CLP mapping) — needed to configure agent sendouts
- [ ] Investigate CRM stage data in Dynamics 365 test environment
