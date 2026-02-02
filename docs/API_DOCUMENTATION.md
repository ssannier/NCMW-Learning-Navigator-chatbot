# API Documentation

## Authentication

All REST API endpoints require JWT authentication via AWS Cognito.

**Headers Required:**
```
Authorization: Bearer <JWT_ID_TOKEN>
Content-Type: application/json
```

---

## REST API Endpoints

### Base URL
```
https://<api-id>.execute-api.<region>.amazonaws.com/prod
```

### Files Management

#### List All Files
```http
GET /files
```

**Response:**
```json
{
  "files": [
    {
      "key": "pdfs/document.pdf",
      "size": 1048576,
      "lastModified": "2026-02-01T10:00:00Z"
    }
  ]
}
```

#### Upload File
```http
POST /files
Content-Type: multipart/form-data

file: <binary>
```

#### Get File
```http
GET /files/{key}
```

#### Delete File
```http
DELETE /files/{key}
```

#### Get Presigned URL
```http
POST /presigned-url
{
  "key": "pdfs/document.pdf"
}
```

**Response:**
```json
{
  "url": "https://s3.amazonaws.com/...",
  "expiresIn": 3600
}
```

### Knowledge Base Operations

#### Manual Sync
```http
POST /sync
```

**Response:**
```json
{
  "jobId": "abc-123",
  "status": "IN_PROGRESS"
}
```

### Session Logs

#### Get All Sessions
```http
GET /session-logs
```

**Query Parameters:**
- `limit` - Number of records (default: 100)
- `startDate` - ISO date string
- `endDate` - ISO date string

**Response:**
```json
{
  "sessions": [
    {
      "session_id": "sess_123",
      "timestamp": "2026-02-01T10:00:00Z",
      "user_id": "user_456",
      "message_count": 5,
      "feedback": "positive"
    }
  ]
}
```

#### Get Single Session
```http
GET /session-logs/{sessionId}
```

### Escalated Queries

#### List Escalated Queries
```http
GET /escalated-queries?status=pending
```

**Response:**
```json
{
  "queries": [
    {
      "query_id": "q_123",
      "email": "user@example.com",
      "query": "Complex question...",
      "timestamp": "2026-02-01T10:00:00Z",
      "status": "pending",
      "agent_response": "Partial answer..."
    }
  ]
}
```

#### Get Single Query
```http
GET /escalated-queries/{queryId}
```

#### Update Query Status
```http
PUT /escalated-queries
{
  "query_id": "q_123",
  "status": "resolved"
}
```

### User Profiles

#### Get User Profile
```http
GET /user-profile
```

#### Create/Update Profile
```http
POST /user-profile
{
  "role": "instructor",
  "preferences": {
    "language": "en",
    "notifications": true
  }
}
```

#### Get Recommendations
```http
GET /recommendations
```

**Response:**
```json
{
  "recommendations": [
    {
      "title": "Youth MHFA Training",
      "description": "...",
      "relevance": 0.95
    }
  ]
}
```

---

## WebSocket API

### Connection URL
```
wss://<api-id>.execute-api.<region>.amazonaws.com/production
```

### Connect
```javascript
const ws = new WebSocket('wss://...');

ws.onopen = () => {
  console.log('Connected');
};
```

### Send Message
```javascript
ws.send(JSON.stringify({
  action: 'sendMessage',
  message: 'What are MHFA requirements?'
}));
```

### Receive Messages
```javascript
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Response:', data.message);
};
```

### Message Format
```json
{
  "action": "sendMessage",
  "message": "User question",
  "session_id": "optional_session_id"
}
```

### Response Format
```json
{
  "message": "AI response text",
  "citations": ["doc1.pdf", "doc2.pdf"],
  "confidence": 95,
  "session_id": "sess_123"
}
```

---

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Bad Request - Invalid input |
| 401 | Unauthorized - Invalid/missing JWT |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 500 | Internal Server Error |

---

## Rate Limits

- WebSocket: 100 messages/minute per connection
- REST API: 1000 requests/minute per user
- File Upload: 10 files/minute, max 50MB each

---

## SDK Examples

### JavaScript/TypeScript
```typescript
import { CognitoIdentityClient } from "@aws-sdk/client-cognito-identity";
import axios from 'axios';

// Get JWT token from Cognito
const token = await Auth.currentSession().getIdToken().getJwtToken();

// Call API
const response = await axios.get('https://api.../files', {
  headers: { Authorization: `Bearer ${token}` }
});
```

### Python
```python
import boto3
import requests

# Get JWT token
cognito = boto3.client('cognito-idp')
response = cognito.initiate_auth(...)

# Call API
headers = {'Authorization': f'Bearer {token}'}
response = requests.get('https://api.../files', headers=headers)
```

---

For implementation examples, see [Modification Guide](MODIFICATION_GUIDE.md)
