# Chapter 10 – Governance: risk, recovery and compliance

> The final chapter steps back from the console: it measures the lab's risks, **proves** that a critical system can be recovered from backup, and checks the airport against **GDPR** and **NIS2** – turning nine chapters of technical work into evidence that management, auditors and regulators can read.

**Status:** Completed – lab version 1.0

---

## From controls to governance

Chapters 1–9 built the airport's security: segmentation, firewalls, monitoring, a SIEM, attack simulation and incident response. Governance answers a different set of questions: *Which risks are left, and who decided to accept them? Can we get back on our feet after a disaster – and how fast? Are we following the law?*

If the earlier chapters built the airport's fences, cameras and response team, this chapter writes the **safety manual**, runs the **evacuation drill** and checks the **regulatory paperwork**.

## Deliverables

| Document | ID | Question it answers |
|---|---|---|
| [`risk-register.md`](risk-register.md) · [`risk-register.xlsx`](risk-register.xlsx) | RISK-REG-2026-001 | What can go wrong, how bad is it, and what did we decide? |
| [`disaster-recovery-plan.md`](disaster-recovery-plan.md) | DRP-2026-001 | Can the SIEM be restored from backup, and how fast? (includes a real test) |
| [`gdpr-compliance.md`](gdpr-compliance.md) | GDPR-2026-001 | Which personal data does the lab process, and was the chapter 9 incident a reportable breach? |
| [`nis2-gap-analysis.md`](nis2-gap-analysis.md) | NIS2-2026-001 | Is the airport in scope of NIS2, and where are the gaps? |

## 1. Risk assessment

The lab's known issues, collected chapter by chapter, were turned into a formal **risk register** using the method of **NIST SP 800-30** and **ISO/IEC 27005**: likelihood × impact on a 5×5 scale, and one of the four **ISO/IEC 27001** treatment options for every risk (mitigate, accept, avoid, transfer). The register is published both in Markdown and as an Excel workbook, where scores and levels are formulas and the level thresholds can be edited.

| Level | Count |
|---|---|
| Critical | 0 |
| High | 4 |
| Medium | 8 |
| Low | 3 |

The four High risks: the firewall GUI still reachable from the check-in zone (R-01), hosts without a SIEM agent (R-06), and two risks that **did not exist on paper until the recovery test found them** – backups covering only the SIEM (R-14) and checkpoints that restore servers with a stale network identity and clock (R-15).

> Scores are reasoned estimates for the airport scenario, not measured values, and the register says so explicitly. A risk register is a record of decisions, not a precision instrument.

## 2. Disaster recovery – a real test

This is the hands-on part of the chapter. The question: *the SIEM's disk has failed – can we restore it, and within what time?*

**Checkpoint ≠ backup.** Hyper-V checkpoints live on the same disk as the VM: if the disk dies, the VM and every checkpoint die together. Following the **3-2-1 rule**, the SIEM was exported to an **external disk that is disconnected after use** – an offline copy that ransomware cannot reach.

**Targets first.** Recovery objectives were fixed *before* the test, so the result could not be bent to fit them: **RTO 60 minutes** (time to restore the service) and **RPO 24 hours** (maximum data loss).

**Integrity before use.** The backup was sealed with a **SHA-256 hash** at creation time and verified again before the restore. A backup that cannot be shown to be unaltered is not restored.

![SHA-256 of the backup verified before the restore](images/dr-backup-hash-verified.png)

**What happened.** The first attempt was aborted and documented: the PowerShell console was frozen by an accidental click (QuickEdit mode), so the timing could not be trusted. In the second attempt the backup was verified and imported as a new VM – and the restored SIEM was **unreachable**.

The cause took ten minutes to find. The checkpoints were of type **Standard**, which also saves the VM's memory: the restored server *resumed* with the **original MAC address**, while Hyper-V had assigned the copy a new one. Hyper-V's virtual switch drops frames from a MAC that does not belong to the VM, so every packet was silently discarded. The same memory state left the server's clock **a day behind**, while `timedatectl` still reported *synchronized: yes*. A reboot fixed both. Enabling MAC spoofing would have been quicker – and would have switched off a security control for convenience.

![Hyper-V assigned a new MAC address to the restored copy](images/dr-restore-mac-mismatch.png)

The test ended when the restored SIEM received a **new alert** from the check-in workstation (a deliberate failed logon).

![The restored SIEM receives a new alert – end of the recovery clock](images/dr-restore-logon-failure-alert.png)

| Objective | Target | Measured | Result |
|---|---|---|---|
| RTO | 60 min | **52 min** | ✅ Met |
| RPO | 24 h | **22 h 20 min** | ✅ Met |

The targets were met, but narrowly. The recovery runbook was revised with every problem found, and two new risks entered the register. The full timeline, root-cause analysis and corrective actions are in the **[DR plan](disaster-recovery-plan.md)**.

## 3. GDPR

Security tools process personal data too: the SIEM stores user names, logons and command lines of employees. The assessment covers:

- **Roles** – the airport as controller (staff, SIEM) and as processor for the airlines at check-in, with the software vendor as sub-processor: the Collins Aerospace lesson in legal terms (Art. 28).
- **Record of processing activities** (Art. 30) – five activities, including the **backups**, which inherit the retention limits of the data they contain.
- **Security of processing** (Art. 32) – each requirement mapped to lab evidence; the DR test is the proof of "the ability to restore the availability and access to personal data in a timely manner".
- **Breach procedure** (Art. 33–34) – the 72-hour clock starts when the controller is *reasonably certain* of a breach.
- **The chapter 9 incident, assessed:** clearing the Security log destroyed employee data, so it was treated as an availability breach – but the SIEM already held a copy. Risk to individuals: unlikely. Result: **no notification to the authority, recorded in the breach register**.

Points that depend on legal interpretation (for example, Italian rules on monitoring employees) are listed as open points for a DPO, not presented as settled.

## 4. NIS2

NIS2 protects **services**, not data. Airport managing bodies are listed in Annex I of the directive; as an assumed medium-sized operator, Monteverde would be an **important entity**.

The gap analysis against the ten risk-management areas of **Article 21(2)** found **2 implemented, 7 partial, 1 missing** (multi-factor authentication). The technical foundations are in place; most gaps are organisational – policies, training, supplier security.

## One incident, three lenses

The chapter 9 incident was re-examined from every angle of this chapter:

| Lens | Question | Answer for IR-2026-001 |
|---|---|---|
| Security (ch. 9) | Was the attack detected and contained? | Yes – all three techniques detected, host restored |
| GDPR | Was it a personal data breach that must be notified? | Breach recorded; **no notification** (no risk to individuals) |
| NIS2 | Was it a significant incident to report within 24 h? | **No** – no disruption of the check-in service |
| NIS2 (counterfactual) | What if the check-in vendor had been hit, as in the Collins case? | **Significant** – early warning within 24 h, even though the attack started at the supplier |

## Known limitations

- Only the **SIEM** was backed up and recovery-tested; backups are manual and there is no off-site copy
- The runbook step that avoids the MAC/clock problem (delete the saved state before the first start) is **documented but not yet tested**
- Risk scores are estimates for the scenario; entity size and NIS2 category are assumptions
- Italian NIS2 implementation details come from secondary sources and must be verified on the ACN portal
- Organisational controls (policies, training, supplier contracts, management approval) are documented as gaps, not simulated
- This is a portfolio exercise, not legal advice

## Lessons learned

- **A checkpoint is not a backup:** only a copy on separate, disconnected media survives a disk failure or ransomware
- **Test the restore, not just the backup:** the backup was flawless – both real problems appeared only during recovery
- **Verify values, not status flags:** *synchronized: yes* was true in memory and false in reality
- **Imperfect tests are results:** an aborted attempt and two failures produced the most useful content of the chapter – and two new entries in the risk register
- **Centralised logging changes legal outcomes:** because the SIEM held a copy, the deleted log was not a loss of data
- **The supplier's incident is your incident:** both GDPR (Art. 28) and NIS2 (Art. 21(2)(d)) make supply chain security an explicit obligation

## Files

| File | Purpose |
|---|---|
| [`risk-register.md`](risk-register.md) | Risk register (RISK-REG-2026-001) |
| [`risk-register.xlsx`](risk-register.xlsx) | Same register as an Excel workbook (download) |
| [`disaster-recovery-plan.md`](disaster-recovery-plan.md) | DR plan and test record (DRP-2026-001, DR-TEST-2026-001) |
| [`gdpr-compliance.md`](gdpr-compliance.md) | GDPR assessment and breach register (GDPR-2026-001) |
| [`nis2-gap-analysis.md`](nis2-gap-analysis.md) | NIS2 applicability and gap analysis (NIS2-2026-001) |
