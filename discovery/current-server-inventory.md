# Current GIS Server Inventory

## Purpose

Document the existing GIS server environment before migration. This inventory informs VM sizing, network configuration, and ESRI migration planning for the new Azure Local environment.

## Known Information

From prior engagements and the ntua-coop-pentest report:

- **GIS2APP1** — GIS application server (confirmed in pentest inventory)
- **GIS6APP1** — GIS application server (confirmed in pentest inventory)
- OS is believed to be **Windows Server 2012 R2** (end of life — a key driver for this migration)
- Servers are currently joined to `ntua.local` domain
- Persistent **ESRI communication failures** on these servers, suspected to be caused by GPO and/or Cisco security controls applied in `ntua.local`
- Total server count may be more than 2 — **need to confirm full list with Lee**

## Server Details

| Hostname | OS | vCPU | RAM | Disk | IP Address | Role | Notes |
|---|---|---|---|---|---|---|---|
| GIS2APP1 | Win Server 2012 R2 (?) | TODO | TODO | TODO | TODO | GIS App Server | Confirmed in pentest report |
| GIS6APP1 | Win Server 2012 R2 (?) | TODO | TODO | TODO | TODO | GIS App Server | Confirmed in pentest report |
| TODO | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee |
| TODO | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee |

## Action Items

- [ ] Confirm full server list with Lee (we know 2, need specs for all 4)
- [ ] Verify OS versions on each server
- [ ] Document current ESRI software versions and licensing
- [ ] Capture current IP addresses, VLAN assignments, and DNS entries
- [ ] Document installed roles/features on each server
- [ ] Identify data volumes and storage utilization
- [ ] Screenshot or export current ESRI configuration
