# Monteverde Airport – Cybersecurity Home Lab

> A hands-on cybersecurity project: designing, defending and attacking the IT infrastructure of a small, fictional regional airport in an isolated virtual lab.

**Status:** Work in progress – version 0.8 completed

---

## The scenario

**Monteverde Airport** is a fictional small regional airport. Like any real airport, it relies on IT systems to operate: check-in desks, boarding gates, flight information displays, staff offices and a public Wi-Fi network for passengers.

The project is inspired by a real event: in **September 2025**, a ransomware attack on **Collins Aerospace's MUSE** check-in and boarding platform disrupted operations at several European airports, including London Heathrow, Brussels and Berlin, forcing staff to switch back to manual check-in.

The key lesson: the airports were not attacked directly – **their supplier was**. This lab explores how a small airport can protect itself, detect attacks and keep operating when something goes wrong.

## Goals

- Design a **segmented network** where passengers, operations and offices are kept separate
- Build and **harden** the airport's core systems
- Collect logs and **detect attacks** with a SIEM
- Simulate a realistic attack chain (phishing → check-in operator → check-in system)
- Respond to the incident and write a clear **incident report**
- Assess risks, plan **disaster recovery** and consider **GDPR** and **NIS2** requirements

## Lab architecture

**Version 0.8 (current):**

| Machine | Zone | Role in the scenario | OS | IP address |
|---|---|---|---|---|
| `fw-monteverde` | – | Perimeter firewall, gateway, DNS and time source for all zones | OPNsense | 10.10.2.65 (check-in) · 10.10.2.97 (SecOps) |
| `checkin-srv` | Check-in | Check-in system (provided by an external supplier), protected by a host firewall | Ubuntu Server | 10.10.2.66/27 |
| `checkin-pc01` | Check-in | Check-in desk operator's workstation | Windows 11 Enterprise | 10.10.2.80/27 |
| `mon-srv` | Security Operations | Monitoring: Prometheus and Grafana | Ubuntu Server | 10.10.2.98/28 |
| `wazuh-srv` | Security Operations | SIEM / SOC: Wazuh (server, indexer, dashboard) | Ubuntu Server 24.04 | 10.10.2.99/28 |
| `soc-ws01` | Security Operations | Analyst workstation | Xubuntu | 10.10.2.100/28 |
| `kali-redteam` | WAN side or check-in (per test) | Attacker machine for simulated attacks | Kali Linux | DHCP (WAN side) · 10.10.2.81/27 (insider) |

All machines run on **Hyper-V**. Each zone sits on its own isolated private switch (`Checkin-Zone`, `SecOps-Zone`): traffic between zones and towards the internet can only pass through the firewall. The attacker VM `kali-redteam` is moved between the WAN side and the check-in zone depending on the test.

## Roadmap

- [x] **[Chapter 1 – Network design](01-network-design/README.md):** network zones and IP addressing plan
- [x] **[Chapter 2 – Core systems](02-core-systems/README.md):** Ubuntu server and Windows workstation on an isolated network
- [x] **[Chapter 3 – Perimeter firewall](03-perimeter-firewall/README.md):** OPNsense gateway, NAT and DNS
- [x] **[Chapter 4 – Firewall rules](04-firewall-rules/README.md):** default deny, centralized time sync and a real log investigation
- [x] **[Chapter 5 – Host firewall](05-host-firewall/README.md):** ufw on the check-in server – defense in depth
- [x] **[Chapter 6 – Monitoring](06-monitoring/README.md):** Security Operations zone, Prometheus and Grafana, chrony on the firewall
- [x] **[Chapter 7 – SOC](07-soc-wazuh/README.md):** Wazuh SIEM, agents on Linux and Windows, attack detection
- [x] **[Chapter 8 – Attack simulation](08-attack-simulation/README.md):** reconnaissance from Kali Linux (external, insider, cross-zone) – a detection blind spot and a firewall rule flaw found and fixed
- [ ] **Chapter 9 – Incident response:** containment and incident report
- [ ] **Chapter 10 – Governance:** risk assessment, disaster recovery plan, GDPR and NIS2

Each chapter will have its own folder with documentation, configuration files and screenshots.

## Tools

Hyper-V · OPNsense · Ubuntu Server · Windows · PowerShell · chrony · ufw · Prometheus · Grafana · Xubuntu · Wazuh · Kali Linux · Nmap

## Disclaimer

Monteverde Airport is **entirely fictional**. All activities, including attack simulations, take place in an **isolated lab** on my own computer. No real systems, networks or organizations are involved.

## About me

I'm **Andrei Nicolae Telea**, currently training as an **ICT Security Specialist** (IFTS course, Italy). This project is where I put into practice what I study.

🔗 [LinkedIn](https://www.linkedin.com/in/andreitelea/)
