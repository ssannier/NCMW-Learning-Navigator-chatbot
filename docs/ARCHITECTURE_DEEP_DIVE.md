# Architecture Deep Dive

> For quick technical reference, see [TECHNICAL_DOCUMENTATION.md](../TECHNICAL_DOCUMENTATION.md)

## System Components

This document provides an in-depth look at each component of the Learning Navigator architecture.

### 1. Frontend Layer (React + Amplify)

**Technology Stack:**
- React 18 with Hooks
- WebSocket for real-time communication
- AWS Amplify for hosting and auth
- Material-UI components

**Key Components:**
- `ChatInterface.jsx` - Main chat UI
- `AdminDashboard.jsx` - Admin portal
- `FileManager.jsx` - Document management
- `SessionLogs.jsx` - Analytics view

**State Management:**
```
User Input → WebSocket → Real-time Response → UI Update
```

### 2. API Gateway Layer

**WebSocket API:**
- Manages persistent connections
- Routes messages to Lambda
- Handles connection lifecycle

**REST API:**
- Cognito JWT authentication
- CORS configuration
- Rate limiting and throttling

### 3. Lambda Functions (Compute Layer)

#### chatResponseHandler (Node.js)
- **Purpose**: Orchestrates Bedrock Agent calls
- **Timeout**: 120s
- **Memory**: 1024MB
- **Key Functions**:
  - Invoke Bedrock Agent
  - Stream responses
  - Log to DynamoDB

#### websocketHandler (Python)
- **Purpose**: WebSocket connection management
- **Invokes**: chatResponseHandler
- **Handles**: Connect, disconnect, send

#### email/NotifyAdminFn (Python)
- **Purpose**: Admin email notifications
- **Triggers**: Bedrock action group
- **Integrates**: SES, DynamoDB

#### adminFile (Python)
- **Purpose**: File upload/download/delete
- **Integrates**: S3, Bedrock KB sync

#### kb-sync (Python)
- **Purpose**: Auto-sync Knowledge Base
- **Trigger**: S3 event notification
- **Process**: Start Bedrock ingestion job

#### sessionLogs (Python)
- **Purpose**: Daily analytics export
- **Trigger**: EventBridge (cron)
- **Process**: CloudWatch Logs → S3 → DynamoDB

### 4. AI Layer (AWS Bedrock)

**Bedrock Agent Configuration:**
```typescript
{
  model: Claude 4 Sonnet v1.0,
  crossRegion: US inference profile,
  userInputEnabled: true,
  guardrails: {
    inputFiltering: HIGH,
    outputFiltering: MEDIUM
  },
  actionGroups: ['notify-admin'],
  knowledgeBases: [kb]
}
```

**Knowledge Base:**
- **Embeddings**: Titan v2 (1024 dimensions)
- **Vector Store**: OpenSearch Serverless
- **Parsing**: Bedrock Data Automation
- **Data Source**: S3 (national-council bucket)

**RAG Pipeline:**
```
1. User query → Embedding
2. Vector similarity search (OpenSearch)
3. Retrieve top-k documents (k=5)
4. Context injection to Claude
5. Generate response with citations
```

### 5. Data Layer

**S3 Buckets:**
1. **national-council** - Knowledge Base documents
2. **EmailStorageBucket** - Incoming SES emails
3. **SupplementalDataBucket** - Bedrock multimodal data
4. **DashboardLogsBucket** - Exported CloudWatch logs

**DynamoDB Tables:**
1. **NCMWDashboardSessionlogs**
   - Partition: session_id
   - Sort: timestamp
   - Purpose: Chat history

2. **NCMWEscalatedQueries**
   - Partition: query_id
   - Sort: timestamp
   - GSI: StatusIndex
   - Purpose: Admin follow-ups

3. **NCMWUserProfiles**
   - Partition: userId
   - Purpose: User preferences

**OpenSearch Serverless:**
- Auto-created by Bedrock KB
- Stores vector embeddings
- Handles similarity search

### 6. Security Layer

**Authentication Flow:**
```
User → Cognito User Pool → JWT Token
  ↓
Identity Pool → Temporary AWS Credentials
  ↓
API Gateway Authorizer → Lambda
```

**IAM Roles:**
- **BedrockAgentRole**: S3 read, Bedrock invoke
- **LambdaExecutionRole**: DynamoDB, S3, SES access
- **CognitoAuthenticatedRole**: S3 upload only

**Encryption:**
- S3: Server-side encryption (SSE-S3)
- DynamoDB: Encryption at rest
- API Gateway: TLS 1.2+
- Secrets Manager: Encrypted secrets

### 7. Monitoring & Observability

**CloudWatch:**
- Lambda function logs
- API Gateway access logs
- Custom metrics (response time, errors)

**EventBridge:**
- Daily log export (11:59 PM UTC)
- Custom event rules
- Lambda triggers

### 8. CI/CD Pipeline

**CodeBuild Process:**
```
1. Git push to main
2. CodeBuild triggered
3. Install dependencies (npm, pip)
4. CDK synth & deploy backend
5. Build React frontend
6. Deploy to Amplify
7. Output deployment info
```

**Build Phases:**
- Install: Node.js 20, Python 3.12, CDK
- Pre-build: Validate AWS credentials
- Build: Deploy CDK stack
- Post-build: Build & deploy frontend

---

## Performance Characteristics

### Latency
- Cold start: ~500ms
- Warm: ~50ms
- Bedrock response: 1-3s
- Full round-trip: <5s

### Scalability
- Lambda: 1000 concurrent executions
- DynamoDB: On-demand (unlimited)
- API Gateway: 10,000 RPS
- Bedrock: Regional quotas apply

### Availability
- Multi-AZ deployment
- Cross-region inference (Bedrock)
- 99.9% SLA (target)

---

## Cost Optimization

**Strategies:**
1. Lambda: Right-sized memory allocation
2. DynamoDB: On-demand billing
3. S3: Lifecycle policies (Glacier transition)
4. CloudWatch: 30-day log retention
5. Bedrock: Cross-region for availability (no extra cost)

**Estimated Monthly Cost:**
- Lambda: $50-100
- Bedrock: $200-500 (usage-based)
- DynamoDB: $20-50
- S3: $10-30
- Other services: $50-100
- **Total**: ~$330-780/month

---

## Future Enhancements

1. **Multi-language Support**: Integrate Amazon Translate
2. **Voice Input/Output**: Add Polly and Transcribe
3. **Advanced Analytics**: QuickSight dashboards
4. **Mobile App**: React Native application
5. **Caching Layer**: ElastiCache for responses
6. **CDN**: CloudFront for global delivery

---

For service connections and workflows, see [TECHNICAL_DOCUMENTATION.md](../TECHNICAL_DOCUMENTATION.md)
For deployment procedures, see [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
