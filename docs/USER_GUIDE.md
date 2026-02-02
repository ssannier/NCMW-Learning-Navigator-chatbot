# User Guide

## Getting Started with Learning Navigator

### For End Users (Chat Interface)

#### Accessing the Application
Visit: `https://main.<your-amplify-domain>.amplifyapp.com`

#### Starting a Conversation
1. Type your question in the chat input
2. Press Enter or click Send
3. Receive real-time streaming response
4. View citations and source documents

#### Example Questions
- "What are MHFA certification requirements?"
- "How do I access my course materials?"
- "What's the process to renew my instructor certification?"
- "Find training resources for youth mental health"

#### Features
- **Real-time responses** - See answers as they're generated
- **Citations** - All answers include source references
- **Conversation history** - Previous messages saved in session
- **Feedback** - Thumbs up/down on responses

---

### For Administrators (Dashboard)

#### Accessing Admin Dashboard
1. Navigate to `/admin-dashboard`
2. Login with Cognito credentials
3. Complete MFA if enabled

#### Document Management
**Upload Documents**
```
1. Go to Files section
2. Click "Upload Document"
3. Select PDF/document
4. Wait for auto-sync to complete (~2-5 min)
5. Document now searchable in chat
```

**Delete Documents**
```
1. Go to Files section
2. Select document
3. Click Delete
4. Confirm deletion
```

#### User Management
- View user profiles
- Track user activity
- Manage preferences
- Export user data

#### Analytics & Monitoring
- View session logs
- Check query confidence scores
- Track escalated queries
- Monitor feedback ratings

#### Escalated Queries
**Handling Escalations**
```
1. Check Escalated Queries section
2. View user email and original question
3. Mark as "In Progress" or "Resolved"
4. Respond via email if needed
```

---

## Best Practices

### For Users
- Be specific in questions
- Provide context when needed
- Use feedback buttons to improve responses
- Check cited sources for official information

### For Administrators
- Keep Knowledge Base updated
- Monitor escalated queries daily
- Review analytics weekly
- Update user profiles regularly

---

## Common Tasks

### Syncing Knowledge Base
```bash
aws bedrock-agent start-ingestion-job \
  --knowledge-base-id <KB_ID> \
  --data-source-id <DS_ID>
```

### Creating New Users
```bash
aws cognito-idp admin-create-user \
  --user-pool-id <POOL_ID> \
  --username user@example.com \
  --temporary-password TempPass123!
```

### Viewing Logs
```bash
aws logs tail /aws/lambda/chatResponseHandler --follow
```

---

For API integration, see [API Documentation](API_DOCUMENTATION.md)
