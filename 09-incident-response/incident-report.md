# Incident Report — Suspicious activity on check-in workstation `checkin-pc01`

| | |
|---|---|
| **Report ID** | IR-2026-001 |
| **Classification** | Internal – lab exercise |
| **Status** | Closed |
| **Date of incident** | 2026-10-09 |
| **Date of report** | 2026-10-09 |
| **Author** | SOC analyst (Monteverde Airport home lab) |
| **Affected system** | `checkin-pc01` (10.10.2.80), check-in zone |
| **Severity** | Medium (contained, no confirmed impact) |

> **Exercise disclaimer.** This is a controlled attack simulation performed in an isolated lab against the author's own virtual machines. Attacker actions were executed with [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) and native Windows tools, under an **assume breach** scenario: the initial access vector (phishing) and any exploitation or lateral movement were **not** performed. See *Scope and limitations*.

---

## 1. Executive summary

On 2026-10-09, the SOC detected a sequence of suspicious actions on `checkin-pc01`, a check-in operator workstation. Starting from a simulated existing foothold, an actor performed **host and user reconnaissance**, established **persistence** through a registry *Run* key, and then **cleared the Windows Security event log** in an attempt to remove its tracks.

Every stage was detected by the Wazuh SIEM, which receives Sysmon telemetry from the host. The log-clearing attempt failed to hide the activity: the events had already been forwarded to the SIEM and remained available for analysis. The incident was contained by isolating the host from the network, the persistence mechanism was eradicated, and the workstation was recovered from a known-good state. No real impact occurred; the persistence entry pointed to a non-existent executable.

---

## 2. Timeline

Times are local (Europe/Rome, UTC+2) as recorded in the SIEM.

| Time (2026-10-09) | Phase | Event | Detection (Wazuh rule) |
|---|---|---|---|
| ~19:17 | Attack | System owner/user discovery (`whoami`, `wmic`, `quser`, `qwinsta`) | 92032 (lvl 3), 92022 (lvl 3), 92052 (lvl 4) |
| 19:23 | Attack | Persistence: registry *Run* key created via `reg.exe` | 92302 (lvl 6), 92041 (lvl 10) |
| 19:31 | Attack | Defense evasion: Windows Security log cleared (`wevtutil cl Security`) | 63103 (lvl 5) |
| 19:3x | Response | Evidence preserved (VM checkpoint `post-attack-compromised`) | — |
| 19:3x | Response | Host isolated from the network (containment) | — |
| 19:3x | Response | Persistence key removed; recon artifacts cleaned | — |
| 19:4x | Response | Host recovered from known-good checkpoint; agent confirmed active | — |

---

## 3. Attack narrative (MITRE ATT&CK)

The observed activity maps to three ATT&CK techniques across three tactics:

| Tactic | Technique | Observed action |
|---|---|---|
| Discovery (TA0007) | **T1033** – System Owner/User Discovery | `cmd.exe /c whoami`, `wmic useraccount get /ALL`, `quser`, `qwinsta` enumerating users and sessions |
| Persistence (TA0003) | **T1547.001** – Registry Run Keys / Startup Folder | `reg.exe` added value `Atomic Red Team` under `HKCU\...\CurrentVersion\Run` |
| Defense Evasion (TA0005) | **T1070.001** – Clear Windows Event Logs | `wevtutil cl Security` cleared the Security event log |

**Key observation.** The persistence command was issued by `reg.exe`, spawned by `cmd.exe`, in turn started by an abnormal parent process (not normal user activity). Wazuh flagged both the registry modification and the Base64-like appearance of the written value (rule 92041, level 10), and separately flagged the abnormal process ancestry (rule 92052).

---

## 4. Indicators of compromise (IOCs)

| Type | Indicator |
|---|---|
| Registry key | `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` |
| Registry value name | `Atomic Red Team` |
| Registry value data | `C:\Path\AtomicRedTeam.exe` *(non-existent path; simulated payload)* |
| Process | `C:\Windows\System32\reg.exe` adding a *Run* value, parent `cmd.exe` |
| Host indicator | Security event log cleared (Windows Event ID 1102) |

---

## 5. Detection

Detection relied on the monitoring stack built in Chapters 7–8, extended for this chapter:

- **Sysmon** (sysmon-modular configuration) installed on `checkin-pc01`, providing process-creation, registry and network telemetry that standard Windows logging does not capture.
- The **Wazuh agent** forwarding the `Microsoft-Windows-Sysmon/Operational` channel, enabled through **centralized group configuration** on the manager (`agent.conf` for the `windows` group) rather than per-host edits.
- The **Wazuh manager** correlating events and raising alerts, reviewed by the analyst on `soc-ws01`.

All three attack stages produced alerts of level 3 or higher, confirming the pipeline works end to end.

---

## 6. Response actions

Actions follow the NIST SP 800-61 incident response lifecycle.

**6.1 Preparation** — Visibility in place before the incident: Sysmon + Wazuh agent on the host, centralized log collection, analyst dashboard on `soc-ws01`.

**6.2 Detection & analysis** — Alerts triaged on the Wazuh dashboard; the level-10 persistence alert was pivoted to extract the exact registry key, value and responsible process (Section 4).

**6.3 Containment** — The compromised state was first preserved as a Hyper-V checkpoint (`post-attack-compromised`) for forensic reference. The host's virtual network adapter was then disconnected, isolating it from all zones while leaving the VM running for local investigation.
> *Trade-off:* once isolated, the host stops reporting to the SIEM. This is acceptable because the evidence had already been forwarded, and remediation was performed from the local console.

**6.4 Eradication** — The persistence value was removed with `reg delete` targeting only the malicious value (not the whole `Run` key, which also holds legitimate entries). Removal was confirmed by re-querying the key. Reconnaissance artifacts were cleaned.

**6.5 Recovery** — The workstation was restored from the known-good checkpoint `pre-atomic`, which predates the attack and retains the monitoring agent. This mirrors the enterprise practice of rebuilding from a trusted image rather than trusting a once-compromised host. The Wazuh agent returned to **Active** and resumed reporting.

---

## 7. Impact assessment

- **Confidentiality:** No data exfiltration observed or simulated. Reconnaissance was limited to local user/session enumeration.
- **Integrity:** The Security event log on the host was cleared. No other integrity impact; the persistence payload did not exist.
- **Availability:** None. The host remained operational throughout.
- **Scope:** Single host (`checkin-pc01`). No lateral movement was observed — and none was attempted, per the exercise scope.

---

## 8. Lessons learned & recommendations

**What worked**

- **Centralized logging defeated anti-forensics.** The actor cleared the local Security log, but the SIEM already held the evidence. This is the single strongest argument for shipping logs off-host in real time.
- **Host telemetry is essential inside a zone.** Intra-zone traffic never reaches the perimeter firewall; without Sysmon on the endpoint, this activity would have been invisible.

**Recommendations**

| # | Recommendation | Rationale |
|---|---|---|
| 1 | Deploy the Wazuh agent to `soc-ws01` and `mon-srv` | These hosts currently log only locally; the same blind spot would apply to them |
| 2 | Add MITRE ATT&CK tags to the custom rules from Chapter 8 | Make the ATT&CK mapping visible directly in alerts |
| 3 | Enable Wazuh agent enrollment password | Prevent unauthorized agent registration |
| 4 | Consider application allow-listing on operator workstations | Directly mitigates *Run*-key persistence and untrusted execution |
| 5 | Enable real-time File Integrity Monitoring on critical paths | Current FIM reports only at scheduled scans |

---

## 9. Scope and limitations

- **Assume breach.** The exercise starts from an existing foothold. Phishing (initial access), exploitation and lateral movement were **not** simulated.
- **Log-clearing step** was executed with the native `wevtutil` command, because the corresponding Atomic Red Team test (T1070.001) is not included in the installed atomics set. The command is the same one the test would have run.
- **PowerShell execution policy** was set to `RemoteSigned` (current user) to run the testing framework — a deliberate, scoped lab change, not a disabling of the control.
- **Microsoft Defender remained enabled** throughout; the techniques used rely on legitimate Windows tools, not malware, so no antivirus exclusion was required.
- All activity was confined to the isolated lab, against the author's own virtual machines.
