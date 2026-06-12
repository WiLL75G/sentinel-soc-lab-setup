# Microsoft Sentinel SOC Lab Setup

## Overview
Stood up a Microsoft Sentinel workspace on Azure as the foundation for a 6-week SOC detection lab. Day 1 covers infrastructure deployment and capturing a clean baseline before any data ingestion.

## Architecture
Azure → Resource Group (rg-soc-lab) → Log Analytics Workspace (law-soc-lab) → Microsoft Sentinel

- **Log Analytics** is the data platform: stores and indexes logs, queried with KQL.
- **Microsoft Sentinel** is the SIEM/SOAR layer on top: detection, incidents, threat hunting, and investigation tooling.

## What I Did
- Provisioned an Azure free subscription
- Created a resource group (`rg-soc-lab`) as the lab container
- Deployed a Log Analytics workspace (`law-soc-lab`)
- Deployed Microsoft Sentinel onto the workspace
- Captured a pre-ingestion baseline confirming the workspace was empty

## Baseline
Before connecting any data sources, the workspace was confirmed empty ("All tables are currently empty"), and a KQL query was run in the Logs editor to record the starting state. This baseline is the reference point for identifying new telemetry from Day 2 onward.

Baseline query run in the Logs (KQL) editor:

```kql
Usage
| take 10
```

See `notes/baseline-tables.md` for details.

## Repository Structure
