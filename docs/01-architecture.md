# 01 — Lab Architecture

## Overview

The lab runs entirely inside **Hyper-V** on a single laptop. It has two virtual
machines: one acts as the security monitoring server, the other as the computer being
monitored.

```
                    Hyper-V (laptop)
   ┌─────────────────────────────────────────────┐
   │                                               │
   │   ┌───────────────┐        ┌───────────────┐  │
   │   │  Wazuh Server  │        │  Windows 11    │  │
   │   │  (Ubuntu 24.04)│◄──────►│  Endpoint      │  │
   │   │  SIEM          │  logs  │  Wazuh agent   │  │
   │   └───────────────┘        └───────────────┘  │
   │            ▲                                    │
   │            │ internal lab network               │
   └────────────┼────────────────────────────────────┘
                │
        Browser on the laptop
        opens the Wazuh dashboard (HTTPS)
```

## Machines

| Machine | Role | OS | Resources |
|---|---|---|---|
| `wazuh-server` | SIEM (indexer + manager + dashboard) | Ubuntu Server 24.04 | ~4–6 GB RAM, 2 vCPU, ~120 GB disk |
| `Win11-endpoint` | Monitored endpoint | Windows 11 Pro | ~3 GB RAM |

## Network

Both machines sit on the same internal lab network so they can reach each other while
staying isolated from everything else. The Windows endpoint sends its logs to the
Wazuh server; the server analyses them and shows them in the dashboard.

| Component | Purpose |
|---|---|
| Wazuh agent (on Windows) | Collects Windows Event Logs and sends them to the server |
| Wazuh manager (on Ubuntu) | Receives events, runs detection rules, raises alerts |
| Wazuh indexer | Stores the events so they can be searched |
| Wazuh dashboard | Web interface (HTTPS) to view and investigate events |

## Ports that matter

| Port | Used for |
|---|---|
| 1514 | Agent sends events to the manager |
| 1515 | Agent enrollment / registration |
| 443 | Wazuh dashboard (web interface) |

> Note on addressing: getting stable IP addressing between the two VMs on Hyper-V's
> virtual switches was the trickiest part of the setup. See
> [`05-lessons.md`](05-lessons.md) for what I learned.
