# Live Azure Snapshot — `rg-AzureLocal-GIS_Production`

> **Captured:** 2026-06-05 via `az connectedmachine list` against the live NTUA tenant.
> Refresh this page any time by re-running the commands in
> [az-cli-setup.md](az-cli-setup.md#snapshot-of-all-gis-vms-paste-into-discovery-docs).

## Subscription / tenant (verified from live login)

| Field | Value |
|---|---|
| Subscription name | `Azure subscription 1` (NTUA's only sub) |
| Subscription ID   | `d6520ce9-5566-4091-920b-4348d4e708b4` |
| Tenant ID         | `19154b1a-aa93-4c98-aa77-e950e4f0d817` |
| Tenant default domain | `ntuaops.onmicrosoft.com` |
| Tenant display name | Navajo Tribal Utility Authority |

> **Doc discrepancy note:** The NTUA-Security-Project as-built lists the subscription
> as `d6520ce9-5566-4091-920b-4348d4`**`c708b4`**. The live API returns
> `…d4`**`e708b4`**. Treat the live value as authoritative.

## Azure Local cluster

| Field | Value |
|---|---|
| Cluster name | `NTUA-OPS-HCI1-C` |
| Resource group | `rg-hci-fd` (security-project doc says `rg-hci-id` — **live wins**) |
| Location | South Central US |

## All resource groups starting with `rg-AzureLocal-` / `rg-hci-` (live)

```
rg-hci-fd
rg-AzureLocal-AMI_Production
rg-AzureLocal-GIS_Production
rg-AzureLocal-DEUCE_Production
rg-AzureLocal-OPS_Infrastructure
rg-AzureLocal-migration
rg-AzureLocal-OSI_DEV
rg-AzureLocal-OSI_Quality
rg-AzureLocal-OPS_Security
```

## `rg-AzureLocal-GIS_Production` contents (live)

8 Arc machines + 1 virtual hard disk. All 8 machines have `kind = HCI` (i.e., Azure
Local guest VMs registered with Arc).

| Name | Type | Kind |
|---|---|---|
| `ops-gis-por1` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gis106ds` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gis106svr` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gis106web` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gisdb` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gisdb-prd` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gisfmlicprod` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-gisweb01` | `Microsoft.HybridCompute/machines` | HCI |
| `ops-107portal-OSdisk-0-77f8-seed` | `Microsoft.AzureStackHCI/virtualHardDisks` | — |

## Arc agent reporting status

All 8 Arc records have empty agent-reported properties (`osName`, `osVersion`,
`status`, `dnsFqdn`, `lastStatusChange` all `null`). Example for `ops-gisdb-prd`:

```json
{
  "name": "ops-gisdb-prd",
  "kind": "HCI",
  "osProfile": {},
  "agentConfiguration": {},
  "provisioningState": "Succeeded",
  "identity": { "type": "SystemAssigned", "principalId": "934ca440-…" },
  "licenseProfile": { "esuProfile": { "esuEligibility": "Unknown", "licenseAssignmentState": "NotAssigned" } },
  "tags": {
    "ASRv2": "AzStackHCI",
    "Application": "Infrastructure",
    "CostCenter": "IT Operations",
    "Criticality": "Medium",
    "DataClassification": "Internal",
    "Environment": "Production",
    "Owner": "leanderb"
  }
}
```

**Interpretation:** These VMs are Arc-registered shells — the Arc agent inside the guest
isn't reporting (or never was). This matches the analysis in the security-project
[azure-local-guest-management-status.md](../../NTUA-Security-Project/azure-local-guest-management-status.md):
they were registered direct-to-Arc (not via Arc Resource Bridge), so the VM Config Agent
can't be enabled and full visibility never lit up.

**Implication for this project:** When we build the 4 new GIS VMs the right way (via WAC
on the Azure Local cluster), they should appear with `osName`, `status`, etc. populated
because they'll be true Azure Local VMs with the VM Config Agent enabled. That's the
deliberate quality bar for the new servers — if the same fields come back `null` after
deployment, treat it as a Phase-2 defect.

## Tag baseline observed on existing GIS VMs

All 8 existing GIS VMs share this tag schema (carry it forward to the 4 new VMs):

| Tag | Value (existing) | Value for new VMs |
|---|---|---|
| `Owner` | `leanderb` | `leanderb` |
| `Environment` | `Production` | `Production` |
| `Application` | `Infrastructure` | TBD — likely `GIS` or `ESRI` |
| `Criticality` | `Medium` | TBD with Lee |
| `CostCenter` | `IT Operations` | `IT Operations` (or `AMI/GIS`?) |
| `DataClassification` | `Internal` | `Internal` |
| `ASRv2` | `AzStackHCI` | `AzStackHCI` |

## Open follow-ups

- [ ] Decide whether to re-onboard the existing 8 GIS VMs through Arc Resource Bridge in
  a future phase (out of current 40-hr scope; flag in `future-work/`)
- [ ] Pull live data on `OPS-DC1`, `OPS-DC2`, and the HCI nodes (different RGs) once Lee
  confirms scope for cross-checking DNS/IP records
- [ ] Confirm the `10.23.62.0/?` subnet exists in NTUA's network design (it doesn't match
  the cluster mgmt range `10.23.61.0/24` documented in the security-project)
