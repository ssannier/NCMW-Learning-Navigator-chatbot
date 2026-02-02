# Modification Guide

## Customizing Learning Navigator

### 1. Changing the AI Model

**Location:** `cdk_backend/lib/cdk_backend-stack.ts`

```typescript
// Current: Claude 4 Sonnet
const CONFIG = {
  BEDROCK_AGENT_MODEL: bedrock.BedrockFoundationModel.ANTHROPIC_CLAUDE_4_SONNET_V1_0,
  // ...
};

// Change to Claude Opus:
BEDROCK_AGENT_MODEL: bedrock.BedrockFoundationModel.ANTHROPIC_CLAUDE_OPUS_V2_0,

// Change to Claude Haiku (faster, cheaper):
BEDROCK_AGENT_MODEL: bedrock.BedrockFoundationModel.ANTHROPIC_CLAUDE_HAIKU_V1_0,
```

**Re-deploy:**
```bash
cdk deploy
```

---

### 2. Modifying Agent Instructions

**Location:** `cdk_backend/lib/cdk_backend-stack.ts` (line ~183)

```typescript
const prompt_for_agent = `
You are Learning Navigator, an AI-powered assistant...

// ADD CUSTOM INSTRUCTIONS HERE
// Example:
// - Always respond in a friendly, conversational tone
// - Prioritize instructor questions over learner questions
// - Include motivational quotes in responses
`;
```

---

### 3. Adding New Lambda Functions

**Step 1: Create Lambda Directory**
```bash
mkdir cdk_backend/lambda/myNewFunction
cd cdk_backend/lambda/myNewFunction
```

**Step 2: Create Handler**
```python
# handler.py
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Hello'})
    }
```

**Step 3: Add to CDK Stack**
```typescript
const myNewFn = new lambda.Function(this, 'MyNewFunction', {
  runtime: lambda.Runtime.PYTHON_3_12,
  handler: 'handler.lambda_handler',
  code: lambda.Code.fromAsset('lambda/myNewFunction'),
  timeout: cdk.Duration.seconds(30),
});
```

**Step 4: Add API Endpoint**
```typescript
const myResource = AdminApi.root.addResource('my-endpoint');
myResource.addMethod('GET', new apigateway.LambdaIntegration(myNewFn), {
  authorizer: userPoolAuthorizer,
  authorizationType: apigateway.AuthorizationType.COGNITO,
});
```

---

### 4. Adding New DynamoDB Tables

```typescript
const myTable = new dynamodb.Table(this, 'MyTable', {
  tableName: 'MyTableName',
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'timestamp', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});

// Grant Lambda access
myTable.grantReadWriteData(myNewFn);
```

---

### 5. Customizing Frontend

**Change Theme Colors**
```css
/* frontend/src/App.css */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --background: #f8f9fa;
}
```

**Add New Components**
```bash
cd frontend/src/components
mkdir MyComponent
cd MyComponent
touch MyComponent.jsx MyComponent.css
```

```jsx
// MyComponent.jsx
import React from 'react';
import './MyComponent.css';

export default function MyComponent() {
  return <div>My Custom Component</div>;
}
```

---

### 6. Adding Knowledge Base Sources

**Add New S3 Bucket**
```typescript
const newDocsBucket = new s3.Bucket(this, 'NewDocsBucket', {
  enforceSSL: true,
  encryption: s3.BucketEncryption.S3_MANAGED,
});

const newDataSource = new bedrock.S3DataSource(this, 'NewDataSource', {
  bucket: newDocsBucket,
  knowledgeBase: kb,
  parsingStrategy: bedrock.ParsingStrategy.bedrockDataAutomation(),
});
```

---

### 7. Modifying Guardrails

**Change Content Filtering Levels**
```typescript
const DEFAULT_INPUT = bedrock.ContentFilterStrength.MEDIUM; // Was HIGH
const DEFAULT_OUTPUT = bedrock.ContentFilterStrength.LOW;    // Was MEDIUM
```

**Add Custom Topics**
```typescript
guardrail.addContextualGroundingPolicy({
  threshold: 0.8,
  filters: [
    bedrock.ContextualGroundingFilter.groundednessFilter(),
    bedrock.ContextualGroundingFilter.relevanceFilter(),
  ],
});
```

---

### 8. Adding Email Templates

**Location:** `cdk_backend/lambda/email/handler.py`

```python
def create_email_html(query, email, response):
    return f"""
    <html>
      <body>
        <h1>Custom Email Template</h1>
        <p>User asked: {query}</p>
        <p>Contact: {email}</p>
        <p>Partial answer: {response}</p>
      </body>
    </html>
    """
```

---

### 9. Adding Analytics Events

```typescript
// Add EventBridge rule
const customRule = new events.Rule(this, 'CustomAnalyticsRule', {
  schedule: events.Schedule.cron({
    minute: '0',
    hour: '12', // Daily at noon
  }),
});

customRule.addTarget(new targets.LambdaFunction(myAnalyticsFn));
```

---

### 10. Environment-Specific Configuration

**Add Context Parameters**
```bash
cdk deploy \
  -c environment=production \
  -c logLevel=error \
  -c enableDebug=false
```

**Use in Stack**
```typescript
const environment = this.node.tryGetContext('environment') || 'development';
const logLevel = this.node.tryGetContext('logLevel') || 'info';

// Conditional resources
if (environment === 'production') {
  // Enable WAF, CloudFront, etc.
}
```

---

## Testing Changes

### Local Testing
```bash
# Test Lambda locally
cd cdk_backend/lambda/myFunction
python handler.py

# Test CDK synth
cd cdk_backend
cdk synth

# Test frontend locally
cd frontend
npm start
```

### Integration Testing
```bash
# Deploy to dev environment
cdk deploy -c environment=dev

# Run integration tests
npm run test:integration
```

---

## Best Practices

1. **Always test locally first**
2. **Use environment variables for configuration**
3. **Add comprehensive comments**
4. **Follow existing code patterns**
5. **Update documentation**
6. **Create git branches for changes**
7. **Test in dev before production**

---

## Common Customizations

### Change Response Timeout
```typescript
const chatResponseHandler = new lambda.Function(this, 'ChatHandler', {
  timeout: cdk.Duration.seconds(180), // Increased from 120
});
```

### Add Custom Logging
```typescript
myFn.addEnvironment('LOG_LEVEL', 'DEBUG');
```

### Change DynamoDB Capacity
```typescript
myTable = new dynamodb.Table(this, 'MyTable', {
  billingMode: dynamodb.BillingMode.PROVISIONED,
  readCapacity: 5,
  writeCapacity: 5,
});
```

---

For architecture details, see [Architecture Deep Dive](ARCHITECTURE_DEEP_DIVE.md)
