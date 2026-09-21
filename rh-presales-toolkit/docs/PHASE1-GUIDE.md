# Pre-Sales Toolkit — Phase 1 Guide
## Discovery Console + Recommendation Engine

Version 1.0 · 2026-07-20 · Schema v1.0

---

## 1. What Phase 1 delivers

Two single-file web apps, two sample engagement records, and this guide.

| File | Role |
|---|---|
| `discovery-console.html` | Structured intake for the full discovery workshop. Produces `engagement.json`. |
| `recommendation-engine.html` | Consumes `engagement.json`, scores Private/Public/Hybrid fit, recommends platforms, appends its output to the same file. |
| `samples/meridian-industries.engagement.json` | Generic enterprise walkthrough profile (default). |
| `samples/crescent-bank.engagement.json` | Banking walkthrough profile (regulated framing). |

Everything runs from a laptop file system or GitHub Pages. No server, no build step. The one external dependency is the docx library loaded from CDN for Word export; Print/PDF export works fully offline. To be fully offline for Word too, download `https://unpkg.com/docx@8.5.0/build/index.umd.js`, save it next to the HTML files, and change the `<script src=...>` line in each file to `src="index.umd.js"`.

### Deployment on GitHub Pages
1. Create a repo, drop the `rh-presales-toolkit` folder contents in.
2. Settings → Pages → deploy from branch → root.
3. Share the URL. Data never leaves the engineer's browser; nothing is posted anywhere.

---

## 2. Component specifications

### 2.1 Discovery Console

**Purpose.** Capture a complete, gap-flagged discovery record in one workshop, in a format every downstream component consumes. Kill the "notes in five notebooks" problem.

**Inputs.** Live client conversation; RVTools/CMDB extracts read out during the session; a previously exported `engagement.json` (via Import) to resume or revise.

**Outputs.**
- `engagement.json` (schema v1.0) — the portable record and the toolkit's data spine
- Discovery Summary as Word (.docx) and Print/PDF
- Autosave in browser localStorage (convenience only; the JSON export is the record)

**Sections.** 1 Engagement · 2 Qualification (BANT 0–20 gate + MEDDPICC 0–80) · 3 Executive discovery (drivers, outcomes, stakeholder map) · 4 Technical discovery (estate, workloads, container ambition, compliance, growth, DR) · 5 VMware estate (hosts, sockets, cores, VMs, vCenter/ESXi versions, products, ELA status, renewal date and quote, uplift) · 6 Review & export (gap flags, assumptions log).

**Scoring model.** BANT 4 items × 0–5, MEDDPICC 8 items × 0–10. Total 0–100. Bands: 80+ Commit, 60–79 Pursue, 40–59 Develop, below 40 Nurture. Working rule: below 8 on BANT, stop and requalify before spending workshop hours.

**Gap engine.** Every required field that is empty (respecting conditional visibility, e.g. VMware fields only when VMware is present) raises a flag, plus three qualitative checks: no champion in the stakeholder map, economic buyer scored ≤ 3, VMware present but no renewal quote captured. Gaps travel inside the JSON so downstream components can refuse to pretend the record is complete.

**Connects to.** Recommendation Engine (Phase 1), Blueprint Generator and BOQ Builder (Phase 2), TCO (Phase 3), Proposal Assembler (Phase 4). The computed core-pair figure shown in the VMware section is the bridge into subscription sizing.

### 2.2 Recommendation Engine

**Purpose.** Turn the discovery record into a defensible deployment model recommendation (Private / Public / Hybrid) and a platform recommendation, with every scoring point traceable to a named discovery field.

**Inputs.** `engagement.json` via file load, paste, or same-browser autosave.

**Outputs.**
- Updated `engagement.json` with a `recommendation` block (scores, model, platforms, justification, caveats, honesty flags, full contribution table)
- Recommendation document as Word (.docx) and Print/PDF

**How scoring works.** A visible rule table, additive points per dimension, clamped 0–100. No hidden weights. The contribution table in the UI and both exports lists rule, points, and source schema field, which is exactly what you hand a CTO who asks "why hybrid?"

**Positioning toggle behavior.** Set in the Discovery Console header, carried in `meta.positioning`.
- `redhat_default`: recommends OpenShift Virtualization Engine as the baseline for VM-dominant estates (upgrade path stated), OCP + OpenShift Virtualization where container ambition is committed, MTV when VMware is present, RHACM + RHACS (Platform Plus path) for multi-site or platform strategies, ROSA/ARO for public-with-containers.
- `neutral`: same model scoring, but the platform section becomes criteria-led shortlists (OpenShift Virtualization listed among peers, not ahead of them).

**Honesty checks (both lenses).** Small estate + no ambition + no compliance → "OpenShift is likely heavier than needed." VM-only public migration → "does not need OpenShift." Nurture-band qualification → "directional only, not a proposal basis." These print in the client-facing exports on purpose; credibility is the product.

**Connects to.** The updated JSON is the input for the Phase 2 Blueprint Generator and BOQ Builder.

---

## 3. Live workshop walkthrough — Meridian Industries

Fictional generic-enterprise profile. To follow along, open `discovery-console.html` and import `samples/meridian-industries.engagement.json`. To rehearse the banking variant, import the Crescent Bank file instead; the flow is identical, only the answers change.

**Setting.** 2.5-hour workshop. In the room: CIO (Farah), Infrastructure Manager (Adeel, your champion), Head of Applications (Sana). CFO (Imran) joins the last 30 minutes.

**Before the workshop (15 min).** Create the record: client name, industry, engagement ID `MER-202607-OCPV`, your name. Confirm positioning lens (Red Hat default here; you would flip to Neutral for a bank running a strictly vendor-agnostic RFP). Score BANT from the qualification call: budget 4 (renewal budget exists), authority 4 (CIO owns it), need 5 (65% uplift quote in hand), timeline 4 (renewal 31 Dec). 17/20 — proceed.

**Segment 1, executive framing (30 min, section 3).** Ask Farah for outcomes in her words and type them verbatim. Check drivers as they surface: cost reduction, VMware exit, modernization, vendor consolidation. Build the stakeholder map live but neutrally (do not read "Skeptic" aloud next to the CFO's name; keep dispositions for the debrief). Capture the November–January change freeze under constraints, it will shape the migration plan in Phase 3.

**Segment 2, technical estate (45 min, section 4).** Adeel has an RVTools export: 24 hosts, 420 VMs, vSAN plus a legacy FC SAN. Enter the workload table as groups, not individual VMs: ERP stays VM (Oracle sensitivity), MES is the containerization candidate (dev team already on Docker), dev/test is the obvious first migration wave. Container ambition: "Committed — some apps", 12 candidates. Compliance: ISO 27001 only, residency: none. DR: warm site, RPO 1 h, RTO 8 h.

**Segment 3, VMware estate (20 min, section 5).** The heart of this deal. 24 hosts × 2 sockets × 16 cores = 768 cores; the console shows ≈ 384 core-pairs, which you note aloud as the sizing bridge for the BOQ. vCenter 7.0 U3, ESXi 7.0.3 (past end of general support, an upgrade is unavoidable either way, which neutralizes "do nothing"). ELA expiring within 12 months, renewal 2026-12-31, quote 1,180,000 USD, uplift 65%. Log the quote's source in the assumptions table immediately: "Adeel Khan, quote email" tagged `input`. When the CFO later challenges the status quo cost, you point at that row.

**Segment 4, review with the CFO present (20 min, section 6).** Open Review & export. Walk the gap list aloud, this reads as rigor, not weakness: "We still owe you a confirmed growth figure; today's 10% is the CIO's planning number, logged as directional." Add remaining assumptions. Export JSON, and export the Word Discovery Summary as the meeting record you send within the hour.

**After the workshop (10 min).** Open `recommendation-engine.html`, load the JSON (or "Load last autosave"). Expected result for Meridian: Private fit leads (VMware exit driver, 420-VM scale, mission-critical workloads, warm DR), Hybrid close behind, Public low. Recommended platforms under the Red Hat lens: OCP + OpenShift Virtualization (ambition is Committed), MTV for migration. Read the honesty checks; for Meridian none should fire except caveats on any open gaps. Export the updated JSON — this is now the single file you carry into Phase 2 — and the Word recommendation for your champion to circulate.

**Crescent Bank contrast, one paragraph.** Same flow, different physics: residency mandated, SBP + PCI in scope, hot/hot DR, platform-level container strategy on an unsupported DIY Kubernetes. Private scores near ceiling, hybrid rises on the DR pattern, and the engine adds RHACM + RHACS with the Platform Plus note. The caveat about the missing renewal quote fires, matching the assumptions row that uses a 100% uplift midpoint tagged `directional`.

---

## 4. Engineer checklists

### Before the workshop
- [ ] Qualification call done, BANT ≥ 8, scores entered
- [ ] Positioning lens confirmed (Red Hat default vs Neutral) and set in the header
- [ ] Engagement ID assigned, record created, autosave verified (edit a field, see "Saved")
- [ ] Client asked to bring RVTools/CMDB export and last VMware renewal quote
- [ ] Laptop tested offline: console opens from file system, Print/PDF works

### During the workshop
- [ ] Outcomes captured in the client's words, not yours
- [ ] Stakeholder map filled (dispositions silently)
- [ ] Workload table as groups with criticality and modernization flags
- [ ] VMware block complete: hosts/sockets/cores, versions, products, ELA status, renewal date and quote
- [ ] Every disclosed number gets an assumptions row with source and tag (`input` / `cited` / `directional`)
- [ ] Gap list reviewed aloud before closing

### After the workshop
- [ ] JSON exported and filed (the JSON is the record, not localStorage)
- [ ] Word Discovery Summary sent to attendees same day
- [ ] Recommendation Engine run; honesty checks and caveats read, not skipped
- [ ] Updated JSON (with recommendation block) exported and versioned
- [ ] Recommendation Word doc shared with champion; gaps assigned owners and dates

---

## 5. Schema reference (engagement.json v1.0)

Top-level keys: `schema_version`, `meta` (client, industry, profile, engineer, positioning, currency incl. `pkr_rate` 279.40 default, updated 2026-07-20), `qualification` (bant, meddpicc, computed `scores`), `business` (drivers, outcomes, constraints, budget_range, decision_timeline, success_criteria), `stakeholders[]`, `technical` (estate, workloads[], container_ambition, compliance[], data_residency, growth, dr), `vmware` (present, hosts, sockets_per_host, cores_per_socket, computed `total_cores`, vms_total, vms_powered_on, avg profile, versions, products[], ela_status, renewal_date, renewal_quote_usd, uplift_pct, pain), `assumptions[]` (text, source, tag), `gaps[]` (computed), `recommendation` (written by the engine).

Phase 2 will append `architecture` and `boq` blocks; the schema changelog will record every addition. Existing files remain loadable (missing blocks are treated as empty).

**PKR handling (applies toolkit-wide).** USD is the base currency everywhere. PKR appears only as an appended conversion block: editable rate, timestamp, and the standing disclaimer — indicative only, subject to prevailing rate on day of quotation, exclusive of applicable Pakistan taxes and import duties unless the duty layer is enabled (arrives with the BOQ in Phase 2).

---

## 6. Known limits, stated plainly

- localStorage autosave is per-browser and per-machine. The exported JSON is the only portable record. This is deliberate: no server, no data leaves the room.
- Word export needs the CDN (or the vendored library file). Print/PDF always works.
- The recommendation rules are transparent by design and therefore simple by design. They rank fit; they do not replace an architect's judgment, and the exports say so via caveats.
- Core-pair figures shown are directional sizing bridges. Subscription counts are computed properly in the Phase 2 BOQ Builder and must be validated against the current Red Hat price book before quoting.
