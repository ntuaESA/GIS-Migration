# ESRI Dependencies & Requirements

## Purpose

Document all ESRI software dependencies, network requirements, and integration points to ensure a successful migration to the new Azure Local VMs. This information is critical for Phase 3 (ESRI Install & Migration) and Phase 4 (Security Baseline — knowing what ports/protocols to allow).

## ESRI Software Components

| Component | Version | License Type | Servers | Notes |
|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO |

> TODO — Confirm with Lee and ESRI support:
> - ArcGIS Server version
> - ArcGIS Portal (if applicable)
> - ArcGIS Data Store (if applicable)
> - Any desktop components (ArcGIS Pro, ArcMap)
> - License server / license manager details

## Network Requirements

| Port | Protocol | Direction | Purpose | Source → Destination |
|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO |

> TODO — Document:
> - ESRI inter-server communication ports
> - Client-to-server ports
> - External endpoints for licensing, updates, basemaps
> - Database connectivity ports
> - Any ports that were blocked by the suspected Cisco/GPO controls on legacy servers

## Service Accounts Required

> TODO — Cross-reference with [service-accounts.md](../docs/service-accounts.md):
> - ArcGIS Server service account
> - Portal service account (if applicable)
> - Data Store service account (if applicable)
> - Required permissions (local admin, log on as service, etc.)

## Database Dependencies

> TODO — Document:
> - Database engine (SQL Server, PostgreSQL, etc.)
> - Database server location (local or remote)
> - Connection strings / endpoints
> - Authentication method (Windows auth, SQL auth)
> - Geodatabase configuration

## Known Issues

### Communication Failures on Legacy Servers

The legacy GIS servers in `ntua.local` experienced persistent communication failures that affected ESRI functionality. Root cause is suspected to be:

1. **Group Policy Objects** applied in the `ntua.local` domain that restrict network traffic or service behavior
2. **Cisco security controls** (firewall rules, ACLs, IPS/IDS) that may be blocking ESRI traffic patterns

This is a primary reason for joining the new servers to `ntua-ops.local` instead of `ntua.local`, and for sequencing the project so ESRI is confirmed working **before** security baselines are applied.

> TODO — During migration, document:
> - Specific error messages from legacy servers
> - Which ESRI functions were affected
> - Whether the issue is resolved on the new servers (expected: yes, since ntua-ops.local won't have the restrictive policies)

## ESRI Support Contact Info

> TODO — Document:
> - ESRI support case number (if opened)
> - ESRI technical contact assigned
> - NTUA ESRI account / customer number
> - Support tier and response SLA
