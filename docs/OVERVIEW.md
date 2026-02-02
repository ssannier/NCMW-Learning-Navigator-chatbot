# Overview

## Learning Navigator - MHFA Learning Ecosystem AI Assistant

### What is Learning Navigator?

Learning Navigator is an enterprise-grade AI chatbot designed to support the Mental Health First Aid (MHFA) Learning Ecosystem. Built on AWS Bedrock's Claude 4 Sonnet, it provides intelligent, context-aware assistance to instructors, learners, and administrators.

### Key Features

🤖 **AI-Powered Conversations**
- Claude 4 Sonnet for natural, human-like responses
- Context-aware answers based on MHFA training materials
- Real-time streaming responses via WebSocket

📚 **Intelligent Knowledge Base**
- Retrieval-Augmented Generation (RAG) for accurate answers
- Automatic document indexing when new materials are uploaded
- Semantic search across PDFs, documents, and FAQs

🔐 **Secure & Scalable**
- AWS Cognito authentication with MFA support
- Serverless architecture that scales automatically
- Content guardrails to ensure safe, on-topic responses

📊 **Admin Dashboard**
- Real-time analytics and session logs
- Document management and Knowledge Base updates
- User profile management and escalated query tracking

📧 **Smart Escalation**
- Automatic email notifications for complex queries
- Admin follow-up system for unanswered questions
- Query confidence scoring

### Who Uses Learning Navigator?

**Instructors**
- Find teaching resources and course materials
- Get guidance on certification requirements
- Access training best practices

**Learners**
- Navigate course requirements
- Find answers to common questions
- Get personalized learning recommendations

**Administrators**
- Manage content and documents
- Track user engagement and feedback
- Handle escalated queries efficiently

### Technology Stack

**Frontend**
- React 18 with modern hooks
- AWS Amplify hosting
- Real-time WebSocket communication

**Backend**
- AWS Bedrock (Claude 4 Sonnet)
- Lambda functions (Python & Node.js)
- API Gateway (REST + WebSocket)

**Data & Storage**
- Amazon S3 for documents
- DynamoDB for session logs
- OpenSearch for vector search

**Security & Auth**
- AWS Cognito for authentication
- IAM for access control
- Content filtering with Bedrock Guardrails

### System Highlights

⚡ **Performance**
- < 2 second response time
- Real-time streaming responses
- Auto-scaling serverless architecture

🛡️ **Security**
- End-to-end encryption
- Content filtering and guardrails
- Compliance-ready architecture

💰 **Cost-Efficient**
- Pay-per-use pricing
- No idle costs
- Automatic scaling to zero

🔄 **CI/CD Pipeline**
- Automated deployments from GitHub
- Infrastructure as Code (CDK)
- Zero-downtime updates

### Use Cases

**1. Training Support**
```
User: "What are the requirements to become an MHFA instructor?"
Assistant: [Provides detailed requirements with citations from KB]
```

**2. Course Navigation**
```
User: "How do I access my certification?"
Assistant: [Guides through MHFA Connect with direct links]
```

**3. Administrative Guidance**
```
User: "How do I update my instructor profile?"
Assistant: [Step-by-step instructions with form links]
```

**4. Document Search**
```
User: "Find information about youth mental health programs"
Assistant: [Searches KB, provides relevant excerpts with sources]
```

### Architecture Overview

```
User → React Frontend (Amplify)
  ↓
WebSocket/REST API Gateway
  ↓
Lambda Functions
  ↓
Bedrock Agent (Claude 4) → Knowledge Base (RAG)
  ↓
Response with citations
```

### Getting Started

1. **For Users**: Visit the application URL and start chatting
2. **For Admins**: Login to admin dashboard to manage content
3. **For Developers**: See [Deployment Guide](DEPLOYMENT_GUIDE.md)

### Documentation

- [Architecture Diagram](ARCHITECTURE_DIAGRAM.md) - Visual system design
- [Architecture Deep Dive](ARCHITECTURE_DEEP_DIVE.md) - Detailed technical docs
- [Deployment Guide](DEPLOYMENT_GUIDE.md) - Setup instructions
- [User Guide](USER_GUIDE.md) - How to use the application
- [API Documentation](API_DOCUMENTATION.md) - API reference
- [Modification Guide](MODIFICATION_GUIDE.md) - Extend and customize

### Support

- **Technical Issues**: Check [Troubleshooting](../TECHNICAL_DOCUMENTATION.md#7-troubleshooting-guide)
- **Questions**: Open a GitHub issue
- **Security**: Report to admin email

---

**Version**: 1.0
**Last Updated**: February 2026
**Powered by**: AWS Bedrock Claude 4 Sonnet
