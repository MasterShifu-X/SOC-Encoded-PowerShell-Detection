# Installation Guide

## Overview

This document describes how to deploy the complete lab environment used in the AI-Assisted SOC Detection and Triage for Encoded PowerShell Attacks project.

The environment consists of:

- Ubuntu Server (Splunk Enterprise)
- Windows 11 Victim Endpoint
- Sysmon
- Splunk Universal Forwarder
- n8n
- VirusTotal API
- Gemini API
- Telegram Bot

---

# Lab Architecture

## Splunk Server

Operating System:

- Ubuntu Server 24.04 LTS

Installed Components:

- Splunk Enterprise
- n8n

---

## Victim Endpoint

Operating System:

- Windows 11

Installed Components:

- Sysmon
- Splunk Universal Forwarder

---

# Prerequisites

## Ubuntu Server

Minimum Requirements:

- 2 CPU Cores
- 4 GB RAM
- 50 GB Storage

Recommended:

- 4 CPU Cores
- 8 GB RAM
- 100 GB Storage

---

## Windows Endpoint

Minimum Requirements:

- 2 CPU Cores
- 4 GB RAM
- 40 GB Storage

---

# Step 1 – Install Splunk Enterprise

Download Splunk Enterprise:

https://www.splunk.com

Install:

```bash
sudo dpkg -i splunk-<version>.deb
```

Start Splunk:

```bash
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt
```

Enable boot startup:

```bash
sudo /opt/splunk/bin/splunk enable boot-start
```

Access Splunk:

```text
http://<SERVER-IP>:8000
```

---

# Step 2 – Install Sysmon

Download Sysmon:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Install Sysmon:

```powershell
sysmon64.exe -i
```

Verify installation:

```powershell
Get-Service Sysmon64
```

Check logs:

```text
Event Viewer
→ Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

---

# Step 3 – Install Splunk Universal Forwarder

Download Splunk Universal Forwarder:

https://www.splunk.com

Install the forwarder on the Windows endpoint.

Configure forwarding:

```powershell
splunk add forward-server <SPLUNK-IP>:9997
```

Configure inputs:

```powershell
splunk add monitor WinEventLog://Microsoft-Windows-Sysmon/Operational
```

Restart forwarder:

```powershell
splunk restart
```

---

# Step 4 – Configure Splunk Receiving Port

On the Splunk server:

Navigate to:

Settings
→ Forwarding and Receiving
→ Configure Receiving

Create a new receiver:

```text
Port: 9997
```

Verify incoming events are received.

---

# Step 5 – Verify Sysmon Logs in Splunk

Run:

```spl
index="mastershifu-ai"
```

Confirm Sysmon events appear.

---

# Step 6 – Install n8n

Install Docker:

```bash
sudo apt update
sudo apt install docker.io -y
```

Run n8n:

```bash
docker run -it --name n8n \
-p 5678:5678 \
-v ~/.n8n:/home/node/.n8n \
docker.n8n.io/n8nio/n8n
```

Access:

```text
http://<SERVER-IP>:5678
```

---

# Step 7 – Create VirusTotal API Key

Create an account:

https://www.virustotal.com

Generate:

```text
API Key
```

Store the key securely.

---

# Step 8 – Create Gemini API Key

Create an account:

https://aistudio.google.com

Generate:

```text
Gemini API Key
```

Store the key securely.

---

# Step 9 – Create Telegram Bot

Open Telegram.

Search:

```text
@BotFather
```

Create a bot:

```text
/newbot
```

Save:

- Bot Token
- Chat ID

---

# Step 10 – Import n8n Workflow

Navigate to:

```text
n8n/
workflow_export.json
```

Import the workflow into n8n.

Configure:

- VirusTotal API Key
- Gemini API Key
- Telegram Bot Token

---

# Step 11 – Import Detection Rule

Navigate to:

```text
splunk/
encoded_powershell_detection.spl
```

Create a new Splunk alert using the detection query.

Configure:

- Scheduled Alert
- Webhook Action
- n8n Webhook URL

---

# Step 12 – Validate the Workflow

Open the phishing website.

Download:

```text
invoice.bat
```

Execute:

```text
invoice.bat
```

Verify:

- Sysmon Event ID 1 generated
- Splunk alert triggered
- n8n workflow executed
- VirusTotal enrichment completed
- Gemini triage generated
- Telegram alert received

---

# Expected Outcome

After successful deployment:

- Encoded PowerShell activity is detected
- Threat intelligence enrichment is performed
- AI-assisted triage is generated
- SOC alert is delivered to Telegram

The complete detection and response workflow should execute automatically.
