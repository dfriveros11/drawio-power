# Prompt File Specification

The prompt file (`prompt-*.md`) is the single source of truth for diagram regeneration. It must be kept in sync with the `.drawio` file at all times.

## Required Sections

Every prompt file must contain these sections in order:

1. Canvas — title, flow direction, width, height
1. Groups — id, label, type, parent, x, y, width, height
1. Nodes — id, parent, icon, label, sublabel, x, y
1. Connections — id, source, target, exit-port, entry-port, label, style, step
1. Style Notes — free-text rendering rules

## Column Schemas

### Canvas

| Column | Type | Description |
|--------|------|-------------|
| Property | string | Canvas property name |
| Value | string/int | Property value |

### Groups

| Column | Type | Allowed Values |
|--------|------|----------------|
| ID | string | Unique group identifier |
| Label | string | Display label |
| Type | string | `aws-cloud`, `vpc`, `subnet-public`, `subnet-private` |
| Parent | string/null | Parent group ID or `null` |
| X, Y | int | Absolute position |
| Width, Height | int | Dimensions |

### Nodes

| Column | Type | Description |
|--------|------|-------------|
| ID | string | Unique node identifier (used in connections) |
| Parent | string | Group ID or `(root)` for external actors |
| Icon | string | MCP shape key, `user`, or `square` |
| Label | string | Primary label |
| Sublabel | string | Secondary label (below primary) |
| X, Y | int | Position relative to parent group |

### Connections

| Column | Type | Allowed Values |
|--------|------|----------------|
| ID | string | Unique connection identifier |
| Source | string | Node ID from Nodes table (never sub-cell IDs) |
| Target | string | Node ID from Nodes table (never sub-cell IDs) |
| Exit-Port | string | `top`, `bottom`, `left`, `right` |
| Entry-Port | string | `top`, `bottom`, `left`, `right` |
| Label | string | Connection label with step number prefix |
| Style | string | `sync`, `async`, `optional` |
| Step | int | Sequential step number |

## Style Notes

Free-text rendering rules describing connection styles and visual conventions used in the diagram. Use this section to document any diagram-specific rendering decisions such as dashed lines for async flows, dotted lines for optional paths, color conventions, or label placement preferences.

## Rules

- Connections reference node IDs, never sub-cell IDs (`-icon`, `-label`) — sub-cell resolution is handled by the MCP engine automatically based on port direction
- External actors have `(root)` as parent
- Node icon values must match MCP shape keys or special values (`user`, `square`)
- Step numbers are consecutive starting from 1, no gaps
- When syncing `.drawio` → prompt: parse XML, map sub-cells back to node IDs, convert child-relative positions to parent-relative coordinates

