# Wazuh Security Monitoring & Threat Detection Lab

A hands-on security monitoring lab built with **Wazuh 4.11.2**, Windows Security Event Logs, Sysmon, PowerShell Script Block Logging, custom detection rules, and controlled security simulations.

The project demonstrates endpoint monitoring, security event collection, threat detection, Windows activity investigation, and event correlation in an isolated lab environment.

## Project Objectives

- Deploy and validate a Wazuh security monitoring environment.
- Monitor Windows and Linux endpoints.
- Collect and analyze Windows Security and Application events.
- Integrate Sysmon telemetry for enhanced process monitoring.
- Monitor PowerShell activity through Script Block Logging.
- Develop custom Wazuh detection rules.
- Detect account management and Windows service creation activity.
- Investigate network reconnaissance through Windows Filtering Platform events.
- Validate event correlation using custom rules.
- Practice threat hunting using the Wazuh dashboard.

## Lab Architecture

### Core Components

| Component | Role |
|---|---|
| Wazuh Manager | Security event analysis and rule processing |
| Wazuh Indexer | Security event storage and indexing |
| Wazuh Dashboard | Security monitoring and threat hunting |
| SEC01 | Ubuntu-based Wazuh server |
| DC01 | Windows Server / Active Directory endpoint |
| WIN-TARGET01 | Windows endpoint used for controlled security simulations |
| Kali Linux | Controlled security testing and network reconnaissance |

### Network Environment

The Windows security testing environment uses an isolated VMware network.

| Host | IP Address | Purpose |
|---|---|---|
| Kali Linux | 192.168.250.20 | Controlled testing |
| WIN-TARGET01 | 192.168.250.30 | Windows target |

**Security boundary:** The SECURITYLAB network (`192.168.250.0/24`) is isolated from the Internet and used for controlled lab activities.

## Detection Engineering

Custom Wazuh rules were developed to identify specific Windows and Linux behaviors.

### Custom Detection Rules

| Rule ID | Description | Level |
|---|---|---:|
| 100100 | Suspicious service control: sc.exe executed from PowerShell | 10 |
| 100103 | Suspicious Windows service configured to execute cmd.exe | Not confirmed in available evidence |
| 100200 | Controlled Sysmon process creation test | 3 |
| 100201 | Windows whoami process execution detected | 3 |
| 100202 | Windows system information discovery detected | 5 |
| 100203 | PowerShell encoded command parameter detected | Not confirmed in available evidence |
| 100400 | Synthetic lab correlation test event | 3 |
| 100401 | Correlation of three matching events confirmed | 10 |

The custom rules demonstrate detection engineering for process execution, service control, system discovery, encoded PowerShell parameters, and event correlation.

### Event Correlation

A custom correlation rule was implemented to identify repeated synthetic events within a defined time window.

**Rule 100400**
- Matches the `LAB_CORRELATION_TEST` string.
- Generates a level 3 event.

**Rule 100401**
- Uses `if_matched_sid` to reference rule 100400.
- Requires three matching events within 60 seconds.
- Generates a level 10 alert.

The correlation was validated through:

1. Synthetic event generation using Linux `logger`.
2. Rule testing with `wazuh-logtest`.
3. Event processing by the Wazuh Manager.
4. Alert generation in `alerts.json`.
5. Verification in the Wazuh Dashboard.

This demonstrates an end-to-end custom correlation workflow.

## Windows Security Monitoring

### 1. PowerShell Script Block Logging

PowerShell activity was monitored through:

`Microsoft-Windows-PowerShell/Operational`

Relevant event:

- Event ID: `4104`

Tested commands included:

```powershell
Get-ChildItem C:\Windows
```

```powershell
Get-ChildItem C:\Windows -Recurse
```

Relevant native Wazuh rules:

| Rule ID | Description | Level |
|---|---|---:|
| 91819 | PowerShell script searching filesystem | 4 |
| 91820 | PowerShell script recursively collected files from a filesystem search | 4 |

The recursive search rule includes MITRE ATT&CK mappings:

- T1083 — File and Directory Discovery
- T1119 — Automated Collection

### 2. Process Creation Monitoring

Windows process creation was monitored through:

- Event ID: `4688`
- Native rule description: `A process was created.`

The collected telemetry includes process names, process IDs, account information, and token elevation details.

A historical dashboard query returned 569 matching events in a 24-hour window.

This provides visibility into process execution and supports investigation of suspicious parent-child process relationships.

### 3. Windows Account Management

Relevant event:

- Event ID: `4720`

A controlled account creation test was performed using:

- Account created: `RedTeam-Test`
- Initiating account: `Labadmin`
- Endpoint: `WIN-TARGET01`

The event was collected and classified by Wazuh.

MITRE ATT&CK reference:

- T1136 — Create Account

The available evidence does not establish that the test account was granted administrative privileges.

### 4. Windows Service Creation

Relevant event:

- Event ID: `7045`
- Native Wazuh rule: `61138`
- Description: `New Windows Service Created`

Controlled test services included:

- `RedTeam-TestSvc`
- `Suspicious-TestSvc`
- `WazuhRuleTest_DO_NOT_EXIST`
- `WazuhRuleTest_100103`
- `Detection-TestSvc`

Two documented service configurations used:

- `cmd.exe` as the executable.
- Automatic startup.
- `LocalSystem` as the service account.

Custom detection rule 100103 was also validated against a service configured to execute `cmd.exe`.

MITRE ATT&CK reference:

- T1543.003 — Windows Service

These were controlled detection tests, not evidence of an actual compromise.

### 5. Authentication Monitoring

Relevant Windows Security events:

| Event ID | Activity | Wazuh Rule |
|---|---|---|
| 4624 | Successful logon | 60118 |
| 4625 | Failed logon | 60122 |
| 4634 | Logoff | Observed in dashboard |
| 4672 | Special privileges assigned to a new logon | Observed in dashboard |

The lab collected both successful and failed authentication events.

An isolated failed logon was observed with the description:

`Logon Failure - Unknown user or bad password`

A single authentication failure does not establish brute-force activity.

## Sysmon Monitoring and Threat Hunting

Sysmon telemetry was collected from:

`Microsoft-Windows-Sysmon/Operational`

A historical query returned 31 matching events during the selected period.

Observed activity included:

- Process creation tests.
- `whoami` execution.
- Windows system information discovery.
- `net.exe` account discovery commands.
- `net group` discovery activity.
- PowerShell process execution involving an encoded command.
- Application Compatibility Database activity.

Relevant detection rules included:

| Rule ID | Description |
|---|---|
| 100200 | Controlled Sysmon process creation test |
| 100201 | Windows whoami process execution detected |
| 100202 | Windows system information discovery detected |
| 100203 | PowerShell encoded command parameter detected |
| 92039 | A net.exe account discovery command was initiated |
| 92031 | Discovery activity executed |

The Sysmon integration provides additional context for investigating process execution and system discovery.

Encoded PowerShell parameters are treated as investigation indicators, not proof of malicious execution.

## Network Reconnaissance and Windows Firewall Telemetry

A controlled network reconnaissance test was performed from Kali Linux against the Windows target.

Command:

```bash
sudo nmap -sS -sV 192.168.250.30
```

Observed Nmap output:

- Target host was reachable.
- 1,000 scanned TCP ports were reported as filtered.
- No responses were received for those filtered ports.

Windows Filtering Platform telemetry was then investigated through Wazuh.

Relevant Windows Security events:

| Event ID | Description |
|---|---|
| 5152 | Windows Filtering Platform blocked a packet |
| 5157 | Windows Filtering Platform blocked a connection |

The collected events included traffic originating from:

`192.168.250.20`

and targeting:

`192.168.250.30`

The dashboard showed multiple matching events for the source address.

MITRE ATT&CK reference:

- T1046 — Network Service Discovery

**Important:** The evidence confirms that Windows Filtering Platform logged blocked network activity and that Wazuh collected those events. It does not independently prove that Wazuh identified Nmap or that every scanned port generated an individual event.

## Dashboard and Security Overview

The Wazuh Dashboard was used for:

- Threat Hunting.
- Agent monitoring.
- Security event investigation.
- Rule-based filtering.
- MITRE ATT&CK visibility.
- Authentication monitoring.
- Process and service investigation.
- Event correlation validation.

A historical dashboard snapshot from November 17, 2026 showed:

| Metric | Value |
|---|---:|
| Active agents | 2 |
| Disconnected agents | 0 |
| Total alerts | 8,586 |
| Critical alerts | 0 |
| High severity alerts | 1 |
| Medium severity alerts | 94 |
| Low severity alerts | 8,490 |
| Authentication failures | 32 |
| Authentication successes | 3,271 |

These are historical dashboard counters, not unique confirmed security incidents.

## Validation and Evidence

The project was validated through:

- Wazuh agent event collection.
- Native Windows event monitoring.
- PowerShell Script Block Logging.
- Sysmon event collection.
- Custom rule execution.
- Custom correlation testing.
- Windows account creation monitoring.
- Windows service creation monitoring.
- Process creation investigation.
- Windows Filtering Platform event collection.
- Threat Hunting dashboard queries.

Screenshots and selected event details document the validation process.

## Security Considerations

All simulations were performed in a controlled lab environment.

- The SECURITYLAB network is isolated from the Internet.
- Test activities are restricted to authorized lab endpoints.
- Synthetic events are used to validate detection logic.
- Test accounts and services are not treated as real compromises.
- Sensitive credentials, private keys, and authentication material should never be committed to the repository.

## Skills Demonstrated

- SIEM deployment and administration.
- Wazuh Manager and agent management.
- Windows Event Log analysis.
- Sysmon integration.
- PowerShell security monitoring.
- Custom detection engineering.
- Event correlation.
- Threat hunting.
- Windows authentication investigation.
- Service and process monitoring.
- Network security monitoring.
- MITRE ATT&CK mapping.
- Security event validation.

## Project Status

The lab has demonstrated event collection, native and custom rule detection, and controlled correlation testing.

Further improvements can include:

- Expanding custom detection coverage.
- Documenting rule XML and detection logic.
- Adding additional endpoint telemetry.
- Developing investigation playbooks.
- Integrating additional threat detection use cases.

---

**Project:** Wazuh Security Monitoring & Threat Detection Lab  
**Platform:** VMware  
**SIEM:** Wazuh 4.11.2  
**Testing Environment:** Isolated Windows and Linux lab
