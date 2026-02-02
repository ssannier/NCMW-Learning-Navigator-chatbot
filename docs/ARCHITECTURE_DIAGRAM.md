# Architecture Diagram

## System Architecture Overview

### High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                          END USERS                                    │
│                    (Instructors, Learners, Admins)                   │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      FRONTEND LAYER                                   │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │           React Application (Amplify Hosting)                   │ │
│  │  • Chat Interface        • Admin Dashboard                      │ │
│  │  • User Profiles         • Document Management                  │ │
│  │  • Analytics View        • Real-time Updates                    │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────┬────────────────────────────┬──────────────────────────┘
               │                            │
               │ (WSS)                      │ (HTTPS)
               ▼                            ▼
┌──────────────────────────┐  ┌────────────────────────────────────┐
│   WebSocket API Gateway  │  │      REST API Gateway              │
│   • Real-time chat       │  │      • File management             │
│   • Streaming responses  │  │      • User operations             │
│   • Connection mgmt      │  │      • Analytics queries           │
└────────────┬─────────────┘  └──────────────┬─────────────────────┘
             │                               │
             │                               │ (Cognito JWT)
             │                               ▼
             │                    ┌──────────────────────┐
             │                    │  AWS Cognito         │
             │                    │  • User Pools        │
             │                    │  • Identity Pools    │
             │                    │  • Authentication    │
             │                    └──────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      COMPUTE LAYER (Lambda Functions)                 │
│  ┌────────────────┐  ┌─────────────────┐  ┌───────────────────────┐ │
│  │ WebSocket      │  │ Chat Response   │  │  Admin Functions      │ │
│  │ Handler        │  │ Handler         │  │  • File Upload        │ │
│  │ (Python)       │  │ (Node.js)       │  │  • User Profile       │ │
│  └────────────────┘  └─────────────────┘  │  • KB Sync            │ │
│                                            │  • Email Notif        │ │
│                                            │  • Session Logs       │ │
│                                            └───────────────────────┘ │
└──────────────┬────────────────────────────┬──────────────────────────┘
               │                            │
               ▼                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          AI LAYER                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │              AWS Bedrock Agent (Claude 4 Sonnet)               │ │
│  │  • Cross-Region Inference                                      │ │
│  │  • Content Guardrails (HIGH/MEDIUM filtering)                  │ │
│  │  • Action Groups (Email escalation)                            │ │
│  │  • User input enabled                                          │ │
│  └──────────────────────┬──────────────────────┬──────────────────┘ │
└─────────────────────────┼──────────────────────┼────────────────────┘
                          │                      │
                          ▼                      ▼
         ┌─────────────────────────┐  ┌──────────────────────┐
         │  Bedrock Knowledge Base │  │  Email Lambda        │
         │  • Titan Embeddings v2  │  │  • SES Integration   │
         │  • OpenSearch Serverless│  │  • Admin Alerts      │
         │  • RAG Pipeline         │  │  • DynamoDB Logging  │
         └────────────┬────────────┘  └──────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      DATA LAYER                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ S3 Buckets  │  │  DynamoDB    │  │  OpenSearch  │  │   SES    │ │
│  │ • KB Docs   │  │  • Sessions  │  │  • Vectors   │  │ • Email  │ │
│  │ • Emails    │  │  • Queries   │  │  • Search    │  │ • Rules  │ │
│  │ • Logs      │  │  • Profiles  │  │  • Index     │  │          │ │
│  │ • Suppl.    │  │              │  │              │  │          │ │
│  └─────────────┘  └──────────────┘  └──────────────┘  └──────────┘ │
└──────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│                   MONITORING & LOGGING                                │
│  ┌────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │
│  │  CloudWatch    │  │  EventBridge    │  │   X-Ray (Optional)  │  │
│  │  • Logs        │  │  • Cron Jobs    │  │   • Tracing         │  │
│  │  • Metrics     │  │  • Events       │  │   • Performance     │  │
│  │  • Alarms      │  │                 │  │                     │  │
│  └────────────────┘  └─────────────────┘  └─────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### 1. User Chat Query Flow

```
┌─────────┐
│  User   │ "What are MHFA certification requirements?"
└────┬────┘
     │ 1. Send message
     ▼
┌────────────────┐
│ React Frontend │
│ (WebSocket)    │
└────┬───────────┘
     │ 2. WSS connection
     ▼
┌────────────────────┐
│ API Gateway        │
│ (WebSocket)        │
└────┬───────────────┘
     │ 3. Route to Lambda
     ▼
┌───────────────────────┐
│ websocketHandler      │
│ (Python)              │
│ • Validate connection │
│ • Parse message       │
└────┬──────────────────┘
     │ 4. Invoke async
     ▼
┌──────────────────────────┐
│ chatResponseHandler      │
│ (Node.js)                │
│ • Invoke Bedrock Agent   │
│ • Stream responses       │
└────┬─────────────────────┘
     │ 5. Query agent
     ▼
┌─────────────────────────────┐
│ Bedrock Agent               │
│ • Apply guardrails          │
│ • Query Knowledge Base      │
│ • Calculate confidence (%)  │
└────┬────────────────────────┘
     │ 6. Search documents
     ▼
┌────────────────────────────┐
│ Bedrock Knowledge Base     │
│ • Create query embedding   │
│ • Search OpenSearch        │
│ • Retrieve relevant docs   │
└────┬───────────────────────┘
     │ 7. Return context
     ▼
┌─────────────────────────────┐
│ Claude 4 Sonnet             │
│ • Generate response         │
│ • Add citations             │
│ • Include URLs if present   │
└────┬────────────────────────┘
     │ 8. Stream response
     ▼
┌──────────────────────────┐
│ Lambda → API Gateway     │
│ (WebSocket push)         │
└────┬─────────────────────┘
     │ 9. Real-time update
     ▼
┌────────────────┐
│ React Frontend │
│ • Display      │
│ • Citations    │
└────┬───────────┘
     │ 10. Log session
     ▼
┌────────────┐
│ DynamoDB   │
│ (Sessions) │
└────────────┘
```

### 2. Document Upload & KB Sync Flow

```
┌────────┐
│ Admin  │ Upload PDF
└───┬────┘
    │ 1. Upload via dashboard
    ▼
┌──────────────────┐
│ React Dashboard  │
└───┬──────────────┘
    │ 2. POST /files
    ▼
┌────────────────────┐
│ REST API Gateway   │
│ + Cognito Auth     │
└───┬────────────────┘
    │ 3. Validate JWT
    ▼
┌───────────────────┐
│ adminFile Lambda  │
│ • Validate file   │
│ • Upload to S3    │
└───┬───────────────┘
    │ 4. Put object
    ▼
┌──────────────────────────┐
│ S3 Bucket                │
│ (national-council)       │
│ /pdfs/document.pdf       │
└───┬──────────────────────┘
    │ 5. S3 Event Notification
    ▼
┌─────────────────────────┐
│ kb-sync Lambda          │
│ (Triggered by S3)       │
└───┬─────────────────────┘
    │ 6. Start ingestion job
    ▼
┌──────────────────────────────┐
│ Bedrock Knowledge Base       │
│ • Extract text from PDF      │
│ • Chunk into segments        │
│ • Generate embeddings        │
└───┬──────────────────────────┘
    │ 7. Store vectors
    ▼
┌────────────────────────┐
│ OpenSearch Serverless  │
│ • Vector index         │
│ • Metadata             │
└───┬────────────────────┘
    │ 8. Ready for queries
    ▼
┌──────────────┐
│ Available in │
│ Chat Agent   │
└──────────────┘
```

### 3. Authentication Flow

```
┌─────────┐
│  User   │ Login
└────┬────┘
     │ 1. Enter credentials
     ▼
┌────────────────┐
│ React Login    │
│ (Amplify Auth) │
└────┬───────────┘
     │ 2. Authenticate
     ▼
┌──────────────────────┐
│ Cognito User Pool    │
│ • Verify credentials │
│ • Check MFA (opt)    │
└────┬─────────────────┘
     │ 3. Issue JWT tokens
     ▼
┌────────────────────────┐
│ Frontend receives:     │
│ • ID Token             │
│ • Access Token         │
│ • Refresh Token        │
└────┬───────────────────┘
     │ 4. Exchange for AWS creds
     ▼
┌──────────────────────────┐
│ Cognito Identity Pool    │
│ • Temporary credentials  │
│ • IAM role assumed       │
└────┬─────────────────────┘
     │ 5. Access AWS resources
     ▼
┌────────────────────┐
│ S3 Upload          │
│ • Presigned URLs   │
│ • Direct uploads   │
└────────────────────┘
```

---

## Component Interaction Matrix

| Component | Interacts With | Protocol | Purpose |
|-----------|---------------|----------|---------|
| Frontend | WebSocket API | WSS | Real-time chat |
| Frontend | REST API | HTTPS + JWT | Admin operations |
| Frontend | Cognito | HTTPS | Authentication |
| WebSocket API | Lambda (WS) | Event | Connection handling |
| REST API | Lambda (Admin) | Event | CRUD operations |
| Lambda | Bedrock Agent | AWS SDK | AI inference |
| Bedrock Agent | Knowledge Base | Internal | RAG queries |
| Knowledge Base | S3 | Internal | Read documents |
| Knowledge Base | OpenSearch | Internal | Vector search |
| Lambda | DynamoDB | AWS SDK | Data persistence |
| Lambda | SES | AWS SDK | Email notifications |
| S3 | Lambda (KB Sync) | Event | Auto-sync trigger |
| EventBridge | Lambda (Logs) | Scheduled | Daily exports |

---

## Network Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Internet                              │
└───────────────┬─────────────────────────────────────────┘
                │
                │ HTTPS/WSS (TLS 1.2+)
                ▼
┌─────────────────────────────────────────────────────────┐
│              AWS CloudFront (Optional CDN)               │
└───────────────┬─────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────┐
│                    Amplify Hosting                       │
│                    (Static Assets)                       │
└───────────────┬─────────────────────────────────────────┘
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
┌──────────────┐  ┌─────────────┐
│ WebSocket API│  │  REST API   │
│   Gateway    │  │  Gateway    │
│              │  │  + WAF      │
└──────┬───────┘  └──────┬──────┘
       │                 │
       │  VPC (Optional) │
       ▼                 ▼
┌─────────────────────────────┐
│     Lambda Functions         │
│     (Security Groups)        │
└─────────────────────────────┘
```

---

## Security Layers

```
┌────────────────────────────────────────────┐
│         Layer 1: Network Security          │
│  • HTTPS/WSS only                          │
│  • CloudFront (optional)                   │
│  • WAF rules                               │
└────────────┬───────────────────────────────┘
             ▼
┌────────────────────────────────────────────┐
│      Layer 2: Authentication & AuthZ       │
│  • Cognito User Pools (JWT)                │
│  • MFA (optional)                          │
│  • API Gateway Authorizers                 │
└────────────┬───────────────────────────────┘
             ▼
┌────────────────────────────────────────────┐
│         Layer 3: Application Security      │
│  • IAM roles (least privilege)             │
│  • Resource policies                       │
│  • Bedrock Guardrails                      │
└────────────┬───────────────────────────────┘
             ▼
┌────────────────────────────────────────────┐
│          Layer 4: Data Security            │
│  • S3 encryption at rest                   │
│  • DynamoDB encryption                     │
│  • Secrets Manager                         │
│  • TLS in transit                          │
└────────────────────────────────────────────┘
```

---

## Deployment Architecture

```
┌──────────────┐
│   GitHub     │ Push to main
└──────┬───────┘
       │
       │ Webhook
       ▼
┌──────────────────┐
│   CodeBuild      │
│ • npm install    │
│ • cdk synth      │
│ • cdk deploy     │
│ • npm build      │
└──────┬───────────┘
       │
       ├──────────────┬─────────────────┐
       ▼              ▼                 ▼
┌──────────────┐ ┌──────────┐   ┌────────────┐
│ CloudFormation│ │ Lambda   │   │  Amplify   │
│ • Stack      │ │ • Deploy │   │  • Deploy  │
│ • Resources  │ │ • Update │   │  • Build   │
└──────────────┘ └──────────┘   └────────────┘
```

---

**For detailed component specifications, see [Architecture Deep Dive](ARCHITECTURE_DEEP_DIVE.md)**
