# NTUA GIS Server Migration to Azure Local

| Field | Detail |
|---|---|
| **Opportunity** | OPP-2026-05-025 |
| **Purchase Order** | PO 200825 |
| **Budget** | 40 hours / $6,000 |
| **Client** | NTUA Electrical Operations / AMI / GIS |
| **Client Contact** | Lee Begaye — Sr. Systems Architect, AMI/GIS · leanderb@ntua.com · (928) 729-6219 |
| **Partner** | Mind's Angle LLC (Kutlu Gulamber) |
| **Contractor** | Round the House Technology (RTH) |

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

## Related

- [ntua-coop-pentest](https://github.com/yourorg/ntua-coop-pentest) — Security baseline / GPO reference from prior NTUA engagement
- OPP-2026-05-025 — Opportunity record
