# Monteverde Airport – GDPR Compliance Assessment

**Document ID:** GDPR-2026-001 (includes breach register entry DB-2026-001)
**Scope:** Monteverde Airport simulated IT environment (isolated Hyper-V lab)
**Date:** 2026-10-10
**Owner:** SOC analyst (lab author)
**Legal references:** Regulation (EU) 2016/679 (GDPR); EDPB Guidelines 9/2022 on personal data breach notification (v2.0, adopted 28 March 2023); Regulation (EC) No 1107/2006 (passengers with reduced mobility); Italian Law 300/1970, Art. 4 (remote monitoring of workers)

> **Disclaimer.** This is a portfolio exercise written from a security
> analyst's point of view, not legal advice. Where a conclusion depends on a
> legal interpretation, it is marked as an **open point** for a Data Protection
> Officer (DPO) or a lawyer. No real personal data exists in the lab: all users,
> hosts and events are simulated.

---

## 1. Why GDPR matters for a security lab

Security work is personal data work. A SIEM collects user names, IP
addresses, logon events and command lines of real employees. A check-in
system processes passenger identities. GDPR therefore matters in two directions:

- **Security protects personal data** – Article 32 requires appropriate
  technical and organisational measures (the firewall, segmentation, SIEM and
  backups built in this lab).
- **Security also processes personal data** – the monitoring itself needs a
  legal basis, a retention limit and transparency towards the people monitored.

## 2. Roles: controller and processor

| Term (GDPR Art. 4) | Meaning |
|---|---|
| **Controller** (Art. 4(7)) | Decides the purposes and means of the processing |
| **Processor** (Art. 4(8)) | Processes personal data on behalf of a controller |

**Scenario assumption.** Real role allocation depends on contracts and is
assessed case by case. For this lab:

- **Monteverde Airport** is the **controller** for its own staff data, its
  security monitoring, and assistance to passengers with reduced mobility
  (the airport managing body is responsible for that assistance under
  Regulation (EC) No 1107/2006, Art. 8(1)).
- For **passenger check-in**, Monteverde provides common-use check-in on behalf
  of the airlines: the **airlines are controllers**, the airport acts as a
  **processor**, and the **check-in software vendor** (simulated by
  `checkin-srv`) is a **sub-processor**.

**Why this matters – the Collins Aerospace lesson.** In September 2025 the
attack hit the check-in *vendor*, not the airports. In GDPR terms, a supplier
in the processor chain is exactly where a breach can start. The GDPR answer is
Article 28: a controller may only use processors "providing sufficient
guarantees" of appropriate security measures (Art. 28(1)), under a binding
contract (Art. 28(3)), and a processor must notify the controller "without
undue delay" after becoming aware of a breach (Art. 33(2)).

---

## 3. Record of processing activities (Article 30)

**Is the record required?** Article 30(5) exempts organisations with fewer than
250 employees, but **not** when the processing "is not occasional" or "includes
special categories of data". Security monitoring is continuous and assistance
to passengers with reduced mobility involves health data, so the exemption
does **not** apply: the record is required.

| ID | Processing activity | Role | Data subjects | Personal data | Legal basis | Retention (proposed) | Systems |
|---|---|---|---|---|---|---|---|
| P-01 | Passenger check-in (on behalf of airlines) | Processor (airline = controller); vendor = sub-processor | Passengers | Name, booking reference, travel document data, seat, baggage | Determined by the airline as controller | As instructed by the airline (Art. 28) | `checkin-srv`, `checkin-pc01` |
| P-02 | Assistance to passengers with reduced mobility | Controller | Passengers with reduced mobility | Identity, flight, **type of assistance needed (health data – Art. 9)** | Legal obligation – Reg. (EC) 1107/2006 (Art. 6(1)(c)); Art. 9(2) condition: **open point** | Proposed: until the end of the journey + short follow-up period | (not yet implemented) |
| P-03 | Security monitoring (SIEM) | Controller | Employees, vendor staff | User names, IP addresses, host names, logon events, process and command-line data (Sysmon), firewall logs | Legitimate interest (Art. 6(1)(f)) – network and information security, explicitly recognised in Recital 49 | **Proposed: 90 days online** in the SIEM, then deletion (to be validated) | `wazuh-srv`, agents, `checkin-pc01` (Sysmon) |
| P-04 | Infrastructure monitoring | Controller | (Mostly none) | Technical metrics of servers; host names and IPs of servers. **Limited or no personal data** | Legitimate interest (Art. 6(1)(f)) | Proposed: 30 days | `mon-srv` (Prometheus/Grafana) |
| P-05 | Backups for disaster recovery | Controller | Same as P-03 | Copies of the SIEM data | Same as the source data (P-03) | **Must not exceed the source retention**: backups older than 90 days to be destroyed | Offline backup disk (see DRP-2026-001) |

**Notes on the record**

- **P-05 is easy to forget.** Backups are copies of personal data. A retention
  limit that is respected in the SIEM but not in the backups is not respected.
- **P-03 needs a balancing test.** Legitimate interest (Art. 6(1)(f)) applies
  "except where such interests are overridden by the interests or fundamental
  rights and freedoms of the data subject". Monitoring must be limited to what
  is "strictly necessary and proportionate" for security (Recital 49).
- **P-04 is listed on purpose,** even with little personal data: deciding that a
  system holds *no* personal data is also a decision worth documenting.

### 3.1 Italian specifics: monitoring of employees

In Italy, Article 4 of Law 300/1970 (Workers' Statute) applies on top of GDPR.
Tools "from which the possibility of remote monitoring of workers' activity
also derives" may be used only for organisational and production needs, work
safety or protection of company assets, and require a **collective agreement
with the union representatives** or, failing that, an **authorisation from the
labour inspectorate**. Tools that the worker uses "to perform the work" are
excluded from this requirement (paragraph 2). In all cases, information may be
used only if workers received **adequate information** on how the tools are
used and how checks are carried out (paragraph 3).

**Open point:** whether endpoint telemetry such as Sysmon and a SIEM fall under
paragraph 1 (agreement or authorisation needed) or paragraph 2 (work tool) is a
legal interpretation. The conservative approach for Monteverde would be to
treat them under paragraph 1 and to inform employees in writing.

---

## 4. Security of processing (Article 32) – mapped to the lab

Article 32(1) lists measures "as appropriate". The lab provides concrete
evidence for each:

| Art. 32(1) | Requirement | Lab evidence | Chapter |
|---|---|---|---|
| (a) | Pseudonymisation and encryption | HTTPS on firewall GUI and Wazuh dashboard; encrypted Wazuh agent traffic. **Gap:** Grafana over HTTP (risk R-05) | 3, 6, 7 |
| (b) | Ongoing confidentiality, integrity, availability and resilience | Network segmentation, default-deny firewall, host firewalls, SIEM monitoring | 2–8 |
| (c) | Ability to restore availability and access to personal data in a timely manner | Offline backup and **tested restore of the SIEM: RTO 52 min** (DR-TEST-2026-001) | 10 |
| (d) | Regular testing, assessing and evaluating of the measures | Attack simulation (ch. 8), incident response exercise (ch. 9), DR test (ch. 10), risk register | 8–10 |

Article 32(1)(c) and (d) are where the DR test of this chapter becomes a
compliance argument: the ability to restore was **demonstrated**, not assumed.

---

## 5. Personal data breach procedure (Articles 33 and 34)

### 5.1 Key definitions

- **Personal data breach** (Art. 4(12)): "a breach of security leading to the
  accidental or unlawful destruction, loss, alteration, unauthorised disclosure
  of, or access to, personal data".
- The EDPB classifies breaches as **confidentiality** (unauthorised disclosure
  or access), **integrity** (unauthorised alteration) and **availability**
  (loss or destruction) breaches. One incident can be more than one type.
- The controller becomes **aware** of a breach when it is **reasonably certain**
  that a security incident has occurred (EDPB Guidelines 9/2022). The 72-hour
  clock starts from that moment, not from the attack.

### 5.2 Procedure

1. **Detect and confirm** – SOC investigation (see IR-2026-001 process).
2. **Assess whether personal data are involved** and which breach type applies.
3. **Assess the risk** to the rights and freedoms of the people concerned.
4. **Decide:**

| Risk to individuals | Notify the supervisory authority (Art. 33) | Inform the individuals (Art. 34) | Document internally (Art. 33(5)) |
|---|---|---|---|
| Unlikely to result in a risk | No | No | **Yes** |
| Risk | **Yes – within 72 hours** of awareness | No | Yes |
| High risk | Yes – within 72 hours | **Yes – without undue delay** (unless an Art. 34(3) exception applies) | Yes |

5. **Notify** (if required) the Italian supervisory authority, the *Garante per
   la protezione dei dati personali*, through its **online procedure**
   (mandatory since 1 July 2021). The notification must at least describe the
   nature of the breach, the DPO/contact point, the likely consequences and the
   measures taken (Art. 33(3)); information may be provided in phases (Art. 33(4)).
   A notification sent after 72 hours must include the reasons for the delay.
6. **If Monteverde acts as processor** (passenger check-in, P-01): notify the
   **airlines** (controllers) without undue delay (Art. 33(2)). The airlines,
   not the airport, decide on notification to the authority.
7. **Record** every breach in the breach register, **including those not
   notified** (Art. 33(5); EDPB Guidelines 9/2022).

---

## 6. Breach register

### DB-2026-001 – linked to incident IR-2026-001

| Field | Value |
|---|---|
| Incident | IR-2026-001 – compromised check-in workstation (`checkin-pc01`): discovery, persistence via Run key, Security event log cleared |
| Date of the incident | 2026-10-09, ≈19:17–19:31 (local time) |
| Awareness | 2026-10-09, evening – during SOC analysis of the Wazuh alerts |
| Personal data involved | **Employee data**: the Windows Security log contains account names and logon events. **No passenger data**: the check-in application is not deployed in this lab version |
| Breach type | **Availability** (unlawful destruction of the local Security log). Possible **confidentiality** (unauthorised access to a workstation); no evidence that data were viewed or exfiltrated |
| Mitigating factors | The log events had **already been forwarded to the SIEM** before the deletion, so no data were actually lost; the persistence mechanism was removed and the host restored to a clean state |
| Risk assessment | **Unlikely to result in a risk** to the rights and freedoms of the employees concerned |
| Notification to the Garante | **Not required** (Art. 33(1) exception) |
| Communication to data subjects | **Not required** (no high risk – Art. 34(1)) |
| Remedial action | Containment, eradication and recovery per IR-2026-001 |
| Classification approach | Treated as a personal data breach under a **conservative reading** of Art. 4(12) (deliberate destruction of stored personal data), even though a complete copy existed |

**Lesson:** centralised logging is not only a detection control. Because the
SIEM already held a copy, the deletion of the local log did not become a loss
of data – this directly changed the GDPR outcome from "possible breach with
data loss" to "breach with no impact on individuals".

**Counterfactual (if the check-in application had been running).** If
passenger data had been processed on the compromised workstation, Monteverde
would have acted as **processor**: it would have had to inform the airlines
without undue delay, and the airlines would have assessed whether to notify
the Garante within 72 hours of their awareness.

---

## 7. Open points and known limitations

| # | Open point | Owner |
|---|---|---|
| 1 | Art. 9(2) condition for health data of passengers with reduced mobility (P-02) | DPO / legal |
| 2 | Whether Sysmon/SIEM require a union agreement or labour-inspectorate authorisation under Art. 4 Law 300/1970 | DPO / legal / HR |
| 3 | Retention periods (90 days SIEM, 30 days metrics) are **proposals**, not validated values | DPO |
| 4 | Whether a **DPO** must be designated (Art. 37) depends on the nature and scale of the airport's core activities | Legal |
| 5 | A **Data Protection Impact Assessment** (Art. 35) is advisable for systematic monitoring of employees (P-03) | DPO |
| 6 | Controller/processor allocation for check-in is a **scenario assumption**; in reality it follows the contracts with airlines and the vendor | Legal |
| 7 | No privacy notice to employees has been written yet | DPO / HR |

## 8. Lessons learned

1. **Security tools are also personal data processing.** A SIEM must respect
   purpose limitation, retention and transparency like any other system.
2. **Backups inherit the obligations of the data they contain.**
3. **The 72-hour clock starts at awareness.** Fast detection (chapter 7) and
   structured analysis (chapter 9) are what make the deadline realistic.
4. **Not every breach is notified, but every breach is documented.**
5. **The supply chain is part of the compliance perimeter.** Processor
   contracts (Art. 28) are the legal counterpart of vendor risk management.
