# AI Integration & Deployment Guide

## Quick Start: 5-Minute Integration Overview

OmniChannelAI integrates with existing customer support infrastructure via REST APIs and webhooks. No code changes to existing systems required.

### Architecture Diagram
```
Your CRM                    Your Chat Widget               Your Email System
  (Salesforce)              (Intercom)                     (Gmail/Outlook)
        │                        │                               │
        └────────────────────────┴───────────────────────────────┘
                                 │
                        [Webhook Gateway]
                                 │
                    ┌────────────┴────────────┐
                    │                         │
            [Message Queue]            [OmniChannelAI]
            (Kafka/RabbitMQ)           Core Platform
                    │                         │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
            [Agent Dashboard]          [Analytics API]
            (React/Vue.js)             (Data Export)
```

---

## API Integration Guide

### 1. Receiving Messages

**Endpoint:** `POST /api/v1/messages`

**Authentication:** Bearer token (provided at setup)

**Request:**
```json
{
  "message_id": "msg_abc123",
  "timestamp": "2024-03-15T14:30:00Z",
  "customer_id": "cust_123",
  "channel": "whatsapp|email|chat|twitter|sms",
  "text": "My order hasn't arrived yet",
  "sender": {
    "id": "john@example.com",
    "name": "John Smith",
    "phone": "+1-555-0123"
  },
  "context": {
    "order_id": "ORD-2023-456",
    "thread_id": "conv_789"
  }
}
```

**Response:**
```json
{
  "status": "received",
  "message_id": "msg_abc123",
  "processing_id": "proc_xyz789",
  "estimated_processing_time_ms": 500
}
```

**Integration Examples:**

**Webhook from Twilio (SMS/WhatsApp):**
```
POST /api/v1/messages

{
  "message_id": "SM1234567890abcdef",
  "timestamp": "2024-03-15T14:30:00Z",
  "customer_id": "CUST123",
  "channel": "whatsapp",
  "text": "Hi, I need help with my order",
  "sender": {
    "id": "whatsapp:+15550123",
    "phone": "+1-555-0123"
  },
  "context": {
    "twilio_sid": "SM1234567890abcdef"
  }
}
```

**Webhook from Freshdesk (Email tickets):**
```
POST /api/v1/messages

{
  "message_id": "freshdesk_ticket_12345",
  "timestamp": "2024-03-15T14:30:00Z",
  "customer_id": "FRESHDESK_CUST_789",
  "channel": "email",
  "text": "I have a question about my order #ORD-2023-456",
  "sender": {
    "id": "john@example.com",
    "name": "John Smith",
    "email": "john@example.com"
  },
  "context": {
    "freshdesk_ticket_id": "12345",
    "freshdesk_requester_id": "789"
  }
}
```

**Webhook from custom web chat:**
```
POST /api/v1/messages

{
  "message_id": "chat_conv_123_msg_456",
  "timestamp": "2024-03-15T14:30:00Z",
  "customer_id": "web_user_678",
  "channel": "chat",
  "text": "Can you track my order?",
  "sender": {
    "id": "web_session_abc123",
    "name": "Anonymous"
  },
  "context": {
    "session_id": "web_session_abc123"
  }
}
```

---

### 2. Requesting Recommendations

**Endpoint:** `GET /api/v1/recommendations/{message_id}`

**Purpose:** Get AI recommendations (routing, response, escalation)

**Query Parameters:**
```
?include=routing&include=response&include=escalation
```

**Response:**
```json
{
  "message_id": "msg_abc123",
  "recommendations": {
    "routing": {
      "agent_id": "agent_sarah",
      "agent_name": "Sarah Chen",
      "confidence": 0.87,
      "alternatives": [
        {
          "agent_id": "agent_marcus",
          "confidence": 0.78
        }
      ]
    },
    "response": {
      "suggested_text": "Hi John! I'd be happy to help track your order...",
      "confidence": 0.89,
      "tone": "empathetic"
    },
    "escalation": {
      "recommended": false,
      "urgency_score": 68,
      "escalation_triggers": []
    },
    "context": {
      "customer_csat": 4.1,
      "previous_resolutions": ["Order tracking -> Resolution"],
      "kb_articles": [
        {
          "title": "How to Track Your Order",
          "url": "/help/track-order"
        }
      ]
    }
  }
}
```

**Implementation:**
```python
# Python example using requests library
import requests

def get_recommendations(message_id):
    response = requests.get(
        f"https://api.omnichannelai.com/api/v1/recommendations/{message_id}",
        params={
            "include": "routing,response,escalation"
        },
        headers={
            "Authorization": "Bearer YOUR_API_KEY"
        }
    )
    return response.json()
```

---

### 3. Sending Agent Feedback

**Endpoint:** `POST /api/v1/feedback`

**Purpose:** Provide feedback on recommendations (accepted, rejected, modified)

**Request:**
```json
{
  "message_id": "msg_abc123",
  "action": "accepted|rejected|modified",
  "feedback": {
    "routing_accepted": true,
    "response_modified": false,
    "escalation_override": false,
    "outcome": {
      "resolved": true,
      "resolution_time_minutes": 8,
      "customer_satisfaction": 5,
      "escalation_occurred": false
    }
  }
}
```

**Response:**
```json
{
  "status": "received",
  "feedback_id": "fb_abc123",
  "message": "Thank you for feedback. Your input improves our models."
}
```

---

### 4. Customer Portal API

**Endpoint:** `GET /api/v1/customer/{customer_id}/conversations`

**Purpose:** Fetch consolidated conversation history for customer portal

**Response:**
```json
{
  "customer_id": "cust_123",
  "conversations": [
    {
      "thread_id": "conv_789",
      "created_at": "2024-03-15T14:30:00Z",
      "issue_category": "Order Status Query",
      "status": "resolved",
      "messages": [
        {
          "timestamp": "2024-03-15T14:30:00Z",
          "channel": "whatsapp",
          "sender": "customer",
          "text": "My order hasn't arrived yet"
        },
        {
          "timestamp": "2024-03-15T14:35:00Z",
          "channel": "whatsapp",
          "sender": "agent",
          "text": "I'm happy to help. Here's your tracking: 1Z999AA..."
        }
      ],
      "assigned_agent": "Sarah Chen",
      "resolved_at": "2024-03-15T14:42:00Z",
      "customer_satisfaction": 5
    }
  ]
}
```

---

## Deployment Options

### Option 1: Cloud-Hosted (Recommended for SMB)

**Setup time:** 30 minutes
**Cost:** $5K-50K/month (scales with usage)
**Maintenance:** Fully managed by OmniChannelAI

**Process:**
1. Sign up for account
2. Connect your CRM/ticketing system via OAuth
3. Map customer ID fields
4. Test with sample messages
5. Enable for production

**What we host:**
- API servers, Load balancers
- NLP models, LLM integrations
- Database, Vector DB
- Monitoring, backups

**What you host:**
- Your CRM/ticketing system
- Your customer data stays with you
- Webhook ingestion server (optional)

---

### Option 2: Hybrid Deployment (Enterprise)

**Setup time:** 4-6 weeks
**Cost:** $200K+ (engineering + infrastructure)
**Maintenance:** Shared responsibility

**Your infrastructure:**
- Intent/Sentiment models (run on premise)
- NER model (on premise)
- Vector DB (managed or self-hosted)
- Agent dashboard (you host)

**OmniChannelAI managed:**
- LLM API (GPT-4/Claude via API)
- Routing engine (lightweight, on premise)
- Model updates/improvements

**Benefits:**
- Full control over models
- No customer data sent to cloud
- Lower latency (models run locally)
- Compliance with data residency laws

---

### Option 3: On-Premise (Enterprise with max control)

**Setup time:** 8-12 weeks
**Cost:** $300K+ (infrastructure + full ML team)
**Maintenance:** Your team owns everything

**You host everything:**
- All NLP models (fine-tuned on your data)
- Vector DB
- LLM server (fine-tuned LLaMA or Mistral)
- Agent dashboard, monitoring, backups

**Model training:**
- Custom fine-tuning on your data
- Weekly retraining cycles
- A/B testing framework

**Trade-offs:**
- Maximum control & privacy
- Highest total cost of ownership
- Requires ML/DevOps team
- Slower updates from OmniChannelAI

---

## Channel-Specific Integration Examples

### WhatsApp Business API Integration

**Via Twilio:**
```javascript
// Twilio webhook handler (Node.js)
const express = require('express');
const app = express();

app.post('/whatsapp-webhook', async (req, res) => {
  const message = req.body.Messages[0];

  // Forward to OmniChannelAI
  const response = await fetch('https://api.omnichannelai.com/api/v1/messages', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      message_id: message.id,
      timestamp: new Date().toISOString(),
      customer_id: message.from,
      channel: 'whatsapp',
      text: message.body,
      sender: {
        id: message.from,
        phone: message.from
      }
    })
  });

  res.status(200).send('OK');
});

app.listen(3000);
```

### Freshdesk Ticket Integration

```python
# Freshdesk webhook → OmniChannelAI
# Webhook event: "ticket.created" or "note.added"

import requests
import json

def freshdesk_webhook_handler(event_data):
    ticket = json.loads(event_data['ticket'])

    # Get latest message from ticket
    latest_note = ticket['notes'][-1]['body']

    # Send to OmniChannelAI
    omnichannelai_payload = {
        'message_id': f"freshdesk_ticket_{ticket['id']}",
        'timestamp': ticket['created_at'],
        'customer_id': f"freshdesk_{ticket['requester_id']}",
        'channel': 'email',
        'text': latest_note,
        'sender': {
            'id': ticket['requester']['email'],
            'email': ticket['requester']['email'],
            'name': ticket['requester']['name']
        },
        'context': {
            'freshdesk_ticket_id': ticket['id']
        }
    }

    response = requests.post(
        'https://api.omnichannelai.com/api/v1/messages',
        json=omnichannelai_payload,
        headers={'Authorization': f'Bearer {API_KEY}'}
    )

    return response.json()
```

### Zendesk Integration

```javascript
// Zendesk app (custom integration)
ZAFClient.on('app.registered', async function() {
  const client = ZAFClient.instance();

  // Listen for ticket creation
  client.on('ticket.save', async function(e) {
    // Get ticket details
    const ticketData = await client.get([
      'ticket.id',
      'ticket.description',
      'ticket.requester.email',
      'ticket.requester.name'
    ]);

    // Forward to OmniChannelAI
    const response = await fetch(
      'https://api.omnichannelai.com/api/v1/messages',
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${OMNICHANNELAI_API_KEY}`
        },
        body: JSON.stringify({
          message_id: `zendesk_${ticketData['ticket.id']}`,
          customer_id: `zendesk_${ticketData['ticket.requester.email']}`,
          channel: 'email',
          text: ticketData['ticket.description'],
          sender: {
            email: ticketData['ticket.requester.email'],
            name: ticketData['ticket.requester.name']
          }
        })
      }
    );
  });
});
```

---

## Testing & Validation

### Pre-Production Checklist

- [ ] API authentication working (test with invalid token → 401 error)
- [ ] Message ingestion functional (send 10 test messages)
- [ ] Recommendations retrieved correctly (verify routing/response returned)
- [ ] Feedback loop working (send feedback, verify logged)
- [ ] Latency acceptable (<500ms for recommendations)
- [ ] Webhook security validated (IP whitelisting, HMAC signing)
- [ ] PII redaction working (masked tokens seen in logs)
- [ ] Error handling tested (retry logic, circuit breaker)

### Load Testing

```
Target: 100 concurrent agents, 10K messages/day (0.11 msgs/sec)

Test scenarios:
1. Steady state: Constant 0.11 msgs/sec for 8 hours
   → Latency P50: <150ms, P99: <400ms

2. Peak load: 1 msgs/sec for 1 hour (9x normal)
   → Latency P99: <1s, no errors

3. Spike: 5 msgs/sec for 5 minutes
   → System recovers within 2 minutes

Tools: Apache JMeter, K6.io
```

### Monitoring Post-Deployment

```
Key metrics to track:
[CHECK] API response time (target <500ms P99)
[CHECK] Message processing success rate (target >99%)
[CHECK] Model accuracy (target >92%)
[CHECK] Escalation rate (target <10%)
[CHECK] Agent adoption (target >70% use AI suggestions)
[CHECK] CSAT improvement (target +0.5 points)

Dashboards:
- Real-time operations: Latency, throughput, errors
- Business metrics: CSAT, escalations, handle time
- Model performance: Intent accuracy, routing success
- System health: Memory, CPU, database connections
```

---

## Troubleshooting Guide

### "API returned 401 Unauthorized"
**Cause:** Invalid API key
**Fix:** Verify API key in dashboard, regenerate if needed

### "Messages not appearing in recommendations"
**Cause:** Message format invalid
**Fix:** Validate JSON schema, check sample payloads in docs

### "Recommendations taking >1 second"
**Cause:** Model inference slow, network latency
**Fix:** Check GPU availability, reduce batch size, test locally

### "Low agent adoption of suggestions"
**Cause:** Suggestions not good quality, poor confidence
**Fix:** Review rejected suggestions, retrain models on feedback

### "False positives in escalation detection"
**Cause:** Urgency threshold too low
**Fix:** Adjust escalation rules in admin panel, add custom rules

---

## Support & Resources

**Documentation:** https://docs.omnichannelai.com
**API Reference:** https://api-docs.omnichannelai.com
**Status Page:** https://status.omnichannelai.com
**Support Email:** support@omnichannelai.com
**Slack Community:** Join our user community for peer support

This integration guide ensures smooth deployment and ongoing success with OmniChannelAI.
