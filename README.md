# Draw.io Architect — Kiro Power

Create and manage AWS architecture diagrams from natural language using 18 MCP tools, 300+ AWS service icons, auto edge routing, sync/async connection styles, validation, and auto-fix. Describe your architecture and the agent builds the `.drawio` file for you.

## Prerequisites

- [Node.js 18+](https://nodejs.org/)
- [Kiro](https://kiro.dev/)
- [draw.io desktop](https://www.drawio.com/) (optional, for viewing generated diagrams)

## Installation

1. Open Kiro
1. Go to the Powers panel
1. Click "Add from GitHub"
1. Paste the repository URL: `https://github.com/dfriveros11/drawio-power`
1. The power installs automatically and configures the MCP server

## Quick Start

1. Install the power using the steps above
1. Verify the MCP server is working by asking the agent: "Call `list_aws_shapes` to verify the drawio-architect MCP server"
1. Try your first diagram — copy and paste this prompt into Kiro chat:

```text
Use the drawio-architect power to create an AWS architecture diagram for an order processing system using AWS Step Functions for orchestration.

The flow starts with Amazon API Gateway receiving order requests. It invokes an AWS Lambda function that starts an AWS Step Functions state machine. The state machine orchestrates three parallel branches:
- Branch 1: AWS Lambda "Validate Payment" → Amazon DynamoDB "Payments" table
- Branch 2: AWS Lambda "Check Inventory" → Amazon DynamoDB "Inventory" table
- Branch 3: AWS Lambda "Fraud Check" → external API call

After all branches complete, a final AWS Lambda "Fulfill Order" writes to Amazon DynamoDB "Orders" table and sends a notification via Amazon SNS.

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring.

Connection styles: sync for API Gateway to Lambda and Lambda to DynamoDB, async for SNS notification.
```

> **Tip:** Start your prompt with "Use the drawio-architect power" so Kiro activates the power and loads the steering files automatically. This produces better results.

## Example Prompts

Copy any prompt below into Kiro chat. Start with "Use the drawio-architect power" so Kiro activates the power automatically.

### 1. VPC Multi-Tier Web Application

```text
Use the drawio-architect power to create an AWS architecture diagram for a multi-tier web application inside a VPC.

The architecture has three tiers inside a VPC:
- Public subnet: Application Load Balancer receiving traffic from an external user
- Private subnet (app tier): Two AWS Lambda functions — one for the API backend and one for background processing
- Private subnet (data tier): Amazon RDS (PostgreSQL) as the primary database and Amazon ElastiCache (Redis) for session caching

Additional services:
- Amazon Cognito for user authentication, connected to the ALB
- AWS WAF protecting the ALB from the public internet

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring, Amazon SQS for background processing.

Connection styles: sync for the main request path (User → WAF → ALB → API Lambda → RDS and ElastiCache), async for SQS to background Lambda.
```

### 2. Step Functions Orchestration

```text
Use the drawio-architect power to create an AWS architecture diagram for an order processing system using AWS Step Functions for orchestration.

The flow starts with Amazon API Gateway receiving order requests. It invokes an AWS Lambda function that starts an AWS Step Functions state machine. The state machine orchestrates three parallel branches:
- Branch 1: AWS Lambda "Validate Payment" → Amazon DynamoDB "Payments" table
- Branch 2: AWS Lambda "Check Inventory" → Amazon DynamoDB "Inventory" table
- Branch 3: AWS Lambda "Fraud Check" → external API call

After all branches complete, a final AWS Lambda "Fulfill Order" writes to Amazon DynamoDB "Orders" table and sends a notification via Amazon SNS.

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring.

Connection styles: sync for API Gateway to Lambda and Lambda to DynamoDB, async for SNS notification.
```

### 3. Streaming Analytics

```text
Use the drawio-architect power to create an AWS architecture diagram for a real-time streaming analytics pipeline.

Data flows from IoT devices through Amazon Kinesis Data Streams for ingestion. An AWS Lambda function enriches each record and writes to Amazon S3 (raw data lake). Amazon Kinesis Data Firehose delivers a copy to Amazon OpenSearch Service for real-time dashboards.

For batch analytics: AWS Glue crawls the S3 data lake, catalogs it, and Amazon Athena runs SQL queries against the catalog. Query results go to a separate Amazon S3 bucket.

Amazon QuickSight connects to both Amazon OpenSearch (real-time) and Amazon Athena (batch) for unified dashboards.

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring.

Connection styles: async for streaming paths (Kinesis, Firehose), sync for direct writes (S3, OpenSearch, Athena).
```

### 4. ECS Microservices

```text
Use the drawio-architect power to create an AWS architecture diagram for a containerized microservices application running on Amazon ECS inside a VPC.

The architecture has:
- Public subnet: Application Load Balancer receiving traffic from an external user
- Private subnet (app tier): Amazon ECS cluster with three Fargate services — "User Service", "Order Service", and "Payment Service". Each service has its own target group on the ALB.
- Private subnet (data tier): Amazon RDS (MySQL) for the Order Service, Amazon DynamoDB for the User Service, and Amazon ElastiCache (Redis) shared by all services for caching

Supporting services (no connections needed, just show them): Amazon ECR for container images, Amazon CloudWatch for monitoring, AWS Secrets Manager for credentials.

Data flow: User → ALB → ECS services (routed by path). Each service connects to its own database. All services share the ElastiCache cluster.

Connection styles: sync for the main request path and database queries.
```

### 5. GenAI RAG with Bedrock

```text
Use the drawio-architect power to create an AWS architecture diagram for a Retrieval-Augmented Generation (RAG) application using Knowledge Bases for Amazon Bedrock.

The architecture has two external actors and multiple flows:

Query flow (sync, top area):
- A Chatbot client selects a model and posts a question
- The request goes through AWS WAF for protection
- AWS WAF forwards to Amazon API Gateway
- Amazon API Gateway invokes AWS Lambda which calls Knowledge Bases for Amazon Bedrock to retrieve an answer
- Knowledge Bases for Amazon Bedrock searches Amazon OpenSearch Serverless (vector store) for relevant embeddings
- Knowledge Bases for Amazon Bedrock invokes Amazon Bedrock (Foundation Model) to generate the answer using the retrieved context
- The answer is returned to the Chatbot

Ingestion flow A — document upload (async, left side):
- An Upload Client uploads content to Amazon S3
- An AWS Lambda function triggers ingestion on Knowledge Bases for Amazon Bedrock
- Knowledge Bases for Amazon Bedrock loads the content, generates embeddings, and stores them in Amazon OpenSearch Serverless

Ingestion flow B — web crawler (async, right side):
- Amazon EventBridge fires a recurring web sync event
- An AWS Lambda function triggers webcrawler ingestion on Knowledge Bases for Amazon Bedrock
- Webcrawler results are stored in Amazon OpenSearch Serverless

Knowledge Bases for Amazon Bedrock is the central node connecting to Amazon OpenSearch Serverless for vector storage.

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring.

Connection styles: sync for the query flow, async for both ingestion flows.
```

### 6. Hybrid Cloud Migration

```text
Use the drawio-architect power to create an AWS architecture diagram for a hybrid cloud architecture connecting an on-premises data center to AWS.

On-premises side (outside AWS Cloud):
- Corporate Data Center with an application server and an Oracle database
- The data center connects to AWS through AWS Direct Connect

AWS side (inside AWS Cloud, inside a VPC):
- Private subnet: AWS Database Migration Service (DMS) replicates data from the on-premises Oracle database to Amazon Aurora (PostgreSQL)
- Private subnet: Amazon EC2 instances running the modernized application, behind an internal Application Load Balancer
- Public subnet: External-facing Application Load Balancer for internet users
- Amazon S3 stores migrated files and backups from the on-premises file server

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring, AWS Systems Manager for EC2 management.

Data flow: On-prem app server → Direct Connect → internal ALB → EC2. On-prem Oracle → Direct Connect → DMS → Aurora. External User → public ALB → EC2 → Aurora.

Connection styles: sync for the main request paths, async for DMS replication.
```

### 7. Security and Compliance

```text
Use the drawio-architect power to create an AWS architecture diagram for a security and compliance monitoring architecture.

The architecture monitors and protects a VPC-based application:
- Amazon VPC with a public subnet (Application Load Balancer) and a private subnet (Amazon EC2 instances running the application)
- AWS WAF protects the ALB from web attacks (inline, in the traffic path)
- AWS Shield Advanced provides DDoS protection on the ALB (monitoring overlay, NOT in the traffic path — show as a standalone node associated with the ALB, not inline)

Security monitoring chain:
- Amazon GuardDuty analyzes VPC Flow Logs, AWS CloudTrail logs, and DNS logs for threat detection
- AWS Security Hub aggregates findings from GuardDuty, Amazon Inspector, and AWS Config
- Amazon Inspector scans EC2 instances for vulnerabilities
- AWS Config monitors resource configuration compliance

Alerting and response:
- AWS Security Hub sends critical findings to Amazon EventBridge
- Amazon EventBridge triggers an AWS Lambda function "Security Responder" that sends alerts via Amazon SNS and creates tickets

Supporting services (no connections needed, just show them): AWS CloudTrail for API logging, Amazon CloudWatch for monitoring, AWS Shield Advanced for DDoS protection.

Data flow: External traffic → WAF → ALB → EC2. GuardDuty, Inspector, and Config feed findings to Security Hub. Security Hub → EventBridge → Lambda → SNS.

Connection styles: sync for the traffic path, async for security findings flow.
```

### 8. Multi-Tenant SaaS

```text
Use the drawio-architect power to create an AWS architecture diagram for a multi-tenant serverless SaaS application based on the AWS SaaS Factory reference architecture (pool model).

Authentication and authorization flow:
- A tenant user authenticates with Amazon Cognito (shared user pool, tenant ID as custom claim in JWT)
- The JWT token is passed with each request to Amazon API Gateway
- A Lambda Authorizer validates the JWT, extracts the tenant ID, and generates tenant-scoped credentials via AWS STS (Security Token Service)
- The scoped credentials are passed to downstream Lambda functions

Application flow (sync):
- Amazon API Gateway routes requests to AWS Lambda application services (e.g., Product Service, Order Service)
- Lambda functions use the tenant-scoped credentials to access Amazon DynamoDB (pooled table with tenant partition key) with fine-grained access control
- Amazon S3 stores per-tenant files with prefix-based isolation

Supporting services (no connections needed, just show them): Amazon CloudWatch for monitoring and tenant metrics, AWS CodePipeline for CI/CD.

Connection styles: sync for the main request path, async for the authentication flow (Cognito → JWT).
```

### 9. Contact Center

```text
Use the drawio-architect power to create an AWS architecture diagram for an intelligent contact center using Amazon Connect.

The architecture handles both voice and chat channels:

Customer interaction flow:
- A customer calls or chats, entering Amazon Connect (contact center)
- Amazon Connect routes to Amazon Lex (chatbot) for initial intent classification
- If the chatbot can handle it, Amazon Lex invokes AWS Lambda "Bot Fulfillment" to query Amazon DynamoDB "Customer" table and respond
- If escalation is needed, Amazon Connect transfers to a human agent

Real-time analytics:
- Amazon Connect streams Contact Trace Records (CTRs) to Amazon Kinesis Data Streams
- AWS Lambda "CTR Processor" enriches the records and writes to Amazon S3 (data lake)
- Amazon QuickSight provides dashboards from the S3 data lake

Additional services:
- Amazon CloudWatch monitors all Lambda functions
- Amazon SNS sends alerts when queue wait times exceed thresholds

Data flow: Customer → Amazon Connect → Lex → Lambda → DynamoDB. Amazon Connect → Kinesis → Lambda → S3 → QuickSight. CloudWatch monitors Lambdas.

Connection styles: sync for the customer interaction path, async for the analytics streaming path, optional for CloudWatch monitoring and SNS alerts.
```

### 10. Multi-Region Disaster Recovery

```text
Use the drawio-architect power to create an AWS architecture diagram for a multi-region active-passive disaster recovery architecture.

Primary region (us-east-1):
- Amazon Route 53 routes traffic to the primary region (failover routing policy)
- Application Load Balancer in a public subnet
- Amazon ECS Fargate cluster running the application in a private subnet
- Amazon Aurora (PostgreSQL) as the primary database with a writer instance
- Amazon S3 for application assets

Secondary region (us-west-2) — passive standby:
- Application Load Balancer in a public subnet (receives traffic only during failover)
- Amazon ECS Fargate cluster with scaled-down capacity (minimum tasks)
- Amazon Aurora read replica (cross-region replication from primary)
- Amazon S3 with cross-region replication from the primary bucket

Cross-region services:
- Amazon Route 53 performs health checks on the primary ALB and fails over to secondary
- Amazon Aurora cross-region replication keeps the secondary database in sync
- Amazon S3 Cross-Region Replication keeps assets in sync
- Amazon CloudWatch monitors both regions and triggers alarms

Data flow: User → Route 53 → Primary ALB → ECS → Aurora. Aurora replicates to secondary region. S3 replicates to secondary region. On failover: Route 53 → Secondary ALB → ECS → Aurora (promoted to writer).

Connection styles: sync for the main request path within each region, async for cross-region replication (Aurora, S3), optional for CloudWatch monitoring and Route 53 health checks.
```

## Disclaimer

This tool uses generative AI to accelerate the creation of architecture diagrams. While it follows AWS best practices and validates output automatically, AI-generated diagrams may contain inaccuracies in service placement, connection routing, or architectural patterns. Always review the final diagram before using it in customer-facing deliverables, design reviews, or production documentation. The human architect remains responsible for the correctness of the architecture.

## License

This project is licensed under the [MIT License](LICENSE).
