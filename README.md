# OshoBot Telegram - Enhanced AI Chatbot

## 🧘‍♂️ Overview

OshoBot is an AI-powered Telegram chatbot that responds to users in the voice and wisdom of Osho (the spiritual teacher). This enhanced version includes performance optimizations, advanced analytics, and improved user experience features.

## ✨ Key Features

### Core Functionality
- **AI-Powered Responses**: Uses GPT-4 with Osho's wisdom from Pinecone vector database
- **Multi-language Support**: Hindi/English language detection and responses
- **User Management**: Comprehensive user tracking and subscription management
- **Payment Integration**: Razorpay integration for premium access

### New Enhancements
- **Performance Monitoring**: Response time tracking and rate limiting
- **Message Categorization**: Automatic categorization of user questions
- **Advanced Analytics**: Enhanced logging and user behavior insights
- **Rate Limiting**: Prevents abuse and improves system stability

## 🏗️ Architecture

### Workflow Components
1. **Telegram Trigger** → **Start Timer** → **Rate Limit Check** → **User_DB**
2. **Embed Query** → **Pinecone Query** → **Categorize Message** → **Build Prompt**
3. **AI Agent** → **Calculate Response Time** → **Send Message** → **Update User**

### Data Flow
```
User Message → Authentication → Rate Limiting → AI Processing → Response → Analytics
```

## 📊 Analytics & Monitoring

### Performance Metrics
- **Response Time**: Tracks AI response latency in milliseconds
- **User Engagement**: Message frequency and session duration
- **Content Performance**: Question categories and user satisfaction
- **System Health**: Rate limiting and error tracking

### Data Collection
- **User Behavior**: Message patterns, preferred topics, session data
- **Content Insights**: Most requested topics, seasonal patterns
- **Business Metrics**: Conversion rates, churn analysis, revenue tracking

## 🚀 Quick Start

### Prerequisites
- N8N instance (self-hosted or cloud)
- OpenAI API key
- Pinecone vector database
- Telegram Bot API token
- Razorpay account
- Google Sheets API access

### Installation
1. Import the `OshoBot_telegram_enhanced.json` workflow
2. Configure credentials for all external services
3. Set up webhook endpoints for Telegram and Razorpay
4. Deploy and activate the workflow

### Configuration
```json
{
  "openai_api_key": "your_openai_key",
  "pinecone_api_key": "your_pinecone_key",
  "telegram_bot_token": "your_bot_token",
  "razorpay_key": "your_razorpay_key"
}
```

## 📈 Business Model

### Freemium Structure
- **Free Tier**: 5 questions per user
- **Premium**: ₹300/month (~$4) for unlimited access
- **Features**: Full AI responses, priority support, advanced features

### Revenue Streams
- Monthly subscriptions
- Premium content access
- Enterprise partnerships

## 🔧 Technical Details

### API Integrations
- **OpenAI**: GPT-4 for AI responses and embeddings
- **Pinecone**: Vector similarity search for context
- **Telegram**: Bot API for messaging
- **Razorpay**: Payment processing
- **Google Sheets**: User database and analytics

### Performance Optimizations
- **Rate Limiting**: 3-second minimum interval between messages
- **Caching**: User data caching to reduce API calls
- **Batch Processing**: Efficient database operations
- **Error Handling**: Graceful fallbacks and user feedback

## 📊 Analytics Dashboard

### Key Metrics
- **Daily Active Users**: Unique users per day
- **Response Quality**: User satisfaction and follow-up rates
- **Revenue Metrics**: Conversion rates and subscription growth
- **Content Performance**: Most engaging topics and responses

### Data Sources
- Google Sheets (Users, PromptLog, PaymentRequest)
- PostgreSQL (Chat memory and session data)
- Telegram API (User interaction data)
- Razorpay (Payment and subscription data)

## 🛠️ Maintenance & Monitoring

### Daily Tasks
- Monitor response times and error rates
- Check user engagement metrics
- Review payment processing status

### Weekly Tasks
- Analyze user behavior patterns
- Review content performance
- Update rate limiting parameters

### Monthly Tasks
- Revenue analysis and forecasting
- User retention optimization
- Content strategy refinement

## 🔒 Security & Privacy

### Data Protection
- User data encryption in transit and at rest
- GDPR compliance for EU users
- Secure API key management
- Regular security audits

### Access Control
- Role-based access to analytics
- API rate limiting and abuse prevention
- Secure webhook endpoints

## 📚 API Documentation

### Endpoints
- `POST /telegram-webhook`: Telegram message processing
- `POST /razorpay-webhook`: Payment confirmation
- `GET /analytics`: Performance metrics and insights

### Webhook Payloads
```json
// Telegram Webhook
{
  "message": {
    "text": "User question",
    "from": { "id": 123456, "first_name": "John" },
    "chat": { "id": 123456 }
  }
}

// Razorpay Webhook
{
  "payload": {
    "payment": {
      "entity": {
        "status": "captured",
        "notes": { "telegram_id": "123456" }
      }
    }
  }
}
```

## 🐛 Troubleshooting

### Common Issues
1. **Rate Limiting Errors**: User sending messages too quickly
2. **API Timeouts**: External service delays
3. **Payment Failures**: Razorpay integration issues
4. **Memory Issues**: PostgreSQL connection problems

### Debug Steps
1. Check workflow execution logs
2. Verify API credentials and quotas
3. Monitor external service status
4. Review user feedback and error reports

## 🤝 Contributing

### Development Guidelines
- Follow N8N best practices
- Add comprehensive error handling
- Include performance monitoring
- Document all new features

### Testing
- Test with various user scenarios
- Validate payment flows
- Monitor performance under load
- Verify analytics accuracy

## 📞 Support

### Contact Information
- **Developer**: Tanay Nandi
- **Email**: tanay.nandi87@gmail.com
- **Technical Issues**: Check N8N logs and workflow execution
- **Business Questions**: Review analytics dashboard
- **Feature Requests**: Submit through project repository

### Resources
- [N8N Documentation](https://docs.n8n.io/)
- [OpenAI API Reference](https://platform.openai.com/docs)
- [Pinecone Documentation](https://docs.pinecone.io/)
- [Telegram Bot API](https://core.telegram.org/bots/api)

## 📄 License

This project is proprietary software. All rights reserved.

---

**Built with ❤️ and 🧘‍♂️ for spiritual seekers worldwide**
