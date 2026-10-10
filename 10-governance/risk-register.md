# Monteverde Airport – Information Security Risk Register

**Document ID:** RISK-REG-2026-001
**Scope:** Monteverde Airport simulated IT environment (isolated Hyper-V lab)
**Date:** 2026-10-10
**Owner:** SOC analyst (lab author)
**Method:** NIST SP 800-30 Rev. 1 and ISO/IEC 27005 (qualitative), risk treatment per ISO/IEC 27001:2022

---

## 1. Purpose

This register records the information security risks of the Monteverde Airport
lab, the controls already in place, and the treatment decision for each risk.
A risk register is not a to-do list: its purpose is to make a **conscious,
documented decision** for every risk (reduce it, accept it, avoid it, or
transfer it), so that residual risk is accepted on purpose and not by neglect.

## 2. Method

Risk is assessed as **Likelihood × Impact**, each on a 1–5 qualitative scale.

**Likelihood**

| Score | Meaning |
|---|---|
| 1 | Rare |
| 2 | Unlikely |
| 3 | Possible |
| 4 | Likely |
| 5 | Almost certain |

**Impact** (on confidentiality, integrity, availability, or on the airport's operations)

| Score | Meaning |
|---|---|
| 1 | Negligible |
| 2 | Minor |
| 3 | Moderate |
| 4 | Major |
| 5 | Severe (operations stopped / large data breach) |

**Risk level** (lab convention, stated explicitly, not a standard):

| Score (L×I) | Level |
|---|---|
| 1–4 | Low |
| 5–9 | Medium |
| 10–15 | High |
| 16–25 | Critical |

**Treatment options** (ISO/IEC 27001:2022):
*Mitigate* (add/strengthen a control), *Accept* (retain the risk on purpose),
*Avoid* (stop doing the risky activity), *Transfer* (share the risk, e.g. insurance or a supplier).

## 3. Scoping assumption (stated honestly)

Likelihood and impact are estimated **as if Monteverde were a real regional
airport**, because that is the scenario this lab simulates and what makes the
assessment meaningful. In the actual isolated lab the real-world likelihood of
most threats is near zero. These scores are a **reasoned estimate**, not a
measured value.

---

## 4. Risk register

| ID | Asset / area | Threat | Vulnerability | Existing controls | L | I | Score | Level | Treatment |
|---|---|---|---|---|---|---|---|---|---|
| R-01 | OPNsense firewall (management GUI) | Unauthorised access to firewall management from the check-in zone | Anti-lockout rule leaves the GUI reachable from the whole check-in LAN; Office/IT admin zone not yet built | Web rules exclude `PRIVATE_NETS`; GUI over HTTPS; SecOps admin restricted to soc-ws01 | 3 | 5 | 15 | High | Mitigate |
| R-02 | Wazuh manager (agent enrollment) | Rogue or spoofed agent registers with the SIEM | Agent enrollment has no password/authentication | Enrollment ports (1514/1515) reachable only from the two active zones | 2 | 4 | 8 | Medium | Mitigate |
| R-03 | checkin-srv / checkin-pc01 (file integrity) | Undetected tampering with critical files | FIM runs on a schedule, not in real time | Wazuh FIM active on a schedule; Sysmon on checkin-pc01 | 3 | 3 | 9 | Medium | Mitigate |
| R-04 | Internal time service (NTP) | Time spoofing that corrupts log correlation and certificate validation | Internal NTP (firewall → hosts) is not authenticated | Single internal time source (firewall), Hyper-V time sync disabled on guests | 2 | 3 | 6 | Medium | Accept (for now) |
| R-05 | Grafana (mon-srv) | Interception of monitoring credentials / data | Grafana served over plain HTTP | Reachable only from soc-ws01 (ufw + firewall rule) | 2 | 2 | 4 | Low | Mitigate |
| R-06 | mon-srv, soc-ws01, firewall (telemetry) | Attacker activity on hosts without a SIEM agent goes unseen | No Wazuh agent on mon-srv/soc-ws01; firewall perimeter blocks visible only in OPNsense live view; soc-ws01 ufw blocks stay local | Host ufw active everywhere; perimeter firewall logging; Sysmon on the endpoint | 3 | 4 | 12 | High | Mitigate |
| R-07 | Network segmentation | Lateral movement / admin traffic outside a controlled zone | Office/IT zone not built; VM administration done from the Hyper-V console on the host | Stateful firewall, default-deny between zones; only two zones active | 2 | 4 | 8 | Medium | Accept (lab scope) |
| R-08 | checkin-pc01 (Windows 11 evaluation) | Loss of the endpoint / lab environment | Evaluation licence expires ~end December 2026 | Checkpoints taken; chapter work completed before expiry | 4 | 2 | 8 | Medium | Accept |
| R-09 | Wazuh detection rules | Missed detection or alert fatigue | Custom rule 100101 (port scan) not fully tuned; MITRE tags incomplete on custom rules | Baseline custom rules tested with wazuh-logtest; sysmon-modular ruleset | 3 | 3 | 9 | Medium | Mitigate |
| R-10 | Host firewall logging (ufw) | Security-relevant packets not recorded | ufw logging set to "low" (samples packets, not all) | ufw active on servers and soc-ws01; blocks forwarded to Wazuh on checkin-srv | 3 | 2 | 6 | Medium | Accept |
| R-11 | checkin-pc01 (host configuration) | Residual weakening of endpoint hardening after lab exercises | PowerShell execution policy left at `RemoteSigned` (CurrentUser) after chapter 9 | Atomic Red Team removed via `pre-atomic` checkpoint; Defender stayed active throughout | 2 | 2 | 4 | Low | Accept |
| R-12 | Hyper-V host (VM management) | Slow or error-prone recovery actions under pressure | Hyper-V PowerShell module not installed on the host; management is GUI-only | Documented GUI procedures for checkpoint/network/restore | 2 | 2 | 4 | Low | Accept |
| R-13 | checkin-srv (business application) | The core business service cannot be protected or monitored in context | Check-in application not yet deployed on checkin-srv | Host hardened (ufw, static IP, NTP, Wazuh agent) ready to host it | 3 | 2 | 6 | Medium | Accept (future scope) |

---

## 5. Risk treatment plan (actions for "Mitigate" items)

| ID | Level | Planned action | Status |
|---|---|---|---|
| R-01 | High | Build the Office/IT zone and restrict firewall management to it; tighten the anti-lockout exposure | Planned |
| R-06 | High | Install Wazuh agents on mon-srv and soc-ws01; forward OPNsense logs (syslog) to Wazuh | Planned |
| R-02 | Medium | Enable Wazuh agent enrollment password | Planned |
| R-03 | Medium | Set FIM to `realtime` on critical directories | Planned |
| R-05 | Low | Issue a TLS certificate for Grafana | Planned |
| R-09 | Medium | Tune rule 100101; add MITRE technique tags to custom rules | Planned |

## 6. Accepted risks (residual risk)

R-04, R-07, R-08, R-10, R-11, R-12 and R-13 are **accepted** for the current lab
version. Each is accepted on purpose: either it reflects deliberate lab scope
(R-07, R-13), a documented operational constraint (R-08, R-12), or a low-level
residual risk whose mitigation cost is not justified at this stage (R-04, R-10,
R-11). Accepted risks are reviewed when the lab scope changes.

## 7. Out of scope / informational

- **QUIC (UDP 443) blocked from the active zones** — a deliberate design choice
  for traffic visibility (forces HTTP/HTTPS over inspectable TCP). Documented as
  an accepted limitation, not scored as a risk.

## 8. Review

This register is reviewed at the end of each lab version and whenever a new zone,
host, or service is added.
