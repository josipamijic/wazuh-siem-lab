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

## Limitations

This is a learning lab, and naming its limits is part of understanding the tool:

- **Single endpoint, single-node server.** Not representative of a real, multi-machine
  environment.
- **Default logging only.** Out of the box Wazuh does not record every process launch or
  file change, so things like "Word spawning PowerShell" are not yet visible.
- **No custom detection rules.** All alerts come from Wazuh's built-in rule set.
- **Self-signed certificate.** Fine for a lab, not for production.

## Next steps (planned v2)

1. **Deploy Sysmon + File Integrity Monitoring.** This gives detailed process and file
   visibility — the biggest single upgrade in detection power, and what makes realistic
   attack chains visible.
2. **Write a custom detection rule.** For example: alert on 5+ failed logins within
   2 minutes (brute-force detection). This moves from *using* the SIEM to *tuning* it
   (detection engineering).
3. **Simulate and detect an attack.** Run a controlled attack against a target and show
   the before/after in Wazuh (a small Purple Team exercise).

These are deliberately left as next steps so v1 could be finished and published rather
than endlessly polished.
