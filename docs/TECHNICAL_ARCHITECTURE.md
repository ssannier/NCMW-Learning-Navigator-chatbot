# Technical Architecture - Learning Navigator

This document provides a detailed explanation of the Learning Navigator architecture.

---

## Architecture Overview

Learning Navigator is built on a fully serverless AWS architecture, designed for scalability, cost-efficiency, and enterprise-grade security. The system leverages AWS Bedrock's Claude 4 Sonnet with Retrieval-Augmented Generation (RAG) to provide intelligent responses about MHFA training resources.

The application consists of three main user flows:

1. **Public Chat Flow** – Users interact with the AI chatbot via a real-time WebSocket interface
2. **Admin Dashboard Flow** – Staff access analytics, manage documents, and handle escalated queries at `/admin-dashboard`
3. **Knowledge Ingestion Flow** – MHFA training documents are automatically indexed via S3 event triggers

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     USERS (Public / Admins)                          │
└────────────────┬────────────────────────────┬────────────────────────┘
                 │                            │
                 │ (WSS)                      │ (HTTPS + JWT)
                 ▼                            ▼
┌────────────────────────────┐   ┌───────────────────────────────┐
│   AWS Amplify              │   │   AWS Cognito                 │
│   • React Frontend         │   │   • User Pool                 │
│   • Static Hosting         │   │   • Identity Pool             │
│   • Auto-build from Git    │   │   • JWT Tokens                │
└────────────────┬───────────┘   └───────────────────────────────┘
                 │                            │
                 ▼                            ▼
┌────────────────────────────┐   ┌───────────────────────────────┐
│   WebSocket API Gateway    │   │   REST API Gateway            │
│   • Real-time chat         │   │   • File management           │
│   • Streaming responses    │   │   • User profiles             │
│   • Connection mgmt        │   │   • Analytics                 │
└────────────────┬───────────┘   └───────────┬───────────────────┘
                 │                            │
                 ▼                            ▼
┌────────────────────────────────────────────────────────────────┐
│                   AWS LAMBDA FUNCTIONS                          │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────┐ │
│  │ WebSocket      │  │ Chat Response  │  │  Admin Functions │ │
│  │ Handler        │  │ Handler        │  │  • adminFile     │ │
│  │ (Python)       │  │ (Node.js)      │  │  • userProfile   │ │
│  │                │  │                │  │  • sessionLogs   │ │
│  └────────────────┘  └────────────────┘  │  • kb-sync       │ │
│                                           │  • email         │ │
│                                           └──────────────────┘ │
└────────────────┬──────────────────────────┬────────────────────┘
                 │                          │
                 ▼                          ▼
┌─────────────────────────────────────────────────────────────────┐
│              AWS BEDROCK (AI/ML SERVICES)                        │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           Bedrock Agent (Claude 4 Sonnet)                │  │
│  │  • Cross-Region Inference Profile (US)                   │  │
│  │  • Content Guardrails (HIGH input / MEDIUM output)       │  │
│  │  • Action Groups: notify-admin                           │  │
│  └───────────────┬──────────────────┬───────────────────────┘  │
│                  │                  │                           │
│                  ▼                  ▼                           │
│  ┌────────────────────────┐  ┌──────────────────────────────┐ │
│  │  Knowledge Base        │  │  Email Action Lambda         │ │
│  │  • Titan Embeddings v2 │  │  • SES Integration           │ │
│  │  • OpenSearch          │  │  • DynamoDB Logging          │ │
│  │  • RAG Pipeline        │  └──────────────────────────────┘ │
│  └────────────┬───────────┘                                    │
└───────────────┼────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      DATA LAYER                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│  │ S3 Buckets   │  │  DynamoDB    │  │  OpenSearch         │  │
│  │ • KB Docs    │  │  • Sessions  │  │  Serverless         │  │
│  │ • Emails     │  │  • Queries   │  │  • Vector Search    │  │
│  │ • Logs       │  │  • Profiles  │  │  • Auto-managed     │  │
│  │ • Supplemental│ │              │  │                     │  │
│  └──────────────┘  └──────────────┘  └─────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Amazon SES (Email Service)                   │  │
│  │  • Receipt Rule Set                                       │  │
│  │  • Email notifications for escalated queries             │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Architecture Flow

### 1. User Interaction (Instructors, Learners, Admins)

Users access the chatbot through a web interface hosted on **AWS Amplify**:

- **Frontend** is a React application with:
  - Main chat interface for real-time conversations
  - Admin dashboard at `/admin-dashboard` for content management
  - User authentication via AWS Cognito
- Users send messages over **WebSocket** and receive **streaming** responses with citations
- **Feedback collection** (thumbs up/down) on responses
- **Query escalation** for questions requiring expert attention

### 2. WebSocket API (Chat) and HTTP API (Admin)

**Amazon API Gateway** provides two entry points:

#### WebSocket API
- **Route**: `sendMessage`
- Handles real-time bidirectional communication
- Maintains persistent connections during chat sessions
- Streams responses chunk-by-chunk from Bedrock Agent
- Single Lambda (**websocketHandler**) manages connections

#### HTTP API (REST)
- **Files Management**:
  - `GET /files` – List all Knowledge Base documents
  - `POST /files` – Upload new documents
  - `DELETE /files/{key}` – Remove documents
  - `POST /presigned-url` – Get secure upload URLs
  - `POST /sync` – Manually trigger KB sync

- **Session Analytics**:
  - `GET /session-logs` – Retrieve conversation history
  - `GET /session-logs/{sessionId}` – Get specific session

- **Escalated Queries**:
  - `GET /escalated-queries?status=pending` – List by status
  - `GET /escalated-queries/{queryId}` – Get single query
  - `PUT /escalated-queries` – Update status (pending/resolved)

**Authentication**: All admin endpoints require Cognito JWT via Authorization header

### 3. WebSocket Handler Lambda (Chat)

The **websocketHandler** Lambda (`lambda/websocketHandler/handler.py`) implements the core chat flow:

- **Connect**: Establishes WebSocket connection, logs connectionId
- **Disconnect**: Cleans up connection state
- **sendMessage**:
  - Validates input
  - Invokes **chatResponseHandler** Lambda asynchronously
  - Returns immediate acknowledgment to client

**Why async?** Direct Bedrock Agent invocation can take 5-30 seconds; invoking synchronously would cause WebSocket timeout. The async pattern ensures:
- Fast WebSocket response
- Long-running Bedrock processing
- Response pushed back via WebSocket callback

### 4. Chat Response Handler Lambda (AI Orchestration)

The **chatResponseHandler** Lambda (`lambda/chatResponseHandler/handler.js`) orchestrates AI interactions:

```javascript
// Flow:
1. Receive message from WebSocket Handler
2. Invoke Bedrock Agent with user query
3. Agent applies guardrails (content filtering)
4. Agent queries Knowledge Base (RAG)
5. Claude 4 Sonnet generates response with citations
6. Stream response back to user via WebSocket
7. Log session to DynamoDB for analytics
8. Invoke logclassifier Lambda for sentiment analysis
```

**Key Functions**:
- `invokeBedrockAgent()` – Calls Bedrock Agent with streaming
- `sendToWebSocket()` – Pushes chunks to API Gateway WebSocket
- `logToClassifier()` – Triggers session analytics
- Error handling with fallback responses

**Bedrock Agent Configuration**:
```typescript
{
  agentId: "<AGENT_ID>",
  agentAliasId: "<ALIAS_ID>",
  sessionId: "session_<uuid>",
  inputText: "user query",
  enableTrace: false,
  endSession: false
}
```

### 5. Bedrock Agent & Knowledge Base

**Amazon Bedrock Agent** provides conversational AI with RAG:

- **Foundation Model**: `amazon.nova-pro-v1:0` (Claude 4 Sonnet via cross-region inference)
- **Embedding Model**: `amazon.titan-embed-text-v2:0` (1024 dimensions)
- **Guardrail**: Content filtering
  - Input: HIGH (blocks harmful content)
  - Output: MEDIUM (allows some nuance)
- **Action Groups**: `notify-admin` for email escalation
- **User Input**: Enabled for iterative conversations

**Knowledge Base RAG Pipeline**:
```
1. User query → Embedding (Titan v2)
2. Vector similarity search (OpenSearch Serverless)
3. Retrieve top-5 relevant document chunks
4. Inject context into Claude 4 Sonnet prompt
5. Generate response with source citations
6. Return answer + citation URLs
```

**Agent Instructions** (simplified):
```
You are Learning Navigator for MHFA.

1. On every question:
   - Query Knowledge Base
   - Calculate confidence (1-100)
   - Include citations

2. If confidence >= 90:
   - Provide direct answer
   - Include source URLs exactly as found

3. If confidence < 90:
   - Say "I don't have much information"
   - Ask for email to escalate to admin

4. If out of scope:
   - Say "This is out of scope"
   - Do NOT ask for email

5. CRITICAL: Preserve URLs exactly from sources
```

### 6. OpenSearch Serverless

**Amazon OpenSearch Serverless** is the vector store:

- **Collection**: Auto-generated by Bedrock KB (format: `bedrock-knowledge-base-*`)
- **Index**: Vector index with 1024-d embeddings
- **Vector Algorithm**: L2 distance (cosine similarity)
- **Mappings**:
  - `AMAZON_BEDROCK_TEXT_CHUNK` – Document text
  - `AMAZON_BEDROCK_METADATA` – Source URL, title, etc.
- **Scaling**: Automatic; no cluster management
- **Access**: IAM-based; Bedrock Knowledge Base has read/write

### 7. Data Sources (Knowledge Base)

The Knowledge Base is populated via **S3 data source**:

- **Source Bucket**: `s3://national-council/`
- **Parsing Strategy**: Bedrock Data Automation (BDA)
  - Handles PDFs, DOCX, TXT, HTML
  - Extracts text, images, tables
  - Chunks intelligently
- **Sync Trigger**: Automatic via S3 event notification
  - Lambda **kb-sync** (`lambda/kb-sync/handler.py`)
  - Triggered on `s3:ObjectCreated:*` and `s3:ObjectRemoved:*`
  - Calls `StartIngestionJob` API
- **Manual Sync**: Via admin dashboard or AWS CLI

**Ingestion Process**:
```
1. Admin uploads PDF via dashboard
2. S3 PutObject event
3. kb-sync Lambda triggered
4. StartIngestionJob called
5. Bedrock:
   - Downloads document
   - Parses with BDA
   - Chunks text (overlap for context)
   - Generates embeddings (Titan v2)
   - Stores vectors in OpenSearch
6. Document available for queries (~2-5 minutes)
```

### 8. Admin Flow

Admins use the **`/admin-dashboard`**:

#### Authentication
```
1. User logs in → Cognito User Pool
2. Receives JWT tokens (ID, Access, Refresh)
3. Frontend stores tokens in session storage
4. All API calls include Authorization header
```

#### File Management
```
Admin → POST /files (multipart form) → adminFile Lambda
                                            ↓
                                    Upload to S3
                                            ↓
                                    S3 event trigger
                                            ↓
                                    kb-sync Lambda
                                            ↓
                                    Knowledge Base sync
```

#### Analytics Dashboard
```
Admin → GET /session-logs → retrieveSessionLogs Lambda
                                    ↓
                            Query DynamoDB
                            • SessionLogsTable (date index)
                            • NCMWResponseFeedback (join)
                                    ↓
                            Return metrics:
                            • Total sessions
                            • Avg response time
                            • Feedback distribution
                            • Top queries
```

#### Escalated Queries
```
Admin → GET /escalated-queries?status=pending
                ↓
        escalatedQueries Lambda
                ↓
        Query DynamoDB (StatusIndex GSI)
                ↓
        Return: [{
          query_id, email, query, timestamp,
          status, agent_response
        }]
```

### 9. Email Escalation Flow

When Bedrock Agent determines low confidence:

```
1. Agent asks user for email
2. User provides email
3. Agent calls action group: notify-admin
4. Lambda (NotifyAdminFn) invoked
5. Lambda:
   - Saves to DynamoDB (NCMWEscalatedQueries)
   - Sends email via SES
   - Email template includes:
     * User's email
     * Original question
     * Agent's partial answer
6. Admin receives email notification
7. Admin views in dashboard (GET /escalated-queries)
8. Admin updates status via dashboard (PUT /escalated-queries)
```

**Email Template** (`lambda/email/handler.py`):
```python
def create_email(query, user_email, response):
    return {
        'Subject': 'New Escalated Query - Learning Navigator',
        'Body': f'''
        A user needs expert assistance.

        User Email: {user_email}
        Question: {query}
        Agent Response: {response}

        Please respond directly to the user.
        '''
    }
```

### 10. Data Storage (DynamoDB)

Three DynamoDB tables store application data:

#### Session Logs (`NCMWDashboardSessionlogs`)
- **Keys**: `session_id` (PK), `timestamp` (SK)
- **Attributes**: `user_id`, `message_count`, `response_time_ms`, `date`
- **Billing**: On-demand (pay-per-request)
- **Purpose**: Analytics, session history, performance metrics

#### Escalated Queries (`NCMWEscalatedQueries`)
- **Keys**: `query_id` (PK), `timestamp` (SK)
- **GSI**: `StatusIndex` (PK: `status`, SK: `timestamp`)
- **Attributes**: `email`, `query`, `agent_response`, `status`, `date`
- **Purpose**: Admin follow-up queue

#### User Profiles (`NCMWUserProfiles`)
- **Keys**: `userId` (PK)
- **Attributes**: `role`, `preferences`, `last_login`, `created_at`
- **Purpose**: Personalization, recommendations

---

## Cloud Services / Technology Stack

### Frontend

- **React 18**: Modern hooks-based architecture
  - `ChatInterface.jsx` – Main chat UI with WebSocket
  - `AdminDashboard.jsx` – Admin portal with Cognito auth
  - `FileManager.jsx` – Document upload/delete
  - `SessionLogs.jsx` – Analytics visualization

- **AWS Amplify**: Frontend hosting and CI/CD
  - Builds from `frontend/` on git push
  - Environment variables from CDK outputs:
    - `REACT_APP_WS_ENDPOINT`
    - `REACT_APP_API_ENDPOINT`
    - `REACT_APP_COGNITO_USER_POOL_ID`
    - `REACT_APP_COGNITO_CLIENT_ID`
  - Auto-deploys on main branch push

### Backend Infrastructure

- **AWS CDK**: Infrastructure as Code (TypeScript)
  - Stack: `LearningNavigatorStack`
  - 900+ lines defining all resources
  - Context parameters: `githubOwner`, `githubRepo`, `adminEmail`

- **Amazon API Gateway**:
  - **WebSocket API** for real-time chat
  - **REST API** with Cognito JWT authorizer for admin endpoints
  - CORS enabled for frontend domain

- **AWS Lambda** (12 functions):
  | Function | Runtime | Purpose | Timeout |
  |----------|---------|---------|---------|
  | websocketHandler | Python 3.12 | WebSocket mgmt | 120s |
  | chatResponseHandler | Node.js 20 | Bedrock orchestration | 120s |
  | adminFile | Python 3.12 | File operations | 30s |
  | email/NotifyAdminFn | Python 3.12 | Email escalation | 60s |
  | emailReply | Python 3.12 | Process SES replies | 120s |
  | kb-sync | Python 3.12 | Auto KB sync | 300s |
  | logclassifier | Python 3.12 | Sentiment analysis | 30s |
  | sessionLogs | Python 3.12 | Daily export | 30s |
  | retrieveSessionLogs | Python 3.12 | Query logs | 10s |
  | escalatedQueries | Python 3.12 | Query management | 10s |
  | updateQueryStatus | Python 3.12 | Status updates | 10s |
  | userProfile | Python 3.12 | Profile mgmt | 30s |

### AI/ML Services

- **Amazon Bedrock**:
  - **Agent**: Claude 4 Sonnet via cross-region inference
  - **Knowledge Base**: RAG with Titan embeddings
  - **Guardrails**: Content filtering
  - **Action Groups**: Email escalation integration

- **Amazon OpenSearch Serverless**:
  - Vector store for embeddings
  - Auto-managed, no clusters

### Data Storage

- **Amazon S3** (4 buckets):
  | Bucket | Purpose | Retention |
  |--------|---------|-----------|
  | national-council | KB documents | Permanent |
  | EmailStorageBucket | SES emails | 90 days |
  | SupplementalDataBucket | Bedrock multimodal | Auto-delete |
  | DashboardLogsBucket | Exported logs | 1 year |

- **Amazon DynamoDB** (3 tables):
  - Pay-per-request billing (scales to zero)
  - Encryption at rest enabled
  - Point-in-time recovery (optional)

### Security & Authentication

- **AWS Cognito**:
  - **User Pool**: Email-based auth, password policy
  - **Identity Pool**: Temporary AWS credentials for S3 uploads
  - **JWT tokens**: ID token for API Gateway authorizer

- **IAM**: Least-privilege roles
  - Lambda execution roles scoped to required services
  - S3 bucket policies block public access
  - API Gateway invocation permissions

### Monitoring & Observability

- **CloudWatch**:
  - Lambda function logs (all invocations)
  - API Gateway access logs
  - Custom metrics: response time, error rate
  - Log retention: 30 days (configurable)

- **EventBridge**:
  - Daily cron: `DailySessionLogsScheduler` (11:59 PM UTC)
  - Triggers: sessionLogs Lambda for analytics export

---

## Infrastructure as Code

This project uses **AWS CDK** to define and deploy infrastructure.

### CDK Stack Structure

```
cdk_backend/
├── bin/
│   └── cdk_backend.ts              # CDK app entry point
├── lib/
│   └── cdk_backend-stack.ts        # Main stack definition (980 lines)
├── lambda/
│   ├── websocketHandler/
│   │   ├── handler.py              # WebSocket handler
│   │   └── requirements.txt        # boto3
│   ├── chatResponseHandler/
│   │   ├── handler.js              # Chat orchestration
│   │   └── package.json            # aws-sdk
│   ├── adminFile/
│   │   ├── handler.py              # File management
│   │   └── requirements.txt
│   ├── email/
│   │   ├── handler.py              # Email notifications
│   │   └── requirements.txt
│   ├── kb-sync/
│   │   ├── handler.py              # Auto KB sync
│   │   └── requirements.txt
│   ├── sessionLogs/
│   │   ├── handler.py              # Daily export
│   │   └── requirements.txt
│   ├── retrieveSessionLogs/
│   │   ├── handler.py              # Query logs
│   │   └── requirements.txt
│   ├── escalatedQueries/
│   │   ├── handler.py              # Query mgmt
│   │   └── requirements.txt
│   ├── userProfile/
│   │   ├── handler.py              # Profile mgmt
│   │   └── requirements.txt
│   ├── emailReply/
│   │   ├── handler.py              # SES processing
│   │   └── requirements.txt
│   ├── logclassifier/
│   │   ├── handler.py              # Sentiment
│   │   └── requirements.txt
│   └── notify-admin-schema.yaml    # Bedrock action schema
├── cdk.json
├── package.json
└── tsconfig.json
```

### Deployment Automation

- **CDK**: `cdk deploy` creates all backend resources
- **CodeBuild**: Automated via `buildspec.yml` on git push
  - Phase 1: Deploy CDK stack
  - Phase 2: Build React frontend
  - Phase 3: Deploy to Amplify
- **Amplify**: Auto-builds frontend on git push (alternative to CodeBuild)
  
---

## Data Flow Diagrams

### Chat Request Flow

```
User → React (Amplify)
        ↓ (WebSocket send)
WebSocket API Gateway
        ↓ (Route: sendMessage)
websocketHandler Lambda
        ↓ (Async invoke)
chatResponseHandler Lambda
        ↓ (InvokeAgent)
Bedrock Agent
        ├─→ Guardrail (input validation)
        ├─→ Knowledge Base (RAG)
        │      ├─→ Titan Embeddings (query vector)
        │      ├─→ OpenSearch Serverless (similarity search)
        │      └─→ Retrieve top-5 documents
        └─→ Claude 4 Sonnet (generate with context)
                ↓
        Response + Citations
                ↓
chatResponseHandler
        ↓ (WebSocket callback)
API Gateway → User (streaming chunks)
        ↓
chatResponseHandler
        ↓ (Save session)
DynamoDB (SessionLogsTable)
```

### Feedback Flow

```
User (thumbs up/down)
        ↓
React → API (POST /feedback)
        ↓
feedbackHandler Lambda
        ↓
DynamoDB (NCMWResponseFeedback)
        ↓
Response → User (confirmation)
```

### Admin Document Upload Flow

```
Admin → React (/admin-dashboard)
        ↓ (POST /files with multipart form)
REST API Gateway + JWT Auth
        ↓ (Cognito authorizer validates)
adminFile Lambda
        ↓ (PutObject)
S3 (national-council bucket)
        ↓ (S3 event: ObjectCreated)
kb-sync Lambda
        ↓ (StartIngestionJob)
Bedrock Knowledge Base
        ├─→ Download PDF from S3
        ├─→ Parse with BDA (extract text/images/tables)
        ├─→ Chunk into segments (~500 tokens, 20% overlap)
        ├─→ Generate embeddings (Titan v2)
        └─→ Store vectors in OpenSearch Serverless
                ↓
        Document ready for queries
```

### Email Escalation Flow

```
User: "Complex question"
        ↓
chatResponseHandler → Bedrock Agent
        ↓ (Confidence < 90%)
Agent: "I don't have much information. Provide your email?"
        ↓
User: "user@example.com"
        ↓
Agent: Calls action group "notify-admin"
        ↓ (InvokeFunction)
NotifyAdminFn Lambda (email/handler.py)
        ├─→ Save to DynamoDB (EscalatedQueriesTable)
        │      Fields: query_id, email, query, agent_response, timestamp
        └─→ Send email via SES
                ↓
Admin receives email notification
        ↓
Admin → /admin-dashboard → GET /escalated-queries?status=pending
        ↓
escalatedQueries Lambda
        ↓ (Query DynamoDB via StatusIndex GSI)
Display list in dashboard
        ↓
Admin: "Mark as resolved"
        ↓ (PUT /escalated-queries)
updateQueryStatus Lambda
        ↓ (UpdateItem)
DynamoDB (status = "resolved")
```

### Knowledge Base Ingestion Flow

```
S3 Bucket (national-council)
        ├─→ /pdfs/document1.pdf
        ├─→ /pdfs/document2.docx
        └─→ /faqs/faq.html
                ↓ (S3 event)
kb-sync Lambda (on ObjectCreated/Removed)
        ↓ (Call StartIngestionJob API)
Bedrock Knowledge Base
        ├─→ Fetch documents from S3
        ├─→ Bedrock Data Automation (BDA) parsing
        │      ├─→ Text extraction
        │      ├─→ Table/image extraction (multimodal)
        │      └─→ Metadata extraction (title, URL)
        ├─→ Chunking strategy
        │      ├─→ Semantic chunking (~500 tokens)
        │      ├─→ Overlap for context (20%)
        │      └─→ Preserve citations (source URL)
        ├─→ Generate embeddings
        │      └─→ Titan v2 (1024-d vectors)
        └─→ Store in OpenSearch Serverless
                ├─→ Vector field: embeddings
                ├─→ Text field: chunk content
                └─→ Metadata: source URL, title, date
                        ↓
        Ingestion complete (2-5 min for ~10 PDFs)
                ↓
Documents available for RAG queries
```

### Admin Analytics Flow

```
Admin → /admin-dashboard
        ↓ (GET /session-logs)
REST API Gateway + JWT Auth
        ↓
retrieveSessionLogs Lambda
        ├─→ Query SessionLogsTable (date-timestamp-index)
        │      Aggregate by date: count, avg response time
        ├─→ Query NCMWResponseFeedback
        │      Count thumbs up/down
        └─→ Join results
                ↓
Return JSON:
{
  "totalSessions": 1523,
  "avgResponseTime": 3200,
  "feedbackPositive": 1124,
  "feedbackNegative": 89,
  "topQueries": [...]
}
                ↓
React Dashboard renders:
  - Total sessions chart (last 30 days)
  - Response time trend
  - Feedback distribution (pie chart)
  - Top queries table
```

### Daily Log Export Flow

```
EventBridge Rule (DailySessionLogsScheduler)
        ↓ (Cron: 11:59 PM UTC daily)
sessionLogs Lambda
        ├─→ Query CloudWatch Logs
        │      Log group: /aws/lambda/chatResponseHandler
        │      Time range: Last 24 hours
        │      Filter: Extract session_id, timestamp, response_time
        ├─→ Aggregate data
        │      Group by session_id
        │      Calculate metrics per session
        ├─→ Write to S3
        │      Bucket: DashboardLogsBucket
        │      Key: logs/YYYY-MM-DD.json
        └─→ Write summary to DynamoDB
                Table: SessionLogsTable
                ↓
Logs archived for admin dashboard queries
```

---

## Related Documentation

- [Overview](./OVERVIEW.md) – Project introduction and features
- [Architecture Diagram](./ARCHITECTURE_DIAGRAM.md) – Visual architecture
- [Deployment Guide](./DEPLOYMENT_GUIDE.md) – How to deploy the application
- [API Documentation](./API_DOCUMENTATION.md) – WebSocket and REST API reference
- [User Guide](./USER_GUIDE.md) – How to use the chatbot and admin dashboard
- [Modification Guide](./MODIFICATION_GUIDE.md) – How to customize the application

---