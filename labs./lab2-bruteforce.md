# Lab 2 — SSH Brute Force Attack Detection

## Objective

Simulate an SSH brute force attack from the Arch host against the Ubuntu VM using Hydra, and observe Wazuh detect and log the full attack chain in real time.

---

## Setup

### On the Ubuntu VM (target)

**Enable SSH:**
```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

**Verify SSH is running:**
```bash
sudo systemctl status ssh
```

**Get the VM's IP address:**
```bash
ip a
```
Note the IP (e.g. `192.168.1.x`) — this is the attack target.

---

### On the Arch Host (attacker)

**Install Hydra:**
```bash
sudo pacman -S hydra
```

**Create a password wordlist:**
```bash
cat > ~/passwords.txt << 'EOF'
123456
password
admin
letmein
welcome
qwerty
abc123
monkey
dragon
master
YOUR_ACTUAL_VM_PASSWORD
EOF
```

Replace `YOUR_ACTUAL_VM_PASSWORD` with the actual Ubuntu VM user password so Hydra eventually succeeds and generates the full attack story.

---

## Running the Attack

```bash
hydra -l vboxuser -P ~/passwords.txt ssh://192.168.1.x -t 4 -V
```

| Flag | Meaning |
|---|---|
| `-l vboxuser` | Username to attack |
| `-P ~/passwords.txt` | Password list file (capital P = file, lowercase p = single password) |
| `ssh://192.168.1.x` | Target IP |
| `-t 4` | 4 parallel threads |
| `-V` | Verbose — shows each attempt live |

---

## What Wazuh Catches

Open the dashboard at `https://localhost` → **Security Events** before running the attack and watch alerts appear in real time.

| Alert | What it means |
|---|---|
| `sshd: authentication failed` | Hydra trying wrong passwords |
| `PAM: User login failed` | Linux auth system confirming failures |
| `unix_chkpwd: Password check failed` | Password verification failing |
| `sshd: authentication success` | Hydra found the correct password |
| `PAM: Login session opened` | Attacker successfully logged in |
| `PAM: Login session closed` | Session ended |

The full attack story — from first failed attempt to successful login — is visible in one dashboard view. This is exactly what a SOC analyst would see during a real brute force incident.

---

## Evidence

See `/evidence/bruteforce-alerts.png` for screenshot of the alerts flooding in during the attack.
See `/evidence/successful-login-caught.png` for screenshot of the successful login event.
