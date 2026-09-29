# Monteverde Airport – Cybersecurity Home Lab

> A hands-on cybersecurity project: designing, defending and attacking the IT infrastructure of a small, fictional regional airport in an isolated virtual lab.

**Status:** Work in progress – version 0.2 completed ✅

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

**Version 0.2 (current):**

| Machine | Role in the scenario | OS | IP address |
|---|---|---|---|
| `fw-monteverde` | Perimeter firewall and gateway of the check-in zone | OPNsense | 10.10.2.65/27 (LAN) |
| `checkin-srv` | Check-in system (provided by an external supplier) | Ubuntu Server | 10.10.2.66/27 |
| `checkin-pc01` | Check-in desk operator's workstation | Windows 11 Enterprise | 10.10.2.80/27 |

All machines run on **Hyper-V**. The check-in zone sits on the isolated private switch `Checkin-Zone`: its only way out is through the firewall.

## 🗺️ Roadmap

- [x] **[Chapter 1 – Network design](01-network-design/README.md):** network zones and IP addressing plan
- [x] **[Chapter 2 – Core systems](02-core-systems/README.md):** Ubuntu server and Windows workstation on an isolated network
- [x] **[Chapter 3 – Perimeter firewall](03-perimeter-firewall/README.md):** OPNsense gateway, NAT and DNS
- [ ] **Chapter 4 – Firewall rules:** default deny, only the traffic the airport actually needs
- [ ] **Chapter 5 – Monitoring:** system availability with Prometheus and Grafana
- [ ] **Chapter 6 – SOC:** log collection and detection with Wazuh
- [ ] **Chapter 7 – Attack simulation:** phishing and lateral movement from Kali Linux
- [ ] **Chapter 8 – Incident response:** containment and incident report
- [ ] **Chapter 9 – Governance:** risk assessment, disaster recovery plan, GDPR and NIS2

Each chapter will have its own folder with documentation, configuration files and screenshots.

## Tools

Hyper-V · OPNsense · Ubuntu Server · Windows · PowerShell · *(coming next: Prometheus, Grafana, Wazuh, Kali Linux)*

## Disclaimer

Monteverde Airport is **entirely fictional**. All activities, including attack simulations, take place in an **isolated lab** on my own computer. No real systems, networks or organizations are involved.

## About me

I'm **Andrei Nicolae Telea**, currently training as an **ICT Security Specialist** (IFTS course, Italy). This project is where I put into practice what I study.

🔗 [LinkedIn](https://www.linkedin.com/in/andreitelea/)
