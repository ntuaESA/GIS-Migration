# Current GIS Server Inventory

## Scope (confirmed with Lee, 2026-06-05)

**Build 4 net-new Windows Server 2022 VMs on the NTUA-OPS-HCI1-C Azure Local cluster, joined to `ntua-ops.local`, to replace 4–8 existing legacy GIS servers.**

- New VM IP range (from Lee): **`10.23.62.130` – `10.23.62.134`** (5 IPs allocated for 4 VMs + 1 spare/reserve)
- New VM resource group: `rg-AzureLocal-GIS_Production`
- New VM domain: `ntua-ops.local`
- Legacy servers will be decommissioned after cutover

## Purpose of this document

Document the existing (legacy) GIS server environment so we have the source-of-truth
before we build replacements. This inventory drives VM sizing, network design, and the
ESRI migration plan for the new Azure Local environment.

> **Target environment** — where the new VMs will land — is documented separately in
> [azure-local-environment.md](azure-local-environment.md). Read that page first if you
> haven't already.

## Known Information (Legacy `ntua.local` servers)

From prior engagements and the ntua-coop-pentest report:

- **GIS2APP1** — GIS application server (confirmed in pentest inventory)
- **GIS6APP1** — GIS application server (confirmed in pentest inventory)
- OS is believed to be **Windows Server 2012 R2** (end of life — a key driver for this migration)
- Servers are currently joined to `ntua.local` domain
- Persistent **ESRI communication failures** on these servers, suspected to be caused by GPO and/or Cisco security controls applied in `ntua.local`
- Total legacy server count is somewhere between **4 and 8** — need Lee to confirm the full list so we know what's being decommissioned

## Legacy server details (to be replaced / decommissioned)

| Hostname | Domain | OS | vCPU | RAM | Disk | IP Address | Role | Notes |
|---|---|---|---|---|---|---|---|---|
| GIS2APP1 | ntua.local | Win Server 2012 R2 (?) | TODO | TODO | TODO | TODO | GIS App Server | Confirmed in pentest report |
| GIS6APP1 | ntua.local | Win Server 2012 R2 (?) | TODO | TODO | TODO | TODO | GIS App Server | Confirmed in pentest report |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee (4–8 total) |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee (4–8 total) |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee (4–8 total) |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee (4–8 total) |

## New VM IP assignments (proposed)

| New VM | Replaces (legacy) | IP | Notes |
|---|---|---|---|
| `ops-gisapp2` | `GIS2APP1` (+ ?) | `10.23.62.130` | Per Lee's allocated range |
| `ops-gisapp6` | `GIS6APP1` (+ ?) | `10.23.62.131` | Per Lee's allocated range |
| `ops-gisapp?` | TBD | `10.23.62.132` | Confirm role with Lee |
| `ops-gisapp?` | TBD | `10.23.62.133` | Confirm role with Lee |
| _reserved_ | _spare_ | `10.23.62.134` | Hold for future / cutover overlap |

## Already in Azure Local — for reference, NOT in migration scope

The security-project repo enumerated these GIS VMs already running in
`rg-AzureLocal-GIS_Production` on the `NTUA-OPS-HCI1-C` cluster:

`ops-gisweb01`, `ops-gisfmlicprod`, `ops-gisdb-prd`, `ops-gisdb`,
`ops-gis106ds`, `ops-gis106svr`, `ops-gis106web`, `ops-gis-por1`

These were migrated previously from VMware via VMware Migrate. They are registered
directly with Azure Arc (not through Arc Resource Bridge), so they don't appear in the
Azure Local management plane. See
[azure-local-environment.md](azure-local-environment.md#gis-vms-already-in-this-environment).

**These VMs are out of scope for this migration** — they stay where they are. The 4
new VMs will run alongside them in the same resource group.

## Action Items

- [ ] Confirm full legacy server list with Lee (we have 2, the total is 4–8)
- [ ] Verify OS versions on each legacy server
- [ ] Document current ESRI software versions and licensing (also captured in [esri-dependencies.md](esri-dependencies.md))
- [ ] Capture current IP addresses, VLAN assignments, and DNS entries for the legacy servers
- [ ] Document installed roles/features on each legacy server
- [ ] Identify data volumes and storage utilization
- [ ] Screenshot or export current ESRI configuration
- [ ] Confirm VLAN and subnet mask for the `10.23.62.0/?` network (looks like a guest VLAN distinct from the cluster's mgmt `10.23.61.0/24`)
- [ ] Run `az resource list --resource-group rg-AzureLocal-GIS_Production ...` (see [az-cli-setup.md](az-cli-setup.md)) and paste current state of existing Azure Local GIS VMs
- [ ] Finalize naming for the 4 new VMs with Lee (proposed: `ops-gisapp{2,6,?,?}`)
