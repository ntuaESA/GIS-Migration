# SOP: Deploying and Managing VMs on Azure Local

## Purpose

This Standard Operating Procedure documents how to create, configure, and manage virtual machines on the NTUA Azure Local (Dell APEX) cluster using Windows Admin Center (WAC). The goal is to enable Lee Begaye and the NTUA GIS team to independently deploy and manage VMs without requiring RTH involvement for routine operations.

## Audience

- **Primary:** Lee Begaye, Sr. Systems Architect, AMI/GIS
- **Secondary:** NTUA GIS team members who may assist with VM management

## Prerequisites

> TODO — Document:
> - WAC URL and access requirements
> - Azure Local cluster name and node details
> - Required permissions / AD group membership
> - Network prerequisites (VLANs, DNS, DHCP reservations)

## Procedure (Windows Admin Center)

### Creating a New VM

> TODO — Step-by-step with screenshots:
> - Navigate to cluster → Virtual Machines
> - New VM wizard (name, generation, memory, vCPU, network, disk)
> - Naming convention for NTUA GIS VMs
> - Recommended defaults (Gen 2, dynamic memory, etc.)

### Configuring VM Settings

> TODO — Document:
> - Adjusting vCPU and memory after creation
> - Adding/removing virtual network adapters
> - Configuring boot order
> - Integration services settings

### Managing VM Lifecycle (Start / Stop / Checkpoint)

> TODO — Document:
> - Starting and stopping VMs gracefully
> - Creating and reverting checkpoints (snapshots)
> - When to use checkpoints vs. backups
> - Live migration between cluster nodes

### Storage Management

> TODO — Document:
> - Attaching additional virtual disks
> - Expanding existing disks
> - Storage pool / volume overview
> - Shared storage considerations for ESRI

### Networking

> TODO — Document:
> - VLAN assignment
> - Static IP configuration inside the VM
> - DNS registration in ntua-ops.local
> - Firewall rules relevant to ESRI

## Appendix A: Azure Portal Reference

> TODO — Document portal-based visibility for the same operations. This is supplementary — WAC is the primary management tool. Include:
> - Navigating to Azure Local resources in the portal
> - Viewing VM status and metrics
> - When the portal is useful vs. when WAC is required

## Appendix B: az CLI Reference

> TODO — Document equivalent CLI commands for automation scenarios:
> - `az stack-hci vm create` basics
> - Common management commands
> - When CLI is preferable to WAC (scripting, bulk operations)

## Troubleshooting

> TODO — Document common issues:
> - VM won't start (resource contention, storage errors)
> - VM lost network connectivity
> - WAC connectivity issues
> - Checkpoint/snapshot management problems
