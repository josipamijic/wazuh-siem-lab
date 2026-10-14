# 02 — Installing the Wazuh Server

This describes how I installed the Wazuh server on Ubuntu Server 24.04 running as a
Hyper-V virtual machine.

## 1. Create the Ubuntu VM

- Hyper-V → New Virtual Machine, **Generation 2**.
- Memory: 4096 MB (later raised to 6144 MB — see note below), **Dynamic Memory off**.
- 2 virtual processors.
- Attach the Ubuntu Server 24.04 ISO.
- **Secure Boot:** change the template to *Microsoft UEFI Certificate Authority*,
  otherwise the Ubuntu installer will not boot on a Generation 2 VM.

## 2. Install Ubuntu Server

Standard installation. Points worth noting:

- Chose **Ubuntu Server** (not minimized).
- Used the whole disk with LVM, then **expanded the logical volume to use the full
  disk** (by default LVM only claims about half, which would waste space Wazuh wants
  for logs).
- Enabled **OpenSSH server** during install — this later let me manage the server
  remotely from PowerShell, which was far more reliable than typing inside the Hyper-V
  console window.

## 3. Update the system

```bash
sudo apt update && sudo apt upgrade -y
```

## 4. Install Wazuh (all-in-one)

Download and run the official installation assistant:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

The `-a` flag installs all central components (indexer, manager, dashboard) on this
single host — ideal for a single-node lab.

> **Hardware requirement:** the installer checks for at least 4 GB RAM and 2 CPU cores.
> With memory assigned exactly at 4 GB the check can fail (the OS uses some itself),
> so I raised the VM to 6 GB during installation.

When it finishes, the installer prints the dashboard login:

```
User: admin
Password: <generated password>
```

I saved this immediately — it cannot be recovered easily if lost.

## 5. Verify the server is running

```bash
sudo systemctl status wazuh-manager
```

Expected: **active (running)**.

## 6. Open the dashboard

From the laptop browser:

```
https://<wazuh-server-ip>
```

The browser shows a certificate warning (the lab uses a self-signed certificate, which
is expected). After accepting it, the Wazuh login screen appears. Logging in with
`admin` and the generated password opens the dashboard.

At this point the SIEM is running but has **no agents** yet — that comes next.
