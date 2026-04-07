name: "drawio-architect"
displayName: "Draw.io Architect"
description: "Create AWS architecture diagrams from natural language using 18 MCP tools, 300+ AWS service icons, auto layout, validation, and fix."
keywords: ["drawio", "draw.io", "diagram", "architecture", "aws", "icon", "canvas", "layout", "validate", "fix", "mxgraph", "architecture diagram", "cloud diagram"]
# Draw.io Architect
Create and manage draw.io architecture diagrams with AWS service icons from natural language. Describe your architecture and the agent builds the `.drawio` file for you.
18 MCP tools | 300+ AWS service icons | Auto edge routing | Sync/async connection styles | Validation and auto-fix
## Onboarding
### Step 1: Verify Node.js 18+
```bash
node --version
```
If Node.js is not installed or below v18, install it before proceeding.
### Step 2: Verify MCP connectivity
Call `list_aws_shapes` from the drawio-architect MCP server. If it returns a list of AWS shapes grouped by category, the server is working. If it fails, wait 10 seconds and retry once. If it still fails, verify Node.js version and MCP server configuration.
### Step 3: Smoke test — build a 3-node diagram
Run this exact sequence to verify the full pipeline:
1. `create_diagram` — title: "Smoke Test", direction: "LR"
1. `add_group` — group_id: "aws-cloud", label: "AWS Cloud", group_type: "aws-cloud"
1. `add_node` — node_id: "user", label: "User" (parent_group: "", outside aws-cloud)
1. `add_node` — node_id: "apigw", label: "Amazon API Gateway", aws_service: "api-gateway", parent_group: "aws-cloud"
1. `add_node` — node_id: "lambda", label: "AWS Lambda", aws_service: "lambda", parent_group: "aws-cloud"
1. `add_node` — node_id: "dynamodb", label: "Amazon DynamoDB", aws_service: "dynamodb", parent_group: "aws-cloud"
1. `add_connection` — source: "user", target: "apigw", style: "sync", label: "1. HTTPS"
1. `add_connection` — source: "apigw", target: "lambda", style: "sync", label: "2. Invoke"
1. `add_connection` — source: "lambda", target: "dynamodb", style: "sync", label: "3. Read/Write"
1. `validate_diagram`
1. `fix_diagram`
Open the generated `.drawio` file in draw.io desktop to confirm it renders correctly. If any step fails, verify Node.js version, check npx connectivity, and retry MCP server installation.
## MCP Tools (18)
| Group | Count | Tools |
|-------|-------|-------|
| Creation | 6 | `create_diagram`, `add_node`, `add_connection`, `add_group`, `add_page`, `add_legend` |
| Editing | 5 | `edit_node`, `edit_edge`, `remove_node`, `set_cell_shape`, `set_cell_data` |
| Layers | 3 | `create_layer`, `move_cell_to_layer`, `list_layers` |
| Utility | 4 | `read_diagram`, `list_aws_shapes`, `validate_diagram`, `fix_diagram` |
## When to Load Steering Files
Read steering files sequentially. Each phase builds on the previous one.
| Trigger | Load File |
|---------|-----------|
| Starting any diagram or modifying existing | `diagram-rules.md` |
| Planning nodes and connections (Phase 2) | `plan-icons-and-connections.md` |
| Positioning and building (Phase 3) | `layout-and-routing.md` |
| Writing or validating a prompt file | `prompt-file-spec.md` |
## Prompt Guide
### Template
```text
I need an AWS architecture diagram for [brief description].
[Flow/path name] (sync/async):
- Service A receives requests from [source]
- Service A invokes Service B
- Service B writes to Service C
[Second flow name] (sync/async):
- Service D triggers Service E
- Service E stores results in Service F
Supporting services (no connections needed): Service G, Service H.
Connection styles: sync for [path], async for [path].
Save to [path/to/file.drawio].
```
### Do's
1. Use official AWS service names: "Amazon API Gateway", "AWS Lambda", "Amazon DynamoDB"
1. Describe the data flow naturally: "User sends requests to API Gateway, which invokes Lambda"
1. Specify connection styles: "sync for the request path, async for the streaming path"
1. Group related services into flows: "Customer interaction flow", "Analytics flow"
1. Mention VPC/subnets if needed: "inside a VPC with public and private subnets"
1. Say "supporting services (no connections needed)" for monitoring/logging services
1. Use parenthetical examples for service roles: "Lambda application services (e.g., Product Service, Order Service)" — creates one node with a sublabel
### Don'ts
1. Don't include layout instructions: "put X below Y", "same X column", "row 1"
1. Don't specify coordinates or pixel values — describe the architecture, not the visual layout
1. Don't use unofficial abbreviations: "DDB", "CFN", "APIGW"
### Connection Styles
| Style | Arrow | When to use | Example |
|-------|-------|-------------|---------|
| sync | solid | Request-response, direct invocations, queries | API GW → Lambda, Lambda → DynamoDB |
| async | dashed | Event-driven, streaming, notifications | SQS → Lambda, Kinesis → Lambda, SNS publish |
| optional | dotted | Monitoring, health checks (< 8 nodes only) | CloudWatch metrics |
### Authentication Pattern
Describe authentication naturally:
```text
Authentication:
- A user authenticates with Amazon Cognito (shared user pool)
- The JWT token is passed with each request to Amazon API Gateway
- A Lambda Authorizer validates the JWT and extracts the tenant ID
```
The agent places Cognito above the main flow and the Lambda Authorizer below API Gateway as a vertical side branch.
### Multi-Region Structure
For DR architectures, describe each region separately:
```text
Primary region (us-east-1):
- ALB in public subnet, ECS Fargate in private subnet
- Aurora PostgreSQL as primary database
Secondary region (us-west-2) — passive standby:
- ALB in public subnet (receives traffic only during failover)
- ECS Fargate with scaled-down capacity
- Aurora read replica (cross-region replication)
```
### Example Prompts
**Simple (3-5 nodes):**
```text
I need an AWS architecture diagram for a serverless REST API.
An external user sends requests to Amazon API Gateway, which
invokes AWS Lambda. Lambda writes to Amazon DynamoDB.
Connection styles: all sync. Save to docs/api-rest.drawio.
```
**Medium (8-12 nodes):**
```text
I need an AWS architecture diagram for a multi-tenant serverless
SaaS application. A tenant user authenticates with Amazon Cognito.
The JWT is passed to Amazon API Gateway. A Lambda Authorizer
validates the JWT and generates tenant-scoped credentials via
AWS STS. API Gateway routes requests to AWS Lambda application
services (e.g., Product Service, Order Service). Lambda accesses
Amazon DynamoDB (pooled table with tenant partition key) and
Amazon S3 for per-tenant file storage. Supporting services (no
connections needed): Amazon CloudWatch, AWS CodePipeline.
Connection styles: sync for the request path, async for auth.
Save to docs/multi-tenant-saas.drawio.
```
**Complex (12+ nodes):**
```text
I need an AWS architecture diagram for a RAG application using
Knowledge Bases for Amazon Bedrock. Query flow: Chatbot → AWS WAF
→ Amazon API Gateway → AWS Lambda → Knowledge Bases for Amazon
Bedrock → Amazon OpenSearch Serverless. KB Bedrock invokes Amazon
Bedrock (Foundation Model) for answer generation. Ingestion flow A:
Upload Client → Amazon S3 → AWS Lambda → KB Bedrock → OpenSearch.
Ingestion flow B: Amazon EventBridge → AWS Lambda → KB Bedrock →
OpenSearch. Supporting services: Amazon CloudWatch. Connection
styles: sync for query flow, async for ingestion flows.
Save to docs/genai-rag-bedrock.drawio.
```
## Workflow
### Workflow A — Create from scratch
**Phase A — Gather requirements:**
1. Classify intent: `CREATE_FROM_SCRATCH`, `CREATE_FROM_PROMPT`, `CREATE_FROM_PNG`, `FIX_EXISTING`, `MODIFY_EXISTING`
1. Determine save location from context — do NOT ask unless ambiguous
1. Ask clarifying questions about the architecture (services, data flow, VPC, sync/async)
1. Go back and forth until the user confirms
1. Write the confirmed architecture to a `_diagram-spec.md` file
1. Search for existing `prompt-*.md` files in the target directory. If found, offer to reuse
**Phase B — Build the diagram:**
Read steering files in order: `diagram-rules.md` → `plan-icons-and-connections.md` → `layout-and-routing.md`. Then:
1. Identify patterns: scan the prompt for fan-out/fan-in/hub/convergence triggers. List each pattern found BEFORE proceeding
1. Plan nodes, connections, groups — applying the identified patterns to determine layout
1. Calculate layout (canvas sizing, column/row spacing)
1. Write the prompt-file (`prompt-<name>.md`) following `prompt-file-spec.md`
1. Build: `create_diagram` → `add_group` → `add_node` → `add_connection` → `add_legend`
1. Validate and fix: `validate_diagram` → `fix_diagram`
1. Present the result and ask "What would you change?"
**Build order:** create_diagram → add_group (all groups) → add_node (all nodes) → add_connection (all connections) → add_legend → validate_diagram → fix_diagram
When the node count exceeds 15, suggest splitting into multiple pages via `add_page`.
### Workflow B — Create from PNG/image
1. Analyze the provided image to identify AWS services, groups, and connections
1. Identify all services visible in the image — include only what is visible, skip enrichment suggestions
1. Confirm the service list and connections with the user before building
1. Preserve text annotations from the original image
1. Continue from Phase B step 1 (planning)
### Workflow C — Fix existing diagram
1. Call `read_diagram` to load the existing `.drawio` file structure
1. Call `validate_diagram` to identify all violations
1. Present findings to the user in a table: violation type, affected element, suggested fix
1. Apply fixes via `fix_diagram` and targeted `edit_node` or `edit_edge` calls based on user-approved changes
1. Repeat validate and fix cycle until the user confirms satisfaction or no violations remain
### Building from an existing prompt-file
Skip Phase A. Read the prompt-file tables and go directly to Phase B step 4 (build).
## Adjustment Cycle
After building and validating a diagram:
1. Ask "What would you change?"
1. **Simple changes** (move, relabel, or restyle 1-3 nodes): apply directly via `edit_node` or `edit_edge` without re-reading the diagram
1. **Complex changes** (add/remove nodes, change connections, restructure groups): apply changes, then call `validate_diagram` followed by `fix_diagram`
1. Mandatory final pass: always run `validate_diagram` → `fix_diagram` before closing the cycle
1. Continue the loop until the user explicitly confirms the diagram is complete
## Interaction Style
- Use a conversational tone — avoid robotic or overly formal language
- Ask clarifying questions BEFORE building, not after presenting a completed diagram
- When the user provides a vague description, offer two concrete options to choose from
- Use tables instead of paragraphs when presenting service lists, connection plans, or validation results
- Ask "What would you change?" after presenting every built diagram
## Key Rules
1. All diagram operations go through MCP tools — never edit draw.io XML directly
1. Generate a `prompt-<name>.md` file alongside every diagram for regeneration
1. Only add connections explicitly described in the user prompt — do not add best-practice suggestions
1. External actors (User, Customer) go OUTSIDE the aws-cloud group; all AWS managed services go INSIDE
1. After building, always call `validate_diagram` then `fix_diagram`
1. When unsure which `aws_service` key to use, call `list_aws_shapes` to find the correct key
## Known Limitations
- **VPC subnet misplacement**: Services may end up in the wrong subnet when multiple subnets have similar services
- **Label collisions**: Dense areas with nested groups can produce overlapping labels
- **Edge crossing**: The orthogonal router may route edges through unrelated nodes when space is limited
- **Convergence pattern errors**: Diagrams with 12+ nodes using fan-out with convergence may produce incorrect post-convergence node placement
## Disclaimer
This power uses generative AI to accelerate the creation of architecture diagrams. While it follows AWS best practices and validates output automatically, AI-generated diagrams may contain inaccuracies in service placement, connection routing, or architectural patterns. Always review the final diagram before using it in design reviews or production documentation. The human architect remains responsible for the correctness of the architecture.

## License and Support

This power uses [drawio-architect-mcp](https://www.npmjs.com/package/drawio-architect-mcp) (MIT).

- [Privacy Policy](https://github.com/dfriveros11/drawio-power/blob/main/LICENSE)
- [Support](https://github.com/dfriveros11/drawio-power/issues)

We handle your information as described in our Privacy Notice.
