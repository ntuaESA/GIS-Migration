# NTUA Azure Local Environment — Overview for Lee

> **Audience:** Lee Begaye. This page is the "what am I looking at?" map of the Azure
> Local environment your new GIS VMs will run on. Read this first; everything else in this
> repo assumes you understand what's on this page.

## TL;DR

| Question | Answer |
|---|---|
| What is the cluster? | `NTUA-OPS-HCI1-C` — a 3-node Dell APEX Azure Local cluster |
| Who owns it? | NTUA Operations (you, via the `ntua-ops.local` domain) |
| Where is it in Azure? | Subscription `d6520ce9-5566-4091-920b-4348d4e708b4`, tenant `ntuaops.onmicrosoft.com`, region South Central US |
| What domain will the new GIS VMs join? | `ntua-ops.local` (NOT `ntua.local`) |
| What tool will we use to create the VMs? | Windows Admin Center (WAC) — primary. Azure Portal and `az` CLI for read-only / advanced use. |

> **Verified live on 2026-06-05.** Two values in the NTUA-Security-Project as-built
> are stale and have been corrected above: the subscription ID was `...d4c708b4`
> (should be `...d4e708b4`) and the cluster resource group was `rg-hci-id` (should be
> `rg-hci-fd`). Full live capture in
> [azure-live-snapshot-2026-06-05.md](azure-live-snapshot-2026-06-05.md).

> **Why `ntua-ops.local` and not `ntua.local`?** See the "Why this domain" section in
> [docs/as-built.md](../docs/as-built.md#why-ntua-opslocal-not-ntualocal). Short version:
> you are admin in `ntua-ops.local`, and the legacy ESRI comm failures are believed to be
> caused by GPO/Cisco controls in `ntua.local`.

## Cluster facts (from the as-built)

The full Azure Local cluster as-built lives in the security project repo. Treat that as
the single source of truth and update it there if anything changes:

> [NTUA-Security-Project / azure-local-cluster-config.md](../../NTUA-Security-Project/01-Network-Segmentation/docs/infrastructure/azure-local-cluster-config.md)

Key facts copied here for convenience:

| Item | Value |
|---|---|
| Cluster name | `NTUA-OPS-HCI1-C` |
| Platform | Dell APEX Azure Local (3-node) |
| Azure region | South Central US |
| Azure tenant | `19154b1a-aa93-4c98-aa77-e950e4f0d817` (`ntuaops.onmicrosoft.com`) |
| Subscription name | `Azure subscription 1` (the only sub in the NTUA tenant) |
| Subscription ID | `d6520ce9-5566-4091-920b-4348d4e708b4` |
| Cluster resource group | `rg-hci-fd` |
| Key Vault | `kv-hci-c` |
| Domain | `ntua-ops.local` |
| HCI host OU | `OU=HCIHosts,OU=Servers,DC=ntua-ops,DC=local` |
| Domain controllers | `OPS-DC1`, `OPS-DC2` |
| DNS servers | `10.23.61.101`, `10.23.61.102` |
| WAC / witness host | `OPS-AMDHOST.ntua-ops.local` (`10.23.61.5`) |
| ACP Manager VM | `NTUA-OPS-ACM-1` (`192.168.10.80`) |

### HCI nodes

| Node | OS mgmt IP (VLAN 10) | iDRAC IP (VLAN 241) |
|---|---|---|
| NTUA-OPS-HCI1   | 192.168.10.70 | 192.168.241.70 |
| NTUA-OPS-HCI1-2 | 192.168.10.71 | 192.168.241.71 |
| NTUA-OPS-HCI1-3 | 192.168.10.72 | 192.168.241.72 |

### Networks the cluster uses

| Purpose | VLAN | Subnet | Notes |
|---|---|---|---|
| iDRAC (out-of-band) | 241 | 192.168.241.0/24 | Hardware management only |
| OS management | 10 | 192.168.10.0/24 | Host OS + WAC |
| Storage 1 | 711 | 10.71.1.0/24 | East-west between nodes |
| Storage 2 | 712 | 10.71.2.0/24 | East-west between nodes |
| Guest VM — internal server tier | TODO — confirm | `10.23.62.0/?` | New ArcGIS internal tier (Portal/Server/Data Store/+1) uses `10.23.62.130–134` |
| Guest VM — DMZ (web adapters) | **TBD** | **TBD** | Web Adapter VMs sit in a DMZ segmented by Lee's **Palo Alto NGFW**. Need to confirm next week whether the DMZ already exists in his Palo Alto config. |

## GIS VMs already in this environment

The security project already enumerated GIS VMs in resource group
`rg-AzureLocal-GIS_Production`:

`ops-gisweb01`, `ops-gisfmlicprod`, `ops-gisdb-prd`, `ops-gisdb`,
`ops-gis106ds`, `ops-gis106svr`, `ops-gis106web`, `ops-gis-por1`

> **Important note from the security project:** These VMs were migrated from VMware using
> VMware Migrate and are registered directly with Azure Arc — NOT through the Arc Resource
> Bridge. That means they don't show up in the Azure Local management plane and can't be
> managed with `az stack-hci-vm`. For *this* migration, we will create the 4 new GIS VMs
> the right way (via WAC → Azure Local) so they appear in the management plane from day
> one. See [azure-local-guest-management-status.md](../../NTUA-Security-Project/azure-local-guest-management-status.md)
> for full context.

## Two "Arc-ness" tiers — important to understand

Azure shows VMs in two different shapes, and they look the same until you try to manage
them:

| Tier | Resource type | Created how | Manageable from Azure Local plane? |
|---|---|---|---|
| **Arc-enabled VM (server)** | `Microsoft.HybridCompute/machines` | Arc agent installed on any VM (on-prem, cloud, VMware) | No |
| **Azure Local VM** | `Microsoft.HybridCompute/machines` **+** `Microsoft.AzureStackHCI/virtualMachineInstances/default` | Created via WAC → Azure Local cluster (or Arc Resource Bridge) | Yes |

We want **Azure Local VMs** for the new GIS servers. The SOP in
[docs/sop-vm-deployment.md](../docs/sop-vm-deployment.md) walks through exactly that flow.

## How to get hands on this environment

1. **WAC (primary tool)** — Connect to `https://OPS-AMDHOST.ntua-ops.local` from a
   machine joined to `ntua-ops.local`, signed in as a domain admin.
2. **Azure Portal (read-only-ish)** — Sign in at <https://portal.azure.com>, switch to
   subscription `d6520ce9-...`, browse to resource group `rg-AzureLocal-GIS_Production`.
3. **`az` CLI (advanced / scripting)** — Follow [az-cli-setup.md](az-cli-setup.md) to log
   in via device code and select the NTUA subscription.

## Open items for Lee to confirm

- [ ] VLAN ID and subnet mask for the `10.23.62.0/?` server network (IPs `10.23.62.130–134` already allocated)
- [ ] **Whether a DMZ for the web adapter tier already exists on Lee's Palo Alto NGFW**, or whether we need to design one (subnet, VLAN, firewall rules between DMZ → internal tier)
- [ ] Storage volume / pool to use for the new VM disks
- [ ] Final role / hostname for the 4th internal-tier VM at `10.23.62.133`
- [ ] Disposition of `GIS2APP1` / `GIS6APP1` legacy servers (separate from the DEUCES stack)
