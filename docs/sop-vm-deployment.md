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
| Guest VM VLAN/subnet | `10.23.62.0/?` — **VLAN ID + mask TBD with Lee** (new GIS VM IPs `10.23.62.130 / .132 / .134`) |
| Storage | Hyperconverged — Storage Spaces Direct (S2D) across the 3 Dell APEX nodes. **3 cluster shared volumes**; new VMs are placed round-robin across them. Pick the CSV with the most free space at create time. |

### Access requirements

- Domain account in `ntua-ops.local` with Domain Admin (Lee, David) or membership in the
  HCI management group (`OPS-ACPManager`)
- Network reachability from your workstation to `OPS-AMDHOST.ntua-ops.local` on TCP 443
- Azure Portal access to subscription `d6520ce9-...` (for visibility — VM creation goes
  through WAC)

### Naming convention for new GIS VMs

**Convention (per Lee, 2026-06-05):** `ops2-` prefix to match the gen-2 AMI server
naming pattern, followed by a conventional ESRI role identifier (`portal`, `server`,
`datastore`, `webportal`, `webserver`). Lowercase. The `2` in `ops2-` denotes
second-generation infrastructure; the earlier proposals using `*12` and the
DEUCES-derived names (`ops-deuce*` — "Deuce" was the original 2nd-gen working name)
are retired.

Replaces the legacy DEUCES ArcGIS Enterprise stack in `ntua.local`.

**Two tiers** — the internal application tier on the existing server VLAN, and a DMZ
web-adapter tier on a separate subnet behind Lee's Palo Alto NGFW:

#### Internal tier (3 VMs on `10.23.62.130 / .132 / .134`)

| New VM | IP | ESRI role | Replaces (legacy) | Legacy IP |
|---|---|---|---|---|
| `ops2-gisportal`    | `10.23.62.130` | ArcGIS Portal     | `ops-deucepor01.ntua.local` | `10.23.62.130` |
| `ops2-gisserver`    | `10.23.62.132` | ArcGIS Server     | TBD with Lee                | `10.23.62.132` |
| `ops2-gisdatastore` | `10.23.62.134` | ArcGIS Data Store | `ops-deucesds01.ntua.local` | `10.23.62.134` |

#### DMZ tier (Web Adapter VMs — IPs TBD)

Web adapters sit in a DMZ, on a **separate IP range from the internal tier**,
with traffic between the DMZ and the internal tier segmented by Lee's Palo Alto NGFW.
Follows the standard [ESRI base ArcGIS Enterprise reference architecture](https://doc.esri.com/en/arcgis-enterprise/latest/plan/base-arcgis-enterprise-deployment.html?pivots=os-windows).

| New VM | IP | ESRI role | Replaces (legacy) |
|---|---|---|---|
| `ops2-giswebportal` | **TBD** | Web Adapter → Portal (TCP/7443 to internal) | TBD with Lee |
| `ops2-giswebserver` | **TBD** | Web Adapter → Server (TCP/6443 to internal) | TBD with Lee |

> **Open questions for next-week sync with Lee:**
> - Does the Palo Alto already have a DMZ zone configured (subnet, VLAN, rules) we can
>   reuse, or do we need to design one? If new, pick a non-overlapping IP range
>   (something distinct from `10.23.61.0/24` and `10.23.62.0/24`).
> - Sanity-check the convention against AMI server naming. The closest pattern in the
>   security project is `ops-amiAFP1` / `ops-amiITP1` (`ops-` + 3-letter app + tier + #).
>   `ops2-` is unambiguous as gen-2, but if AMI is also moving to `ops2-`, confirm what
>   the AMI naming will be so we stay aligned.

> **Decisions needed from Lee:** Confirm or adjust the naming scheme and the
> legacy-to-new mapping before any VM is created — renaming after domain-join is painful.

## End-to-end build procedure

This is the order to build each of the new VMs. The first VM (`ops2-gisportal`)
should be done together so Lee sees each step once; the remaining VMs can be
done the same way solo.

The flow has six steps, each documented below:

1. [Create the VM shell in Windows Admin Center](#step-1--create-the-vm-shell-windows-admin-center)
2. [First boot — rename, static IP, DNS, domain join](#step-2--first-boot-rename-static-ip-dns-domain-join)
3. [Move the computer object into the GIS OU](#step-3--move-the-computer-object-into-the-gis-ou)
4. [Apply the security baseline GPO](#step-4--apply-the-security-baseline-gpo)
5. [Onboard to Azure (Arc, AMA, Update Manager, Defender)](#step-5--onboard-to-azure-arc-ama-update-manager-defender)
6. [Per-role ESRI prep (handoff)](#step-6--per-role-esri-prep-handoff)

### Per-VM build sheet (fill in before starting)

Print or copy this for each VM before clicking anything. Sizing is a starting
point from the [ESRI base deployment](https://doc.esri.com/en/arcgis-enterprise/latest/install/windows/system-requirements-for-arcgis-enterprise.htm)
— adjust if Lee has different guidance.

| VM | IP | Tier | OU (planned) | vCPU | RAM | OS disk | Data disk |
|---|---|---|---|---|---|---|---|
| `ops2-gisportal`    | `10.23.62.130` | Internal | `OU=GIS,OU=Servers,DC=ntua-ops,DC=local`         | 8  | 16 GB | 100 GB | 200 GB |
| `ops2-gisserver`    | `10.23.62.132` | Internal | `OU=GIS,OU=Servers,DC=ntua-ops,DC=local`         | 8  | 16 GB | 100 GB | 200 GB |
| `ops2-gisdatastore` | `10.23.62.134` | Internal | `OU=GIS,OU=Servers,DC=ntua-ops,DC=local`         | 8  | 32 GB | 100 GB | 500 GB |
| `ops2-giswebportal` | TBD            | DMZ      | `OU=DMZ,OU=GIS,OU=Servers,DC=ntua-ops,DC=local`  | 4  |  8 GB | 100 GB | —      |
| `ops2-giswebserver` | TBD            | DMZ      | `OU=DMZ,OU=GIS,OU=Servers,DC=ntua-ops,DC=local`  | 4  |  8 GB | 100 GB | —      |

> **Decision needed before Step 3:** the security project uses an `OU=AMI` tree
> under `OU=Servers`. For GIS, plan is a parallel `OU=GIS` (with a `OU=DMZ`
> child for the two web adapters). Confirm with Lee, and create the OUs before
> any VMs are joined so the baseline GPO inheritance lands correctly. The
> security-project script
> [`Create-AMI-OU-Structure.ps1`](../../NTUA-Security-Project/scripts/Create-AMI-OU-Structure.ps1)
> is the template — copy and adapt it to make `Create-GIS-OU-Structure.ps1`.

### Step 1 — Create the VM shell (Windows Admin Center)

1. Browse to `https://OPS-AMDHOST.ntua-ops.local` and sign in with a Domain
   Admin account.
2. Tools pane → **Azure Arc resources** → cluster `NTUA-OPS-HCI1-C`.
3. **Virtual machines → New → Add VM**.
4. Fill in the wizard from the build sheet above:
   - **Resource group:** `rg-AzureLocal-GIS_Production`
   - **Image:** Windows Server 2022 Datacenter (Azure edition) — gallery image
     already published to the cluster
   - **Generation:** Gen 2
   - **vCPU / Memory:** from the build sheet; leave dynamic memory **off**
     (ArcGIS prefers fixed allocations)
   - **Storage path:** pick the CSV with the most free space (3 CSVs span the
     S2D pool across all 3 nodes — keep VMs spread roughly round-robin)
   - **OS disk:** 100 GB, dynamic
   - **Data disk:** add a second disk per the build sheet on the same CSV as
     the OS disk (skip for the two web adapters)
   - **Network:**
     - Internal-tier VMs → logical network on the guest VM VLAN
       (`10.23.62.0/?` — value pending Lee), **static IP** from the build sheet
     - DMZ-tier VMs → DMZ logical network behind the Palo Alto (TBD)
   - **Local admin:** set a strong password, store in the password vault
   - **Domain join in the wizard:** leave **off** — we'll do the domain join
     manually after IP/DNS are verified
5. Click **Create**. Wait for status `Succeeded` in WAC / Azure portal.

### Step 2 — First boot, rename, static IP, DNS, domain join

Connect with **Connect → Console** in WAC (or RDP once the IP is set).

```powershell
# 1) Rename to match the build sheet (skip if you set it in the wizard).
Rename-Computer -NewName 'ops2-gisportal' -Restart

# After reboot, log back in as local admin.

# 2) Confirm the static IP came through; if not, set it explicitly.
Get-NetIPConfiguration
# If needed:
New-NetIPAddress -InterfaceAlias 'Ethernet' -IPAddress 10.23.62.130 `
    -PrefixLength 24 -DefaultGateway 10.23.62.1
Set-DnsClientServerAddress -InterfaceAlias 'Ethernet' `
    -ServerAddresses 10.23.61.101,10.23.61.102

# 3) Sanity-check DNS resolves the DCs.
Resolve-DnsName ops-dc1.ntua-ops.local
Resolve-DnsName ops-dc2.ntua-ops.local

# 4) Join the domain. Will prompt for a domain account that can join.
Add-Computer -DomainName ntua-ops.local -Restart
```

> The security project has a packaged variant of the above as
> [`scripts/Join-Domain-Local.ps1`](../../NTUA-Security-Project/scripts/Join-Domain-Local.ps1)
> — copy it into the VM if you'd rather run a known script than type the commands.

After reboot, log in with a domain account (`ntua-ops\<you>`) and verify:

```powershell
(Get-WmiObject Win32_ComputerSystem).Domain   # ntua-ops.local
nltest /sc_verify:ntua-ops.local              # NERR_Success
```

### Step 3 — Move the computer object into the GIS OU

By default, the new computer lands in `CN=Computers,DC=ntua-ops,DC=local`,
which is **not** linked to the security baseline GPO. Move it.

From a DC (or any box with the `ActiveDirectory` RSAT module):

```powershell
Import-Module ActiveDirectory

# Internal-tier VM example:
Get-ADComputer ops2-gisportal |
    Move-ADObject -TargetPath 'OU=GIS,OU=Servers,DC=ntua-ops,DC=local'

# DMZ-tier VM example:
Get-ADComputer ops2-giswebportal |
    Move-ADObject -TargetPath 'OU=DMZ,OU=GIS,OU=Servers,DC=ntua-ops,DC=local'
```

Reference template:
[`scripts/Domain-Join-Checklist.ps1`](../../NTUA-Security-Project/scripts/Domain-Join-Checklist.ps1)
in the security project — uses the same `Move-ADObject` pattern for AMI VMs.

### Step 4 — Apply the security baseline GPO

The security project already maintains
[`AMI-Servers-Security-Baseline`](../../NTUA-Security-Project/04-Configuration-Hardening/GPOs/AMI-Servers-Security-Baseline/)
— Microsoft Security Baseline v3.0 with NTUA tweaks (14-char passwords, RDP /
WinRM / SQL firewall rules, audit logging, Defender exclusions for SCADA).

Plan: link the same GPO to the new `OU=GIS` tree so GIS gets the identical
posture as the AMI servers. Two options:

- **Easiest (recommended):** link the existing GPO to `OU=GIS,OU=Servers,...`
  with the **Enforced** flag, same as it's linked on `OU=Servers` /
  `OU=AMI`. No duplication, future baseline updates flow automatically.
- **If GIS needs different rules later:** copy the GPO to `GIS-Servers-Security-Baseline`
  and link that copy on `OU=GIS` instead. Diverge only when there's a real
  reason — every divergence is something to keep in sync by hand.

Apply and verify on each VM:

```powershell
gpupdate /force
gpresult /r /scope:computer | Select-String 'Applied Group Policy'
# Should list the baseline GPO under "Applied Group Policy Objects"
```

A single reboot after the first `gpupdate` is the cleanest way to make sure
firewall and audit policy settings are fully in effect.

### Step 5 — Onboard to Azure (Arc, AMA, Update Manager, Defender)

Because the VM was created through Arc Resource Bridge on Azure Local, the
Azure resource (`Microsoft.HybridCompute/machines`) already exists. The
remaining step is enabling the **VM config agent** so Azure Policy can push
the extensions (Guest Configuration, Azure Monitor Agent, Windows Patch).

```powershell
# Run from any workstation with Azure CLI signed in to the GIS subscription.
az stack-hci-vm update `
    --name ops2-gisportal `
    --resource-group rg-AzureLocal-GIS_Production `
    --enable-vm-config-agent true
```

Repeat per VM (or loop). Reference:
[`scripts/Enable-AzureLocalGuestManagement.ps1`](../../NTUA-Security-Project/scripts/Enable-AzureLocalGuestManagement.ps1)
does this in bulk for AMI.

Within ~15–20 minutes the following policy-driven things land automatically
(no per-VM action needed — they're already assigned at subscription scope):

| What | Driven by | Verify in portal |
|---|---|---|
| Guest Configuration extension + system-assigned identity | Policy `12794019-7a00-42cf-95c2-882eed337cc8` | VM → Extensions → `AzurePolicyforWindows` |
| Windows Security Baseline (audit) | Built-in `Windows machines should meet requirements for the Azure security baseline` | Guest Assignments tab on the VM |
| Azure Monitor Agent + DCR association → Sentinel | Policy `0d1b56c6-...` + DCR `sentinel-dcr-ami-config` | VM → Extensions → `AzureMonitorWindowsAgent`; logs flowing in `SecurityEvent` table |
| Windows Patch Extension + monthly maintenance window | `MaintenanceConfig-OPS-Production` (3rd Sat, 02:00 MST, 3h55m) | Update Manager → Machines → VM "Healthy" |
| Defender for Servers Plan 2 (MDE, file integrity, vuln scan) | Subscription-level Defender plan | Defender for Cloud → Inventory → VM "Plan 2" |
| Required tags (`Environment`, `Owner`, `CostCenter`, …) | Tag policy initiative | Resource → Tags |

> The DCR (`sentinel-dcr-ami-config`) was built for AMI server log volume.
> Confirm with the Sentinel owner whether GIS events should land in the same
> workspace (`logs-sentinel`, `rg-sentinel`) or a separate workspace before
> association. Default: same workspace, same DCR — least operational overhead.

If a policy hasn't taken effect after ~30 min, trigger a manual evaluation:

```powershell
az policy state trigger-scan --resource-group rg-AzureLocal-GIS_Production
```

### Step 6 — Per-role ESRI prep (handoff)

At this point the VM is a hardened, monitored, patched member of
`ntua-ops.local`. Hand off to the ArcGIS install workflow:

- **Service account:** create / reuse the GIS service account in AD before
  install (Portal, Server, and Data Store all need one — can be a single
  shared account per ESRI guidance).
- **SSL certificate:** issue an internal CA cert for the VM's FQDN
  (Portal/Server insist on HTTPS).
- **Firewall openings inside the VM:** the baseline GPO closes everything
  except RDP/WinRM. Add the ESRI ports the role needs (7443 Portal, 6443
  Server, 2443/9876 Data Store, 11211 GeoEvent if used).
- **Install media + version:** confirm with Lee which ArcGIS Enterprise
  version (matching the existing DEUCES stack to keep the migration
  in-place-upgrade-friendly).

Detailed ESRI install steps are out of scope for this SOP — they live in the
ArcGIS Enterprise install guide and Lee's runbook.

## Operational reference

### Lifecycle (start / stop / checkpoint / live migrate)

Day-to-day operations are all in WAC under the cluster → **Virtual machines**
view:

- **Start / Stop / Restart:** select the VM → action buttons. Always use **Shut
  down** (graceful) before **Stop** (forced).
- **Checkpoints:** Settings → Checkpoints. Use sparingly for short-lived
  rollbacks (e.g., before a patch); not a backup. Delete within days — ESRI
  data churn makes long-lived checkpoints expensive.
- **Live migration:** WAC moves VMs between nodes automatically for
  maintenance. Manually: select VM → **Move** → choose target node.

### Storage

- Storage is hyperconverged — S2D pool spans all 3 Dell APEX nodes, exposed as
  3 cluster shared volumes. No external SAN. Existing VMs are spread roughly
  round-robin across the 3 CSVs; keep that pattern for the new GIS VMs (pick
  the CSV with the most free space when creating each one).
- Check current volume free space in WAC → cluster → **Volumes**, or:
  `Get-ClusterSharedVolume | Get-ClusterSharedVolumeState | ft Name,FileSystemType,StateInfo`
  and `Get-Volume` on a cluster node for raw capacity.
- Add a disk: VM → Settings → Disks → **Add disk** (dynamic, same CSV as the
  OS disk unless that CSV is tight on space).
- Expand a disk: shut the VM down, Settings → Disks → **Expand**, boot, then
  extend the partition in Disk Management.

### Networking

- VLAN/subnet is set by the logical network chosen at VM creation — don't
  change it post-create.
- Static IP is set inside the guest (Step 2). DNS registers automatically into
  `ntua-ops.local` on domain join.
- Firewall: baseline GPO governs the perimeter. Per-role openings happen in
  Step 6 via the same GPO (preferred) or VM-local `New-NetFirewallRule` (only
  for short-term testing).

## Appendix A: Azure portal reference

WAC is the primary tool; the portal is for visibility and policy/extension
status.

- VM: subscription `d6520ce9-...` → resource group `rg-AzureLocal-GIS_Production`
  → resource type `Microsoft.HybridCompute/machines` (the Arc projection).
- Cluster health: `rg-hci-fd` → `NTUA-OPS-HCI1-C`.
- Update status: **Update Manager** → Machines → filter by RG.
- Policy compliance: **Policy** → Compliance → filter by scope = the RG.

## Appendix B: az CLI reference

Common scripted equivalents:

```powershell
# List VMs on the cluster.
az stack-hci-vm list --resource-group rg-AzureLocal-GIS_Production -o table

# Stop / start a VM.
az stack-hci-vm stop  --name ops2-gisportal --resource-group rg-AzureLocal-GIS_Production
az stack-hci-vm start --name ops2-gisportal --resource-group rg-AzureLocal-GIS_Production

# Re-evaluate policy on a VM right now.
az policy state trigger-scan --resource-group rg-AzureLocal-GIS_Production

# Show all installed extensions on a VM.
az connectedmachine extension list --machine-name ops2-gisportal `
    --resource-group rg-AzureLocal-GIS_Production -o table
```

Prefer CLI over WAC when scripting bulk operations or when WAC is
unreachable.

## Troubleshooting

| Symptom | First thing to check |
|---|---|
| Domain join fails with "domain controller cannot be contacted" | DNS — `Resolve-DnsName ntua-ops.local` should return both DC IPs. If not, re-set `Set-DnsClientServerAddress`. |
| `gpresult` doesn't show the baseline GPO | Computer object isn't in `OU=GIS` (Step 3). Confirm with `(Get-ADComputer ops2-gisportal).DistinguishedName`. |
| VM stuck "Provisioning" in WAC | Check node disk pressure (`Get-ClusterSharedVolume`); check the WAC event log on `OPS-AMDHOST`. |
| Extensions never install after Step 5 | `--enable-vm-config-agent true` returned an error, or system-assigned identity didn't get created. Check VM → Identity in the portal. |
| AMA installed but no logs in Sentinel | DCR not associated — `az monitor data-collection rule association list --resource <vm-resource-id>`. |
| Update Manager shows "Unknown" | Windows Patch Extension not deployed yet (give it 30 min) or VM offline during last assessment. |
| WAC won't load at all | `OPS-AMDHOST` is the witness host — confirm it's online with `Test-NetConnection OPS-AMDHOST.ntua-ops.local -Port 443`. |
