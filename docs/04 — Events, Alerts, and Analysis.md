# 04 — Events, Alerts, and Analysis

This is the heart of the project: generating real security events on the endpoint and
analysing them the way an analyst would.

## Generating events

On the Windows endpoint I generated authentication events on purpose. Failed logins
are one of the most common early signals a SOC watches, because a burst of them can
mean a brute-force or password-spraying attack.

To create **failed logins**, I ran an authentication attempt with a wrong password
several times from an Administrator command prompt:

```cmd
net use \\localhost\IPC$ /user:testuser WrongPassword123
```

Each failed attempt produces a Windows **Event ID 4625**. Normal logins produce
**Event ID 4624**.

## Finding them in Wazuh

In the dashboard → **Threat Hunting**, the events appeared within a minute:

- **Authentication failure:** 2
- **Authentication success:** 2
- A **Top MITRE ATT&CK** breakdown was generated automatically, mapping the events to
  techniques such as *Valid Accounts*, *Account Access Removal*, and *Remote Desktop
  Protocol*.

> Screenshot: `screenshots/04-threat-hunting.png`

## Reading the events table

The security-events table shows exactly what an analyst works with:

| Field | Meaning |
|---|---|
| `timestamp` | When it happened |
| `agent.name` | Which endpoint sent it (`Win11-endpoint`) |
| `rule.description` | What Wazuh interprets the event as |
| `rule.level` | Severity (higher = more serious) |
| `rule.id` | The Wazuh rule that fired |

The failed logins showed as **"Logon Failure – Unknown user or bad password"**,
rule level 5, rule id 60122.

> Screenshot: `screenshots/05-events-table.png`

## Investigating a single alert

Opening one Event ID 4625 alert in **Document Details** gave the full picture:

| Field | Value | What it tells me |
|---|---|---|
| `agent.name` | Win11-endpoint | Which machine was targeted |
| `authenticationPackageName` | NTLM | The authentication method used |
| `logonType` | 3 | A **network** logon — matches my `net use` test |
| `failureReason` | %%2313 | Unknown user name or bad password |
| `status` / `subStatus` | 0xC000006D / 0xC0000064 | The account does not exist |
| `ipAddress` | ::1 | Attempt came from the local machine (my test) |

> Screenshot: `screenshots/06-alert-detail.png`

## Analyst's verdict

Using the standard "is this normal or suspicious?" approach:

- **What happened:** repeated failed network logons for a non-existent user on one
  endpoint, over a very short time window.
- **Normal or suspicious:** a single failure is normal (someone mistypes a password).
  Several failures in seconds, for an unknown user, is the signature of a **brute-force
  / password-spraying** attempt.
- **MITRE ATT&CK:** Credential Access — Brute Force.
- **What an analyst would do:** check how many attempts and from where, confirm whether
  the target account exists and whether any attempt later *succeeded*, and if it looks
  like a real attack, consider locking the account and blocking the source.

This small example shows the full value of a SIEM: the raw Windows event becomes a
searchable, severity-rated, MITRE-mapped alert that an analyst can reason about.
