# Workflow

## Overview

This document describes the complete attack simulation, detection process, enrichment workflow, and notification pipeline used in the AI-Assisted SOC Detection and Triage for Encoded PowerShell Attacks project.

---

# 1. Attack Simulation

## Stage 1 – Phishing Delivery

The attack begins with a phishing website designed to imitate a legitimate corporate document portal.

The website presents a fake invoice download page intended to convince users to download and execute a file named:

```text
invoice.bat
```

The BAT file serves as the initial execution vector within the simulated attack chain.

---

## Stage 2 – Payload Execution

The victim downloads and executes the BAT file.

Upon execution:

* A PowerShell process is launched
* ExecutionPolicy is bypassed
* The command is executed using Base64 encoding

Example behavior:

```text
powershell.exe -NoProfile -ExecutionPolicy Bypass -enc <Base64 Payload>
```

Encoded PowerShell commands are frequently used by attackers to conceal malicious activity and evade basic detection mechanisms.

---

# 2. Endpoint Telemetry Collection

## Sysmon Monitoring

Sysmon is installed on the victim endpoint to provide detailed process creation telemetry.

When the PowerShell process is launched:

* Sysmon records Event ID 1
* Process metadata is collected
* Command-line arguments are logged
* Parent-child process relationships are preserved

Relevant information captured includes:

* Process Name
* Command Line
* Parent Process
* User Account
* Timestamp
* Hash Information

---

# 3. Log Forwarding

## Splunk Universal Forwarder

The Splunk Universal Forwarder is installed on the victim endpoint.

The forwarder continuously monitors Sysmon logs and sends telemetry to the Splunk Enterprise server.

Flow:

Victim Endpoint
→ Sysmon
→ Splunk Universal Forwarder
→ Splunk Enterprise

---

# 4. Detection Logic

## Encoded PowerShell Detection

Splunk continuously analyzes incoming Sysmon events.

The detection rule searches for:

* powershell.exe execution
* pwsh.exe execution
* EncodedCommand usage
* Base64 encoded command parameters

Detection SPL:

```spl
index="mastershifu-ai"
source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=1
(Image="*\\powershell.exe" OR Image="*\\pwsh.exe")
(CommandLine="*-enc*" OR CommandLine="*-e *" OR CommandLine="*-EncodedCommand*")
| table _time, ComputerName, User, CommandLine, ParentImage
```

If matching activity is identified, Splunk generates an alert.

---

# 5. Alert Automation

## Splunk Alert Trigger

When the detection rule identifies a matching event:

* Splunk creates an alert
* A webhook action is triggered
* Event data is sent to n8n

Alert information includes:

* Hostname
* User
* Process Name
* Command Line
* SHA256 Hash
* Timestamp

---

# 6. SOAR Workflow

## n8n Processing

n8n receives the webhook payload from Splunk.

The workflow performs the following actions:

1. Parse alert information
2. Extract relevant indicators
3. Query VirusTotal
4. Gather enrichment data
5. Send context to Gemini AI
6. Deliver final analyst summary to Telegram

This stage acts as the orchestration layer of the workflow.

---

# 7. Threat Intelligence Enrichment

## VirusTotal Integration

The SHA256 hash from the detected process is submitted to VirusTotal.

Returned intelligence includes:

* Malicious detections
* Suspicious detections
* Reputation score
* Community verdicts

The enrichment results provide additional context for analysts during triage.

---

# 8. AI-Assisted Triage

## Gemini AI Analysis

The enriched alert is forwarded to Gemini AI.

Gemini analyzes:

* Process information
* Command behavior
* VirusTotal results
* Detection context

Gemini generates:

* Command summary
* MITRE ATT&CK mapping
* Threat assessment
* Investigation recommendations
* Containment recommendations

The objective is not to replace analysts but to accelerate the initial triage process.

---

# 9. Alert Notification

## Telegram Delivery

The final incident summary is delivered to Telegram.

The alert contains:

* Host information
* User information
* Process information
* VirusTotal results
* AI-generated analysis
* Investigation guidance
* Containment guidance

This allows SOC analysts to receive incident information in near real-time.

---

# 10. End-to-End Workflow

Complete attack flow:

Phishing Website
→ invoice.bat Download
→ PowerShell Execution
→ Sysmon Event ID 1
→ Splunk Universal Forwarder
→ Splunk Enterprise
→ Detection Rule
→ Webhook Alert
→ n8n SOAR Workflow
→ VirusTotal Enrichment
→ Gemini AI Triage
→ Telegram Alert

---

# Outcome

The workflow demonstrates an end-to-end SOC detection and triage pipeline that combines endpoint telemetry, SIEM detection, SOAR automation, threat intelligence enrichment, and AI-assisted analysis to improve incident visibility and accelerate investigation.

