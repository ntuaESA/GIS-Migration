# Time Log — PO 200825 (40 hrs)

| Date | Hours | Phase | Description |
|---|---|---|---|
| 2026-06-05 | 1.0 | Phase 1 | Discovery and planning — repo setup, initial documentation stubs |
| 2026-06-05 | 1.5 | Phase 1 | Cross-referenced NTUA-Security-Project as-built; populated discovery docs with real Azure Local cluster facts (NTUA-OPS-HCI1-C, subscription, domain, DNS, VLANs); authored azure-local-environment.md and az-cli-setup.md as Lee-friendly on-ramps; seeded SOP and as-built with concrete values; added documentation map to README |
| 2026-06-05 | 1.0 | Phase 1 | Scope confirmation from Lee (4 net-new VMs, IP range 10.23.62.130-134); device-code login to NTUA tenant; live snapshot of rg-AzureLocal-GIS_Production via az connectedmachine; corrected stale subscription ID + cluster RG in docs; captured tag baseline from existing 8 GIS VMs; authored azure-live-snapshot-2026-06-05.md |
| 2026-06-05 | 0.5 | Phase 1 | Call with Lee — confirmed Celeste (celestec@ntua.com) joins the project; authored docs/access-and-permissions.md with PIM-based Global Admin + Subscription Owner plan; added team to README; wired GitHub remote `origin` → ntuaESA/GIS-Migration |
| 2026-06-05 | 0.5 | Phase 1 | Recreated Celeste's ArcGIS Enterprise architecture diagram as editable docs/diagrams/arcgis-enterprise-architecture.drawio (drawio extension); added docs/diagrams/README.md with format conventions; flagged IP-range discrepancy (Lee said 130-134, design uses 128-134) for follow-up |
| 2026-06-05 | 0.5 | Phase 1 | Treated Lee's hostname + legacy-server edits as authoritative (DEUCES stack replaces previous GIS2APP1/GIS6APP1 assumption); restructured SOP table into Internal tier (10.23.62.130-134) + DMZ tier (web adapters, IPs TBD) per Lee's plan to use Palo Alto NGFW segmentation; updated diagram and environment overview to match; pending: confirm DMZ existence in Palo Alto config next week |
| 2026-06-05 | 0.25 | Phase 1 | Renamed all new VMs to ops2- prefix + conventional ESRI role names per Lee (ops2-gisportal, ops2-gisserver, ops2-gisdatastore, ops2-giswebportal, ops2-giswebserver); updated SOP, diagram, and inventory |

**Running total:** 5.25 / 40 hrs
