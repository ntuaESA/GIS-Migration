# NTUA GIS Server Migration to Azure Local

| Field | Detail |
|---|---|
| **Opportunity** | OPP-2026-05-025 |
| **Purchase Order** | PO 200825 |
| **Budget** | 40 hours |
| **Client** | NTUA Electrical Operations / AMI / GIS |
| **Client Contact** | Lee Begaye — Sr. Systems Architect, AMI/GIS · leanderb@ntua.com · (928) 729-6219 |
| **NTUA Team** | Celeste Clah — celeste@ntua.com (working migration alongside Lee) |
| **Partner** | Mind's Angle LLC (Kutlu Gulamber) |
| **Contractor** | Round the House Technology (RTH) — David Bjurman-Birr |

## Summary

Migrate 4 ESRI GIS servers from legacy Windows Server infrastructure to new Windows Server 2022 VMs running on Dell APEX / Azure Local. The new VMs will be joined to the `ntua-ops.local` Active Directory domain (not `ntua.local` — Lee does not have admin rights there, and suspected GPO/Cisco security controls in that domain were causing ESRI communication failures on the legacy servers). The `ntua-ops.local` domain gives Lee full administrative control over the environment.

Sequencing is critical: **ESRI workloads must be confirmed working first**, then security baselines are applied afterward so we can clearly distinguish application issues from security-policy issues.

## Project Plan

### Phase 1: Prep & Discovery (6–8 hrs)

- Inventory current GIS servers (known: GIS2APP1, GIS6APP1 — confirm full list with Lee)
- Document existing ESRI configuration, versions, licensing
- Review Azure Local cluster capacity and networking
- Identify suspected Cisco/GPO controls in `ntua.local` that caused legacy comms issues

### Phase 2: Build (8–10 hrs)

- Provision 4× Windows Server 2022 VMs via Windows Admin Center (WAC)
- Join VMs to `ntua-ops.local` domain
- Configure networking, storage, and DNS
- Document the entire process as an SOP so Lee can independently manage VMs going forward

### Phase 3: ESRI Install & Migration (10–14 hrs)

- Coordinate with ESRI support for installation and licensing
- Migrate data and configurations from legacy servers
- Validate that the communication issues from legacy servers are resolved
- Document all service accounts, ports, and protocols

### Phase 4: Security Baseline (4–6 hrs)

- Apply security baselines **after** ESRI is confirmed working
- Follow the GPO-based approach from the `ntua-coop-pentest` repo
- Document any exceptions required for ESRI functionality

### Phase 5: Handoff & Docs (4–6 hrs)

- Finalize SOP for VM deployment and management
- Complete as-built documentation
- Deliver service account inventory
- Knowledge transfer session with Lee

## Status

> **Current Phase: Phase 1 — Discovery**

## Documentation Map

Start here, in order, if you're new to this project (this means you, Lee):

1. [discovery/azure-local-environment.md](discovery/azure-local-environment.md) — The Azure Local cluster your new VMs will live on. **Read first.**
2. [discovery/az-cli-setup.md](discovery/az-cli-setup.md) — One-time setup to query NTUA's Azure from the command line.
3. [discovery/azure-live-snapshot-2026-06-05.md](discovery/azure-live-snapshot-2026-06-05.md) — Verified-live capture of the cluster, RGs, and existing GIS VMs (and where the security-project as-built has drifted from reality).
4. [discovery/current-server-inventory.md](discovery/current-server-inventory.md) — The legacy GIS servers being replaced.
5. [discovery/esri-dependencies.md](discovery/esri-dependencies.md) — ESRI software, ports, accounts, and the legacy comm-failure issue.
6. [docs/access-and-permissions.md](docs/access-and-permissions.md) — Who has access to what, and the PIM model for elevated roles.
7. [docs/sop-vm-deployment.md](docs/sop-vm-deployment.md) — How to deploy and manage VMs on the cluster (the everyday playbook).
8. [docs/service-accounts.md](docs/service-accounts.md) — Service accounts the GIS stack depends on.
9. [docs/diagrams/](docs/diagrams/) — Architecture diagrams (.drawio). Open in VS Code with the [Draw.io Integration](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio) extension.
10. [docs/as-built.md](docs/as-built.md) — The final state of the migrated environment (filled in as we go).

## Related

- [`../NTUA-Security-Project`](../NTUA-Security-Project) — Completed 2025 security engagement. Most-relevant pages:
  - [Azure Local cluster as-built](../NTUA-Security-Project/01-Network-Segmentation/docs/infrastructure/azure-local-cluster-config.md) — source of truth for `NTUA-OPS-HCI1-C`
  - [Existing GIS VM inventory & Arc status](../NTUA-Security-Project/azure-local-guest-management-status.md)
  - [Network segmentation overview](../NTUA-Security-Project/01-Network-Segmentation/README.md)
  - [Configuration hardening (Phase 4 reference)](../NTUA-Security-Project/04-Configuration-Hardening/README.md)
- `ntua-coop-pentest` — Security baseline / GPO reference from prior NTUA engagement
- OPP-2026-05-025 — Opportunity record
