# Monteverde Airport – NIS2 Applicability and Gap Analysis

**Document ID:** NIS2-2026-001
**Scope:** Monteverde Airport simulated IT environment (isolated Hyper-V lab)
**Date:** 2026-10-10
**Owner:** SOC analyst (lab author)
**Legal references:** Directive (EU) 2022/2555 (NIS2); Italian Legislative Decree 4 September 2024, no. 138 (NIS2 transposition); guidance of the Italian National Cybersecurity Agency (ACN)

> **Disclaimer.** Portfolio exercise, not legal advice. Directive articles
> quoted here were checked against the text of the directive. Italian
> implementation details (ACN determinations and deadlines) change over time
> and were taken from secondary sources dated 2025–2026; they must be verified
> on the ACN portal before being relied upon. Entity size and classification
> are **scenario assumptions**.

---

## 1. What NIS2 is, in one paragraph

GDPR protects **personal data**. NIS2 protects **services**: it requires
organisations that society depends on (energy, transport, health, digital
infrastructure and others) to manage cybersecurity risk, report significant
incidents, and make their management accountable for it. The Collins Aerospace
attack of September 2025 is a textbook NIS2 scenario: no airport was breached,
yet check-in at several European airports fell back to manual processing
because a **supplier** was hit.

## 2. Is Monteverde Airport in scope?

**Step 1 – Sector.** Annex I (sectors of high criticality), sector *Transport*,
subsector *Air*, lists "airport managing bodies as defined in Article 2,
point (2), of Directive 2009/12/EC" and "entities operating ancillary
installations contained within airports", alongside air carriers and air
traffic control operators. → **Monteverde's operator is an Annex I entity type.**

**Step 2 – Size.** NIS2 applies to Annex I/II entities that "qualify as
medium-sized enterprises" or exceed those ceilings (Art. 2(1)). Smaller
entities are in scope only in specific cases (Art. 2(2)), for example when
they are the sole provider of a vital service in a Member State, or when a
disruption could significantly affect public safety or security.

**Scenario assumption:** the operator of a small regional airport is a
**medium-sized enterprise** (50–249 employees).

**Step 3 – Category.**

| Category | Rule | Monteverde |
|---|---|---|
| Essential entity | Annex I entities that **exceed** the medium-sized ceilings (Art. 3(1)(a)), plus entities designated by the Member State | No (medium-sized) |
| **Important entity** | Annex I or II entities that do not qualify as essential (Art. 3(2)) | **Yes** |

**Result: Monteverde Airport is an *important entity*** under NIS2, unless the
national authority designates it otherwise. The obligations on security
measures and incident reporting are the same for both categories; the
differences are in supervision (ex-post for important entities) and in the
maximum fines (Art. 34(4)–(5)):

| | Maximum fine (at least) |
|---|---|
| Essential entity | EUR 10 million or 2% of total worldwide annual turnover, whichever is higher |
| Important entity | EUR 7 million or 1.4% of total worldwide annual turnover, whichever is higher |

## 3. Italian implementation (to be verified on the ACN portal)

Italy transposed NIS2 with **Legislative Decree 138/2024**; the competent
authority is the **National Cybersecurity Agency (ACN)** and incidents are
reported to **CSIRT Italia**. According to secondary sources (2025–2026):

- Entities **register every year** on the ACN platform between 1 January and
  28 February; ACN communicates inclusion in the NIS list.
- **Incident notification** obligations started in **January 2026** for
  entities listed in 2025 (January 2027 for those listed in 2026).
- **Basic security measures** must be implemented **by October 2026** for
  entities listed in 2025 (18 months after the inclusion notice), and by
  July 2027 for entities listed in 2026.
- The basic measures were defined by **ACN determination no. 164179 of
  14 April 2025** – reportedly replaced by **determination no. 379907/2025**,
  applicable from 15 January 2026 – with **37 measures / 87 requirements for
  important entities** and **43 measures / 116 requirements for essential
  entities**.

> These figures come from secondary sources and could not be checked on the
> ACN portal at the time of writing. The gap analysis below therefore follows
> the **directive's own list of measures** (Art. 21(2)), which is stable.

## 4. Gap analysis – Article 21(2) risk-management measures

Article 21(1) requires "appropriate and proportionate technical, operational
and organisational measures". Article 21(2) lists the minimum areas.

Status: ✅ Implemented (at lab scale) · 🟡 Partial · ❌ Gap

| Art. 21(2) | Measure | Lab evidence | Status | Gap / linked risk |
|---|---|---|---|---|
| (a) | Policies on risk analysis and information system security | Risk register RISK-REG-2026-001 (15 risks, ISO 27005 method) | 🟡 | No formal, management-approved security policy |
| (b) | Incident handling | Wazuh SIEM with custom rules (ch. 7–8); incident IR-2026-001 handled following NIST SP 800-61 (ch. 9) | ✅ | NIS2 reporting step added in section 5 |
| (c) | Business continuity: backup management, disaster recovery, crisis management | DRP-2026-001; offline, hash-verified backup; **restore tested, RTO 52 min** (ch. 10) | 🟡 | Only the SIEM backed up; no schedule; no crisis-management plan (R-14, R-15) |
| (d) | Supply chain security | Vendor system (`checkin-srv`) isolated in its own zone; vendor scenario analysed (Collins lesson) | 🟡 | No supplier security assessment or contractual security clauses |
| (e) | Security in acquisition, development and maintenance, incl. vulnerability handling | Controlled updates (OPNsense, Wazuh repo disabled after install); downloads verified by hash or digital signature | 🟡 | No vulnerability management process (scanning, patch SLAs) |
| (f) | Procedures to assess the effectiveness of the measures | Attack simulation (ch. 8), incident-response exercise (ch. 9), DR test (ch. 10) | ✅ | Repeat periodically |
| (g) | Basic cyber hygiene and cybersecurity training | Least privilege (standard user `checkin-op01`), host firewalls everywhere | 🟡 | No training programme |
| (h) | Cryptography and encryption | HTTPS for firewall GUI and Wazuh dashboard; encrypted agent traffic | 🟡 | Grafana over HTTP (R-05); unauthenticated NTP (R-04) |
| (i) | HR security, access control, asset management | Asset inventory of all VMs; admin access limited to `soc-ws01`; default-deny between zones | 🟡 | Firewall GUI still reachable from the check-in zone (R-01); no HR security process |
| (j) | Multi-factor authentication, secured communications | – | ❌ | No MFA on any administrative interface |

**Summary:** 2 implemented, 7 partial, 1 gap. The technical foundations are in
place; the main gaps are **organisational** (policies, training, supplier
management) and **MFA**.

## 5. Incident reporting (Article 23)

### 5.1 When an incident must be reported

An incident is **significant** if it "has caused or is capable of causing
severe operational disruption of the services or financial loss for the entity
concerned", or "has affected or is capable of affecting other natural or legal
persons by causing considerable material or non-material damage" (Art. 23(3)).

### 5.2 Reporting timeline (Art. 23(4))

| Deadline (from becoming aware) | Report | Content |
|---|---|---|
| **24 hours** | Early warning | Is the incident suspected to be caused by unlawful or malicious acts? Could it have a cross-border impact? |
| **72 hours** | Incident notification | Update of the early warning, initial assessment (severity, impact), indicators of compromise where available |
| On request | Intermediate report | Status updates |
| **1 month** after the notification | Final report | Detailed description, root cause, mitigation measures, cross-border impact |

In Italy, reports go to **CSIRT Italia**.

### 5.3 NIS2 and GDPR together

The same event can trigger **two different notifications** with **two
different clocks**:

| | NIS2 | GDPR |
|---|---|---|
| Protects | The service | Personal data |
| Trigger | Significant incident | Personal data breach with a risk to individuals |
| Recipient (Italy) | CSIRT Italia | Garante per la protezione dei dati personali |
| First deadline | **24 h** early warning | **72 h** notification |

The incident-response procedure must check **both** questions at the start.

### 5.4 Assessment of IR-2026-001

| Question | Answer |
|---|---|
| Severe operational disruption? | No – one workstation, check-in service not disrupted |
| Financial loss? | No |
| Considerable damage to others? | No |
| **Significant incident?** | **No → no NIS2 report required** |
| Recorded internally? | Yes – IR-2026-001; GDPR assessment in DB-2026-001 |

**Counterfactual – a Collins-type event.** If the check-in vendor system were
encrypted by ransomware and check-in had to fall back to manual processing,
the incident would be **significant** (severe operational disruption):
Monteverde would have to send the early warning within 24 hours, even though
the attack started at the supplier.

## 6. Management accountability (Article 20)

NIS2 makes cybersecurity a **board-level** responsibility: management bodies
must "approve the cybersecurity risk-management measures", "oversee its
implementation" and "can be held liable for infringements" (Art. 20(1)), and
their members "are required to follow training" (Art. 20(2)).

In this lab there is no management body. In a real organisation, the
documents of this chapter (risk register, DR plan, this gap analysis) are
exactly what would be presented to management for **approval**.

## 7. Remediation roadmap

| Priority | Action | Art. 21(2) | Risk |
|---|---|---|---|
| 1 | MFA on firewall GUI, Wazuh dashboard and Grafana | (j) | – |
| 2 | Office/IT zone; firewall management only from there | (i) | R-01 |
| 3 | Back up all systems, scheduled, with an off-site copy; Production checkpoints | (c) | R-14, R-15 |
| 4 | Wazuh agents on all hosts and firewall syslog to the SIEM | (b) | R-06 |
| 5 | Vulnerability management process | (e) | – |
| 6 | Supplier security requirements for the check-in vendor | (d) | – |
| 7 | Information security policy approved by management; training plan | (a), (g) | – |
| 8 | TLS for Grafana; authenticated time source | (h) | R-05, R-04 |

## 8. Known limitations

- Entity size and category are **assumptions**; real classification depends on
  the operator's actual size and on the national authority.
- The analysis uses the directive's ten measure areas, not the detailed ACN
  requirement list (37 measures / 87 requirements for important entities),
  which was not reviewed line by line.
- Italian deadlines come from secondary sources and were not checked on the
  ACN portal.
- Organisational controls (policies, training, contracts) are out of the
  technical scope of the lab and are documented as gaps, not simulated.

## 9. Lessons learned

1. **NIS2 protects services, GDPR protects data** – one incident can require
   both reports, with different deadlines and recipients.
2. **The supplier's incident can be your incident.** Supply chain security is
   an explicit legal requirement (Art. 21(2)(d)), not a best practice.
3. **Testing is a compliance requirement.** Art. 21(2)(f) asks for procedures
   to assess effectiveness: chapters 8–10 of this lab are that evidence.
4. **Technical controls are only half of NIS2.** Most remaining gaps are
   organisational: policies, training, management approval.
