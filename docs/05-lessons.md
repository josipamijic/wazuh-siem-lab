# 05 — Lessons Learned, Limitations & Next Steps

## What I learned

- **The full SIEM chain, end to end.** Not just "install a tool", but understanding how
  an event travels: endpoint → agent → server → rule → alert → analysis.
- **Reading Windows security events.** Event IDs 4625 and 4624, logon types, NTLM, and
  Windows status codes now mean something concrete to me.
- **MITRE ATT&CK in practice.** Seeing Wazuh map raw events onto attacker techniques
  made the framework click.
- **Working methodically.** I rebuilt the server cleanly rather than patching a broken
  setup, used checkpoints as safety nets, and documented as I went.

## Lab challenges (and how I solved them)

- **Memory limits.** Wazuh needs ~4 GB RAM; running the server and a Windows VM together
  on a laptop meant carefully balancing memory and turning Dynamic Memory off for the
  server (its database is unstable if memory is taken away from it).
- **Networking between VMs was the hardest part.** Hyper-V's Default Switch handed the
  two VMs addresses on *different* subnets, so they couldn't see each other. I solved it
  by putting both on a controlled network and setting addressing manually so the agent
  always finds the server.
- **The Hyper-V console blocked pasting and some keys.** I switched to managing the
  Ubuntu server over **SSH from PowerShell**, where copy-paste works — a much more
  reliable and realistic way to administer a Linux server.

## What I learned in v2

- **Sysmon makes the endpoint visible.** Process creation with full command lines and
  hashes turns "something happened" into "this exact program ran this exact command".
- **Detection engineering.** Writing a correlation rule (frequency, timeframe,
  same field) showed me the difference between a single event and a pattern worth an alert.
- **Not every high-severity alert is an attack.** A level-15 alert turned out to be most
  likely normal PowerShell behaviour; assessing it is part of the analyst's job.
- **Troubleshooting in layers.** Network → port → service → logs. And: Hyper-V automatic
  checkpoints can silently fill a disk. See [`08-troubleshooting-v2.md`](08-troubleshooting-v2.md).

## Limitations

This is a learning lab, and naming its limits is part of understanding the tool:

- **Single endpoint, single-node server.** Not representative of a real, multi-machine
  environment.
- **Rules tested with simulated activity only.** Real environments need tuning against
  normal user behaviour.
- **Self-signed certificate.** Fine for a lab, not for production.

## Status of the planned v2

| Planned | Status |
|---|---|
| Deploy Sysmon | ✅ Done — see [`06-sysmon.md`](06-sysmon.md) |
| Custom brute-force rule | ✅ Done — see [`07-custom-rule.md`](07-custom-rule.md) |
| File Integrity Monitoring | ⏳ Not yet — next step |
| Simulate and detect an attack chain (Purple Team) | ⏳ Not yet — next step |
