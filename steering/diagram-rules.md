# Diagram Rules

Phase 1 rules for AWS architecture diagrams. Covers pattern recognition, service placement, spacing, auth chains, monitoring suppression, and enrichment. Load this file when starting any new diagram or modifying an existing one.

## Service Placement Classification

For every service extracted from the user's request, classify as VPC-bound or managed/global using the tables below. This classification controls `parent_group` assignment during layout.

### VPC-bound services (zone: vpc)

Services that require ENI or subnet placement:

| Service | Typical subnet |
|---------|---------------|
| Amazon EC2, Amazon ECS, Amazon EKS, AWS Fargate tasks | public or private |
| Amazon RDS, Amazon Aurora, Amazon ElastiCache, Amazon Redshift | private |
| ALB, NLB (internet-facing) | public |
| NAT Gateway, VPC Endpoint | public |
| AWS Lambda (VPC-attached) | private |

### Managed/global services (zone: managed)

Services that operate without VPC attachment:

| Service | Scope |
|---------|-------|
| Amazon Route 53 | Global DNS |
| Amazon CloudFront | Global CDN edge |
| Amazon S3 | Regional managed storage |
| Amazon CloudWatch | Regional managed monitoring |
| Amazon Cognito | Regional managed auth |
| AWS WAF | Edge/regional firewall |
| AWS IAM | Global identity |
| AWS Step Functions | Regional managed orchestration |
| Amazon SQS, Amazon SNS | Regional managed messaging |
| Amazon EventBridge | Regional managed event bus |
| Amazon DynamoDB | Regional managed NoSQL |
| Amazon API Gateway | Regional managed API |
| AWS Secrets Manager, AWS Systems Manager | Regional managed config |
| AWS KMS | Regional managed encryption |
| AWS CloudFormation | Regional managed IaC |
| Amazon Bedrock, Amazon SageMaker (endpoints) | Regional managed AI/ML |

### Decision rule

If the service requires an ENI or subnet to operate → `vpc-bound`. Otherwise → `managed`.

- Managed services MUST be parented to `aws-cloud`, never to a VPC or subnet group
- During layout (Phase 3), managed nodes must NOT fall visually inside the VPC bounding box

### Subnet ordering for layout

In LR flow, public subnet nodes go to the LEFT of private subnet nodes (traffic flows public → private). This ensures subnets are placed side-by-side, not stacked vertically.

## Service Consolidation (HIGH PRIORITY)

- CRITICAL: When the prompt lists examples in parentheses (e.g., "services (e.g., Service A, Service B)"), create a SINGLE node with the examples in the sublabel, NOT separate nodes for each example.
- Only create separate nodes when the prompt explicitly says they are distinct services with different connections or data flows.

## AWS Service Placement

- CRITICAL: ALL AWS managed services MUST be placed INSIDE the aws-cloud group, even when external users connect to them directly. This includes Amazon Cognito, Amazon CloudFront, AWS WAF, Amazon Route 53, Amazon API Gateway, and any other AWS service.
- Only external actors (User, Customer, Corporate Data Center, IoT Devices) and non-AWS systems (External API, Third-party Service) go OUTSIDE the aws-cloud group.
- Authentication services (Amazon Cognito, AWS IAM Identity Center) should be placed ABOVE the main flow row when they are side services. The User authenticates UP to Cognito, then sends requests RIGHT through the main chain.
- CRITICAL: Inline managed services in the traffic path (AWS WAF, Amazon API Gateway, Amazon CloudFront, Amazon Route 53) MUST be placed on the main flow row, to the LEFT of the VPC boundary. They occupy their own column(s) before the VPC starts. NEVER place inline managed services above or below the VPC — they are part of the horizontal LR flow.

## Auth, Authorizer, and Credential Services

- CRITICAL: Authentication services (Amazon Cognito), credential services (AWS STS, AWS IAM), and Lambda Authorizers are NOT inline in the main traffic path. They are side services called by traffic-path services.
- Place Lambda Authorizers DIRECTLY BELOW Amazon API Gateway (same X column). The connection goes straight DOWN. Use exit_x=0.5, exit_y=1.0 on API Gateway and entry_x=0.5, entry_y=0.0 on the Authorizer.
- If the Authorizer calls other services (e.g., AWS STS for AssumeRole), place them DIRECTLY BELOW the Authorizer in the SAME X column. The vertical chain must use the EXACT same X coordinate: API Gateway X = Authorizer X = STS X. The connection from Authorizer to STS goes straight DOWN: exit_x=0.5, exit_y=1.0 on Authorizer, entry_x=0.5, entry_y=0.0 on STS. Example: if API Gateway is at x=250, then Authorizer is at x=250 y=400, and STS is at x=250 y=600. NEVER offset STS to a different X column.
- The main traffic flow continues horizontally from API Gateway to the next service, skipping the Authorizer. The Authorizer is a vertical side branch.
- NEVER place Cognito, Lambda Authorizer, or STS between two services in the main LR flow.

## Fan-Out, Fan-In, and Hub Patterns (HIGH PRIORITY)

These patterns cause the most layout failures. Identify them DURING PLANNING (Phase B step 1) before assigning any coordinates.

### Pattern Recognition (BLOCKING — check every prompt)

Before planning layout, scan the prompt for these triggers:

- "parallel branches" / "orchestrates N tasks" / "fans out to N" → Fan-out with convergence (Pattern 1)
- A service that appears as a TARGET in 2+ different flows → Hub node (Pattern 2)
- Two independent traffic sources (e.g., "on-prem" AND "external user") reaching the same service → Independent convergence (Pattern 3)
- N sources "feed into" / "aggregate to" / "send findings to" one service → Fan-in (Pattern 4)

### Pattern 1 — Fan-out with convergence

When a node orchestrates parallel branches that reconverge:

- CRITICAL: The hub node fans OUT to N branch targets. ALL branch targets MUST be in the SAME X column, stacked vertically. NEVER place any branch target on the main flow row inline.
- Each branch may have its own chain (e.g., Lambda → DynamoDB) extending rightward from the branch target.
- CRITICAL: After branches complete, a CONVERGENCE connection MUST go from the hub node to the post-convergence node. This connection exits the hub from the BOTTOM (exit_y=1.0) and enters the post-convergence node from the TOP (entry_y=0.0).
- The post-convergence node sits BELOW the lowest branch target, in the SAME X column as the hub or offset slightly to the right. NEVER place the post-convergence node on the main flow row or to the right of any branch chain — it must be visually separated from the branches by being on a lower row.
- CRITICAL: The convergence connection from the hub to the post-convergence node must NOT cross through any branch node. If branch nodes are stacked vertically below the hub, offset the post-convergence node's X to the LEFT of the branch column so the vertical connection has a clear path. Use waypoints if needed to route around branch nodes.
- Example: Step Functions fans out to 3 Lambdas (stacked at same X), each Lambda has its own DynamoDB. Step Functions also connects DOWN to Lambda Fulfill (post-convergence). Lambda Fulfill is BELOW the third branch, offset LEFT so the connection from Step Functions doesn't cross through the branch Lambdas.

### Pattern 2 — Hub node (NEVER bypass)

When a central service appears in multiple flows:

- CRITICAL: ALL flows MUST connect TO the hub node. NEVER bypass the hub by connecting a source directly to the hub's downstream target.
- Even if a connection from hub → downstream already exists from another flow, each upstream flow MUST still connect to the hub. The hub → downstream connection is shared.
- Example: If Query Lambda → Bedrock KB → OpenSearch AND Ingest Lambda → Bedrock KB → OpenSearch, you need: Query Lambda → Bedrock KB, Ingest Lambda → Bedrock KB, Bedrock KB → OpenSearch (ONE shared connection). You MUST NOT create Ingest Lambda → OpenSearch directly.
- Place the hub node at the center of the diagram (vertically between the flows that use it). Flows above connect DOWN to the hub, flows below connect UP to the hub.

### Pattern 3 — Independent path convergence (NEVER chain)

When two independent traffic sources reach the same target:

- CRITICAL: Each source has its OWN path to the shared target. NEVER chain one source's path through another source's infrastructure.
- The shared target receives connections from BOTH paths independently.
- Place the two paths on DIFFERENT rows, converging at the shared target's column.
- Example: "On-prem → Direct Connect → Internal ALB → EC2" AND "External User → Public ALB → EC2". These are TWO separate paths. Public ALB connects to EC2 directly. Public ALB does NOT connect to Internal ALB. Both ALBs independently connect to EC2.

### Pattern 4 — Fan-in

When multiple sources feed into one aggregator:

- Stack sources vertically in the SAME X column.
- Use distributed entry anchors on the target (entry_y spread across 0.2 to 0.8).
- The aggregator sits to the RIGHT of the sources.
- Example: GuardDuty, Inspector, Config (stacked vertically) all connect to Security Hub (to their right).
- CRITICAL: When a fan-in aggregator (e.g., QuickSight, Security Hub) receives from sources on DIFFERENT rows (e.g., a real-time row and a batch row), place the aggregator BELOW both rows or between them — NEVER on the main flow row. The main flow row is for the primary data chain, not for cross-flow aggregators.

## Supporting and Transversal Services (HIGH PRIORITY)

- CRITICAL: ALL supporting/transversal services MUST be placed in a single horizontal row at the BOTTOM of the diagram, below ALL other services and groups.
- NEVER place supporting services to the RIGHT of the last data flow column. They go BELOW, not to the right.
- Use the same Y coordinate for all supporting services in the bottom row.
- NEVER place supporting services on the same row as any data flow service.
- For diagrams with 8+ nodes, the following transversal services must be standalone nodes with NO connections: Amazon ECR, Amazon CloudWatch, AWS Secrets Manager, AWS Systems Manager, AWS KMS, AWS Config, AWS CloudTrail, Amazon Inspector, AWS X-Ray.
- These services should have descriptive sublabels explaining their role (e.g., "Container Images", "Monitoring", "Encryption Keys").
- For diagrams with fewer than 8 nodes: connections to supporting services are acceptable.

## Monitoring (CloudWatch) Connections

- For 8+ nodes: NEVER draw connections to Amazon CloudWatch, Amazon Route 53 health checks, or other monitoring/observability services. Place as standalone with descriptive sublabel. This rule applies even when the user prompt explicitly requests monitoring connections — the diagram becomes too cluttered with 8+ nodes. The prompt's connection style specification (e.g., "optional for CloudWatch monitoring") should be interpreted as "show the service but don't connect it."
- For fewer than 8 nodes: 1-2 optional (dotted) connections are acceptable.

## Node and Connection Spacing

- CRITICAL: Horizontal minimum: 160px center-to-center. Vertical minimum: 160px center-to-center.
- CRITICAL: Minimum 40px clearance between any managed node (parented to aws-cloud) and any sibling group boundary (VPC, subnet, region). This prevents nodes from visually overlapping group borders or labels.
- When a connection has a label: add 40px (200px minimum center-to-center).
- When TWO connections exist between the same pair of nodes (bidirectional): add 80px (240px minimum).
- When stacking nodes vertically (fan-in, parallel services): at least 140px vertical spacing between node centers.
- After planning all node positions, verify EVERY pair of connected nodes meets the minimum spacing.
- For bidirectional connections: model as a SINGLE connection with a combined label (e.g., "Authenticate / JWT") unless the prompt explicitly requires two separate connections.

## General Rules

- Never edit draw.io XML directly or with scripts.
- Generate a `prompt-<name>.md` file alongside every diagram for regeneration.
- CRITICAL: Only add connections explicitly described in the user prompt. Best-practice suggestions should be mentioned in validation warnings, NOT added to the diagram.
- CRITICAL: In LR flow diagrams, ALL services in the main traffic/data path must be placed left-to-right on the same horizontal row.
- CRITICAL: ALL connections in LR flow diagrams must go left-to-right (source X < target X).
- CRITICAL: External actors (User, Customer, Client) must ALWAYS use the label "User" or "Customer" — never add extra words.
- Hub-and-spoke with fan-in from below: place the most-connected target ABOVE the hub, less-connected target BELOW.
- After building, always call `validate_diagram` then `fix_diagram`. The agent can review and adjust with `edit_node`.

## Icon Rules

- When the prompt mentions "IoT devices", "IoT sensors", or similar: use `aws_service: "iot-core"` (NOT the User icon).
- When the prompt mentions "Knowledge Bases for Amazon Bedrock" or "Bedrock KB": use `aws_service: "bedrock"`.
- When a service appears multiple times with different roles: use the same `aws_service` for both, differentiate via sublabel.
- When unsure which `aws_service` key to use: call `list_aws_shapes` to find the correct key.

## Layout Rules

- When a node feeds both a branch AND a continuation chain: keep the continuation chain on the SAME ROW, branch the secondary path to a new row.
- When a diagram has multiple independent flows converging on a shared hub: each flow occupies its own horizontal row with at least 160px vertical separation.
- When two nodes from different flows are at the same X column AND connect to the same target: offset the lower node's X by at least 60px.
- CRITICAL: Nodes inside a group use coordinates RELATIVE to the group's top-left corner. Nodes outside groups use ABSOLUTE canvas coordinates. Always plan in absolute first, then convert.

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

Example for a node at absolute (400, 200) inside a subnet at absolute (350, 160) inside a VPC at absolute (300, 120) inside aws-cloud at (200, 20):

- Node relative to subnet: (400-350, 200-160) = (50, 40) ✓
- Subnet relative to VPC: (350-300, 160-120) = (50, 40) ✓
- VPC relative to aws-cloud: (300-200, 120-20) = (100, 100) ✓

MINIMUM GROUP SIZES: Subnet min 200x200, VPC min 300x300, aws-cloud min 500x400. Always err on the side of MORE space.

MANAGED SERVICE OVERLAP CHECK (BLOCKING): After computing VPC group position and size, verify that NO managed-zone node (Amazon SQS, Amazon SNS, Amazon EventBridge, Amazon DynamoDB, Amazon S3, AWS Step Functions, Amazon CloudWatch, Amazon Cognito, AWS WAF, etc.) falls visually inside the VPC bounding box. If a managed node's absolute position is enclosed by the VPC rectangle, move the node outside (shift Y below VPC bottom + 40px, or shift X left of VPC left edge - 40px) or shrink the VPC. This is the most common VPC layout failure — managed services placed at the same Y level as VPC-bound nodes get accidentally enclosed when the VPC is sized generously.

## Connection Routing

- When target is BELOW source (Y > source Y + 40): exit bottom, entry top.
- When target is ABOVE source: exit top, entry bottom.
- When target is at SAME vertical level: exit right, entry left (LR flow).
- When a node has multiple outgoing connections: distribute exit_y values based on target Y position. Topmost exit → topmost target, bottommost exit → bottommost target.
- When multiple connections go from SAME source to SAME target: use different exit AND entry anchors (top portion vs bottom portion).
- When a node has connections going RIGHT and DOWN: right exits from center-right (1.0, 0.5), down exits from center-bottom (0.5, 1.0).

## Sizing Rules

- CRITICAL: Always calculate sizes bottom-up: subnet → VPC → aws-cloud → canvas. ALWAYS err on the side of MORE space.
- Node size: 120px wide x 120px tall. Horizontal spacing: 160px center-to-center. Vertical spacing: 180px center-to-center.
- Subnet padding: 80px top, 60px sides/bottom. Width: (columns x 160px) + 120px. Height: (rows x 180px) + 140px.
- VPC: sum of subnet widths + gaps + 120px padding. aws-cloud: VPC width + 160px, supporting row Y + 200px.
- Canvas: aws-cloud right edge + 60px width, aws-cloud bottom + 220px height (for legend).
- Round UP to nearest 20px.

## Multi-Row Subnet Layout

- When a subnet contains nodes from multiple data flows: use a grid layout (columns = LR position, rows = flow membership).
- Shared nodes: place once, centered between the flows they serve.
- Empty grid cells stay empty — do not compress the grid.

## Networking Boundary Services

- AWS Direct Connect, AWS Transit Gateway, NAT Gateway, Internet Gateway: inside aws-cloud but OUTSIDE VPC.
- In hybrid cloud: AWS Direct Connect bridges on-prem and VPC, at the VPC boundary edge.

## Multi-Region Diagrams

- Use `group_type="region"` for each region, placed INSIDE aws-cloud.
- Global services (Amazon Route 53, Amazon CloudFront, AWS WAF, AWS Shield, AWS IAM): OUTSIDE region groups but INSIDE aws-cloud.
- For active-passive DR with stacked regions: place global routing services (Amazon Route 53) to the LEFT of regions for clean fan-out. Do NOT place above.
- Mirror layout within each region. Cross-region connections: async (dashed), vertical.

## Legend

- Place OUTSIDE the aws-cloud group at x=20, y=canvas height - 140.
- Only add when 2+ distinct connection styles are used.
- Must never overlap with any group, node, or connection.

## Reference Reproduction Mode

When generating from a reference image or existing diagram:

- ONLY include services visibly present in the original
- Do NOT add companion/monitoring/best-practice services unless in reference
- Skip Enrichment Checklist and Companion Service Defaults (new diagrams only)
- Preserve ALL text annotations: step labels, descriptions, conditions, protocols, contextual notes
- Create annotations as text nodes with `add_node` near their associated element

## Companion Service Questions (conditional)

When the extracted service list includes certain services, ask the user about common companions:

| If architecture includes | Ask about |
|-------------------------|-----------|
| Amazon DynamoDB, Amazon S3, Amazon SQS | "Do you need to show encryption with AWS KMS?" |
| AWS Lambda, Amazon ECS | "Should we add Amazon CloudWatch for monitoring?" |
| Amazon SQS, Amazon SNS, AWS Lambda | "Do the async services need a Dead Letter Queue?" |
| ALB, Amazon API Gateway | "Should we add AWS WAF for protection?" |
| Multi-account | SCPs, AWS IAM Identity Center, AWS Control Tower |

These are prompts directed at the user — do not add services without confirmation.

## Implied Services (auto-add)

Some services architecturally REQUIRE companion services to function. Add these automatically without asking. This rule takes precedence over the "only add connections explicitly described in the user prompt" rule — implied services are not best-practice suggestions, they are mandatory architectural components.

| If architecture includes | Auto-add | Reason |
|-------------------------|----------|--------|
| Knowledge Bases for Amazon Bedrock (query/RAG flow) | Amazon Bedrock (Foundation Model) as a separate node | KB uses a foundation model for answer generation. Show as: KB → Bedrock FM (sublabel "Foundation Model" or "Inference"). Place Bedrock FM BELOW KB and offset to the RIGHT (same X as OpenSearch or between KB and OpenSearch) to avoid interfering with ingestion flows that connect to KB from below/left. Connection label: "Generate" sync |

These are NOT optional companions — they are architecturally required for the service to function. Even if the prompt doesn't mention them explicitly, they MUST appear in the diagram.

## AWS Service Naming

Use official AWS service names. Verify via `list_aws_shapes` if unsure.

- Full name first mention ("Amazon DynamoDB", "AWS Lambda"), short form after
- "Amazon" = customer-facing services; "AWS" = platform/infra services
- No unofficial abbreviations ("DDB", "CFN") in docs or labels
- "AWS Systems Manager Parameter Store" not "SSM Parameter Store"
- "AWS Serverless Application Model (AWS SAM)" first, then "AWS SAM"

## Output

After parsing the user's request, produce:

- Service list: `(service, sublabel)` for each service — sublabel includes network context if applicable (e.g., "Private Subnet")
- External actors and entry points
- Connection list: source → target, type, step number, description
- Confirmed output file path

After parsing, proceed to Phase 2 (plan-icons-and-connections.md).

