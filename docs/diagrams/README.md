# Diagrams

Architecture diagrams for the NTUA GIS Server Migration project.

## Files

| File | What | How to view/edit |
|---|---|---|
| [arcgis-enterprise-architecture.drawio](arcgis-enterprise-architecture.drawio) | Base ArcGIS Enterprise multi-machine deployment for the 4 new NTUA VMs. Based on [ESRI's reference architecture](https://doc.esri.com/en/arcgis-enterprise/latest/plan/base-arcgis-enterprise-deployment.html?pivots=os-windows) and Celeste's draft. | Open in VS Code with the [Draw.io Integration](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio) extension (`hediet.vscode-drawio`). |

## Editing

- **In VS Code:** just open the `.drawio` file — the extension renders the diagram editor inline.
- **In a browser:** drag the file onto <https://app.diagrams.net> (or open it from there). Save back to disk to keep the repo in sync.

## File format choice

- `.drawio` (plain XML) — primary, easy to diff and edit. **Does not render on GitHub.**
- `.drawio.svg` — same diagram embedded inside an SVG. Renders inline in GitHub Markdown previews and PRs. Use this if/when we want a diagram to render in the GitHub web UI.

To produce a `.drawio.svg` from a `.drawio`: open the file in VS Code (drawio extension) → command palette → `Draw.io: Convert To`. Or in <https://app.diagrams.net>: File → Save as → "Editable Bitmap SVG".

## Conventions

- One concept per file. Don't merge unrelated diagrams into one file.
- Filename: lowercase, hyphen-separated, descriptive (`arcgis-enterprise-architecture.drawio`, not `diagram1.drawio`).
- Always include a title block and a legend/notes box in the diagram itself so it stands alone when exported as an image.
- Keep IP addresses, hostnames, and ports in sync with [discovery/current-server-inventory.md](../../discovery/current-server-inventory.md) and [docs/as-built.md](../as-built.md).
