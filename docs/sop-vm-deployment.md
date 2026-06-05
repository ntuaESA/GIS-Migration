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
Proposed for this migration (replaces 4–8 legacy `ntua.local` servers):

| New VM | IP | Replaces (legacy) | Legacy IP |
|---|---|---|---|
| `ops-GisPorta12` | `10.23.62.130` | `ops-deucepor01.ntua.local` (+ ?) | `10.23.62.130` |
| `ops-GisDS12` | `10.23.62.134` | `ops-deucesds01.ntua.local` (+ ?) | `10.23.62.134` |
| `ops-Server12` | `10.23.62.132` | TBD with Lee | `10.23.62.132` |
| `ops-gisapp?` | `10.23.62.133` | TBD with Lee | `10.23.62.133` |
| _spare_ | `10.23.62.131` (?) | held in reserve for cutover overlap | n/a |

> **Open questions on the table above (for Lee):**
> - The Data Store (`ops-GisDS12`) and the spare row both showed `10.23.62.134` after the last edit — changed the spare to `.131` since `.134` is now the live Data Store. Confirm the spare IP.
> - Web Adapter VMs from Celeste's diagram (`OPS-GISWEB01` @ `.129`, `OPS-DEUCESWeb02` @ `.128`) aren't represented here. Are they: (a) colocated on the Portal/Server VMs above, (b) pre-existing servers we reuse, or (c) two more new VMs that should be added to this table?
> - The `12` suffix on `ops-GisPorta12`, `ops-GisDS12`, `ops-Server12` — confirm whether that refers to ArcGIS Enterprise 12 / a sequence number / something else, so it's documented.

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
