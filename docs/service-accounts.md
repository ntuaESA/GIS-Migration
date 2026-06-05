# Service Account Inventory

## Purpose

Document all service accounts used by the NTUA GIS server environment **before** applying security baselines. This ensures that GPO restrictions and password policies do not break running services, and provides a clear record of which accounts need exceptions.

## ESRI Service Accounts

| Account Name | Purpose | Permissions | Servers | Notes |
|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO |

> TODO — Coordinate with ESRI support and Lee to identify all ESRI-related service accounts, their required permissions, and logon-as-service requirements.

## Windows Service Accounts

| Account Name | Service | Permissions | Servers | Notes |
|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO |

> TODO — Document built-in and custom Windows service accounts (e.g., SQL Server, IIS app pools, backup agents).

## Scheduled Tasks

| Task Name | Account | Schedule | Server | Purpose |
|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO |

> TODO — Inventory all scheduled tasks and the accounts they run under.

## Network / Integration Accounts

| Account Name | Purpose | Systems | Notes |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

> TODO — Document accounts used for inter-system communication (database connections, API integrations, LDAP binds, etc.).

## Security Baseline Implications

> TODO — After completing the inventory above, identify:
> - Which accounts need "Log on as a service" rights
> - Which accounts need password-never-expires exceptions
> - Which accounts are impacted by Kerberos delegation policies
> - Which accounts require GPO exceptions in the security baseline
> - Recommended compensating controls for each exception
