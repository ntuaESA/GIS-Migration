# Current GIS Server Inventory

## Scope (confirmed with Lee, 2026-06-05)

**Build 4 net-new Windows Server 2022 VMs on the NTUA-OPS-HCI1-C Azure Local cluster, joined to `ntua-ops.local`, to replace the legacy DEUCES ArcGIS Enterprise stack. Plus 2 web adapter VMs in a DMZ (in front of those 4).**

- **Internal tier (4 new VMs)** — Portal, Server, Data Store, +1 TBD — IPs `10.23.62.130–134` on the existing `ntua-ops.local` server VLAN (5 IPs = 4 VMs + 1 reserve)
- **DMZ tier (2 new web adapter VMs)** — IPs TBD, on a separate DMZ subnet segmented by Lee's **Palo Alto NGFW**. Need to confirm with Lee next week whether the DMZ already exists in his Palo Alto config or needs to be built.
- New VM resource group: `rg-AzureLocal-GIS_Production`
- New VM domain: `ntua-ops.local` (DMZ VMs may use a separate OU — TBD)
- Legacy servers will be decommissioned after cutover

> **Lee's commits are authoritative** for hostnames and the legacy-to-new mapping. Per
> the 2026-06-05 update, the naming convention is now **`ops2-` + ESRI role** (gen-2
> infrastructure, matches the AMI server naming family): `ops2-gisportal`,
> `ops2-gisserver`, `ops2-gisdatastore`, plus one more (TBD) for the internal tier, and
> `ops2-giswebportal` / `ops2-giswebserver` for the DMZ. The mapping table lives in
> [docs/sop-vm-deployment.md](../docs/sop-vm-deployment.md#naming-convention-for-new-gis-vms)
> — that's the single source of truth, this doc just references it.

## Purpose of this document

Document the existing (legacy) GIS server environment so we have the source-of-truth
before we build replacements. This inventory drives VM sizing, network design, and the
ESRI migration plan for the new Azure Local environment.

> **Target environment** — where the new VMs will land — is documented separately in
> [azure-local-environment.md](azure-local-environment.md). Read that page first if you
> haven't already.

## Legacy ArcGIS Enterprise stack (to be replaced / decommissioned)

Per Lee's edits to the SOP, the legacy servers being replaced are the existing DEUCES
ArcGIS Enterprise tier in `ntua.local`, not the previously-assumed `GIS2APP1` /
`GIS6APP1` pair.

- OS is believed to be **Windows Server 2012 R2 (?)** on at least some servers — confirm per host
- Currently joined to `ntua.local` domain
- Persistent **ESRI communication failures** suspected to be caused by GPO and/or Cisco security controls applied in `ntua.local`
- Confirmed legacy mappings from Lee: `ops-deucepor01.ntua.local` (Portal), `ops-deucesds01.ntua.local` (Data Store). Other roles (Server, web adapters) still to be confirmed.

| Hostname | Domain | Role | OS | vCPU | RAM | Disk | IP Address | Notes |
|---|---|---|---|---|---|---|---|---|
| `ops-deucepor01` | ntua.local | ArcGIS Portal | TODO | TODO | TODO | TODO | TODO | Replaced by `ops2-gisportal` |
| `ops-deucesds01` | ntua.local | ArcGIS Data Store | TODO | TODO | TODO | TODO | TODO | Replaced by `ops2-gisdatastore` |
| TODO | ntua.local | ArcGIS Server | TODO | TODO | TODO | TODO | TODO | Replaced by `ops2-gisserver`. Confirm hostname with Lee. |
| TODO | ntua.local | Web Adapter (Portal) | TODO | TODO | TODO | TODO | TODO | Replaced by `ops2-giswebportal` in DMZ. Confirm with Lee. |
| TODO | ntua.local | Web Adapter (Server) | TODO | TODO | TODO | TODO | TODO | Replaced by `ops2-giswebserver` in DMZ. Confirm with Lee. |
| TODO | ntua.local | TODO | TODO | TODO | TODO | TODO | TODO | Confirm with Lee (4–8 total) |

> The original assumption that `GIS2APP1` and `GIS6APP1` were the migration targets came
> from the ntua-coop-pentest report. Lee's commit `560948a` reframes the scope around the
> DEUCES stack. Those two servers may still exist and may need separate disposition, but
> they are **not** the primary subject of this migration. Confirm with Lee.

## Already in Azure Local — for reference, NOT in migration scope

The security-project repo enumerated these GIS VMs already running in
`rg-AzureLocal-GIS_Production` on the `NTUA-OPS-HCI1-C` cluster:

`ops-gisweb01`, `ops-gisfmlicprod`, `ops-gisdb-prd`, `ops-gisdb`,
`ops-gis106ds`, `ops-gis106svr`, `ops-gis106web`, `ops-gis-por1`

These were migrated previously from VMware via VMware Migrate. They are registered
directly with Azure Arc (not through Arc Resource Bridge), so they don't appear in the
Azure Local management plane. See
[azure-local-environment.md](azure-local-environment.md#gis-vms-already-in-this-environment).

**These VMs are out of scope for this migration** — they stay where they are. The new
VMs will run alongside them in the same resource group.

## Action Items

- [ ] Confirm full legacy server list with Lee (Portal + Data Store known; Server + web adapter hostnames TBD)
- [ ] Verify OS versions on each legacy server
- [ ] Document current ESRI software versions and licensing (also captured in [esri-dependencies.md](esri-dependencies.md))
- [ ] Capture current IP addresses, VLAN assignments, and DNS entries for the legacy servers
- [ ] Document installed roles/features on each legacy server
- [ ] Identify data volumes and storage utilization
- [ ] Screenshot or export current ESRI configuration
- [ ] Confirm VLAN and subnet mask for the `10.23.62.0/?` server network (`10.23.62.130–134` allocated)
- [ ] **Confirm with Lee whether a DMZ already exists on the Palo Alto NGFW for the web adapter tier**; if not, design one and pick an IP range
- [ ] Decide disposition of `GIS2APP1` / `GIS6APP1` (from the pentest report) — in scope, out of scope, or already retired?
- [ ] Finalize the 4th internal VM (`.133`) hostname and role with Lee
