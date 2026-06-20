```
███████╗███████╗███╗   ██╗████████╗██╗███╗   ██╗███████╗██╗     
██╔════╝██╔════╝████╗  ██║╚══██╔══╝██║████╗  ██║██╔════╝██║     
███████╗█████╗  ██╔██╗ ██║   ██║   ██║██╔██╗ ██║█████╗  ██║     
╚════██║██╔══╝  ██║╚██╗██║   ██║   ██║██║╚██╗██║██╔══╝  ██║     
███████║███████╗██║ ╚████║   ██║   ██║██║ ╚████║███████╗███████╗
╚══════╝╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚═╝╚═╝  ╚═══╝╚══════╝╚══════╝
```

[![Blue Team Notes](https://img.shields.io/badge/Blue_Team_Notes-WilliamInCyber-1F6FEB?style=flat&logo=github&logoColor=white)](https://github.com/WiLL75G)
[![Microsoft Sentinel](https://img.shields.io/badge/Microsoft-Sentinel-0078D4?style=flat&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/products/microsoft-sentinel)
[![KQL](https://img.shields.io/badge/Query-KQL-005BA1?style=flat)](https://learn.microsoft.com/azure/data-explorer/kusto/query)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-C5221F?style=flat)](https://attack.mitre.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

# Microsoft Sentinel SOC Detection Lab

> Building a Security Operations capability from an empty Azure workspace to two confirmed, documented investigations, covering pipeline engineering, KQL detection, threat hunting, and incident reporting across Linux and Windows.

**Platform:** Microsoft Sentinel (Unified Defender Portal) · Azure East US · Workspace `law-soc-lab`

*This is a quick overview. KQL detections, lab architecture, and full walkthroughs are in the [detection modules](#detections).*

---

## Featured Investigations

Two confirmed incidents worked end to end, from detection through containment and reporting:

**SSH Brute-Force Compromise (MITRE T1110) Linux.** A single source IP produced 88 failed and 8 successful logins against one account. Detected in Syslog, confirmed by correlating failures with successes, and contained at the host firewall.

**Malicious PowerShell Execution (MITRE T1059.001) Windows.** Encoded commands, a download cradle, and an execution-policy bypass detected via PowerShell script block logging defeating base64 obfuscation.

**Full flagship report:** [SOC-Flagship-Investigation-Report.pdf](SOC-Flagship-Investigation-Report.pdf)

---

## The Build, Day by Day

Each stage builds on the last: stand up the SIEM, prove ingestion, learn to query, automate detection, then hunt two real attacks and report them.

| Day | Focus | MITRE | Folder |
|-----|-------|-------|--------|
| 1 | Workspace deployment & baseline | — | [day1-workspace-setup](day1-workspace-setup) |
| 2 | First data connector & ingestion validation | — | [day2-ingestion](day2-ingestion) |
| 3 | KQL fundamentals | — | [day3-kql](day3-kql) |
| 4 | Scheduled analytics rule & incident triage | T1565.001 | [day4-analytics-rule](day4-analytics-rule) |
| 5 | SSH brute-force threat hunt | T1110 | [day5-brute-force-hunt](day5-brute-force-hunt) |
| 6 | Malicious PowerShell endpoint hunt | T1059.001 | [day6-powershell-hunt](day6-powershell-hunt) |
| 7 | Flagship investigation report | — | [SOC-Flagship-Investigation-Report.pdf](SOC-Flagship-Investigation-Report.pdf) |

---

## Capabilities Demonstrated

- **SIEM pipeline engineering** Azure Arc to Azure Monitor Agent to Data Collection Rule ingestion for Linux Syslog and Windows event logs
- **KQL / log analysis** filtering, aggregation, correlation, and timelining across multiple data sources
- **Detection engineering** authored a scheduled analytics rule and indicator-based hunt queries
- **Threat hunting** proactive hunts confirming compromise on both Linux and Windows
- **Incident response** detect, confirm, contain workflow including live firewall containment
- **Documentation** structured reporting with executive summary, IOCs, ATT&CK mapping, and recommendations

---

## Environment

| Component | Detail |
|-----------|--------|
| SIEM | Microsoft Sentinel (Unified Defender Portal) |
| Workspace | law-soc-lab · Azure East US |
| Linux telemetry | Ubuntu auth/authpriv Syslog via Arc → AMA → DCR |
| Windows telemetry | PowerShell Operational (4104) via Arc → AMA → DCR |
| Attacker | Kali Linux |
| Query language | KQL (Kusto Query Language) |
