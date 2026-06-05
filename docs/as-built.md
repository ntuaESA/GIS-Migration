# As-Built Documentation — NTUA GIS Server Environment

## Environment Overview

The new GIS environment runs on the existing **NTUA-OPS-HCI1-C** Azure Local cluster (3-node Dell APEX, South Central US, subscription `d6520ce9-5566-4091-920b-4348d4e708b4` in tenant `ntuaops.onmicrosoft.com`). VMs live in resource group `rg-AzureLocal-GIS_Production` and are joined to the `ntua-ops.local` domain. The cluster as-built — nodes, networks, storage, BitLocker keys — is the authoritative source and is maintained in the security-project repo:

> [NTUA-Security-Project / azure-local-cluster-config.md](../../NTUA-Security-Project/01-Network-Segmentation/docs/infrastructure/azure-local-cluster-config.md)

A project-friendly summary lives at [discovery/azure-local-environment.md](../discovery/azure-local-environment.md).

## Why ntua-ops.local (Not ntua.local)

The new GIS servers are joined to `ntua-ops.local` rather than the enterprise `ntua.local` domain. This is intentional for three reasons:

1. **Administrative control** — Lee Begaye does not have Domain Admin rights in `ntua.local`. Using `ntua-ops.local` gives Lee and the GIS team full administrative control over their servers and GPOs without depending on the enterprise AD team.
2. **Isolation from problematic policies** — The legacy GIS servers in `ntua.local` experienced persistent ESRI communication failures. These are suspected to be caused by Group Policy Objects and/or Cisco security controls applied in that domain. By using a separate domain, the GIS servers are not subject to those policies.
3. **Clean baseline** — Starting in a clean domain allows us to confirm ESRI works correctly first, then layer security baselines incrementally with clear cause-and-effect visibility.

## Server Specifications

*Filled in during Phase 2 (Build). The proposed naming scheme and IP allocations are in [sop-vm-deployment.md](sop-vm-deployment.md#naming-convention-for-new-gis-vms).*

New GIS VM IP range (allocated by Lee): **`10.23.62.130` – `10.23.62.134`** (4 VMs + 1 reserve).

| Hostname | OS | vCPU | RAM | Disk | IP Address | Domain | Role | Resource Group |
|---|---|---|---|---|---|---|---|---|
| `ops-gisapp2` | Windows Server 2022 | TODO | TODO | TODO | `10.23.62.130` | ntua-ops.local | TODO | rg-AzureLocal-GIS_Production |
| `ops-gisapp6` | Windows Server 2022 | TODO | TODO | TODO | `10.23.62.131` | ntua-ops.local | TODO | rg-AzureLocal-GIS_Production |
| TODO | Windows Server 2022 | TODO | TODO | TODO | `10.23.62.132` | ntua-ops.local | TODO | rg-AzureLocal-GIS_Production |
| TODO | Windows Server 2022 | TODO | TODO | TODO | `10.23.62.133` | ntua-ops.local | TODO | rg-AzureLocal-GIS_Production |

## Network Architecture

> TODO — Document:
> - VLAN assignments
> - Firewall rules (inbound/outbound)
> - DNS configuration (ntua-ops.local DNS servers, forwarders)
> - Routing between ntua-ops.local and ntua.local (if any)
> - External connectivity for ESRI licensing / updates

## Active Directory Configuration

> TODO — Document:
> - ntua-ops.local domain controllers
> - OU structure
> - GPO assignments
> - Trust relationships (if any) with ntua.local
> - Admin accounts and delegation

## Storage Configuration

> TODO — Document:
> - Azure Local storage pool / volume layout
> - Per-VM disk assignments
> - Shared storage for ESRI data (if applicable)
> - Backup configuration

## ESRI Application Configuration

> TODO — Document:
> - ESRI software versions installed
> - Licensing model and server
> - Application architecture (which server does what)
> - Data sources and database connections
> - Ports and protocols in use

## Security Baseline Exceptions

> TODO — Document after Phase 4:
> - Baseline GPO applied (reference ntua-coop-pentest approach)
> - Exceptions granted for ESRI functionality
> - Justification for each exception
> - Compensating controls
