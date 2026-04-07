# Layout and Routing

Phase 3 of the diagram workflow. Assign positions and route connections for a clean, readable, overlap-free diagram.

## Reading Direction: Left-to-Right (ALWAYS)

Every architecture diagram reads left-to-right. This is the single most important layout rule:

- Entry points (users, DNS, CDN) on the left
- Compute in the middle
- Data stores and backends on the right
- Monitoring/observability at the far right or top-right
- Primary data flow arrows always point rightward
- Reverse flows (e.g., outbound via NAT Gateway) are the only exception — they point left

This applies to both flat and VPC-aware layouts. TB (top-to-bottom) is only used when explicitly requested or when the architecture is purely vertical (e.g., CI/CD pipeline).

## Input

From Phase 2: node list (with flow_position), connection list (with anchor ports), group list (with computed dimensions), affinity hints, pattern classification.

## Canvas Sizing

Derive canvas size from the flow depth and node density computed in Phase 2.

### Flow-based sizing (preferred)

```text
max_flow_pos = max(node.flow_position for all nodes)
max_stack    = max(count of nodes at same flow_position)

canvas_width  = (max_flow_pos + 1) * 250 + 300    # columns + external actor + padding
canvas_height = max_stack * 170 + 200              # rows + padding for groups/labels
```

Clamp minimum to 500x350. For new diagrams from text, cap at 1400x1000.

### Fallback static sizing

When `flow_position` is not available (reference reproductions):

| Icon count | Canvas size |
|-----------|-------------|
| 1-4 | 500x350 |
| 5-7 | 600x450 |
| 8-10 | 800x600 |
| 11-15 | 1000x800 |
| 16+ | 1200x1000 |

### Reference-aware sizing

For reference reproductions the 1200x1000 cap does not apply. Match original aspect ratio: `width = max(original_width * 0.8, table_width)`, `height = max(original_height * 0.8, table_height)`. Add 100px width and 80px height per nesting level beyond aws-cloud.

## Flow Direction

1. Linear chain: left-to-right, single row
1. Fan-out (1→3+): hub left, targets stacked right
1. Fan-in (3+→1): sources left, target right
1. Event-driven hub: hub top-center, consumers below
1. Request-response with shared data: left-to-right, shared services below
1. Default: left-to-right

## Flat Layout (ALWAYS for planning)

Every diagram starts with flat grid planning. All node positions are computed as if parented directly to `aws-cloud`. This ensures clean LR flow with adjacent nodes.

For serverless architectures (no VPC): nodes stay parented to `aws-cloud`. Single group.
For VPC architectures: after planning the flat grid, wrap VPC/subnet groups around VPC-bound nodes using the two-pass approach (see "VPC/Subnet Visual Annotations"). Nodes get re-parented to their subnet group with converted coordinates.

Every diagram must include an `aws-cloud` group (`group_type="aws-cloud"`) rendering the AWS Cloud boundary with official logo. Mandatory — never omit or substitute.

### Node placement (flat grid planning)

1. External actors: `parent_group=""` (page root), x=40
1. Plan ALL AWS nodes as if `parent_group="aws-cloud"` — compute positions on the flat grid
1. Place nodes in `flow_position` order from Phase 2 — each position maps to a column (left-to-right)
1. Nodes sharing the same `flow_position` stack vertically in the same column
1. After grid is planned: if VPC needed, convert VPC-bound node coordinates to group-relative and set `parent_group` to their subnet

### Flow-position to column mapping

Use `flow_position` from Phase 2 to assign columns. This replaces rigid tier-based X values with data-flow-driven placement.

**Column spacing formula (flat layout):**

```text
col_spacing = 200                    # 200px between column centers
col_x(n) = 50 + n * col_spacing     # X position inside aws-cloud for column n
```

For dense diagrams (>6 columns), reduce to 150px. For sparse diagrams (≤4 columns), increase to 250px.

**Row spacing formula:**

```text
row_spacing = 200                    # 200px between row centers
row_y(n) = 200 + n * row_spacing     # Y position inside aws-cloud for row n
row_y(0) = 30                        # Row 0 = monitoring row (above main flow)
```

Standard row assignments:

- Row 0 (y=30): Monitoring services (CloudWatch, X-Ray) — only when connected via async edge from a main-flow node
- Row 1 (y=200): Main flow (primary LR chain)
- Row 2 (y=400): Fan-out targets (data stores, auth, secondary services)
- Row 3 (y=600): Tertiary (auth chains, archive chains, notification endpoints)

- Each `flow_position` increment = one column step
- Nodes at the same `flow_position` share the same X, stacked vertically (200px apart)
- Affinity hints override: if node_b has affinity `"below"` node_a, they share the same column regardless of `flow_position`

Fallback to tier columns when `flow_position` is not available (e.g., reference reproductions).

## Tier Placement (Flat Layout)

When no VPC/subnet groups exist (flat layout with `aws-cloud` only):

| Tier | Column X (inside aws-cloud) | Examples |
|------|----------------------------|----------|
| Entry | x=50 | API Gateway, CloudFront, ALB |
| Compute | x=400 | Lambda, ECS, EC2 |
| Data | x=750 | DynamoDB, S3, RDS |
| AI/ML | x=1100 | Bedrock, SageMaker |

External actors at x=40 (page coordinates).

## VPC/Subnet Visual Annotations

When the architecture includes VPC-bound services, add VPC and subnet groups using the two-pass approach:

### Pass 1: Plan the flat grid

Place ALL nodes on the flat grid as if there were no groups. Use the standard row/column system:

```text
Row 0 (y=30):   Monitoring (CloudWatch)
Row 1 (y=200):  Main flow LR: CloudFront → ALB → ECS → RDS → ElastiCache
Row 2 (y=400):  Secondary: NAT GW (below ALB), S3 (below CloudFront)
```

For VPC architectures, the flat grid must place public-subnet nodes to the LEFT of private-subnet nodes (following LR flow). This ensures that when groups wrap around nodes in Pass 2, the public subnet is naturally on the left and private subnet on the right.

### Managed service placement around VPC (CRITICAL)

After planning the flat grid, managed services that are NOT inside the VPC must be positioned with clear visual separation from the VPC boundary. The minimum clearance between any managed node edge and the VPC group border is 40px.

**Inline managed services (in the traffic path):**

Services like AWS WAF, Amazon API Gateway, Amazon CloudFront, and Amazon Route 53 that are inline in the main LR flow MUST be placed on the main flow row (same Y as VPC-bound nodes on that row), but to the LEFT of the VPC boundary. They occupy their own column(s) before the VPC starts.

```text
GOOD (WAF inline, left of VPC):
  User → [WAF] → [  VPC: ALB → Lambda → RDS  ]

BAD (WAF above VPC, overlapping):
  [WAF]
  [  VPC: ALB → Lambda → RDS  ]
  ↑ WAF visually overlaps VPC label area
```

**Side managed services (auth, not inline):**

Services like Amazon Cognito that are side services (not inline) should be placed ABOVE the VPC with at least 40px clearance from the VPC top edge.

**Supporting managed services (no connections):**

CloudWatch, SQS (when standalone), and other supporting services go BELOW the VPC with at least 40px clearance from the VPC bottom edge.

**Layout column reservation for VPC diagrams:**

When planning the flat grid for a VPC diagram, reserve the leftmost columns for inline managed services that sit outside the VPC. The VPC starts at the column where the first VPC-bound node appears.

```text
Col 0: WAF (managed, inline)
Col 1: ALB (VPC, public subnet)     ← VPC starts here
Col 2: Lambda (VPC, private subnet)
Col 3: RDS (VPC, private subnet)
```

### Pass 2: Wrap groups around nodes

After planning all node positions, compute VPC and subnet groups that wrap around the VPC-bound nodes:

1. Identify which nodes are VPC-bound: ALB, NAT GW (public subnet), ECS, RDS, ElastiCache (private subnet)
1. Compute the bounding box of each group's nodes
1. Add padding: 40px top (for label), 20px sides, 20px bottom
1. Convert node coordinates from aws-cloud-relative to group-relative

**Coordinate conversion:**

```text
# Node's flat-grid position (inside aws-cloud): node_x, node_y
# Group position (inside aws-cloud): group_x, group_y
# Node's group-relative position: node_x - group_x, node_y - group_y
```

**CRITICAL: All child coordinates are RELATIVE to their parent group, not absolute.** When a subnet is inside a VPC, the subnet's (x, y) is relative to the VPC origin. When a node is inside a subnet, the node's (x, y) is relative to the subnet origin. Never use absolute coordinates for children — the MCP engine adds parent offsets automatically.

**Typical subnet positions inside VPC:**

```text
# Public subnet (left side of VPC):
subnet-public: x=20, y=40 (relative to VPC)

# Private subnet (right side of VPC):
subnet-private: x=200, y=40 (relative to VPC, right of public subnet)

# NOT: x=430, y=160 — these are absolute coordinates, not relative!
```

**Group sizing formula:**

```text
# For each group, find the bounding box of its nodes:
min_x = min(node.x for node in group_nodes)
max_x = max(node.x for node in group_nodes) + 120  # node width
min_y = min(node.y for node in group_nodes)
max_y = max(node.y for node in group_nodes) + 120  # node height

# Group position and size:
group_x = min_x - 20                    # 20px left padding
group_y = min_y - 40                    # 40px top padding (label)
group_width = (max_x - min_x) + 40     # 20px padding each side
group_height = (max_y - min_y) + 60    # 40px top + 20px bottom
```

**VPC wraps subnets (not nodes directly):**

```text
# VPC position (relative to aws-cloud):
vpc_x = 20                              # small offset from aws-cloud left edge
vpc_y = 40                              # below aws-cloud label

# VPC size: just big enough to contain both subnets
vpc_width = subnet-public.width + subnet-private.width + 60   # 20px gap + 20px padding each side
vpc_height = max(subnet-public.height, subnet-private.height) + 60  # 40px top + 20px bottom
```

**When managed services exist outside VPC (monitoring row):**

If the architecture has managed services parented to aws-cloud (not VPC) — like CloudWatch, Route 53, CloudFront — the VPC must start low enough to leave room for those nodes above it.

```text
# Count managed service rows above the VPC:
managed_rows_above = count of managed services placed above VPC (monitoring row, row 0)
node_height = 120                       # 78px icon + 2px gap + 40px label
row_gap = 20                            # gap between node bottom and VPC top

vpc_y = 40 + (managed_rows_above * (node_height + row_gap))

# Examples:
# 0 managed rows above → vpc_y = 40 (tight)
# 1 managed row above  → vpc_y = 40 + 1 * 140 = 180
# 2 managed rows above → vpc_y = 40 + 2 * 140 = 320

# aws-cloud height must accommodate:
aws_cloud_height = vpc_y + vpc_height + 20  # 20px bottom padding
```

**Avoid oversized groups.** The VPC should be just big enough to contain its subnets with 20px padding. Subnets should be just big enough to contain their nodes with 20px padding (40px top for label). Never add hundreds of pixels of empty space.

### Key rules

- Plan flat grid FIRST, then wrap groups — never size groups first and cram nodes inside
- Groups must be large enough that nodes maintain the same visual spacing as the flat grid
- ALL managed/global services stay parented to `aws-cloud` — never inside VPC. This includes SQS, SNS, EventBridge, DynamoDB, S3, Step Functions, API Gateway, Cognito, WAF, CloudWatch, Route 53, CloudFront, KMS, Secrets Manager, Systems Manager, and any service classified as `zone: managed` in Phase 1
- VPC-bound nodes get `parent_group` set to their subnet
- Build order: aws-cloud → VPC → subnets → nodes → connections
- The absolute pixel position of every node must be identical whether groups exist or not

### Post-layout VPC overlap check (BLOCKING)

After computing VPC/subnet group positions and sizes, verify that NO managed-zone node falls visually inside the VPC bounding box. For every managed node:

```text
for each node where zone == "managed":
  node_abs_x = aws_cloud_x + node.x
  node_abs_y = aws_cloud_y + node.y
  vpc_abs_x  = aws_cloud_x + vpc.x
  vpc_abs_y  = aws_cloud_y + vpc.y

  VIOLATION if:
    node_abs_x >= vpc_abs_x AND
    node_abs_x + 120 <= vpc_abs_x + vpc.width AND
    node_abs_y >= vpc_abs_y AND
    node_abs_y + 120 <= vpc_abs_y + vpc.height
```

If a managed node falls inside the VPC bounding box, fix by ONE of:

1. Move the managed node outside the VPC area (preferred — shift Y below VPC bottom edge + 40px, or shift X left of VPC left edge - 40px)
1. Shrink the VPC width/height so it no longer encloses the managed node
1. Reposition VPC-bound nodes to make the VPC tighter

Common offenders: SQS, SNS, EventBridge, DynamoDB, S3, Step Functions — these are managed services that often sit at the same Y level as VPC-bound nodes and get accidentally enclosed when the VPC is sized generously.

### Monitoring and observability service positioning

Monitoring and observability services (Amazon CloudWatch, AWS X-Ray, AWS CloudTrail, Amazon OpenSearch for logs) are regional services that operate outside the VPC. They must always be positioned:

1. Parented to `aws-cloud` — never inside VPC or any subnet group
1. Placed in Row 0 (y=30) — visually above the VPC box
1. Horizontally aligned with the compute service they monitor (same column X as ECS, Lambda, etc.)
1. Connected via dashed/async upward edge from the monitored service

This ensures the diagram correctly represents that monitoring services observe the VPC from outside, not from within. Never place CloudWatch between subnets, inside a subnet, or at the same row as VPC-bound services.

### Subnet ordering (LR flow)

In LR layouts, subnets are placed side-by-side following the data flow direction — NOT stacked vertically:

- Public subnet on the LEFT (receives traffic from external/managed services)
- Private subnet on the RIGHT (receives traffic from public subnet)

This keeps the LR flow intact: User → CloudFront → ALB (public) → ECS (private) → RDS (private) reads naturally left-to-right. Stacking subnets vertically breaks the flow direction and creates awkward diagonal connections.

```text
GOOD (LR flow):
  [Public Subnet]  →  [Private Subnet]
   ALB, NAT GW         ECS, RDS, ElastiCache

BAD (stacked):
  [Public Subnet]
   ALB, NAT GW
  [Private Subnet]
   ECS, RDS, ElastiCache
```

For TB layouts, reverse the rule: public subnet on TOP, private subnet on BOTTOM.

## VPC Coordinate Calculation (CRITICAL — most common failure)

VPC diagrams fail because of wrong relative coordinate math. Follow this exact procedure:

1. Plan ALL node positions in ABSOLUTE canvas coordinates first (as if everything were flat inside aws-cloud)
1. Decide which nodes go in which subnet
1. Calculate group positions and sizes BOTTOM-UP:
    - Subnet position = (leftmost_child_absolute_x - 20, topmost_child_absolute_y - 40)
    - Subnet size = (rightmost_child_right_edge - leftmost_child_x + 40, bottommost_child_bottom_edge - topmost_child_y + 60)
    - VPC position = (leftmost_subnet_x - 20, topmost_subnet_y - 40)
    - VPC size = (rightmost_subnet_right_edge - leftmost_subnet_x + 40, bottommost_subnet_bottom_edge - topmost_subnet_y + 60)
1. Convert node coordinates: node_relative_x = node_absolute_x - subnet_absolute_x, node_relative_y = node_absolute_y - subnet_absolute_y
1. Convert subnet coordinates: subnet_relative_x = subnet_absolute_x - vpc_absolute_x, subnet_relative_y = subnet_absolute_y - vpc_absolute_y
1. Convert VPC coordinates: vpc_relative_x = vpc_absolute_x - aws_cloud_x, vpc_relative_y = vpc_absolute_y - aws_cloud_y

VERIFY before building: every node's relative coordinates must be POSITIVE and within its parent's width/height. If any coordinate is negative or exceeds the parent size, the math is wrong — recalculate.

**Worked example** for a node at absolute (400, 200) inside a subnet at absolute (350, 160) inside a VPC at absolute (300, 120) inside aws-cloud at (200, 20):

- Node relative to subnet: (400-350, 200-160) = (50, 40) ✓
- Subnet relative to VPC: (350-300, 160-120) = (50, 40) ✓
- VPC relative to aws-cloud: (300-200, 120-20) = (100, 100) ✓

MINIMUM GROUP SIZES: Subnet min 200x200, VPC min 300x300, aws-cloud min 500x400. Always err on the side of MORE space.

MANAGED SERVICE OVERLAP CHECK (BLOCKING): After computing VPC group position and size, verify that NO managed-zone node (SQS, SNS, EventBridge, DynamoDB, S3, Step Functions, CloudWatch, Cognito, WAF, etc.) falls visually inside the VPC bounding box. If a managed node's absolute position is enclosed by the VPC rectangle, move the node outside (shift Y below VPC bottom + 40px, or shift X left of VPC left edge - 40px) or shrink the VPC. This is the most common VPC layout failure — managed services placed at the same Y level as VPC-bound nodes get accidentally enclosed when the VPC is sized generously.

## Layout Rules

- When a node feeds both a branch AND a continuation chain: keep the continuation chain on the SAME ROW, branch the secondary path to a new row.
- When a diagram has multiple independent flows converging on a shared hub: each flow occupies its own horizontal row with at least 160px vertical separation.
- When two nodes from different flows are at the same X column AND connect to the same target: offset the lower node's X by at least 60px.
- CRITICAL: Nodes inside a group use coordinates RELATIVE to the group's top-left corner. Nodes outside groups use ABSOLUTE canvas coordinates. Always plan in absolute first, then convert.

## Multi-Row Subnet Layout

- When a subnet contains nodes from multiple data flows: use a grid layout (columns = LR position, rows = flow membership).
- Shared nodes: place once, centered between the flows they serve.
- Empty grid cells stay empty — do not compress the grid.

## Connection Routing

- When target is BELOW source (Y > source Y + 40): exit bottom, entry top.
- When target is ABOVE source: exit top, entry bottom.
- When target is at SAME vertical level: exit right, entry left (LR flow).
- When a node has multiple outgoing connections: distribute exit_y values based on target Y position. Topmost exit → topmost target, bottommost exit → bottommost target.
- When multiple connections go from SAME source to SAME target: use different exit AND entry anchors (top portion vs bottom portion).
- When a node has connections going RIGHT and DOWN: right exits from center-right (1.0, 0.5), down exits from center-bottom (0.5, 1.0).

### Connection-density row assignment

Group nodes sharing 2+ mutual connections onto same row. Highest-density groups in middle rows (y=250, y=420). Sort within tier by primary connection partner's row.

### Vertical stacking

200px center-to-center: row 0 at y=30 (monitoring), row 1 at y=200, row 2 at y=400, row 3 at y=600. Sort by step number within each row.

## Spacing Rules

- Vertical: 200px center-to-center between rows (row 0 at y=30, row 1 at y=200, row 2 at y=400, row 3 at y=600)
- Horizontal: 200px center-to-center between columns (col 0 at x=50, col 1 at x=250, etc.)
- Labels: icon_size + 2px below icon, centered (label_x = icon_x - 21, label_y = icon_y + 80)
- Minimum 10px clearance between all elements
- Minimum 40px clearance between any managed node (parented to aws-cloud) and any sibling group boundary (VPC, region). This prevents nodes from visually overlapping group borders or labels
- No floating text — every label attached to parent
- >4 nodes in column: split into two sub-columns offset 100px

### Edge label overlap prevention

Edge labels sit at the midpoint of the edge. When multiple edges exit from the same node or converge on the same area, labels overlap. Prevent this with spacing:

**Increase horizontal spacing for fan-out nodes:**

When a node has 3+ outgoing edges to nodes in the same column (e.g., Lambda Orders → Step Functions + DynamoDB), increase the horizontal gap between source and targets from 200px to 250px. This gives labels more room along the edge.

**Increase vertical spacing for stacked targets:**

When two targets are stacked vertically (same X, different Y) and both receive edges from the same source, increase vertical spacing from 170px to 200px between them. This separates the edge paths and their labels.

**Short edges need shorter labels:**

When source and target are in adjacent columns (200px apart), edge labels must be ≤ 15 characters. If the label is longer, abbreviate:

| Long label | Short label |
|-----------|-------------|
| "7. Start workflow" | "7. Start" |
| "8. Save order" | "8. Save" |
| "11. Queue result" | "11. Queue" |
| "12. Trigger" | "12. Trigger" (already short) |

**Vertical edges need labels offset to the side:**

For edges that go top→bottom or bottom→top, the label sits on top of the edge line and can overlap with nearby nodes. Use `align=left` or `align=right` in the edge style to push the label to one side of the vertical line.

**Fan-out label stacking rule:**

When a node has 2 edges exiting from split ports (exitY=0.33 and exitY=0.67), the upper edge label goes above the line (`verticalAlign=bottom`) and the lower edge label goes below the line (`verticalAlign=top`). This prevents the two labels from overlapping at the midpoint.

### Overlap prevention

Before finalizing: same column `node_a.y + 120 < node_b.y` (shift 200px if overlap), adjacent columns 120px horizontal clearance. Use absolute coordinates. `fix_diagram` also resolves overlaps.

### Edge clearance from non-connected nodes

See `plan-icons-and-connections.md` "Edge path crossing check" for the authoritative algorithm. The same relaxed diagonal rule applies here: only check horizontal and vertical edges.

## Sizing Rules

- CRITICAL: Always calculate sizes bottom-up: subnet → VPC → aws-cloud → canvas. ALWAYS err on the side of MORE space.
- Node size: 120px wide x 120px tall. Horizontal spacing: 160px center-to-center. Vertical spacing: 180px center-to-center.
- Subnet padding: 80px top, 60px sides/bottom. Width: (columns x 160px) + 120px. Height: (rows x 180px) + 140px.
- VPC: sum of subnet widths + gaps + 120px padding. aws-cloud: VPC width + 160px, supporting row Y + 200px.
- Canvas: aws-cloud right edge + 60px width, aws-cloud bottom + 220px height (for legend).
- Round UP to nearest 20px.

## Absolute Coordinates

Nodes inside `aws-cloud` use local coordinates relative to group origin:

```text
absolute_x = node.x + parent.x
absolute_y = node.y + parent.y
```

## AWS Cloud Group Sizing (flat layout)

Use dimensions from Phase 2 group sizing formulas. Position: x=200, y=20.

## External Actor Positioning

User/Client icon: x=40, y=230 (page coordinates).

## Edge Routing

Engine handles routing automatically via orthogonal styles. `fix_diagram` normalizes edge styles.

## Legend Placement

- Place OUTSIDE the aws-cloud group at x=20, y=canvas_height - 140.
- Only add when 2+ distinct connection styles are used.
- Must never overlap with any group, node, or connection.

## Port-to-Anchor Conversion Table

When converting port directions to MCP anchor coordinates for `add_connection`:

| Port Direction | exit_x / entry_x | exit_y / entry_y |
|----------------|-------------------|-------------------|
| `top` | 0.5 | 0.0 |
| `bottom` | 0.5 | 1.0 |
| `left` | 0.0 | 0.5 |
| `right` | 1.0 | 0.5 |

## Style Conversion Table

When converting connection styles to MCP parameters for `add_connection`:

| Style | Visual | MCP Parameter |
|-------|--------|---------------|
| `sync` | Solid line, filled arrow | `style: "sync"` |
| `async` | Dashed line, open arrow | `style: "async"` |
| `optional` | Dashed gray line, open arrow | `style: "async"` + apply gray color via `edit_edge` |

For `optional` connections: first create the connection with `style: "async"`, then call `edit_edge` to apply gray color styling.

## Group Type Mapping

When calling `add_group`, map the group type as follows:

| Group Type | add_group `group_type` |
|------------|------------------------|
| `aws-cloud` | `aws-cloud` |
| `vpc` | `vpc` |
| `subnet` | `subnet` |
| `az` | `az` |
| `generic` | `generic` |

## Build Sequence

Execute MCP tool calls in this exact order:

1. `create_diagram` — create the diagram file with title and direction
1. If multiple pages needed: `add_page` for each additional page
1. For each group (parent groups first): `add_group` with the mapped `group_type`
1. For each node: `add_node` with label (concatenate sublabel if present: `label + "\n" + sublabel`), position (x, y), parent group, and `aws_service` key
1. For each connection: `add_connection` with mapped port anchors and style. If style is `optional`, follow with `edit_edge` to apply gray color
1. `validate_diagram` to check for structural issues
1. If validation returns errors: `fix_diagram` once
1. If 2+ distinct connection styles exist, or legend explicitly requested: `add_legend`

## Error Handling

- If `create_diagram` fails: stop immediately — the diagram cannot be built
- If an individual `add_node` or `add_connection` fails: retry once with the same parameters
- If the retry fails: log the error and continue with remaining elements
- If `validate_diagram` returns errors after `fix_diagram`: review remaining errors and apply manual fixes with `edit_node` or `edit_edge`

## fix_diagram Order

Applies: (1) model normalization, (2) style fixes, (3) shape fixes, (4) bidirectional arrow removal, (5) wrong user icon correction. Group containers never modified structurally.

All operations MUST go through MCP tools. No direct XML editing.

## Post-Build QC Checks

Run before presenting any diagram. Correct all QC violations regardless of count:

| Check | Rule | Exception |
|-------|------|-----------|
| QC-01 | All nodes have ≥1 connection | Transversal services (CloudWatch, CloudTrail, Config, IAM) |
| QC-02 | Step numbers consecutive, no gaps | None |
| QC-03 | All groups contain ≥1 node | None |
| QC-04 | No connections reference nonexistent nodes | None |
| QC-05 | Canvas dimensions contain all elements | None |
| QC-06 | No edge path crosses through a non-connected node (see algorithm below) | None |

### QC-06 Algorithm: Edge Crossing Check

For each connection (src→tgt), check if any other node's position (X, Y from the prompt file) falls between src and tgt on the edge path. Only check horizontal and vertical edges:

- **Horizontal edges** (same Y ± 10px): check if any node at the same Y has X between src.X and tgt.X
- **Vertical edges** (same X ± 10px): check if any node at the same X has Y between src.Y and tgt.Y
- **Diagonal edges**: SKIP — the orthogonal routing engine handles these with bends

If a violation is found, go back to the planning phase and reorder nodes to eliminate the crossing. After reordering, regenerate the prompt-file and rebuild.

Separate from MCP violations (`validate_diagram`). If >3 MCP violations remain after `fix_diagram`, offer the user: manual correction or full layout recalculation.

## Validation Checks

`validate_diagram` checks: duplicate cell IDs, dangling edges, overlapping nodes, missing parent groups, style violations.

## Readability Checklist

Pipeline scores readability 0-5. Ensure:

- LR flow default; TB only for vertical patterns. Steps follow reading order
- All positions snapped to 10px grid. Same-tier nodes share X; same-row share Y
- Group by logical tier (entry → compute → data → AI/ML) in adjacent columns
- Zero overlaps: bounding-box sweep before `add_connection`
- Minimize edge crossings: connected nodes on same row when possible
- Labels fit parent cells; edge labels have `labelBackgroundColor=none`

## Node Group Pattern

Engine creates each node as transparent group with `-icon` and `-label` children. Connection retargeting to `-icon` child applied automatically. No manual intervention needed.

## Promoted Layout Defaults

Applied automatically by engine at creation time.

| Property | Default | Source |
|----------|---------|--------|
| fontColor | `#232F3E` | AWS standard dark text |
| strokeColor | `#232F3E` | AWS standard border |
| labelBackgroundColor | `none` | Transparent edge label background |
| verticalAlign | `middle` | Centered label alignment |

