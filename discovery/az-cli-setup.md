# Connecting to NTUA's Azure with the `az` CLI

> **Audience:** Lee Begaye and anyone on the project who needs to query NTUA's Azure from
> the command line. Use this once and you're set; subsequent commands just work.

## Why bother with the CLI?

The Azure Portal is great for browsing one VM at a time. The CLI is faster (and
scriptable) for:

- Listing every VM in `rg-AzureLocal-GIS_Production` in one command
- Pulling cluster status and node health
- Enabling the VM Config Agent on a new Azure Local VM (one command, vs. clicking through
  the portal)
- Producing snapshots of the environment to paste into this repo

## Install (one-time)

If `az --version` returns nothing, install it:

```powershell
winget install --id Microsoft.AzureCLI -e
```

Then open a **new** PowerShell window so `az` is on the path.

Verify:

```powershell
az --version
```

You want `azure-cli 2.7x` or newer.

## Sign in (device code, no browser popups)

Device-code auth is the right choice on a server / RDP session / shared workstation —
nothing opens in a browser tab on the wrong machine.

```powershell
az login --use-device-code
```

The CLI prints:

```
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and
enter the code ABC123XYZ to authenticate.
```

Open that URL on **your own** workstation, paste the code, sign in with your
`leanderb@ntua.com` account.

## Point at the NTUA Azure Local subscription

After login, you'll see every subscription your account can touch. Pin yourself to the
NTUA Operations one:

```powershell
az account set --subscription d6520ce9-5566-4091-920b-4348d4e708b4
```

Confirm:

```powershell
az account show --query "{name:name, id:id, tenantId:tenantId, user:user.name}" -o table
```

Expected output (verified 2026-06-05):

```
Name                  TenantId                              User
--------------------  ------------------------------------  ------------------------------
Azure subscription 1  19154b1a-aa93-4c98-aa77-e950e4f0d817  davidb@ntuaops.onmicrosoft.com
```

## Smoke-test commands

These read-only commands prove you're connected to the right place:

```powershell
# All Azure Local clusters in the subscription (should show NTUA-OPS-HCI1-C)
az stack-hci cluster list -o table

# All GIS VMs (Arc machines) in the GIS resource group
az resource list `
  --resource-group rg-AzureLocal-GIS_Production `
  --resource-type Microsoft.HybridCompute/machines `
  --query "[].{name:name, location:location, kind:kind}" -o table

# All resource groups starting with rg-AzureLocal-
az group list --query "[?starts_with(name,'rg-AzureLocal-')].name" -o table
```

## Useful one-liners for this project

### Snapshot of all GIS VMs (paste into discovery docs)

```powershell
az resource list `
  --resource-group rg-AzureLocal-GIS_Production `
  --resource-type Microsoft.HybridCompute/machines `
  --query "sort_by([].{Name:name, OS:properties.osName, OSVersion:properties.osVersion, Status:properties.status, LastSeen:properties.lastStatusChange}, &Name)" `
  -o table
```

### Check whether a VM is a "true" Azure Local VM (has VM Instance resource)

```powershell
# Replace <vm-name>
az resource show `
  --ids "/subscriptions/d6520ce9-5566-4091-920b-4348d4e708b4/resourceGroups/rg-AzureLocal-GIS_Production/providers/Microsoft.HybridCompute/machines/<vm-name>/providers/Microsoft.AzureStackHCI/virtualMachineInstances/default" `
  --query "{name:name, provisioningState:properties.provisioningState}" -o table
```

If this errors with `ResourceNotFound`, the VM is Arc-only and NOT managed through the
Azure Local plane. (Most of the existing `rg-AzureLocal-GIS_Production` VMs are in this
state — see [azure-local-environment.md](azure-local-environment.md).)

### Enable the VM Config Agent on a newly-built Azure Local VM

```powershell
az stack-hci-vm update `
  --name <vm-name> `
  --resource-group rg-AzureLocal-GIS_Production `
  --enable-vm-config-agent true
```

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `az: command not found` after install | Old terminal session | Close and reopen PowerShell |
| `Please run 'az login'` on every command | Multiple Windows accounts; token landed in wrong profile | `az logout` then `az login --use-device-code` again |
| `Subscription d6520ce9-... not found` | Your NTUA account hasn't been granted RBAC on the subscription | Lee / David grants Reader (or higher) on the subscription |
| `(AuthorizationFailed)` on a specific command | Account has access to subscription but not the resource group | Need Reader on that RG specifically |
| `az stack-hci-vm ... not found` | Extension not auto-installed | `az extension add --name stack-hci-vm` |

## Where credentials live

This repo never stores secrets. Service principal IDs, cluster passwords, and BitLocker
keys live in the secured as-built in the security-project repo (and that repo uses
secret placeholders, not real values). See
[azure-local-cluster-config.md](../../NTUA-Security-Project/01-Network-Segmentation/docs/infrastructure/azure-local-cluster-config.md).
