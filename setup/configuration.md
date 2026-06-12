# Configuration Guide

## Overview

This document describes the configuration used in the AI-Assisted SOC Detection and Triage for Encoded PowerShell Attacks project.

The configuration covers:

* Sysmon
* Splunk Enterprise
* Splunk Alerting
* n8n Workflow
* VirusTotal Integration
* Gemini AI Integration
* Telegram Integration

---

# Sysmon Configuration

## Purpose

Sysmon provides detailed endpoint telemetry used by Splunk for threat detection.

Relevant Event:

```text
Event ID 1 - Process Creation
```

Detection relies on:

* Process Name
* Command Line
* Parent Process
* User Context
* Hashes

Verification:

```text
Event Viewer
→ Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

---

# Splunk Configuration

## Receiving Port

Configure receiving:

```text
Settings
→ Forwarding and Receiving
→ Configure Receiving
```

Receiver Port:

```text
9997
```

---

## Index

Project Index:

```text
mastershifu-ai
```

---

## Detection Query

```spl
index="mastershifu-ai"
source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=1
(Image="*\\powershell.exe" OR Image="*\\pwsh.exe")
(CommandLine="*-enc*" OR CommandLine="*-e *" OR CommandLine="*-EncodedCommand*")
| table _time, ComputerName, User, CommandLine, ParentImage
```

Purpose:

* Detect encoded PowerShell execution
* Identify obfuscated command usage
* Generate SOC alerts

---

# Splunk Alert Configuration

## Alert Type

```text
Scheduled Alert
```

## Schedule

```text
Cron Schedule
```

Example:

```text
*/5 * * * *
```

Runs every 5 minutes.

---

## Trigger Condition

```text
Number of Results > 0
```

---

## Trigger Action

```text
Webhook
```

Destination:

```text
n8n Webhook URL
```

Purpose:

* Send detection events to n8n
* Initiate automation workflow

---

# n8n Configuration

## Workflow Components

Workflow includes:

1. Webhook Trigger
2. VirusTotal Lookup
3. Merge Node
4. Gemini AI Analysis
5. Telegram Alert

---

## Webhook

Purpose:

Receive Splunk alert data.

Expected Fields:

* Host
* User
* Process
* Command Line
* SHA256
* Time

---

# VirusTotal Configuration

## Purpose

Provide threat intelligence enrichment.

Data Retrieved:

* Malicious Detections
* Suspicious Detections
* Reputation Score

Input:

```text
SHA256 Hash
```

Output Example:

```text
Malicious: 0
Suspicious: 0
Reputation: -24
```

---

# Gemini Configuration

## Purpose

Generate analyst-friendly triage summaries.

Input:

* Host
* User
* Process
* Command Line
* VirusTotal Results

Output:

* Command Summary
* MITRE ATT&CK Mapping
* Threat Assessment
* Investigation Steps
* Containment Recommendations

---

## Prompt Design

Gemini is configured to:

* Avoid severity scoring
* Avoid unsupported assumptions
* Provide concise analyst guidance
* Use numbered investigation steps
* Use numbered containment steps

---

# Telegram Configuration

## Purpose

Deliver analyst notifications.

Alert Contents:

* Host
* User
* Process
* VirusTotal Results
* AI Analysis
* Timestamp

---

## Message Format

```text
SOC ALERT

Host
User
Process

VirusTotal Results

AI Analysis

Time
```

Purpose:

* Rapid analyst visibility
* Mobile-friendly alerting
* Immediate triage support

---

# Validation Checklist

After configuration is complete:

✓ Sysmon generates Event ID 1

✓ Splunk receives Sysmon events

✓ Detection query identifies encoded PowerShell

✓ Splunk alert triggers

✓ Webhook reaches n8n

✓ VirusTotal enrichment succeeds

✓ Gemini analysis is generated

✓ Telegram notification is delivered

---

# Troubleshooting

## No Events in Splunk

Verify:

* Universal Forwarder is running
* Receiving port 9997 is open
* Sysmon logs exist

---

## Alert Not Triggering

Verify:

* Detection query returns results
* Alert schedule is active
* Trigger condition is configured correctly

---

## n8n Workflow Not Running

Verify:

* Webhook URL is reachable
* Workflow is active
* Execution logs show incoming requests

---

## Telegram Alert Not Delivered

Verify:

* Bot token is valid
* Chat ID is correct
* Message length does not exceed Telegram limits

---

# Outcome

When configured correctly, the workflow automatically detects encoded PowerShell activity, enriches the event with threat intelligence, generates AI-assisted analysis, and delivers an actionable alert to the SOC analyst.

