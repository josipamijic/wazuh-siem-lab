# Wazuh SIEM Home Lab — Monitoring a Windows 11 Endpoint

> **In 30 seconds:** I built a small security monitoring system (a SIEM) at home.
> I installed **Wazuh** on an Ubuntu server, connected a **Windows 11** computer to it,
> and watched real security events flow into a central dashboard.
> In **v2** I added **Sysmon** so Wazuh can see which programs run on the computer, and
> I wrote my **own detection rule** that raises an alert when someone repeatedly guesses
> a password (brute force). I tested both with simulated attacker activity and analysed
> the alerts the way an IT support or SOC analyst would.

---

## What is this?

A **SIEM** (Security Information and Event Management) is a system that collects logs
from computers in one place so you can see, search, and get alerted on security events.
Without it, every machine keeps its own logs and no one is watching them together.

In this lab I set up Wazuh (a free, open-source SIEM) for a fictional small company,
**NordDesk AB**, whose IT support team wants to start monitoring its Windows computers.

```
Windows 11 endpoint (+ Sysmon)  →  Wazuh agent  →  Wazuh server (SIEM)  →  rules  →  alert  →  analysis
```

## Why I built it

I'm studying to become a cybersecurity developer and I'm applying for part-time
IT support / service desk roles. This project shows hands-on skills that matter for
both: installing and configuring systems, working with virtual machines and networking,
reading Windows Event Logs, troubleshooting, and using a real security monitoring tool.

## What I actually did

### v1 — Build the SIEM and monitor logons

- Created an **Ubuntu Server** virtual machine on Hyper-V and installed the full Wazuh
  stack (indexer + manager + dashboard) using the all-in-one installation.
- Connected the server and a Windows 11 endpoint on a Hyper-V lab network.
- Installed the **Wazuh agent** on Windows 11 and registered it with the server.
- **Generated real security events** (failed and successful logons) and analysed them:
  Windows Event IDs, the Wazuh rule that fired, severity, and **MITRE ATT&CK** mapping.

### v2 — Process visibility and my own detection rule

- **Deployed Sysmon** with the community SwiftOnSecurity configuration and configured
  the Wazuh agent to forward Sysmon events.
- **Simulated attacker discovery activity** (`whoami`, `net user`,
  `net localgroup administrators` from PowerShell) and confirmed Wazuh detected it as
  **T1087 Account Discovery**, down to the exact command line and file hashes.
- **Identified a false positive** (a level-15 alert caused by normal PowerShell behaviour)
  and documented how an analyst would verify and tune it.
- **Wrote a custom correlation rule**: alert when the same account fails to log on
  **5+ times within 2 minutes** (MITRE **T1110 Brute Force**), validated it safely
  before loading it, and proved it works with a simulated brute-force attack.
- **Troubleshot real lab problems**: VMs on different subnets, a host disk filled by
  automatic Hyper-V checkpoints, and a Wazuh manager component that had stopped.

## Key things the lab demonstrates

| Skill | Where it shows up |
|---|---|
| Virtualisation & networking | Hyper-V VMs, subnets, DHCP vs static addressing |
| Linux server administration | Ubuntu Server, SSH, systemd services, ports |
| Windows security logging | Event IDs 4624/4625, Sysmon Event ID 1 (process creation) |
| SIEM operation | Wazuh install, agent configuration, searching and analysing alerts |
| Detection engineering | Custom Wazuh correlation rule with MITRE mapping |
| Security analysis | MITRE ATT&CK, false-positive assessment, analyst next steps |
| Troubleshooting | Disk space, Hyper-V checkpoints, agent/manager connectivity |

## Highlight: my custom brute-force rule

Wazuh's built-in rule fires on every single failed logon — but one failed logon is
usually a typo. My rule correlates them:

```xml
<rule id="100100" level="10" frequency="5" timeframe="120">
  <if_matched_sid>60122</if_matched_sid>
  <same_field>win.eventdata.targetUserName</same_field>
  <description>Possible brute force: 5+ failed logons for user $(win.eventdata.targetUserName) within 2 minutes</description>
  <mitre><id>T1110</id></mitre>
</rule>
```

After 16 simulated failed logons, Wazuh raised **3 level-10 alerts** naming the
targeted account:

![Brute-force alert detail](screenshots/v2-09-bruteforce-alert-detail.png)

Full write-up: [`docs/07-custom-rule.md`](docs/07-custom-rule.md)

## Documentation

- [`scenario.md`](scenario.md) — the fictional company and goal
- [`docs/01-architecture.md`](docs/01-architecture.md) — lab design, machines, network
- [`docs/02-installation.md`](docs/02-installation.md) — installing the Wazuh server
- [`docs/03-agent-setup.md`](docs/03-agent-setup.md) — connecting the Windows 11 agent
- [`docs/04-detections.md`](docs/04-detections.md) — v1 events, alerts, and analysis
- [`docs/05-lessons.md`](docs/05-lessons.md) — what I learned, limitations, next steps
- [`docs/06-sysmon.md`](docs/06-sysmon.md) — **v2:** Sysmon process monitoring and detections
- [`docs/07-custom-rule.md`](docs/07-custom-rule.md) — **v2:** custom brute-force rule
- [`docs/08-troubleshooting-v2.md`](docs/08-troubleshooting-v2.md) — **v2:** real problems and how I fixed them
- [`rules/local_rules.xml`](rules/local_rules.xml) — the custom rule itself

## Screenshots

See the [`screenshots/`](screenshots/) folder.

| File | Shows |
|---|---|
| `03-agent-active.png` | v1 — Windows 11 agent registered and **Active** |
| `04-threat-hunting.png` | v1 — Threat Hunting overview with MITRE ATT&CK breakdown |
| `05-events-table.png` | v1 — Table of real security events from the endpoint |
| `06-alert-detail.png` | v1 — A single failed-logon alert, full detail |
| `v2-01-agent-active.png` | v2 — Agent reconnected after troubleshooting |
| `v2-02-sysmon-installed.png` | v2 — Sysmon running and logging events |
| `v2-03-sysmon-dashboard.png` | v2 — Sysmon alerts with MITRE ATT&CK breakdown |
| `v2-04-sysmon-events-list.png` | v2 — List of Sysmon-based detections |
| `v2-05-sysmon-alert-detail.png` | v2 — Discovery alert: command line and hashes |
| `v2-06-sysmon-alert-mitre.png` | v2 — Same alert: MITRE T1087 mapping |
| `v2-07-custom-rule.png` | v2 — The custom brute-force rule on the server |
| `v2-08-bruteforce-alert-list.png` | v2 — Custom rule firing (3 alerts) |
| `v2-09-bruteforce-alert-detail.png` | v2 — Custom rule alert detail, MITRE T1110 |
| `v2-troubleshoot-01..03` | v2 — Disk-full incident: symptom, diagnosis, fix |

## Limitations & next steps

This is a **learning lab**, not a production system:

- **Single endpoint, single-node server.** Not representative of a real, multi-machine
  environment.
- **Rules tested with simulated activity only.** Real environments need tuning against
  normal user behaviour to keep false positives low.
- **No File Integrity Monitoring tuning yet.** Next step: configure FIM for sensitive
  folders.
- **Next step: a small Purple Team exercise** — run a controlled attack chain and show
  what Wazuh detects and what it misses.

## Tech used

Wazuh 4.14 · Sysmon (SwiftOnSecurity config) · Ubuntu Server · Windows 11 · Hyper-V · PowerShell · MITRE ATT&CK

---

*Author: Josipa — cybersecurity student, building a hands-on security portfolio.*
