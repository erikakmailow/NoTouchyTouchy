# NoTouchyTouchy

**Windows User Activity Investigation Toolkit**

> **Who logged into this machine, when did they do it, where did they come from, and what were they doing?**

NoTouchyTouchy is a read-only Windows investigation toolkit designed to reconstruct user activity from native Windows telemetry.

Instead of looking at authentication events, sessions, processes, and remote access independently, WhoTouchedMyPC is designed to correlate them into a chronological picture of activity on a Windows endpoint.

---

## What It Does

NoTouchyTouchy collects and correlates evidence from several Windows activity sources:

- Windows authentication events
- Active Windows sessions
- Running processes and process ownership
- Remote Desktop activity
- Local user accounts
- Chronological activity timelines

The goal is simple:

**Turn scattered Windows telemetry into an understandable answer to "Who touched my PC?"**

---

## Architecture

```text
                    WHOTOUCHEDMYPC
                           │
                           ▼
                  ┌─────────────────┐
                  │ Investigation   │
                  │    Modules      │
                  └────────┬────────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
   LogonHunt          SessionHunt         ProcessHunt
       │                   │                   │
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                           ▼
                     RemoteHunt
                           │
                           ▼
                     AccountHunt
                           │
                           ▼
                  ┌─────────────────┐
                  │ TimelineEngine  │
                  └────────┬────────┘
                           │
                           ▼
                 Activity Reconstruction
```

---

## Current Modules

| Module | Purpose |
|---|---|
| **LogonHunt** | Investigates successful and failed Windows authentication events |
| **SessionHunt** | Identifies active Windows logon sessions |
| **ProcessHunt** | Associates running processes with their Windows user |
| **RemoteHunt** | Investigates RDP and remote interactive activity |
| **AccountHunt** | Inventories local Windows user accounts |
| **TimelineEngine** | Normalizes investigation data into a chronological activity timeline |

---

## LogonHunt

Examines Windows Security events associated with authentication.

Currently focuses on:

- Event ID `4624` — successful logon
- Event ID `4625` — failed logon
- Username
- Domain
- Logon type
- Source address
- Workstation
- Authentication package
- Event timestamp

Example:

```text
User       : Erika
Logon Type : 10 - RemoteInteractive / RDP
Source     : 192.168.1.50
Time       : 2026-10-08 13:42:18
```

---

## SessionHunt

Investigates Windows logon sessions using native CIM/WMI data.

Collected information includes:

- User
- Session ID
- Logon type
- Session start time
- Computer

This provides context around users who currently have or recently established Windows sessions.

---

## ProcessHunt

Enumerates running Windows processes and attempts to identify the account responsible for each process.

Collected information includes:

- Process name
- Process ID
- Parent process ID
- Process owner
- Domain
- Executable path
- Computer

Example:

```text
Process        PID     User
-------        ---     ----
explorer.exe   4216    Erika
powershell.exe 8124    Erika
svchost.exe    1032    SYSTEM
```

This allows process activity to be associated with the user context that launched it.

---

## RemoteHunt

Investigates evidence of remote access to the system.

Current telemetry includes:

- Windows Remote Interactive logons
- RDP authentication events
- Source IP address
- Remote username
- Domain
- Authentication package
- Event timestamp

The goal is to distinguish local activity from activity originating from another system.

---

## AccountHunt

Provides context about local Windows accounts.

Collected information includes:

- Username
- Domain
- SID
- Enabled state
- Built-in account status
- Account description

This allows other investigation modules to distinguish known local accounts from unexpected or disabled accounts.

---

## TimelineEngine

TimelineEngine is the beginning of the correlation layer.

It takes results from the investigation modules and normalizes them into a chronological structure.

```text
13:41:02   Authentication   Erika    LogonHunt
13:41:05   Session          Erika    SessionHunt
13:41:07   Process          Erika    ProcessHunt
13:41:18   RemoteActivity   Erika    RemoteHunt
```

The long-term goal is to transform these individual observations into an investigation narrative.

---

# Design Philosophy

NoTouchyTouchy is intentionally designed around several principles.

### Read-Only

The toolkit should observe the endpoint without modifying system configuration or creating persistence.

### Native Telemetry

Where practical, NoTouchyTouchy uses native Windows telemetry and management interfaces rather than requiring third-party agents.

### Correlation Over Enumeration

Listing events is easy.

Understanding how those events relate to one another is more valuable.

### Investigation First

The project is intended to help answer investigative questions rather than simply produce large amounts of system information.

### Modular Architecture

Each investigation capability is separated into a module so functionality can be tested independently and expanded without rewriting the entire toolkit.

---

# Example Investigation

A future investigation might reconstruct an activity chain such as:

```text
08:14:22
Successful Windows authentication
User: Erika
Logon Type: Interactive

        ↓

08:14:24
Windows session established
User: Erika

        ↓

08:14:27
explorer.exe running
User: Erika

        ↓

08:15:03
powershell.exe running
User: Erika

        ↓

08:15:41
Remote interactive authentication detected
Source: 192.168.1.50
```

Rather than investigating each event independently, NoTouchyTouchy can eventually present this as a single activity sequence.

---

# Roadmap

Planned capabilities include:

- [x] Logon investigation
- [x] Session investigation
- [x] Process ownership investigation
- [x] Remote/RDP investigation
- [x] Local account investigation
- [x] Timeline engine
- [ ] Cross-module user correlation
- [ ] Activity scoring
- [ ] Anomaly detection
- [ ] Investigation summaries
- [ ] JSON reporting
- [ ] HTML reporting
- [ ] Improved RDP session correlation
- [ ] Process creation event correlation
- [ ] Investigation confidence scoring
- [ ] Enterprise endpoint support
- [ ] Automated incident timelines

---

# Project Structure

```text
WhoTouchedMyPC/
│
├── WhoTouchedMyPC.ps1
│
├── Modules/
│   ├── LogonHunt.ps1
│   ├── SessionHunt.ps1
│   ├── ProcessHunt.ps1
│   ├── RemoteHunt.ps1
│   ├── AccountHunt.ps1
│   └── TimelineEngine.ps1
│
├── Reports/
│
├── Tests/
│
└── Examples/
```

---

# Requirements

- Windows
- PowerShell 5.1+
- Appropriate permissions for Windows event log and system telemetry access

Some Windows event sources may require elevated PowerShell privileges or specific audit policies to be enabled.

---

# Security Philosophy

NoTouchyTouchy is designed for **defensive security, incident investigation, endpoint visibility, and security research**.

The project does not intentionally:

- Dump credentials
- Extract password hashes
- Create persistence
- Disable security controls
- Modify system configuration
- Exploit vulnerabilities
- Evade security software

The objective is visibility, correlation, and investigation.

---

# Status

**Version:** `0.1.0`

**Status:** Active Development

NoTouchyTouchy is currently being developed as part of **BLACKBOX**, a modular Windows security engineering toolkit.

---

# BLACKBOX

NoTouchyTouchy is one component of the larger BLACKBOX security engineering project.

Other planned BLACKBOX capabilities include:

- GhostHunt
- ProcessDNA
- IdentityShadow
- AttackPath
- BlastRadius
- BehaviorShift
- CanaryMesh
- PowerShellSentinel
- SecurityDrift
- EnterpriseSecurityGraph

The larger goal is to build practical security engineering tools that move beyond simple enumeration toward **behavioral detection, correlation, and investigation**.

---

## Author

**Erika Mailow**

Security Engineering • IT Infrastructure • AI/LLM

GitHub: `@erikakmailow`

---

## License

MIT License
