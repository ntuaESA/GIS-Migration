# As-Built Documentation — NTUA GIS Server Environment

## Environment Overview

> TODO — High-level summary of the final deployed environment: Azure Local cluster details, VM count, domain, purpose.

## Why ntua-ops.local (Not ntua.local)

The new GIS servers are joined to `ntua-ops.local` rather than the enterprise `ntua.local` domain. This is intentional for three reasons:

1. **Administrative control** — Lee Begaye does not have Domain Admin rights in `ntua.local`. Using `ntua-ops.local` gives Lee and the GIS team full administrative control over their servers and GPOs without depending on the enterprise AD team.
2. **Isolation from problematic policies** — The legacy GIS servers in `ntua.local` experienced persistent ESRI communication failures. These are suspected to be caused by Group Policy Objects and/or Cisco security controls applied in that domain. By using a separate domain, the GIS servers are not subject to those policies.
3. **Clean baseline** — Starting in a clean domain allows us to confirm ESRI works correctly first, then layer security baselines incrementally with clear cause-and-effect visibility.

## Server Specifications

| Hostname | OS | vCPU | RAM | Disk | IP Address | Domain | Role |
|---|---|---|---|---|---|---|---|
| TODO | Windows Server 2022 | TODO | TODO | TODO | TODO | ntua-ops.local | TODO |
| TODO | Windows Server 2022 | TODO | TODO | TODO | TODO | ntua-ops.local | TODO |
| TODO | Windows Server 2022 | TODO | TODO | TODO | TODO | ntua-ops.local | TODO |
| TODO | Windows Server 2022 | TODO | TODO | TODO | TODO | ntua-ops.local | TODO |

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
