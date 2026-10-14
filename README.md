# Wazuh SIEM Home Lab — Monitoring a Windows 11 Endpoint

> **In 30 seconds:** I built a small security monitoring system (a SIEM) at home.
> I installed **Wazuh** on an Ubuntu server, connected a **Windows 11** computer to it,
> and watched real security events — like failed and successful logins — flow from the
> Windows machine into a central dashboard. I then analysed those events the way an
> IT support or SOC analyst would: what happened, is it normal or suspicious, and what
> I would do about it. The project shows I can set up the tools, read Windows logs,
> and understand what they mean.

---

## What is this?

A **SIEM** (Security Information and Event Management) is a system that collects logs
from computers in one place so you can see, search, and get alerted on security events.
Without it, every machine keeps its own logs and no one is watching them together.

In this lab I set up Wazuh (a free, open-source SIEM) for a fictional small company,
**NordDesk AB**, whose IT support team wants to start monitoring its Windows computers.

The full chain I built and tested:

```
Windows 11 endpoint  →  Wazuh agent  →  logs  →  Wazuh server (SIEM)  →  alert  →  analysis
```

## Why I built it

I'm studying to become a cybersecurity developer and I'm applying for part-time
IT support / service desk roles. This project shows hands-on skills that matter for
both: installing and configuring systems, working with virtual machines and networking,
reading Windows Event Logs, and using a real security monitoring tool.

## What I actually did

- Created an **Ubuntu Server 24.04** virtual machine on Hyper-V and installed the full
  Wazuh stack (indexer + manager + dashboard) using the all-in-one installation.
- Created an **isolated internal lab network** in Hyper-V so the server and the Windows
  endpoint could talk to each other in a controlled environment.
- Installed the **Wazuh agent** on a Windows 11 machine and registered it with the server
  (confirmed **Active** in the dashboard).
- **Generated real security events** on Windows (failed logins, successful logins) and
  confirmed they were collected and analysed by Wazuh.
- **Analysed the alerts**: identified the Windows Event IDs, the Wazuh rule that fired,
  the severity level, and the **MITRE ATT&CK** technique Wazuh mapped them to.
- Investigated a single failed-login alert down to the detail level (authentication
  package, logon type, failure reason, Windows status codes).

## Key things the lab demonstrates

| Skill | Where it shows up |
|---|---|
| Virtualisation & networking | Hyper-V VMs, internal switch, static/DHCP addressing |
| Linux server administration | Ubuntu Server install, SSH, service management |
| Windows security logging | Event ID 4625 (failed logon), 4624 (successful logon) |
| SIEM operation | Wazuh install, agent enrollment, dashboard, searching events |
| Security analysis | Reading alerts, severity, MITRE ATT&CK mapping |

## Example: analysing a failed login

When I ran repeated failed authentication attempts on the Windows endpoint, Wazuh
captured each one as **Event ID 4625 — "Logon Failure – Unknown user or bad password"**,
rule level 5. Opening one alert showed useful detail for an analyst:

- **Authentication package:** NTLM
- **Logon type:** 3 (a network logon — matches my test method)
- **Failure reason:** `%%2313` (unknown user name or bad password)
- **Status / sub-status:** `0xC000006D` / `0xC0000064` (the account does not exist)
- **MITRE ATT&CK:** mapped to credential-access / valid-accounts techniques

A single failed login is normal. **Many** failed logins in a short time is a classic
sign of a brute-force or password-spraying attempt — which is exactly the kind of
pattern a SIEM is built to surface.

## Documentation

- [`scenario.md`](scenario.md) — the fictional company and goal
- [`docs/01-architecture.md`](docs/01-architecture.md) — lab design, machines, network
- [`docs/02-installation.md`](docs/02-installation.md) — how I installed the Wazuh server
- [`docs/03-agent-setup.md`](docs/03-agent-setup.md) — connecting the Windows 11 agent
- [`docs/04-detections.md`](docs/04-detections.md) — the events, alerts, and analysis
- [`docs/05-lessons.md`](docs/05-lessons.md) — what I learned, limitations, next steps

## Screenshots

See the [`screenshots/`](screenshots/) folder:

| File | Shows |
|---|---|
| `03-agent-active.png` | Windows 11 agent registered and **Active** in Wazuh |
| `04-threat-hunting.png` | Threat Hunting overview with MITRE ATT&CK breakdown |
| `05-events-table.png` | Table of real security events from the endpoint |
| `06-alert-detail.png` | A single failed-login alert, full detail |

## Limitations & next steps

This is a **learning lab**, not a production system. Being honest about its limits is
part of understanding how the tool really works:

- **No process or file monitoring yet.** By default, Wazuh does not log every process
  that starts or every file that changes. Next step: deploy **Sysmon** and enable
  **File Integrity Monitoring (FIM)** for much deeper endpoint visibility (e.g. detecting
  "Word launches PowerShell" or encoded PowerShell commands).
- **No custom detection rules yet.** Next step: write a custom Wazuh rule, for example
  to alert on **5+ failed logins within 2 minutes** (brute-force detection) — moving from
  using the tool to tuning it (detection engineering).
- **Single endpoint, flat lab network.** A production setup would segment the network
  and monitor many endpoints.

## Tech used

Wazuh 4.x · Ubuntu Server 24.04 · Windows 11 · Hyper-V · MITRE ATT&CK

---

*Author: Josipa — cybersecurity student, building a hands-on security portfolio.*
