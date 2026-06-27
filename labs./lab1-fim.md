# Lab 1 — File Integrity Monitoring (FIM)

## Objective

Configure Wazuh to monitor a directory in real time on both Ubuntu and Windows endpoints, and verify that file creation and deletion events appear as alerts in the dashboard.

---

## What is FIM?

Wazuh's FIM (called Syscheck) watches specified files and directories and alerts when anything is:
- Created
- Modified
- Deleted

Without `realtime="yes"`, Wazuh only scans every 12 hours by default. Real-time mode uses OS-level file watchers (inotify on Linux, ReadDirectoryChangesW on Windows) to trigger alerts instantly.

---

## Ubuntu VM — Setup

**1. Open the agent config file:**
```bash
sudo nano /var/ossec/etc/ossec.conf
```

**2. Find the `<syscheck>` section and add your monitored directory:**
```xml
<directories realtime="yes">/home/vboxuser/Downloads/wazuh-test</directories>
```

**3. Restart the agent to apply changes:**
```bash
sudo systemctl restart wazuh-agent
```

---

## Windows VM — Setup

**1. Open the agent config file:**
```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

**2. Find the `<syscheck>` section and add:**
```xml
<directories realtime="yes">C:\Users\vboxuser\Downloads\wazuh-test</directories>
```

**3. Restart the Wazuh Agent service via Windows Services.**

---

## Testing FIM

**On Ubuntu VM, create a test file:**
```bash
mkdir ~/Downloads/wazuh-test
echo "hello" > ~/Downloads/wazuh-test/test.txt
```

**Then delete it:**
```bash
rm ~/Downloads/wazuh-test/test.txt
```

---

## Expected Alerts in Dashboard

Navigate to: **Agent → Integrity Monitoring**

You should see events for:

| Event | Description |
|---|---|
| `File added` | When test.txt was created |
| `File deleted` | When test.txt was removed |

> **Note:** If you click on a deleted file entry and see "Data could not be fetched" — this is expected. The file no longer exists so Wazuh cannot retrieve its current state. The deletion event itself is correctly logged.

---

## Evidence

See `/evidence/fim-alerts.png` for screenshot of alerts in the dashboard.
