# Microsoft Sentinel SOC Lab Setup

## Incident / Lab Summary
This project documents the deployment of a Microsoft Sentinel cloud SIEM on Microsoft Azure, built as the foundation for hands-on SOC detection lab. Day 1 covers the full infrastructure build subscription, resource group, Log Analytics workspace, and Microsoft Sentinel followed by capturing a clean pre-ingestion baseline of the environment before any log sources are connected.

The goal of Day 1 is not detection yet. It is to stand up a working, query-ready SIEM and to record exactly what the environment looks like while it is empty, so that any future telemetry, alerts, and anomalies can be measured against a known-good starting point.

## Executive Summary
A Microsoft Sentinel workspace was provisioned on Azure using a free subscription. All resources were organised inside a single resource group for clean management and teardown. A Log Analytics workspace was deployed as the underlying data platform, and Microsoft Sentinel was layered on top to provide SIEM and SOAR capability. Before connecting any data connectors, the workspace was validated as empty and the baseline state was recorded using the Logs (KQL) editor. The environment is now ready for data source onboarding in Day 2.

## Architecture
\`\`\`
Azure Subscription
│
└── Resource Group: rg-soc-lab
    └── Log Analytics Workspace: law-soc-lab
        └── Microsoft Sentinel (SIEM / SOAR layer)
\`\`\`

- **Azure Subscription** the billing and management boundary. Without an active subscription, no resources can be created.
- **Resource Group (`rg-soc-lab`)** a logical container that holds every resource in this lab. Deleting the group removes everything inside it in a single action, which keeps the environment clean and prevents stray costs.
- **Log Analytics Workspace (`law-soc-lab`)** the data platform. This is where all logs are stored, indexed, and made queryable with KQL (Kusto Query Language). Sentinel does not store data itself; it reads from this workspace.
- **Microsoft Sentinel** the SIEM/SOAR layer that sits on top of the Log Analytics workspace. It provides detection rules, incidents, threat hunting, and investigation tooling. Sentinel is free to enable; cost is based only on the volume of data ingested into the workspace.

## Environment Details
| Component | Name | Region | Notes |
|-----------|------|--------|-------|
| Resource Group | rg-soc-lab | East US | Lab container for all resources |
| Log Analytics Workspace | law-soc-lab | East US | Data store, queried via KQL |
| Microsoft Sentinel | Attached to law-soc-lab | East US | SIEM/SOAR layer, 31-day free trial active |
| Subscription | Azure subscription 1 | — | Free tier, \$200 credit / 30 days |

## Build Methodology

### Step 1 Provision the Azure Subscription
A Microsoft Azure free account was created, providing a \$200 credit valid for 30 days plus a set of always-free services. The free-trial flow, including identity verification, is the step that actually provisions the subscription. Without a completed subscription, the directory exists but no resources can be deployed.

**SOC Observation:** A common Day 1 failure is an account with a directory but no subscription, shown by a "No subscriptions in this directory" message. The fix is to complete the full free-trial signup so the subscription provisions and attaches.

### Step 2 Create the Resource Group
A resource group named `rg-soc-lab` was created to act as the single container for all lab resources.

![Resource Group created](screenshots/day01-resource-group.png)

**SOC Observation:** Consistent naming (`rg-` prefix) and grouping all resources together is standard operational hygiene. It makes the environment easy to audit and easy to delete cleanly.

### Step 3 — Deploy the Log Analytics Workspace
A Log Analytics workspace named `law-soc-lab` was deployed inside the resource group. This is the data lake that Sentinel reads from.

![Log Analytics Workspace deployed](screenshots/day01-log-analytics-workspace.png)

**SOC Observation:** The workspace must exist before Sentinel can be deployed, because Sentinel attaches to an existing workspace. Build order matters: data platform first, detection layer second.

### Step 4 Deploy Microsoft Sentinel
Microsoft Sentinel was deployed onto the `law-soc-lab` workspace, adding the SIEM/SOAR capability on top of the data platform.

![Microsoft Sentinel deployed on law-soc-lab](screenshots/day01-sentinel-deployed.png)

**SOC Observation:** Enabling Sentinel is free; billing is driven only by data ingestion. A 31-day free trial was activated providing up to 10 GB/day free for both Sentinel and Log Analytics, which comfortably covers this lab.

### Step 5 Capture the Pre-Ingestion Baseline
Before connecting any data sources, the workspace was confirmed empty. The Logs interface reported "All tables are currently empty," and a baseline query was run in the Logs (KQL) editor to record the starting state.

![Empty tables baseline](screenshots/day01-baseline-empty-tables.png)

Baseline query run in the Logs (KQL) editor:

\`\`\`kql
Usage
| take 10
\`\`\`

![Baseline query in Logs editor](screenshots/day01-baseline-query.png)

**SOC Observation:** You cannot identify abnormal activity without first recording what normal looks like. This empty-state snapshot is the reference point against which all future telemetry, in Day 2 onward, will be compared. Capturing a baseline before any change is a core SOC discipline used in threat hunting and detection tuning.

### Step 6 Confirm the Build
The Azure home view confirmed the environment was deployed and the free credit untouched, verifying the full Day 1 build succeeded.

![Home view showing resources and credit](screenshots/day01-home-resources.png)

## Tools Used
- Microsoft Azure (free subscription)
- Microsoft Sentinel (SIEM / SOAR)
- Azure Log Analytics (data platform)
- KQL (Kusto Query Language)

## Learning Outcomes
- Understood the relationship between Azure, Log Analytics, and Microsoft Sentinel (data platform vs detection layer).
- Provisioned cloud SIEM infrastructure from zero, in the correct build order.
- Learned why a resource group is used for clean management and teardown.
- Practised the baseline-first discipline: recording the known-good empty state before any ingestion.
- Diagnosed and resolved a real subscription-provisioning issue during setup.

## Conclusion
Day 1 delivered a fully deployed, query-ready Microsoft Sentinel SIEM on Azure with a documented pre-ingestion baseline. The environment is clean, organised in a single resource group, and ready for data source onboarding. Day 2 will connect the first data connector and validate live log ingestion against this baseline.

## Day 2 First Data Connector & Ingestion Validation

### Objective
Onboard the first telemetry source into Microsoft Sentinel and validate that log data is flowing into the workspace, measured against the Day 1 baseline.

### What I Did
- Reviewed the Sentinel data connectors gallery (8 connectors available)
- Opened the Azure Activity connector (status: Not connected)
- Connected Azure Activity using an Azure Policy assignment scoped to the subscription, streaming Activity logs into law-soc-lab
- Generated control-plane activity (resource group tag changes) to produce ingestable events
- Validated ingestion in the Logs (KQL) editor against the AzureActivity table

### Connector
| Connector | Data Type | Method | Cost | Status |
|-----------|-----------|--------|------|--------|
| Azure Activity | Subscription-level control-plane logs | Azure Policy (diagnostic settings pipeline) | Free | Connected |

### Build Steps

**1. Data connectors gallery**
Reviewed available connectors in Microsoft Sentinel for the law-soc-lab workspace.

![Data connectors gallery](screenshots/day02-data-connectors-gallery.png)

**2. Azure Activity connector page**
Opened the Azure Activity connector, confirming prerequisites (workspace read/write, subscription owner) were met.

![Azure Activity connector page](screenshots/day02-azure-activity-connector-page.png)

**3. Azure Policy assignment**
Assigned the "Configure Azure Activity logs to stream to specified Log Analytics workspace" policy, scoped to the subscription and pointing at law-soc-lab. Assignment succeeded.

![Policy assignment succeeded](screenshots/day02-policy-assignment-succeeded.png)

**4. Generated activity**
Created resource group tag events to produce Azure Activity records for ingestion.

![Activity generated via tags](screenshots/day02-activity-generated.png)

### SOC Observation
Azure Activity is connected via a diagnostic settings pipeline governed by Azure Policy. On first setup, ingestion is not instant the policy assignment, role assignment, and diagnostic setting must fully provision before any data flows, which can take from several minutes up to an hour. Querying immediately returns no results; this is expected pipeline latency, not a failure. The validation approach is to compare against the Day 1 empty baseline: once AzureActivity rows appear where the workspace was previously empty, ingestion is confirmed.

### Validation Query
```kql
AzureActivity
| take 20
```
Run in the Logs (KQL) editor with a time range covering the generated activity. The presence of AzureActivity rows where the Day 1 baseline showed an empty workspace confirms the data pipeline is live.

### Result
A verified telemetry pipeline feeding Microsoft Sentinel from the Azure subscription. The before/after against the Day 1 baseline demonstrates ingestion was proven, not assumed.

### Next
Day 3: KQL fundamentals turning ingested logs into detection queries.
