# Microsoft Sentinel SOC Detection Lab

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![SIEM](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-0078D4.svg)]()
[![Focus](https://img.shields.io/badge/Focus-Detection%20Engineering-blue.svg)]()
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-orange.svg)]()
[![Language](https://img.shields.io/badge/Language-KQL-success.svg)]()
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1110%20%7C%20T1059.001-red.svg)](https://attack.mitre.org/)
[![Labs](https://img.shields.io/badge/Labs-7%20Complete-brightgreen.svg)]()
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen.svg)]()

An empty Azure workspace built into a working Security Operations capability in seven stages, ending in two confirmed, documented investigations. The same two attacks a Kali box runs here, an SSH brute force and a malicious PowerShell execution, are the same ones caught host-side and on the wire in the detection-engineering-labs repo. This is those attacks seen a third way: through a cloud SIEM.

## At a Glance

| Field | Detail |
| --- | --- |
| Work Type | SIEM engineering and detection, cloud |
| SIEM | Microsoft Sentinel, Unified Defender Portal |
| Workspace | law-soc-lab, Azure East US |
| Build Arc | 7 days, empty workspace to two investigations |
| Investigations | SSH brute force (T1110), malicious PowerShell (T1059.001) |
| Telemetry | Linux Syslog, Windows PowerShell 4104, via Arc → AMA → DCR |
| Language | KQL |
| Status | Complete |

## What This Is

Building a Security Operations capability from an empty Azure workspace to two confirmed, documented investigations, covering pipeline engineering, KQL detection, threat hunting, and incident reporting across Linux and Windows.

**Platform:** Microsoft Sentinel (Unified Defender Portal) · Azure East US · Workspace `law-soc-lab`

The value here is the full arc. Anyone can write a KQL query; this lab shows the whole pipeline standing up behind it, from an Azure Arc agent on a host to a Data Collection Rule to a scheduled analytics rule that fires an incident. A detection is only as trustworthy as the ingestion feeding it, and this build proves the ingestion first.

## How This Fits the Portfolio

This lab is the cloud-SIEM view of attacks documented elsewhere in the portfolio at other layers:

- The **SSH brute force (T1110)** hunted here in Sentinel Syslog is the same technique caught host-side by Wazuh and on the wire by Suricata in the `detection-engineering-labs` repo. Three independent stacks, one adversary behavior.
- The **malicious PowerShell (T1059.001)** reconstructed here from PowerShell 4104 logs mirrors the endpoint forensics in that repo's PowerShell investigation, recovering attacker intent from script block logging.

Detecting the same techniques across Wazuh, Suricata, and Sentinel is the point. It demonstrates the skill transfers across tooling rather than being bound to one product.

## Featured Investigations

Two confirmed incidents worked end to end, from detection through containment and reporting:

**SSH Brute-Force Compromise (MITRE T1110), Linux.** A single source IP produced 88 failed and 8 successful logins against one account. Detected in Syslog, confirmed by correlating failures with successes, and contained at the host firewall.

The 8 successes are the finding. A wall of failures alone is a failed attempt; the moment a success appears in the same window from the same source, it is a compromise, and the correlation that catches that transition is what separates a hunt from a log search.

**Malicious PowerShell Execution (MITRE T1059.001), Windows.** Encoded commands, a download cradle, and an execution-policy bypass, detected via PowerShell script block logging, defeating base64 obfuscation.

Script block logging is what makes this possible. On the command line the payload is an opaque base64 blob; in the 4104 event it is plaintext. The hunt does not crack the obfuscation, the logging already recorded what ran underneath it.

**Full flagship report:** [SOC-Flagship-Investigation-Report.pdf](SOC-Flagship-Investigation-Report.pdf)

## The Build, Day by Day

Each stage builds on the last: stand up the SIEM, prove ingestion, learn to query, automate detection, then hunt two real attacks and report them.

| Day | Focus | MITRE | Folder |
| --- | --- | --- | --- |
| 1 | Workspace deployment & baseline | — | [day1-workspace-setup](day1-workspace-setup) |
| 2 | First data connector & ingestion validation | — | [day2-ingestion](day2-ingestion) |
| 3 | KQL fundamentals | — | [day3-kql](day3-kql) |
| 4 | Scheduled analytics rule & incident triage | T1565.001 | [day4-analytics-rule](day4-analytics-rule) |
| 5 | SSH brute-force threat hunt | T1110 | [day5-brute-force-hunt](day5-brute-force-hunt) |
| 6 | Malicious PowerShell endpoint hunt | T1059.001 | [day6-powershell-hunt](day6-powershell-hunt) |
| 7 | Flagship investigation report | — | [SOC-Flagship-Investigation-Report.pdf](SOC-Flagship-Investigation-Report.pdf) |

The order is deliberate and it is the argument. Days 1 through 3 are plumbing, no detections yet, because a detection built on unverified ingestion is a guess. Day 4 automates. Days 5 and 6 hunt. Day 7 writes it up the way Tier 2 would need to read it.

## The SOC Angle

This lab is the detection-and-response half of the job carried into a cloud SIEM, and the pipeline is the part most portfolios skip.

The Arc → AMA → DCR chain is not glamorous, but it is where real detection engineering lives. A KQL query is worthless if the log it queries never arrived, and the majority of failed detections in production are ingestion problems, not logic problems. Proving the pipeline before writing the hunt is the discipline this build is structured around.

The two hunts then mirror the two halves of an analyst's instinct: correlation (the brute force, joining failures to successes across a time window) and reconstruction (the PowerShell, reading obfuscated execution back to plaintext intent). Both end in a report, because an investigation nobody can hand to Tier 2 is an investigation that did not finish.

## Capabilities Demonstrated

- **SIEM pipeline engineering** Azure Arc to Azure Monitor Agent to Data Collection Rule ingestion for Linux Syslog and Windows event logs.
- **KQL / log analysis** filtering, aggregation, correlation, and timelining across multiple data sources.
- **Detection engineering** authored a scheduled analytics rule and indicator-based hunt queries.
- **Threat hunting** proactive hunts confirming compromise on both Linux and Windows.
- **Incident response** detect, confirm, contain workflow including live firewall containment.
- **Documentation** structured reporting with executive summary, IOCs, ATT&CK mapping, and recommendations.

## Environment

| Component | Detail |
| --- | --- |
| SIEM | Microsoft Sentinel (Unified Defender Portal) |
| Workspace | law-soc-lab · Azure East US |
| Linux telemetry | Ubuntu auth/authpriv Syslog via Arc → AMA → DCR |
| Windows telemetry | PowerShell Operational (4104) via Arc → AMA → DCR |
| Attacker | Kali Linux |
| Query language | KQL (Kusto Query Language) |

---

[![LinkedIn](https://img.shields.io/badge/LinkedIn-WilliamInCyber-blue?style=flat&logo=linkedin)](https://linkedin.com/in/WilliamInCyber)
[![X](https://img.shields.io/badge/X-@WilliamInCyber-black?style=flat&logo=x)](https://x.com/WilliamInCyber)
