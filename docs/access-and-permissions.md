# Access & Permissions

> **Purpose:** Single source of truth for who has access to what on this project, what
> role they need, and how access is granted. Keep this current \u2014 access drift is how
> projects get stuck.

## Project team

| Person | Email | Role | Access needed |
|---|---|---|---|
| Lee Begaye | `leanderb@ntua.com` | Client lead \u2014 Sr. Systems Architect, AMI/GIS | Owns the `ntua-ops.local` domain and the Azure tenant; full admin everywhere |
| Celeste Clah | `celeste@ntua.com` | NTUA team — working the migration alongside Lee | Entra ID Global Administrator + Subscription Owner (**via PIM, see below**) |
| David Bjurman-Birr | `davidb@ntuaops.onmicrosoft.com` | RTH (contractor) | Currently signed in; effective Owner across the tenant |
| Kutlu Gulamber | (Mind's Angle LLC) | Partner | TBD \u2014 confirm with Lee whether Kutlu needs Azure access |

> If anyone else needs access during the project, add them here **before** the role gets
> granted, not after.

## NTUA tenant facts (verified 2026-06-05)

| Field | Value |
|---|---|
| Tenant ID | `19154b1a-aa93-4c98-aa77-e950e4f0d817` |
| Tenant default domain | `ntuaops.onmicrosoft.com` |
| Subscription name | `Azure subscription 1` |
| Subscription ID | `d6520ce9-5566-4091-920b-4348d4e708b4` |

## Celeste's access \u2014 plan

Lee approved (2026-06-05) granting Celeste **Global Administrator** in Entra ID and
**Owner** on the subscription. **Both should be assigned through Privileged Identity
Management (PIM) as eligible, not permanent**, so day-to-day she runs without elevated
privileges and activates only when needed.

### Why PIM (and not direct, permanent assignment)

- **Blast radius** \u2014 Global Admin can do anything in the tenant; Owner can do anything
  in the subscription. Standing access means a phished or stolen session is catastrophic.
- **Audit trail** \u2014 Every activation is logged with reason + ticket reference; standing
  roles are not.
- **MFA on activation** \u2014 PIM can require MFA at activation even if the session is
  already MFA-d, giving a real-time consent step before sensitive work.
- **Approvals** \u2014 Optional approver gate for high-risk roles (Lee/David can be set as
  approvers for Global Admin activations).
- **Aligns with security-project posture** \u2014 The completed 2025 engagement deliberately
  introduced PIM for NTUA Operations (see [NTUA-Security-Project / 08-Access-Management](../../NTUA-Security-Project/08-Access-Management/README.md)).

### Roles to assign (eligible, not active)

| Role | Scope | Assignment type | Max activation duration | Activation requirements |
|---|---|---|---|---|
| Global Administrator | Tenant `ntuaops.onmicrosoft.com` | PIM eligible | 4 hours (recommended) | MFA + justification + (optional) Lee approval |
| Owner | Subscription `d6520ce9-...e708b4` | PIM eligible (Azure resources) | 8 hours | MFA + justification |

### Prerequisites for Celeste

- [ ] Entra ID account `celeste@ntua.com` exists and is enabled
- [ ] MFA registered (Authenticator app, FIDO2 key, or Windows Hello for Business)
- [ ] **No** standing assignments to Global Admin or Owner before PIM is set up (avoid the dual-state confusion)

### How to assign (Azure Portal, recommended for this one-off)

**Global Administrator (Entra ID PIM)**

1. Azure Portal \u2192 search **Microsoft Entra Privileged Identity Management**
2. **Microsoft Entra roles** \u2192 **Assignments** \u2192 **Add assignments**
3. Role: `Global Administrator`
4. Member: `celeste@ntua.com`
5. Assignment type: **Eligible**
6. Duration: **Permanently eligible** (so the eligibility doesn't auto-expire mid-project)
7. Open the role's **Role settings** and verify: MFA on activation = **Required**, Justification = **Required**, Max activation duration ≤ 4 hours

**Owner on the subscription (Azure resources PIM)**

1. Azure Portal → the subscription (`Azure subscription 1`) → **Access control (IAM)**
2. **Privileged access (Preview)** tab → **Add assignments**
3. Role: `Owner`
4. Member: `celeste@ntua.com`
5. Assignment type: **Eligible** \u2014 Permanently eligible
6. Confirm the role's settings require MFA + justification on activation

### How to assign (CLI \u2014 for reference / automation)

> CLI for PIM is awkward and inconsistent across role types. For one-off setup, the
> portal is faster and safer. Documenting the shape here only:

```powershell
# Verify Celeste exists in the tenant
az ad user show --id celeste@ntua.com --query "{upn:userPrincipalName,id:id,enabled:accountEnabled}" -o table

# Standing (NOT preferred — use PIM instead): Owner on subscription
# az role assignment create --assignee celeste@ntua.com --role Owner \
#   --scope /subscriptions/d6520ce9-5566-4091-920b-4348d4e708b4
```

For PIM, prefer the Graph API or the Azure Portal. Microsoft Learn:
[Assign Microsoft Entra roles in PIM](https://learn.microsoft.com/azure/active-directory/privileged-identity-management/pim-how-to-add-role-to-user).

### After assignment \u2014 verify with Celeste

- [ ] Celeste can sign in to <https://portal.azure.com> with `celeste@ntua.com`
- [ ] She sees the NTUA tenant and `Azure subscription 1`
- [ ] PIM \u2192 **My roles** shows `Global Administrator` (Eligible) and `Owner` (Eligible) \u2014 both **inactive** until she activates
- [ ] She can activate either role with justification and the role appears as **Active** with the correct expiry
- [ ] Add her UPN to the project team table at the top of this file once verified

## Access tracking

| Person | Standing roles | Eligible (PIM) roles | Granted | Verified | Notes |
|---|---|---|---|---|---|
| David Bjurman-Birr | (existing) | (existing) | pre-project | n/a | RTH contractor; already operational |
| Celeste Clah | none | Global Admin (tenant), Owner (sub `d6520ce9-...`) | **pending** | **pending** | Per Lee, 2026-06-05. UPN confirmed via Lee's commit to ntuaESA/GIS-Migration. |

## Action items

- [ ] Confirm Celeste has an MFA method registered (or coordinate first-time MFA setup)
- [ ] Assign Global Admin (PIM eligible) per portal steps above
- [ ] Assign Owner on subscription (PIM eligible) per portal steps above
- [ ] Confirm activation works end-to-end with Celeste
- [ ] Loop Celeste into this repo (push access on GitHub `ntuaESA/GIS-Migration` is separate from Azure access \u2014 Lee should add her as a collaborator)
- [ ] Decide whether Kutlu (Mind's Angle) needs any Azure access for this project
