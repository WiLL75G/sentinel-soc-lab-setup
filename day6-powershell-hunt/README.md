# Malicious PowerShell Execution Hunt on Windows Endpoint

The expected telemetry never showed up, and the hunt succeeded anyway. When the SecurityEvent table (process creation, 4688) refused to populate on this tenant, the whole investigation pivoted onto PowerShell Script Block Logging (4104), which turned out to be the stronger source, because it records the decoded payload of an obfuscated command that 4688 would only show as an opaque blob. This is the same JAMES-VM endpoint and the same 4104 base64 recovery documented in the detection-engineering-labs PowerShell investigation, run here through a cloud SIEM.

## At a Glance

| Field | Detail |
| --- | --- |
| Incident | Malicious PowerShell, obfuscation, download cradle, policy bypass |
| Severity | High |
| Detection | PowerShell Script Block Logging (Event ID 4104) via AMA → Sentinel |
| Host | JAMES-VM, Windows 11 |
| Pivot | SecurityEvent (4688) not collected on tenant; hunt moved to 4104 |
| Proof | Base64-encoded command recovered in decoded form |
| MITRE | T1059.001, T1105 |
| Status | Closed True Positive (simulated) |

## What This Is

A Windows 11 endpoint onboarded into Microsoft Sentinel to hunt malicious PowerShell execution. During pipeline validation a platform constraint surfaced: the SecurityEvent table (Event ID 4688, process creation) did not populate on this tenant, while the Event table carrying PowerShell Script Block Logging (Event ID 4104) ingested normally.

The hunt was pivoted entirely onto 4104 telemetry, which proved sufficient to detect every simulated technique, including a base64-encoded command, demonstrating that script block logging records the decoded payload regardless of obfuscation. Adapting to the telemetry that is actually flowing, rather than the telemetry the plan assumed, is the real skill on display.

## How This Relates to the Portfolio

This is the cloud-SIEM view of the endpoint forensics documented in the `detection-engineering-labs` PowerShell investigation. Both run on the same host, JAMES-VM, and both hinge on the same technique: recovering an obfuscated PowerShell payload in plaintext from Event ID 4104 script block logging.

The other repo reads 4104 directly on the endpoint; this one forwards it through Azure Arc into Sentinel and hunts it with KQL. Same evidence, same T1059.001 base64 recovery, two collection paths. Paired with Day 5's brute force, this completes the portfolio's demonstration of both flagship attacks, T1110 and T1059.001, across host tooling and a cloud SIEM.

## Affected System

| Attribute | Value |
| --- | --- |
| Hostname | JAMES-VM |
| Operating System | Windows 11 |
| Onboarding | Azure Arc → Azure Monitor Agent → Data Collection Rule |
| Log Source | Microsoft-Windows-PowerShell/Operational (Event ID 4104) |
| Destination | Log Analytics Workspace `law-soc-lab` |

## Investigation Methodology

### Step 1 — Baseline the Windows event tables

Both SecurityEvent and Event (4104) returned zero rows, confirming ingestion had to be built.

![SecurityEvent baseline empty](screenshots/day06-windows-baseline-securityevent.png)
![Event 4104 baseline empty](screenshots/day06-windows-baseline-event4104.png)

### Step 2 — Onboard via Azure Arc and deploy the Azure Monitor Agent

The Windows VM was onboarded to Azure Arc and a Data Collection Rule was associated, deploying the agent.

![Arc machine connected](screenshots/day06-arc-machine-connected.png)
![Arc and AMA on Windows](screenshots/day06-arc-ama-windows.png)

The DCR used custom XPath queries to collect exactly the required event IDs.

![DCR custom XPath queries](screenshots/day06-dcr-xpath-queries.png)

### Step 3 — Enable host-side logging

Audit process creation, command-line inclusion, and PowerShell script block logging were enabled on the endpoint.

![Host logging enabled](screenshots/day06-logging-enabled.png)

### Step 4 — Simulate malicious PowerShell

Three execution techniques were run on the endpoint (benign payloads, harmless target).

Encoded command (obfuscation):

![PowerShell encoded](screenshots/day06-powershell-attack-encoded.png)

Hidden download cradle:

![PowerShell cradle](screenshots/day06-powershell-attack-cradle.png)

Execution-policy bypass:

![PowerShell bypass](screenshots/day06-powershell-attack-bypass.png)

### Step 5 — Hunt the telemetry

Suspicious script block indicators:

![Command-line / indicator hunt](screenshots/day06-commandline-hunt.png)

```kql
Event
| where EventID == 4104
| where EventData has_any ("DownloadString","IEX","Invoke-Expression","FromBase64String","Net.WebClient","SOC-DAY6-ATTACK")
| project TimeGenerated, Computer, EventData
| sort by TimeGenerated asc
```

Obfuscation defeated, the base64-encoded payload recovered in decoded form:

![Decoded script block](screenshots/day06-scriptblock-decoded.png)

Execution timeline:

![Execution timeline](screenshots/day06-execution-timeline.png)

## Indicators of Compromise (IOCs)

| Type | Indicator |
| --- | --- |
| Process | powershell.exe with -EncodedCommand |
| Behaviour | IEX (New-Object Net.WebClient).DownloadString(...) download cradle |
| Flag | -WindowStyle Hidden (window-hiding evasion) |
| Flag | -ExecutionPolicy Bypass |
| Host | JAMES-VM |

## MITRE ATT&CK

| Tactic | Technique | ID |
| --- | --- | --- |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |
| Command and Control | Ingress Tool Transfer | T1105 |

## Findings

- All three simulated PowerShell execution techniques were detected via Event ID 4104.
- The base64-encoded command was recovered in decoded form, defeating obfuscation.
- A telemetry gap was identified: SecurityEvent (4688) is not collected on this tenant; Event (4104) is.
- Hidden-window execution did not evade script block logging, `-WindowStyle Hidden` hides the window from the user, not the payload from the log.

## Analyst Insight

The most valuable lesson was the adaptation. When the expected process-creation telemetry proved unavailable, the hunt pivoted to PowerShell Script Block Logging, which turned out to be the stronger source, because it records the decoded content of obfuscated commands that 4688 alone would only show as an opaque encoded blob.

Real detection work is rarely a clean run against ideal telemetry. The analyst's job is to detect with the data that is actually flowing, not to file a ticket that the perfect log source is missing. A hunt that stops when the planned table is empty is not a hunt; the pivot is the skill.

## The SOC Angle

This investigation makes a point about obfuscation that holds across the whole portfolio: base64 encoding hides a command from a human reading the command line, not from properly configured logging.

On Day 6 the encoded payload is an opaque blob in the process-creation view and plaintext in the 4104 view. The same is true in the endpoint-forensics version of this attack in the other repo. The attacker's evasion assumes nobody enabled script block logging, and the entire hunt is the demonstration that this assumption is where they lose. Enable the right log source in advance, and obfuscation becomes a speed bump rather than a wall.

## Learning Outcome

- Onboarded a Windows endpoint to Sentinel via Azure Arc → AMA → DCR.
- Configured custom XPath event collection for specific Windows Event IDs.
- Enabled audit process creation, command-line inclusion, and script block logging.
- Distinguished the gated SecurityEvent table from the ungated Event table.
- Proved script block logging defeats `-EncodedCommand` obfuscation.

## Conclusion

Day 6 established Windows PowerShell execution monitoring in Microsoft Sentinel and detected three obfuscation and evasion techniques using Event ID 4104, working around a tenant-level telemetry constraint by pivoting to script block logging. This investigation feeds the flagship report synthesizing the Linux brute-force (T1110) and Windows PowerShell (T1059.001) investigations.
