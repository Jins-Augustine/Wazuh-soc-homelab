
# Wazuh SOC Home Lab

A home Security Operations Center (SOC) lab built using Wazuh SIEM to gain hands-on experience with security monitoring, endpoint detection, and blue team operations.

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Arch Linux Host Machine                   │
│                                                             │
│   ┌─────────────────────────┐   ┌───────────────────────┐  │
│   │         Docker          │   │      VirtualBox        │  │
│   │                         │   │                        │  │
│   │  ┌───────────────────┐  │   │  ┌──────────────────┐ │  │
│   │  │  Wazuh Manager    │◄─┼───┼──│  Ubuntu VM       │ │  │
│   │  │  Wazuh Indexer    │  │   │  │  (Wazuh Agent)   │ │  │
│   │  │  Wazuh Dashboard  │◄─┼───┼──│  Windows VM      │ │  │
│   │  └───────────────────┘  │   │  │  (Wazuh Agent)   │ │  │
│   └─────────────────────────┘   │  └──────────────────┘ │  │
│                                  └───────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

| Component | Where It Runs | Role |
|---|---|---|
| Wazuh Manager | Docker (Arch host) | Receives and analyzes agent data |
| Wazuh Indexer | Docker (Arch host) | Stores and indexes all events |
| Wazuh Dashboard | Docker (Arch host) | Web UI for monitoring and analysis |
| Wazuh Agent | Ubuntu VM | Monitored Linux endpoint |
| Wazuh Agent | Windows VM | Monitored Windows endpoint |

---

## Tools & Technologies

- **Wazuh** — open-source SIEM and XDR platform
- **Docker + Docker Compose** — runs the Wazuh server stack
- **VirtualBox** — hosts the monitored endpoint VMs
- **Hydra** — used to simulate SSH brute force attacks
- **Arch Linux** — host OS (pacman-based, no apt)
- **Ubuntu & Windows** — monitored endpoint VMs

---

## What I Implemented

### 1. SIEM Deployment
Deployed the full Wazuh stack (Manager + Indexer + Dashboard) using Docker Compose on an Arch Linux host. Configured SSL certificates and accessed the dashboard via browser at `https://localhost`.

### 2. Endpoint Monitoring
Connected two Wazuh agents — one Ubuntu VM and one Windows VM — both showing as Active in the dashboard. Both endpoints report logs, system events, and security alerts in real time.

### 3. File Integrity Monitoring (FIM)
Configured real-time FIM on both endpoints by editing `ossec.conf` on each agent. Monitored directories for file creation, modification, and deletion events. Verified alerts appeared in the dashboard immediately upon file changes.

### 4. SSH Brute Force Attack Simulation
Used Hydra from the Arch host to simulate a brute force attack against the Ubuntu VM's SSH service. Watched Wazuh catch every attempt live on the dashboard — including `sshd: authentication failed`, `PAM: User login failed`, and ultimately `sshd: authentication success` when the correct password was found.

---

## Lab Evidence

See the `/evidence` folder for screenshots of:
- Both agents showing Active status in the dashboard
- FIM alerts for file creation and deletion
- Brute force alerts flooding in during the Hydra attack
- Successful login caught after brute force

---

## Setup Guide

The full step-by-step setup documentation is in `/docs/setup-guide.docx`. It covers every command run during this lab including explanations for why each step is needed — especially adapted for Arch Linux where the official Wazuh docs assume Ubuntu/Debian.

---

## Lab Walkthroughs

- [`labs/lab1-fim.md`](labs/lab1-fim.md) — File Integrity Monitoring setup and testing
- [`labs/lab2-bruteforce.md`](labs/lab2-bruteforce.md) — SSH brute force simulation and detection

---

## Key Takeaways

- Deploying Wazuh on Arch Linux required adapting Ubuntu-based documentation — manually loading kernel modules, managing groups, and swapping `apt` commands for `pacman`
- FIM with `realtime="yes"` triggers alerts instantly vs the default 12-hour scan interval
- A brute force attack generates a clear pattern in the SIEM: repeated auth failures followed by a session open event — the entire attack story is visible in one dashboard view
- Docker makes the Wazuh server stack easy to spin up and tear down without polluting the host system
