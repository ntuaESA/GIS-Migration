# SOP: Deploying and Managing VMs on Azure Local

## Purpose

This Standard Operating Procedure documents how to create, configure, and manage virtual machines on the NTUA Azure Local (Dell APEX) cluster using Windows Admin Center (WAC). The goal is to enable Lee Begaye and the NTUA GIS team to independently deploy and manage VMs without requiring RTH involvement for routine operations.

## Audience

- **Primary:** Lee Begaye, Sr. Systems Architect, AMI/GIS
- **Secondary:** NTUA GIS team members who may assist with VM management

## Prerequisites

### Environment values you'll need

All values below come from the as-built Azure Local cluster configuration. The
canonical source is the security-project repo —
[azure-local-cluster-config.md](../../NTUA-Security-Project/01-Network-Segmentation/docs/infrastructure/azure-local-cluster-config.md)
— and an overview is in
[discovery/azure-local-environment.md](../discovery/azure-local-environment.md).

| Setting | Value |
|---|---|
| Azure Local cluster | `NTUA-OPS-HCI1-C` (3-node Dell APEX) |
| Azure subscription | `d6520ce9-5566-4091-920b-4348d4e708b4` |
| Cluster resource group | `rg-hci-fd` |
| GIS VM resource group | `rg-AzureLocal-GIS_Production` |
| Domain | `ntua-ops.local` |
| Domain controllers / DNS | `OPS-DC1`, `OPS-DC2` (`10.23.61.101`, `10.23.61.102`) |
| WAC URL | `https://OPS-AMDHOST.ntua-ops.local` (witness host) |
| Guest VM VLAN/subnet | `10.23.62.0/?` — **VLAN ID + mask TBD with Lee** (new GIS VM IPs `10.23.62.130–134`) |
| Storage path / volume | **TODO — confirm with Lee** |

### Access requirements

- Domain account in `ntua-ops.local` with Domain Admin (Lee, David) or membership in the
  HCI management group (`OPS-ACPManager`)
- Network reachability from your workstation to `OPS-AMDHOST.ntua-ops.local` on TCP 443
- Azure Portal access to subscription `d6520ce9-...` (for visibility — VM creation goes
  through WAC)

### Naming convention for new GIS VMs

Follow the existing pattern in `rg-AzureLocal-GIS_Production`: lowercase `ops-gis*`.
Replaces the legacy DEUCES ArcGIS Enterprise stack in `ntua.local`.

**Two tiers** — the internal application tier on the existing server VLAN, and a DMZ
web-adapter tier on a separate subnet behind Lee's Palo Alto NGFW:

#### Internal tier (4 VMs on `10.23.62.130–134`)

| New VM | IP | Replaces (legacy) | Legacy IP |
|---|---|---|---|
| `ops-GisPorta12` | `10.23.62.130` | `ops-deucepor01.ntua.local` (+ ?) | `10.23.62.130` |
| `ops-GisDS12`    | `10.23.62.134` | `ops-deucesds01.ntua.local` (+ ?) | `10.23.62.134` |
| `ops-Server12`   | `10.23.62.132` | TBD with Lee                      | `10.23.62.132` |
| `ops-gisapp?`    | `10.23.62.133` | TBD with Lee                      | `10.23.62.133` |
| _spare_          | `10.23.62.131` | held in reserve for cutover overlap | n/a |

#### DMZ tier (Web Adapter VMs — IPs TBD)

Web adapters should sit in a DMZ, on a **separate IP range from the internal tier**,
with traffic between the DMZ and the internal tier segmented by Lee's Palo Alto NGFW.
Follows the standard [ESRI base ArcGIS Enterprise reference architecture](https://doc.esri.com/en/arcgis-enterprise/latest/plan/base-arcgis-enterprise-deployment.html?pivots=os-windows).

| New VM | IP | Role | Replaces (legacy) |
|---|---|---|---|
| `ops-GisWeb12` _(proposed)_  | **TBD** | Web Adapter → ArcGIS Portal (forwards on TCP/7443) | TBD with Lee |
| `ops-DeucesWeb12` _(proposed)_ | **TBD** | Web Adapter → ArcGIS Server (forwards on TCP/6443) | TBD with Lee |

> **Open questions for next-week sync with Lee:**
> - Does the Palo Alto already have a DMZ zone configured (subnet, VLAN, rules) we can
>   reuse, or do we need to design one? If new, pick a non-overlapping IP range
>   (something distinct from `10.23.61.0/24` and `10.23.62.0/24`).
> - Confirm DMZ web adapter hostnames — the proposed names above keep Lee's `12` suffix
>   and the `Gis` / `Deuces` split visible in Celeste's diagram.
> - Confirm the 4th internal VM (`.133`): is it a second ArcGIS Server node, an
>   identity/STS host, a license manager, or something else?
> - The `12` suffix on `ops-GisPorta12` / `ops-GisDS12` / `ops-Server12` — confirm
>   meaning (ArcGIS Enterprise 12?) so it's documented.

> **Decisions needed from Lee:** Confirm or adjust the naming scheme and the
> legacy-to-new mapping before any VM is created — renaming after domain-join is painful.

## Procedure (Windows Admin Center)

### Creating a New VM

> TODO — Step-by-step with screenshots:
> - Navigate to cluster → Virtual Machines
> - New VM wizard (name, generation, memory, vCPU, network, disk)
> - Naming convention for NTUA GIS VMs
> - Recommended defaults (Gen 2, dynamic memory, etc.)

### Configuring VM Settings

> TODO — Document:
> - Adjusting vCPU and memory after creation
> - Adding/removing virtual network adapters
> - Configuring boot order
> - Integration services settings

### Managing VM Lifecycle (Start / Stop / Checkpoint)

> TODO — Document:
> - Starting and stopping VMs gracefully
> - Creating and reverting checkpoints (snapshots)
> - When to use checkpoints vs. backups
> - Live migration between cluster nodes

### Storage Management

> TODO — Document:
> - Attaching additional virtual disks
> - Expanding existing disks
> - Storage pool / volume overview
> - Shared storage considerations for ESRI

### Networking

> TODO — Document:
> - VLAN assignment
> - Static IP configuration inside the VM
> - DNS registration in ntua-ops.local
> - Firewall rules relevant to ESRI

## Appendix A: Azure Portal Reference

> TODO — Document portal-based visibility for the same operations. This is supplementary — WAC is the primary management tool. Include:
> - Navigating to Azure Local resources in the portal
> - Viewing VM status and metrics
> - When the portal is useful vs. when WAC is required

## Appendix B: az CLI Reference

> TODO — Document equivalent CLI commands for automation scenarios:
> - `az stack-hci vm create` basics
> - Common management commands
> - When CLI is preferable to WAC (scripting, bulk operations)

## Troubleshooting

> TODO — Document common issues:
> - VM won't start (resource contention, storage errors)
> - VM lost network connectivity
> - WAC connectivity issues
> - Checkpoint/snapshot management problems
