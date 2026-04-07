# Plan Icons and Connections

Phase 2 of the diagram workflow. Translate enriched architecture into concrete nodes, groups, and connections before positioning.

## Input

From Phase 1: complete AWS service list, external actors, connection list with types and step numbers.

## Node Planning

### Translate services to nodes

For each service:

1. Determine AWS icon using `list_aws_shapes` (filter by category)
1. Assign descriptive kebab-case node ID: `lambda-auth`, `ddb-orders`, `apigw-main`
1. Set label to official AWS service name (first line) + role sublabel (second line)
1. Set sublabel to service variant if needed: "(WebSocket)", "(On-Demand)"

### Official AWS service naming (MANDATORY)

Node labels MUST use the official AWS service name as the primary label. Use sublabel for the role/purpose. Use "Amazon" for customer-facing services, "AWS" for platform/infra services.

| aws_service key | Label (line 1) | Sublabel (line 2) | Example combined |
|-----------------|---------------|-------------------|-----------------|
| `lambda` | AWS Lambda | Orders | "AWS Lambda\nOrders" |
| `dynamodb` | Amazon DynamoDB | Orders Table | "Amazon DynamoDB\nOrders Table" |
| `s3` | Amazon S3 | Static Assets | "Amazon S3\nStatic Assets" |
| `api-gateway` | Amazon API Gateway | REST API | "Amazon API Gateway\nREST API" |
| `cloudfront` | Amazon CloudFront | CDN | "Amazon CloudFront\nCDN" |
| `cognito` | Amazon Cognito | User Pool | "Amazon Cognito\nUser Pool" |
| `sqs` | Amazon SQS | Queue | "Amazon SQS\nQueue" |
| `sns` | Amazon SNS | Alerts | "Amazon SNS\nAlerts" |
| `step-functions` | AWS Step Functions | Workflow | "AWS Step Functions\nWorkflow" |
| `eventbridge` | Amazon EventBridge | Events | "Amazon EventBridge\nEvents" |
| `cloudwatch` | Amazon CloudWatch | Metrics | "Amazon CloudWatch\nMetrics" |
| `rds` | Amazon RDS | Database | "Amazon RDS\nDatabase" |
| `ecs` | Amazon ECS | Fargate | "Amazon ECS\nFargate" |
| `elasticache` | Amazon ElastiCache | Cache | "Amazon ElastiCache\nCache" |
| `route53` | Amazon Route 53 | DNS | "Amazon Route 53\nDNS" |
| `alb` | ALB | Load Balancer | "ALB\nLoad Balancer" |
| `nat-gateway` | NAT Gateway | Outbound | "NAT Gateway\nOutbound" |
| `bedrock` | Amazon Bedrock | Runtime | "Amazon Bedrock\nRuntime" |
| `kms` | AWS KMS | Encryption | "AWS KMS\nEncryption" |
| `iam` | AWS IAM | Roles | "AWS IAM\nRoles" |
| `waf` | AWS WAF | Firewall | "AWS WAF\nFirewall" |

When multiple instances of the same service exist, differentiate via sublabel:

- `lambda` with sublabel "Orders" → "AWS Lambda\nOrders"
- `lambda` with sublabel "Processor" → "AWS Lambda\nProcessor"
- `lambda` with sublabel "Authorizer" → "AWS Lambda\nAuthorizer"

### Node ID convention

- Descriptive kebab-case: `lambda-auth`, `ddb-orders`, `apigw-main`
- Groups: `aws-cloud` (the ONLY group — no VPC/subnet sub-groups)
- External actors: `user`, `client`, `third-party-api`
- No generic IDs (`node1`, `lambda1`)

### External actors

Outside AWS Cloud group: User/Client/Browser, third-party APIs, on-premises systems, mobile devices, external services.

CRITICAL: External actors and non-AWS systems MUST use `parent_group=""` (page root). This includes:

- User/Client/Customer/Browser icons
- Third-party APIs and external services (e.g., "External API Fraud Service", "Stripe API", "Twilio")
- On-premises systems (e.g., "Corporate Data Center", "Oracle Database")
- IoT devices, mobile devices

These nodes are placed at page-level coordinates (not relative to aws-cloud). Position external actors to the LEFT of aws-cloud (entry points) or to the RIGHT of aws-cloud (external targets like third-party APIs).

External user/client/actor nodes MUST use `client_icon` style (`shape=mxgraph.cisco19.user;fillColor=#005073`). This applies to ALL external actor nodes regardless of their label — "User", "Client", "Chatbot Client", "Upload Client", "Customer", "Tenant User", "Browser", or any other label representing a human user or client application. When calling `add_node` for external actors, always pass `aws_service=""` and the node will render as a generic rounded rectangle — `fix_diagram` then converts it to the client_icon. Alternatively, the agent can set the style directly.

Validation enforces this; `fix_diagram` replaces non-compliant styles for nodes with `parent_group=""` that have labels containing "User", "Client", "Actor", "Customer", or "Browser".

Everything else inside `aws-cloud` with `parent_group="aws-cloud"`.

### External actor node rendering

External actors (nodes outside any group) must preserve their icon type. Supported external icons:

| Icon Value | Rendering |
|------------|-----------|
| `user` | Person silhouette (client/end-user actors) |
| `square` | Plain rectangle (third-party services) |

Never degrade external actors to plain text boxes. Always pass the icon value to `add_node`.

### Node label concatenation

When a node has a non-empty sublabel, concatenate: `label + "\n" + sublabel`. Pass the result to `add_node` as the label parameter.

### Multiple instances of the same service

Each `add_node` call with a unique `node_id` creates a separate node, even if the `aws_service` key matches an existing node. No workaround needed — the engine respects distinct node IDs.

When adding multiple instances, differentiate via sublabel:

- `lambda` with sublabel "Orders" → "AWS Lambda\nOrders"
- `lambda` with sublabel "Processor" → "AWS Lambda\nProcessor"
- `lambda` with sublabel "Authorizer" → "AWS Lambda\nAuthorizer"

### Sub-service normalization

Some AWS services have sub-features that share the parent service icon. When planning nodes, use the sub-service key as `aws_service` — the engine normalizes it to the parent and auto-merges into a single node with a combined label.

Do NOT create generic rounded-rect nodes for sub-services. Always pass the sub-service key so the engine resolves the correct icon and merges labels.

| Sub-service key | Parent service | Example label addition |
|-----------------|---------------|----------------------|
| `connect-did` | `connect` | DID Numbers |
| `contact-lens` | `connect` | Contact Lens |
| `connect-wisdom` | `connect` | Wisdom |
| `connect-voiceid` | `connect` | Voice ID |
| `connect-tasks` | `connect` | Tasks |
| `connect-cases` | `connect` | Cases |
| `api gateway websocket` | `api-gateway` | WebSocket API |
| `api gateway http api` | `api-gateway` | HTTP API |
| `api gateway rest api` | `api-gateway` | REST API |
| `kinesis data streams` | `kinesis` | Data Streams |
| `kinesis data firehose` | `kinesis` | Data Firehose |
| `cloudwatch logs` | `cloudwatch` | Logs |
| `cloudwatch alarms` | `cloudwatch` | Alarms |
| `cognito user pool` | `cognito` | User Pool |
| `cognito identity pool` | `cognito` | Identity Pool |
| `bedrock runtime` | `bedrock` | Runtime |
| `bedrock agent` | `bedrock` | Agent |
| `bedrock model` | `bedrock` | Model |

When multiple sub-services of the same parent appear in the architecture, each `add_node` call with a different sub-service key auto-merges into one node. The label accumulates: "Amazon Connect\nDID Numbers, Contact Lens".

### Node ordering for simple connections

**Core layout principle: place connected nodes adjacent to each other so edges are short, straight lines.** Every connection should ideally be a single horizontal or vertical segment between neighboring nodes — no diagonals, no multi-bend paths, no edges that cross half the canvas.

**How to achieve this:**

1. Connected nodes share the same row (horizontal edge) or same column (vertical edge)
1. If A connects to B, place B in the next column (same row) or next row (same column) — never 2+ columns/rows away if avoidable
1. When a node connects to two targets, one goes right (same row) and one goes below (same column) — both are adjacent
1. The entire diagram reads left-to-right: entry points on the left, compute in the middle, data/backends on the right

**Think of it as a grid:** each node occupies one cell. Connected nodes should be in adjacent cells (horizontally or vertically). If you draw a line between any two connected nodes, it should pass through zero other nodes.

Order the node list so that connections flow naturally without crossings. Phase 3 places nodes in the order they appear in the list, so getting the order right here means shorter edges and fewer bends.

**Ordering rules:**

1. Walk the primary data flow path from entry to exit (e.g., User → Route 53 → CloudFront → ALB → ECS → RDS)
1. Assign each node a `flow_position` (0, 1, 2, ...) along this path
1. Nodes on the same flow step share the same position (fan-out targets get the same position)
1. Branch/secondary flows get positions after the primary path
1. Monitoring/observability services (CloudWatch, X-Ray) go last — they connect via async edges that tolerate longer routes

**Within the same flow position, stack by connection direction:**

- Nodes connected via `right → left` go in the same row (horizontal neighbors)
- Nodes connected via `bottom → top` go in the same column (vertical neighbors)
- Nodes connected via `left → right` (reverse flow, e.g., NAT Gateway outbound) go below their source

**Fan-out placement rule (CRITICAL):**

When a node has 2+ outgoing edges to different targets:

- Primary target (main flow): same row, next column to the right
- Secondary target (data store, side effect): directly BELOW the source node (same column, next row)
- Never place both targets at the same row AND different columns — the edge to the farther target will cross through the nearer one

```text
GOOD: Lambda Orders (col 3, row 2) → Step Fn (col 4, row 2)  [right, same row]
      Lambda Orders (col 3, row 2) → DDB Orders (col 3, row 3) [down, same column]

BAD:  Lambda Orders (col 3, row 2) → Step Fn (col 4, row 2)  [right]
      Lambda Orders (col 3, row 2) → DDB Orders (col 4, row 3) [diagonal through Step Fn area]
```

**Triple fan-out rule (3+ outgoing edges):**

When a node has 3 outgoing edges (e.g., Lambda Orders → Step Fn + DDB Orders + CloudWatch):

- Primary target (main flow): same row, next column to the right → exit right port
- Secondary target (data store): directly BELOW source, same column → exit bottom port
- Tertiary target (monitoring/async): directly ABOVE source, same column → exit top port

The tertiary target goes ABOVE (row 0 or a dedicated monitoring row) to avoid the vertical path being blocked by the secondary target below.

```text
GOOD: CloudWatch (col 3, row 0)     ← Lambda Orders top port
      Lambda Orders (col 3, row 1)   → Step Fn (col 4, row 1)  [right]
      DDB Orders (col 3, row 2)      ← Lambda Orders bottom port

BAD:  Lambda Orders (col 3, row 1)   → Step Fn (col 4, row 1)  [right]
      DDB Orders (col 3, row 2)      ← Lambda Orders bottom port
      CloudWatch (col 3, row 3)      ← Lambda Orders bottom port
      ↑ DDB Orders blocks the vertical path to CloudWatch!
```

**Monitoring row (row 0) pattern:**

When a compute node sends async metrics to CloudWatch/X-Ray, place the monitoring service directly ABOVE the source in a dedicated "row 0" (y=30 inside aws-cloud). This keeps the monitoring edge as a clean vertical line going UP, avoiding all fan-out targets below.

```text
Row 0 (y=30):   CloudWatch (monitoring, above source)
Row 1 (y=200):  Main flow (CloudFront → API GW → Lambda → Step Fn → ...)
Row 2 (y=400):  Fan-out targets (S3, Cognito, DDB, EventBridge, ...)
Row 3 (y=600):  Tertiary (Lambda Auth, S3 Archive, SNS, ...)
```

**Reverse-flow chain placement:**

When a chain of nodes flows right-to-left (e.g., response path back to client, NAT Gateway outbound), place them on the same row with NO other nodes between them.

**Fan-out inside VPC subnets:**

The same fan-out rules apply inside subnet groups. When a compute node connects to multiple data stores within the same subnet:

- Primary data store: same row, next column to the right → exit right port
- Secondary data store: directly BELOW the primary data store (same column as primary) → exit bottom port from primary, or exit right+down from compute

```text
GOOD (inside private subnet):
  ECS (col 1, row 1) → RDS (col 2, row 1)       [right, main query]
                        ElastiCache (col 2, row 2) [below RDS, cache]

BAD (inside private subnet):
  ECS (col 1, row 1) → RDS (col 2, row 1)       [right]
  ECS (col 1, row 1) → ElastiCache (col 3, row 1) [right, stretches subnet too wide]
```

This keeps the subnet compact and avoids unnecessarily wide layouts. The subnet group wraps tightly around a 2-row grid instead of a single wide row.

**Forward vertical chain placement:**

When a chain of nodes flows top-to-bottom in the same column (e.g., SQS → Lambda Notify → SNS), place them in consecutive rows with NO other nodes in the same column between them.

```text
GOOD: SQS (col 6, row 1)
      Lambda Notify (col 6, row 2)
      SNS (col 6, row 3)
      All vertical edges, same column, no blockers
```

**Example — event-driven serverless (difficult benchmark):**

```text
flow_position 0: user (external)
flow_position 1: cloudfront (managed, entry)
flow_position 1: s3-assets (managed, below cloudfront — affinity)
flow_position 2: apigw (managed, entry)
flow_position 2: cognito (managed, below apigw — auth affinity)
flow_position 2: lambda-auth (managed, below cognito — auth chain)
flow_position 3: lambda-orders (managed, main compute)
flow_position 3: ddb-orders (managed, below lambda-orders — fan-out)
flow_position 3: cloudwatch (managed, ABOVE lambda-orders — monitoring row 0)
flow_position 4: step-fn (managed, workflow)
flow_position 4: eventbridge (managed, below step-fn — fan-out)
flow_position 4: s3-archive (managed, below eventbridge — archive chain)
flow_position 5: lambda-proc (managed, processor)
flow_position 5: ddb-inventory (managed, below lambda-proc — fan-out)
flow_position 6: sqs (managed, queue)
flow_position 6: lambda-notify (managed, below sqs — trigger chain)
flow_position 6: sns (managed, below lambda-notify — alert chain)
```

**Output the node list in flow_position order.** Phase 3 uses this order for column/row assignment.

## Icon Rules

- When the prompt mentions "IoT devices", "IoT sensors", or similar: use `aws_service: "iot-core"` (NOT the User icon).
- When the prompt mentions "Knowledge Bases for Amazon Bedrock" or "Bedrock KB": use `aws_service: "bedrock"`.
- When a service appears multiple times with different roles: use the same `aws_service` for both, differentiate via sublabel.
- When unsure which `aws_service` key to use: call `list_aws_shapes` directly to verify icon keys exist.

## Connection Planning

### Reference reproduction

Preserve ALL edge labels (step numbers, protocols, descriptions, conditions). Reproduce exact numbering. Protocol-only labels used as-is.

### Determine connections

For each data flow:

1. Identify source and target node IDs
1. Classify type:
    - Sync (solid): HTTP request-response, SDK calls, direct invocations
    - Async (dashed): event triggers, queue messages, stream processing, SNS notifications
1. Assign step number following primary data flow order
1. Write short label: "REST API", "WebSocket", "Query items", "Invoke model"

### Connection type rules

| Pattern | Type |
|---------|------|
| API Gateway → Lambda | Sync |
| Lambda → DynamoDB (CRUD) | Sync |
| Lambda → Lambda (direct invoke) | Sync |
| Lambda → Bedrock (invoke model) | Sync |
| Lambda → SQS (send message) | Async |
| SQS → Lambda (trigger) | Async |
| SNS → Lambda (notification) | Async |
| EventBridge → Lambda (rule) | Async |
| DynamoDB Streams → Lambda | Async |
| S3 event → Lambda | Async |
| CloudFront → S3 (origin) | Sync |
| API Gateway → Step Functions | Async |

### Connection sub-cell targeting

Each AWS node is a group with two child cells: `{node-id}-icon` (top) and `{node-id}-label` (bottom). The engine's `resolveEdgeEndpoint()` automatically targets the correct sub-cell based on port direction.

In the prompt file, connections reference parent node IDs only — never sub-cell IDs. The engine resolves sub-cells at build time.

| Port Direction | Target Sub-Cell | Reason |
|----------------|-----------------|--------|
| `top` | `{node-id}-icon` | Icon sits at top of container |
| `left` | `{node-id}-icon` | Icon is the visual anchor |
| `right` | `{node-id}-icon` | Icon is the visual anchor |
| `bottom` | `{node-id}-label` | Label sits at bottom of container |

Apply to both source (exit-port) and target (entry-port) independently. For example, a connection with exit-port `bottom` and entry-port `top` uses `{source}-label` → `{target}-icon`.

### Default anchor ports

Choose exit/entry ports based on the **actual visual position** of source and target after layout. Because connected nodes are always adjacent (same row or same column), ports are simple:

**Core principle: architecture reads left-to-right, ALWAYS.** Primary flow enters from the left and exits to the right. Secondary/vertical flows use top/bottom ports.

**Adjacent nodes = simple ports:**

| Layout relationship | Exit port | Entry port | Sub-cell |
|--------------------|-----------|------------|----------|
| Same row, target to the right | right (1.0, 0.5) | left (0.0, 0.5) | both `-icon` |
| Same row, target to the left (reverse) | left (0.0, 0.5) | right (1.0, 0.5) | both `-icon` |
| Same column, target below | bottom (0.5, 1.0) | top (0.5, 0.0) | exit `-label`, enter `-icon` |
| Same column, target above | top (0.5, 0.0) | bottom (0.5, 1.0) | exit `-icon`, enter `-label` |

**Multiple edges from same node (fan-out):**

When a node has 3+ outgoing edges, each target is adjacent in a different direction:

- Primary flow (right): right port (1.0, 0.5) → target `-icon` on left
- Secondary flow (below): bottom port (0.5, 1.0) → target `-icon` on top. Exit from `-label` cell
- Tertiary flow (above, monitoring): top port (0.5, 0.0) → target `-label` on bottom. Exit from `-icon` cell

Because each target is adjacent (same row or same column), every edge is a single straight segment. No diagonals needed.

**Never omit anchors.** Always specify exitX/exitY/entryX/entryY — the engine produces cleaner routes with explicit anchors than with auto-routing.

### Step numbering

- Primary user flow left-to-right gets lowest numbers
- Branch flows get sub-steps or continue sequence
- Return paths (responses) don't get separate numbers
- Bidirectional connections get single number

### Edge label brevity

Keep edge labels short — max 15 characters for edges between adjacent nodes (200px apart). Long labels overlap with nearby nodes and other labels.

Format: `"N. verb"` — step number + single verb or short phrase.

| Good | Bad |
|------|-----|
| "7. Start" | "7. Start workflow execution" |
| "8. Save" | "8. Save order to database" |
| "11. Queue" | "11. Queue processing result" |
| "6. Invoke" | "6. Invoke Lambda function" |

### Fan-out and fan-in

Identify patterns early (fan-out, fan-in, hub, chain) — affects Phase 3 layout.

## Group Planning

### AWS Cloud group

Every diagram needs an `aws-cloud` group (`group_type="aws-cloud"`) rendering the AWS Cloud boundary with official logo. Mandatory — never omit or substitute.

- All AWS nodes parented to `aws-cloud` with flat coordinates
- External actors parented to page root (`parent_group=""`)
- Position: x=200, y=20

### Group sizing formulas

Compute group dimensions from child node count instead of guessing. Node size = 120x120 (icon group).

**AWS Cloud (flat layout, no VPC):**

```text
max_flow_position = max(node.flow_position for all nodes)
cols = max_flow_position + 1
max_rows = max(count of nodes at same flow_position)
width  = cols * 250 + 80          # 250px per column + 80px padding
height = max_rows * 170 + 80      # 170px per row + 80px padding
```

### VPC/subnet groups (two-pass approach)

When the architecture has VPC-bound services, use actual draw.io groups (VPC, subnet) with proper parenting. But plan the flat grid FIRST, then wrap groups around nodes. See `layout-and-routing.md` "VPC/Subnet Visual Annotations" for the two-pass approach and coordinate conversion formulas.

- Plan all positions on the flat grid (as if all nodes were in `aws-cloud`)
- Then compute group sizes to wrap around VPC-bound nodes
- Convert coordinates from aws-cloud-relative to group-relative
- Managed services stay `parent_group="aws-cloud"`, VPC-bound nodes get `parent_group` = their subnet

### Spatial affinity hints

When planning the node list, annotate services that should be placed near each other in Phase 3. The goal: every connection is a single straight segment (horizontal or vertical) between adjacent nodes.

| Service A | Service B | Connection | Placement (simple edge) |
|-----------|-----------|-----------|------------------------|
| Route 53 | CloudFront | DNS resolve | Same row, Route 53 left of CloudFront → horizontal edge |
| CloudFront | S3 (static assets) | Origin fetch | Same row, S3 right of CloudFront → horizontal edge. If CloudFront also connects down to ALB, put S3 below instead → vertical edge |
| CloudFront | ALB/API GW | Forward request | Same row, ALB right of CloudFront → horizontal edge |
| API Gateway | Cognito | Auth check | Same column, Cognito below API GW → vertical edge |
| Cognito | Lambda Auth | Authorize | Same column, Lambda Auth below Cognito → vertical edge |
| Lambda Auth | AWS STS | AssumeRole | Same column, STS DIRECTLY below Lambda Auth → vertical edge. exit_x=0.5, exit_y=1.0 (bottom center), entry_x=0.5, entry_y=0.0 (top center). NEVER exit from left or right port — always straight down |
| API Gateway | Lambda Auth | Validate JWT | Same column, Lambda Auth below API GW → vertical edge. exit_x=0.5, exit_y=1.0, entry_x=0.5, entry_y=0.0 |
| API Gateway | Lambda | Invoke | Same row, Lambda right of API GW → horizontal edge |
| Lambda | DynamoDB | CRUD | Same column, DynamoDB below Lambda → vertical edge |
| Lambda | Step Functions | Start workflow | Same row, Step Fn right of Lambda → horizontal edge |
| Lambda | CloudWatch | Metrics (async) | Same column, CloudWatch ABOVE Lambda (row 0) → vertical edge up |
| SQS | Lambda (consumer) | Trigger | Same column, Lambda below SQS → vertical edge |
| Lambda | SNS | Publish | Same column, SNS below Lambda → vertical edge |
| EventBridge | S3 (archive) | Archive | Same column, S3 below EventBridge → vertical edge |
| CloudFront | WAF | Firewall | Same column, WAF above CloudFront → vertical edge |
| NAT Gateway | VPC compute | Outbound | Same row or adjacent, NAT GW left of compute → horizontal edge |
| QuickSight | Multiple sources (OpenSearch, Athena) | Dashboard fan-in | QuickSight is a fan-in aggregator — place BELOW the rows it reads from, centered between its sources. NEVER on the main flow row when it receives from 2+ rows |
| Athena | S3 (query results) | Write results | Same row, S3 right of Athena → horizontal edge. Athena's output bucket is part of the batch chain, not a dashboard |
| Bedrock KB | OpenSearch + Bedrock FM | RAG fan-out | KB fans out to OpenSearch (search, same row RIGHT) and Bedrock FM (generate, BELOW KB offset RIGHT). Place FM at the same X as OpenSearch or between KB and OpenSearch X, one row below. This avoids FM interfering with ingestion flows that connect to KB from below/left |

## Layout Validation (before building)

After planning all nodes, connections, and positions, run these checks BEFORE calling any MCP tools. Fix violations by adjusting node positions.

### Edge path crossing check

For every connection (source → target), check if any non-connected node blocks the edge path. Only check horizontal and vertical edges — diagonal edges are handled by the orthogonal routing engine.

**Check algorithm:**

```text
for each connection (src, tgt):
  for each node N (where N ≠ src and N ≠ tgt):
    if edge is horizontal (same Y ± 10px):
      VIOLATION if N.y == src.y (± 10px) AND N.x is between src.x and tgt.x
    elif edge is vertical (same X ± 10px):
      VIOLATION if N.x == src.x (± 10px) AND N.y is between src.y and tgt.y
    else:
      SKIP — diagonal edges are routed by the engine with bends
```

**Fix strategy:** Move the blocking node to a different row (shift Y by ±200px). If that creates a new violation, increase column spacing instead.

### Fan-out row separation

When a node has 2+ outgoing edges using split ports (exitY=0.33 and exitY=0.67), the targets MUST be on different rows.

### Node size constraint

Every AWS service node group is exactly 120x120 pixels (78px icon + 2px gap + 40px label). Never create larger node groups. The engine calculates this automatically — do not override with manual sizes.

## Strict Rules

- All operations through MCP tools — never edit XML directly or manually construct XML cells
- When creating multiple nodes of the same AWS service, each `add_node` with a unique `node_id` creates a separate node — no workaround needed
- NEVER add legend, summary box, or metadata rectangle
- AWS naming: use official service names per "Official AWS service naming" table. First line = official name, sublabel = role/purpose
- Sublabels for variants: `sublabel="(WebSocket)"`, `sublabel="Connections"`
- Plan complete step sequence BEFORE calling `add_connection`. Create in step-number order
- Labels matching `"N. description"` placed inline with transparent background at edge midpoint
- Ordering: happy-path first, main interaction middle, cleanup/teardown last

## Confirm Before Building

Present complete plan (nodes, connections, groups, pattern) and ask: "Anything to add or change before I build?" Do not proceed until user confirms.

**Before presenting the plan, you MUST have run all checks in "Layout Validation" above.** If any violation was found, fix it first, then present the corrected plan.

## Worked Example: Event-Driven Serverless (17 nodes)

Reference layout for the difficult benchmark:

| Node | Col | Row | flow_pos | Notes |
|------|-----|-----|----------|-------|
| user | ext | 1 | 0 | External actor |
| cloudfront | 1 | 1 | 1 | Entry |
| s3-assets | 1 | 2 | 1 | Below cloudfront (affinity) |
| apigw | 2 | 1 | 2 | Entry |
| cognito | 2 | 2 | 2 | Below apigw (auth affinity) |
| lambda-auth | 2 | 3 | 2 | Below cognito (auth chain) |
| lambda-orders | 3 | 1 | 3 | Main compute |
| ddb-orders | 3 | 2 | 3 | Below lambda-orders (fan-out) |
| cloudwatch | 3 | 0 | 3 | ABOVE lambda-orders (monitoring row) |
| step-fn | 4 | 1 | 4 | Workflow |
| eventbridge | 4 | 2 | 4 | Below step-fn (fan-out) |
| s3-archive | 4 | 3 | 4 | Below eventbridge (archive chain) |
| lambda-proc | 5 | 1 | 5 | Processor |
| ddb-inventory | 5 | 2 | 5 | Below lambda-proc (fan-out) |
| sqs | 6 | 1 | 6 | Queue |
| lambda-notify | 6 | 2 | 6 | Below sqs (trigger chain) |
| sns | 6 | 3 | 6 | Below lambda-notify (alert chain) |

Key: CloudWatch at row 0 avoids blocking vertical edge through DDB Orders. Auth chain and SQS→SNS chain are vertical in their columns. All labels use official names.

## Output

- Node list (in flow_position order): node_id, label (official name), sublabel (role), aws_service, parent_group, flow_position
- Connection list: source_id, target_id, label, style, step_number, exitX, exitY, entryX, entryY
- Group list with computed dimensions: group_id, label, group_type, width, height
- Spatial affinity hints: list of (node_a, node_b, relative_position) tuples
- Pattern classification: chain, fan-out, fan-in, hub, mixed

Feeds directly into Phase 3 (layout and routing).

