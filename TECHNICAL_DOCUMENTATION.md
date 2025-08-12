# OshoBot Technical Documentation

## 🏗️ System Architecture

### High-Level Overview
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Telegram      │    │      N8N        │    │   External      │
│     Bot         │◄──►│   Workflow      │◄──►│    APIs         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   Data Storage  │
                       │   (Sheets/DB)   │
                       └─────────────────┘
```

### Component Breakdown

#### 1. Input Layer
- **Telegram Trigger**: Webhook endpoint for incoming messages
- **Rate Limiting**: Prevents abuse and ensures system stability
- **User Authentication**: Validates user access and subscription status

#### 2. Processing Layer
- **AI Processing**: OpenAI embeddings and GPT-4 responses
- **Vector Search**: Pinecone similarity search for context
- **Message Categorization**: Automatic topic classification
- **Response Generation**: Context-aware AI responses

#### 3. Output Layer
- **Message Delivery**: Telegram API for user communication
- **Data Logging**: Comprehensive analytics and monitoring
- **User Updates**: Database synchronization and tracking

## 🔧 Node Configuration Details

### Core Nodes

#### Telegram Trigger
```json
{
  "type": "n8n-nodes-base.telegramTrigger",
  "parameters": {
    "updates": ["message"],
    "additionalFields": {}
  }
}
```
**Purpose**: Entry point for all user interactions
**Webhook ID**: `9b495821-ee68-4880-8e9e-5a6061e43480`

#### Start Timer
```json
{
  "type": "n8n-nodes-base.set",
  "parameters": {
    "assignments": [
      {
        "name": "start_time",
        "value": "={{ $now }}"
      }
    ]
  }
}
```
**Purpose**: Performance monitoring start point
**Output**: Timestamp for response time calculation

#### Rate Limit Check
```json
{
  "type": "n8n-nodes-base.code",
  "parameters": {
    "jsCode": "// Basic rate limiting - max 1 message per 3 seconds\nconst lastMessageTime = $json['Last Message Time'] || 0;\nconst now = Date.now();\nconst timeDiff = now - lastMessageTime;\nconst minInterval = 3000; // 3 seconds between messages\n\nif (timeDiff < minInterval) {\n  throw new Error('Rate limit exceeded. Please wait a few seconds.');\n}\n\nreturn [{ json: { ...$json, can_proceed: true, last_message_time: now } }];"
  }
}
```
**Purpose**: Prevents message spam and abuse
**Rate Limit**: 3 seconds minimum between messages
**Error Handling**: Throws descriptive error messages

#### Message Categorizer
```json
{
  "type": "n8n-nodes-base.code",
  "parameters": {
    "jsCode": "// Categorize user message for analytics\nconst question = $('Telegram Trigger').item.json.message.text.toLowerCase();\nlet category = 'general';\n\nif (question.includes('love') || question.includes('relationship') || question.includes('marriage')) {\n  category = 'love';\n} else if (question.includes('meditation') || question.includes('zen') || question.includes('mindfulness')) {\n  category = 'meditation';\n} else if (question.includes('life') || question.includes('meaning') || question.includes('purpose')) {\n  category = 'life_meaning';\n} else if (question.includes('pain') || question.includes('suffering') || question.includes('grief')) {\n  category = 'pain_suffering';\n} else if (question.includes('ego') || question.includes('self') || question.includes('identity')) {\n  category = 'ego_identity';\n} else if (question.includes('death') || question.includes('dying') || question.includes('mortality')) {\n  category = 'death_mortality';\n}\n\nreturn [{ json: { ...$json, message_category: category } }];"
  }
}
```
**Purpose**: Automatic topic classification for analytics
**Categories**: love, meditation, life_meaning, pain_suffering, ego_identity, death_mortality
**Fallback**: 'general' for uncategorized messages

#### Calculate Response Time
```json
{
  "type": "n8n-nodes-base.code",
  "parameters": {
    "jsCode": "const startTime = new Date($('Start Timer').item.json.start_time);\nconst endTime = new Date();\nconst responseTime = endTime - startTime;\n\nreturn [{\n  json: {\n    ...$json,\n    response_time_ms: responseTime,\n    response_time_seconds: Math.round(responseTime / 1000),\n    start_time: $('Start Timer').item.json.start_time,\n    end_time: $now\n  }\n}];"
  }
}
```
**Purpose**: Performance monitoring and optimization
**Metrics**: Response time in milliseconds and seconds
**Data**: Start and end timestamps for detailed analysis

## 📊 Data Flow & Connections

### Primary Workflow Path
```
Telegram Trigger → Start Timer → Rate Limit Check → User_DB → Is New User?
                                                              ↓
                                                          Check Prompt Limit → Embed Query → Pinecone Query
                                                              ↓
                                                          Create User → Send Welcome Message
```

### AI Processing Path
```
Pinecone Query → Categorize Message → Build Prompt → AI Agent → Calculate Response Time → Send Message
```

### Analytics Path
```
Send Message → Update User → Append to PromptLog → Analytics Dashboard
```

## 🗄️ Data Storage Schema

### Google Sheets Structure

#### Users Sheet
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| Telegram ID | String | Unique user identifier | `123456789` |
| Username | String | User's display name | `John Doe` |
| Message Count | Number | Total messages sent | `15` |
| Paid | String | Payment status | `1` or `0` |
| Paid On | Date | Payment date | `2024-01-15` |
| Last Message Time | Timestamp | Last interaction | `1705315200000` |

#### PromptLog Sheet
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| Platform | String | Source platform | `Telegram` |
| ID | String | User identifier | `123456789` |
| Name | String | User's name | `John Doe` |
| Prompt | String | User's question | `What is love?` |
| Time | Timestamp | Question time | `2024-01-15 10:30:00` |
| Response | String | AI response | `Love is...` |
| Response_Time_MS | Number | Processing time | `2500` |
| Message_Category | String | Topic category | `love` |
| User_Type | String | User tier | `free` or `premium` |
| Session_ID | String | Session identifier | `123456789_1705315200` |

#### PaymentRequest Sheet
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| Platform | String | Source platform | `Telegram` |
| Time | Timestamp | Request time | `2024-01-15 10:30:00` |
| ID | String | User identifier | `123456789` |

### PostgreSQL Schema

#### Chat Memory Table
```sql
CREATE TABLE chat_memory (
    session_id VARCHAR(255) PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    messages JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 🔐 Security Implementation

### API Key Management
- **OpenAI**: Stored in N8N credentials with restricted access
- **Pinecone**: API key with read-only permissions
- **Telegram**: Bot token with message permissions only
- **Razorpay**: Live API credentials for production

### Rate Limiting Strategy
```javascript
// Rate limiting implementation
const minInterval = 3000; // 3 seconds
const maxMessagesPerHour = 100; // Per user
const maxConcurrentUsers = 50; // System-wide
```

### Error Handling
```javascript
try {
  // Process user message
  const result = await processMessage(message);
  return result;
} catch (error) {
  if (error.message.includes('Rate limit exceeded')) {
    return { error: 'Please wait before sending another message' };
  }
  if (error.message.includes('API quota exceeded')) {
    return { error: 'Service temporarily unavailable' };
  }
  // Log error for monitoring
  logError(error);
  return { error: 'Something went wrong. Please try again.' };
}
```

## 📈 Performance Monitoring

### Key Metrics
1. **Response Time**: Target < 5 seconds
2. **Success Rate**: Target > 99%
3. **User Engagement**: Daily active users
4. **API Usage**: OpenAI and Pinecone quotas

### Monitoring Dashboard
```javascript
// Performance metrics calculation
const metrics = {
  avgResponseTime: calculateAverageResponseTime(),
  successRate: calculateSuccessRate(),
  activeUsers: getDailyActiveUsers(),
  apiUsage: getAPIUsageStats(),
  errorRate: calculateErrorRate()
};
```

### Alerting
- **High Response Time**: > 10 seconds
- **High Error Rate**: > 5%
- **API Quota Warning**: > 80% used
- **System Down**: No successful executions in 5 minutes

## 🚀 Deployment Guide

### Prerequisites
1. **N8N Instance**: Version 1.0+ with Node.js 18+
2. **External Services**: All API keys and credentials
3. **Database**: PostgreSQL instance for chat memory
4. **Storage**: Google Sheets API access

### Environment Variables
```bash
# N8N Configuration
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=secure_password

# Database Configuration
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=osho_bot
DB_POSTGRESDB_USER=osho_user
DB_POSTGRESDB_PASSWORD=secure_password

# External API Limits
OPENAI_RATE_LIMIT=100
PINECONE_RATE_LIMIT=1000
TELEGRAM_RATE_LIMIT=30
```

### Deployment Steps
1. **Import Workflow**: Load JSON file into N8N
2. **Configure Credentials**: Set up all external service credentials
3. **Test Connections**: Verify all API endpoints are accessible
4. **Activate Workflow**: Enable the workflow for production
5. **Monitor Performance**: Set up monitoring and alerting

### Scaling Considerations
- **Horizontal Scaling**: Multiple N8N instances behind load balancer
- **Database Scaling**: Read replicas for analytics queries
- **Caching**: Redis for frequently accessed user data
- **CDN**: For static assets and media files

## 🧪 Testing Strategy

### Unit Testing
- **Node Functions**: Test individual code nodes
- **API Integrations**: Mock external service responses
- **Data Transformations**: Validate data flow between nodes

### Integration Testing
- **End-to-End Flows**: Complete user interaction scenarios
- **Payment Processing**: Test Razorpay integration
- **Error Handling**: Validate error scenarios and recovery

### Load Testing
- **Concurrent Users**: Test with 100+ simultaneous users
- **Message Volume**: High-frequency message testing
- **API Limits**: Test rate limiting and quota handling

### Test Scenarios
1. **New User Onboarding**: First-time user experience
2. **Premium Upgrade**: Payment flow and access upgrade
3. **Rate Limiting**: Abuse prevention testing
4. **Error Recovery**: System failure and recovery
5. **Performance**: Response time under load

## 🔄 Maintenance Procedures

### Daily Operations
- **Health Checks**: Monitor workflow execution status
- **Error Review**: Analyze failed executions and errors
- **Performance Review**: Check response times and success rates
- **User Support**: Address user issues and feedback

### Weekly Operations
- **Analytics Review**: Analyze user behavior and engagement
- **Performance Optimization**: Identify bottlenecks and optimize
- **Content Review**: Analyze question categories and responses
- **Revenue Review**: Monitor payment processing and subscriptions

### Monthly Operations
- **System Updates**: Update N8N and dependencies
- **Security Review**: Audit access and permissions
- **Capacity Planning**: Plan for growth and scaling
- **Business Review**: Revenue analysis and strategy

## 🐛 Troubleshooting Guide

### Common Issues

#### 1. Rate Limiting Errors
**Symptoms**: Users receive "Rate limit exceeded" messages
**Causes**: Users sending messages too quickly
**Solutions**: 
- Adjust rate limiting parameters
- Implement user education about limits
- Add progressive rate limiting

#### 2. API Timeouts
**Symptoms**: Long response times or failures
**Causes**: External API delays or quotas exceeded
**Solutions**:
- Implement retry logic with exponential backoff
- Add fallback responses
- Monitor API quotas and usage

#### 3. Database Connection Issues
**Symptoms**: Chat memory not working or errors
**Causes**: PostgreSQL connection problems
**Solutions**:
- Check database connectivity
- Verify connection pool settings
- Implement connection retry logic

#### 4. Payment Processing Failures
**Symptoms**: Users can't upgrade to premium
**Causes**: Razorpay integration issues
**Solutions**:
- Verify webhook endpoints
- Check payment link generation
- Validate webhook signatures

### Debug Procedures
1. **Check Execution Logs**: Review N8N workflow execution history
2. **Verify Credentials**: Ensure all API keys are valid and active
3. **Test External Services**: Verify API endpoints are accessible
4. **Monitor Resources**: Check system resources and database performance
5. **User Feedback**: Collect and analyze user error reports

### Performance Optimization
1. **Database Queries**: Optimize Google Sheets operations
2. **API Calls**: Implement caching and batching
3. **Response Generation**: Optimize AI prompt processing
4. **Error Handling**: Reduce unnecessary retries and timeouts

## 📚 API Reference

### Internal Endpoints

#### Telegram Webhook
- **URL**: `/webhook/telegram`
- **Method**: POST
- **Purpose**: Process incoming Telegram messages
- **Authentication**: Webhook signature validation

#### Razorpay Webhook
- **URL**: `/webhook/razorpay`
- **Method**: POST
- **Purpose**: Process payment confirmations
- **Authentication**: Webhook signature validation

### External API Usage

#### OpenAI API
- **Embeddings**: `text-embedding-ada-002` model
- **Chat**: `gpt-4` model
- **Rate Limits**: 100 requests per minute
- **Costs**: ~$0.002 per 1K tokens

#### Pinecone API
- **Index**: `osho-works-1536`
- **Dimensions**: 1536
- **Top K**: 1 (configurable)
- **Rate Limits**: 1000 requests per minute

#### Telegram Bot API
- **Methods**: sendMessage, getUpdates
- **Rate Limits**: 30 messages per second
- **Webhook**: Long polling fallback

#### Razorpay API
- **Payment Links**: Dynamic link generation
- **Webhooks**: Payment status updates
- **Rate Limits**: 100 requests per minute

## 🔮 Future Enhancements

### Planned Features
1. **Multi-language Support**: Hindi, English, and other languages
2. **Voice Messages**: Speech-to-text and text-to-speech
3. **Advanced Analytics**: Machine learning insights and predictions
4. **Personalization**: User preference learning and customization
5. **Enterprise Features**: White-label solutions and API access

### Technical Improvements
1. **Microservices Architecture**: Break down into smaller services
2. **Real-time Analytics**: Live dashboard and monitoring
3. **Advanced Caching**: Redis and CDN implementation
4. **Machine Learning**: Automated content optimization
5. **Security Enhancements**: Advanced encryption and authentication

### Scalability Plans
1. **Global Deployment**: Multi-region deployment
2. **Load Balancing**: Advanced traffic management
3. **Database Sharding**: Horizontal database scaling
4. **Content Delivery**: Global CDN and edge computing
5. **Monitoring**: Advanced observability and alerting

---

**Document Version**: 1.0  
**Last Updated**: August 2025  
**Maintainer**: Tanay

