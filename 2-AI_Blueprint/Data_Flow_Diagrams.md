# Data Flow Diagrams: OmniChannelAI

## Diagram 1: End-to-End Message Processing Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ CUSTOMER MESSAGE INGESTION & PROCESSING FLOW                                │
└─────────────────────────────────────────────────────────────────────────────┘

1. MESSAGE ARRIVES FROM CHANNEL
┌─────────────────────────────────────────────────────────────────────────────┐
│ WhatsApp / Email / Chat / Twitter / SMS                                     │
│ "My order hasn't arrived yet. Order #ORD-2023-456"                         │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
2. CHANNEL ADAPTER NORMALIZES MESSAGE
┌─────────────────────────────────────────────────────────────────────────────┐
│ WhatsApp Adapter / Email Mapper / Chat Parser                              │
│ Extract: timestamp, sender_id, message_text, channel_metadata             │
│                                                                             │
│ Output: Standardized JSON schema                                           │
│ {                                                                           │
│   message_id: "msg_abc123",                                                │
│   timestamp: "2024-03-15T14:30:00Z",                                       │
│   customer_id: "CUST123",                                                  │
│   channel: "whatsapp",                                                     │
│   text: "My order hasn't arrived yet. Order #ORD-2023-456",               │
│   metadata: {...}                                                          │
│ }                                                                           │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
3. KAFKA MESSAGE QUEUE (Buffering & Reliability)
┌─────────────────────────────────────────────────────────────────────────────┐
│ Kafka Topic: "customer_messages"                                            │
│ - Partition by customer_id for ordering                                    │
│ - Replication factor: 3 (for durability)                                   │
│ - Retention: 7 days                                                         │
│ - SLA: Message available for processing within 2 seconds                   │
│                                                                             │
│ Deduplicator (prevents reprocessing if message replayed)                  │
│ Idempotent key: message_id                                                 │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ├─────────────────────────────────────────────────────────┐
                     │   (Broadcast to parallel processors)                    │
                     │                                                         │
        ┌────────────┘                                                        │
        │                      ┌──────────────────┐                          │
        │                      │ (Also store in   │                          │
        │                      │  PostgreSQL for  │                          │
        │                      │  audit log)      │                          │
        │                      └──────────────────┘                          │
        │                                                                     │
        ↓                                                                     │
4A. NLP PROCESSING (Parallel)
┌─────────────────────────────────────────────────────────────────────────────┐
│ Step 4A.1: Intent Classification                                            │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Input: "My order hasn't arrived yet. Order #ORD-2023-456"   │           │
│ │ Model: Fine-tuned RoBERTa (via HuggingFace API)             │           │
│ │ Processing Timeline: 150ms                                   │           │
│ │ Output:                                                      │           │
│ │ {                                                            │           │
│ │   intent: "Order Status Query",                             │           │
│ │   confidence: 0.96,                                         │           │
│ │   alternatives: [                                           │           │
│ │     {intent: "Complaint", confidence: 0.03},               │           │
│ │     {intent: "Return Request", confidence: 0.01}           │           │
│ │   ]                                                         │           │
│ │ }                                                            │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 4A.2: Sentiment Analysis                                              │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Input: Same text                                             │           │
│ │ Model: DistilBERT classifier                                │           │
│ │ Process Timeline: 100ms                                      │           │
│ │ Output:                                                      │           │
│ │ {                                                            │           │
│ │   sentiment: "negative",                                    │           │
│ │   confidence: 0.85,                                         │           │
│ │   intensity: 0.72,  # How strongly negative (0-1)          │           │
│ │   urgency_score: 68  # 0-100 scale                          │           │
│ │ }                                                            │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 4A.3: Named Entity Recognition                                        │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Input: Same text                                             │           │
│ │ Model: SpaCy NER                                             │           │
│ │ Process Timeline: 80ms                                       │           │
│ │ Output:                                                      │           │
│ │ {                                                            │           │
│ │   entities: {                                               │           │
│ │     ORDER_ID: "ORD-2023-456",                               │           │
│ │     LOCATION: "hasn't arrived" (implicit delivery issue)     │           │
│ │   }                                                          │           │
│ │ }                                                            │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Combined NLP Output → Merged into single document:                         │
│ {                                                                           │
│   intent: {primary: "Order Status", confidence: 0.96},                    │
│   sentiment: {label: "negative", confidence: 0.85},                       │
│   urgency_score: 68,                                                       │
│   entities: {ORDER_ID: "ORD-2023-456"}                                    │
│ }                                                                           │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
        ┌────────────┘
        │
        ↓
4B. CONTEXT RETRIEVAL (RAG)
┌─────────────────────────────────────────────────────────────────────────────┐
│ Step 4B.1: Generate Embedding                                               │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Input text: "My order hasn't arrived yet. Order #ORD-2023"   │           │
│ │ Model: Sentence-transformers (all-MiniLM-L6-v2)             │           │
│ │ Output: [0.23, -0.45, 0.67, ..., 0.12] (384 dimensions)    │           │
│ │ Process Timeline: 40ms                                       │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 4B.2: Vector Similarity Search (Pinecone)                             │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Find top-5 most similar past interactions:                   │           │
│ │                                                              │           │
│ │ Result #1 (Similarity: 0.92)                                 │           │
│ │ "Customer asked about order delivery on Mar 10. Resolved     │           │
│ │  with tracking number and ETA in 10 minutes."              │           │
│ │                                                              │           │
│ │ Result #2 (Similarity: 0.87)                                │           │
│ │ "Order delayed by 2 days. Offered $20 refund + free return.│           │
│ │  Customer satisfied."                                       │           │
│ │                                                              │           │
│ │ Process Timeline: 150ms                                      │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 4B.3: Customer Profile Lookup (PostgreSQL)                            │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Query: SELECT * FROM customers WHERE id = 'CUST123'         │           │
│ │ Process Timeline: 50ms (cached)                              │           │
│ │ Returns:                                                     │           │
│ │ {                                                            │           │
│ │   customer_id: "CUST123",                                   │           │
│ │   name: "John Smith",                                       │           │
│ │   account_status: "Standard",                              │           │
│ │   lifetime_value: "$8,500",                                │           │
│ │   previous_contacts_30d: 2,                                │           │
│ │   previous_issues: ["Order Status", "Refund Status"],      │           │
│ │   preferred_channel: "WhatsApp",                            │           │
│ │   language: "English",                                      │           │
│ │   vip: false,                                               │           │
│ │   satisfaction_score: 4.1                                   │           │
│ │ }                                                            │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 4B.4: Knowledge Base Search                                           │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Query Elasticsearch for relevant KB articles                │           │
│ │ Search: "order status query" + "delivery delay"             │           │
│ │ Top result:                                                 │           │
│ │ {                                                            │           │
│ │   title: "How to Track Your Order",                         │           │
│ │   slug: "track-order",                                      │           │
│ │   content: "Go to www.site.com/orders, enter Order ID...", │           │
│ │   relevance: 0.93                                           │           │
│ │ }                                                            │           │
│ │ Process Timeline: 80ms                                       │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Context Retrieval Complete:                                                │
│ {                                                                           │
│   customer_profile: {...},                                                 │
│   similar_interactions: [5 results],                                       │
│   kb_articles: [top 3 articles],                                          │
│   retrieval_time_ms: 280                                                   │
│ }                                                                           │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
5. RECOMMENDATION ENGINE
┌─────────────────────────────────────────────────────────────────────────────┐
│ Input: NLP results + Context + Agent availability                          │
│                                                                             │
│ Step 5A: Intelligent Routing Decision (Thompson Sampling)                  │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Filter eligible agents:                                      │           │
│ │  - Expertise: "Order Status" ✓                              │           │
│ │  - Status: Only "Available" agents                          │           │
│ │  - Language: English ✓                                       │           │
│ │                                                              │           │
│ │ Score eligible agents:                                       │           │
│ │  Agent Sarah:                                                │           │
│ │   - Expertise in "Order Status": 95/100                     │           │
│ │   - Historical CSAT: 4.7/5                                  │           │
│ │   - Availability: Available now                             │           │
│ │   - Recent success rate: 92%                                │           │
│ │   - Current queue: 2 tickets                                │           │
│ │   - Score: 4.87 / 5.0 → Confidence: 87%                    │           │
│ │                                                              │           │
│ │  Agent Marcus:                                               │           │
│ │   - Expertise in "Order Status": 78/100                     │           │
│ │   - Historical CSAT: 4.5/5                                  │           │
│ │   - Availability: With customer                             │           │
│ │   - Recent success rate: 85%                                │           │
│ │   - Current queue: 4 tickets                                │           │
│ │   - Score: 4.2 / 5.0 → Confidence: 75% (lower due to busy) │           │
│ │                                                              │           │
│ │ Recommendation: Route to Sarah (87% confidence)             │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 5B: Response Generation (LLM)                                         │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Assemble prompt with:                                        │           │
│ │  - Customer name & context                                  │           │
│ │  - Intent: "Order Status Query"                             │           │
│ │  - Previous resolutions (from context)                      │           │
│ │  - KnowledgeBase info (tracking help)                       │           │
│ │  - Company tone guidelines                                  │           │
│ │                                                              │           │
│ │ Call GPT-4 API with assembled prompt                        │           │
│ │ Model: GPT-4-0125-preview                                   │           │
│ │ Temperature: 0.7 (balance consistency & naturalness)        │           │
│ │                                                              │           │
│ │ Output (Suggested Response):                                │           │
│ │ "Hi John! I'd be happy to help track your order ORD-2023-  │           │
│ │  456. I'm connecting you with Sarah, our shipping expert,   │           │
│ │  who can get you the tracking details right away. She'll    │           │
│ │  help make sure you get your order ASAP!"                  │           │
│ │                                                              │           │
│ │ Process Timeline: 1.2s (including API latency)               │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Step 5C: Escalation Decision                                               │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Decision criteria:                                            │           │
│ │  - Intent confidence 96% → ✓ (high, no escalation)          │           │
│ │  - Urgency score 68 → ✓ (medium, not critical)              │           │
│ │  - Sentiment: Negative but confidence high → Can handle      │           │
│ │  - Keywords: No legal threats, no regulatory language        │           │
│ │  - Customer value: $8.5k (not VIP, normal care)             │           │
│ │                                                              │           │
│ │ Escalation Recommended: NO                                   │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ Final Recommendation Package:                                              │
│ {                                                                           │
│   message_id: "msg_abc123",                                                │
│   customer_id: "CUST123",                                                  │
│   routing: {                                                               │
│     agent: "Sarah Chen",                                                   │
│     confidence: 0.87,                                                      │
│     reason: "87% success with order queries, available now"               │
│   },                                                                       │
│   response: {                                                              │
│     suggested_text: "Hi John! I'd be happy to help...",                   │
│     confidence: 0.89                                                       │
│   },                                                                       │
│   escalation_recommended: false,                                          │
│   context: {...}                                                          │
│ }                                                                           │
│                                                                             │
│ Total Processing Time: 487ms (4A: 245ms, 4B: 280ms, 5: 130ms, overhead) │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
6. DELIVER TO AGENT INTERFACE
┌─────────────────────────────────────────────────────────────────────────────┐
│ Agent Dashboard Display:                                                   │
│                                                                             │
│ CUSTOMER: John Smith (CUST123) | VIP: No | Value: $8.5k                  │
│ SENTIMENT: Negative (85%) | URGENCY: Medium (68/100)                      │
│ CHANNEL: WhatsApp                                                         │
│ PREVIOUS CONTACTS: 2 in last 30 days                                      │
│                                                                             │
│ MESSAGE: "My order hasn't arrived yet. Order #ORD-2023-456"               │
│                                                                             │
│ BULB AI RECOMMENDATION (87% confident)                                      │
│ Agent: Sarah Chen (Available now)                                         │
│ Suggested Response:                                                       │
│ ┌──────────────────────────────────────────────────────────────┐           │
│ │ Hi John! I'd be happy to help track your order ORD-2023-456.│           │
│ │ I'm connecting you with Sarah, our shipping expert, who can │           │
│ │ get you the tracking details right away.                    │           │
│ │                                                              │           │
│ │ [Accept & Send] [Edit First] [Reject] [Routing]            │           │
│ └──────────────────────────────────────────────────────────────┘           │
│                                                                             │
│ RELATED ARTICLES:                                                          │
│ • How to Track Your Order (relevance: 93%)                                 │
│ • Shipping & Delivery FAQ (relevance: 87%)                                │
│                                                                             │
│ PREVIOUS RESOLUTION:                                                       │
│ "Mar 10: Customer asked about order status. Resolved with tracking in 10 min" │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
7. AGENT ACTION & FEEDBACK
┌─────────────────────────────────────────────────────────────────────────────┐
│ Agent's Action:                                                            │
│  [Clicked "Accept & Send"]                                                 │
│                                                                             │
│ Feedback Captured:                                                         │
│ {                                                                           │
│   agent_id: "agent_sarah",                                                 │
│   message_id: "msg_abc123",                                                │
│   action: "accepted_suggestion",  # or "rejected", "modified", "escalated" │
│   routing_accepted: true,                                                  │
│   response_modification: null,    # If agent edited response                │
│   timestamp: "2024-03-15T14:30:45Z",                                       │
│   interaction_outcome: {                                                   │
│     (Will be filled after customer responds)                               │
│     resolved: true,                                                        │
│     resolution_time: "8 minutes",                                          │
│     customer_satisfaction: 5,                                              │
│     escalation_occurred: false                                             │
│   }                                                                         │
│ }                                                                           │
│                                                                             │
│ Feedback loops back to:                                                    │
│  1. Training data for next model iteration                                 │
│  2. Routing algorithm (Thompson Sampling) - reinforces Sarah's expertise  │
│  3. Response quality model - validates suggested response                  │
│  4. Agent performance metrics - tracked for performance reviews             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Diagram 2: Intelligent Routing Decision Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ INTELLIGENT ROUTING ENGINE FLOW                                             │
└─────────────────────────────────────────────────────────────────────────────┘

INCOMING TICKET
│
├─ Intent Detected: "Refund Request"
├─ Urgency Score: 85/100 (HIGH)
├─ Sentiment: Angry (92% confidence)
├─ Customer: VIP (High lifetime value)
│
↓
[ROUTING ALGORITHM DECISION TREE]

┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Filter Eligible Agents                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Available Agents in System:                                    │
│  1. Agent Sarah (Specialization: Orders, Refunds)             │
│  2. Agent Marcus (Specialization: Refunds, Returns)           │
│  3. Agent Jane (Specialization: Technical Support)            │
│  4. Agent Alex (Specialization: Billing)                      │
│  5. Agent Lisa (Specialization: Orders, Refunds, VIP support) │
│                                                                 │
│ Filter by criteria:                                            │
│                                                                 │
│  Expertise match: Intent = "Refund Request"                   │
│    ✓ Sarah (refunds primary)                                  │
│    ✓ Marcus (refunds primary)                                 │
│    ✓ Lisa (refunds + VIP training)                            │
│    X Jane (technical only)                                    │
│    X Alex (billing, not refunds)                              │
│                                                                 │
│  Availability check:                                           │
│    ✗ Sarah: NOT AVAILABLE (on call with customer, 12 min wait)│
│    ✓ Marcus: AVAILABLE (queue: 2 tickets, wait: 3 min)       │
│    ✓ Lisa: AVAILABLE (queue: 0 tickets, wait: 0 min)        │
│                                                                 │
│  Language: English ✓ (all qualified)                          │
│                                                                 │
│  Skill Level match VIP customer: Request level 8+ / 10        │
│    ✓ Lisa: VIP certification level 9/10 STAR STAR STAR              │
│    ~ Marcus: Regular level 6/10                              │
│    ~ Sarah: Regular level 6/10 (but unavailable)             │
│                                                                 │
│  ELIGIBLE AGENTS: Marcus, Lisa                                │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: Score Eligible Agents (Thompson Sampling)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ AGENT MARCUS                                                   │
│ ├─ Expertise with "Refund Requests": 87/100                   │
│ ├─ Historical CSAT with refunds: 4.6/5 STAR                     │
│ ├─ Refund resolution rate: 82%                                │
│ ├─ Current workload: 2 tickets (moderate)                     │
│ ├─ Availability: NOW (0 min wait) → Bonus +10%              │
│ ├─ Recent performance trend: Improving ↗                      │
│ ├─ Confidence interval (Thompson): [0.76, 0.84]              │
│ │                                                              │
│ │ SCORE CALCULATION:                                          │
│ │ Score = (0.35 * 0.87) +        # Expertise weight          │
│ │         (0.30 * 0.92) +        # CSAT (4.6/5 = 0.92 norm) │
│ │         (0.15 * 0.82) +        # Resolution rate           │
│ │         (0.10 * 0.70) +        # Workload (lower is better)│
│ │         (0.10 * 1.0)           # Availability bonus        │
│ │                                                              │
│ │ Score = 0.3045 + 0.276 + 0.123 + 0.07 + 0.1 = 0.8735     │
│ │ Normalized: 87.35%                                         │
│                                                                 │
│ AGENT LISA                                                     │
│ ├─ Expertise with "Refund Requests": 92/100                   │
│ ├─ Historical CSAT with refunds: 4.9/5 STAR STAR STAR               │
│ ├─ Refund resolution rate: 94%                                │
│ ├─ Current workload: 0 tickets (no queue) → Bonus +15%      │
│ ├─ Availability: NOW + prioritized for VIP (0 min) → Bonus  │
│ ├─ VIP certification: Level 9/10 (highest)                    │
│ ├─ Recent performance trend: Excellent ↗↗                    │
│ ├─ Confidence interval (Thompson): [0.88, 0.96]              │
│ │                                                              │
│ │ SCORE CALCULATION:                                          │
│ │ Score = (0.35 * 0.92) +        # Expertise weight          │
│ │         (0.30 * 0.98) +        # CSAT (4.9/5 = 0.98 norm) │
│ │         (0.15 * 0.94) +        # Resolution rate           │
│ │         (0.10 * 1.0) +         # Workload (no queue)       │
│ │         (0.10 * 1.0 + 0.05)    # VIP bonus                 │
│ │                                                              │
│ │ Score = 0.322 + 0.294 + 0.141 + 0.1 + 0.15 = 1.007       │
│ │ Normalized: 91.2% (capped at ~92% max realistic)           │
│                                                                 │
│ RANKING:                                                       │
│ 1. GOLD MEDAL LISA: 91.2% confidence RECOMMENDED                   │
│ 2. SILVER MEDAL MARCUS: 87.35% confidence (fallback)                    │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: Consider Urgency & Escalation Modifiers               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Ticket characteristics:                                        │
│  - Urgency: 85/100 (HIGH)                                     │
│  - Sentiment: ANGRY (92% confidence)                          │
│  - Keywords: "NEED REFUND NOW", slight threat undertone      │
│  - Escalation risk: 72% (medium-high)                         │
│                                                                 │
│ Decision Point: Route to specialist or escalate?              │
│                                                                 │
│ Lisa's VIP & refund expertise mitigates escalation            │
│  → Can handle this high-urgency, angry customer               │
│  → No escalation needed if Lisa takes it                      │
│  → Marcus would require escalation supervision                │
│                                                                 │
│ FINAL DECISION: Route to Lisa (no escalation needed)          │
│ Confidence: 91.2%                                              │
│ Expected outcome: Based on historical CSAT, 94% resolution    │
│ Expected satisfaction: 4.8/5                                   │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: Send Notification to Agent                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Agent Lisa receives alert:                                     │
│                                                                 │
│ Desktop notification:                                          │
│ ┌─────────────────────────────────────┐                      │
│ │ RED HIGH PRIORITY TICKET              │                      │
│ │ Customer: Marcus Todd (VIP)          │                      │
│ │ Issue: Refund Request (URGENT)      │                      │
│ │ Sentiment: Angry WARN                  │                      │
│ │ Expected Resolution in: 8 min        │                      │
│ └─────────────────────────────────────┘                      │
│ (WebSocket real-time notification)                            │
│                                                                 │
│ Dashboard highlight:                                           │
│ ┌────────────────────────────────────────────────────────┐    │
│ │ NEW TICKET FOR YOU                                     │    │
│ │ Marcus Todd - VIP Customer                             │    │
│ │ "I need my refund right now. This is unacceptable!"  │    │
│ │ Refund Request | Urgent 🔴 (85/100)                  │    │
│ │ [ACCEPT & START] [Decline] [Transfer]                │    │
│ └────────────────────────────────────────────────────────┘    │
│                                                                 │
│ Smart suggestion ready:                                        │
│ "Based on customer VIP status and refund policy, you can     │
│  immediately approve a [AMOUNT] refund. Would you like       │
│  suggested response?"                                          │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 5: Monitor & Learn from Outcome                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Agent Lisa accepts ticket and resolves it in 6 minutes        │
│ Customer satisfaction: 5/5 STAR STAR STAR STAR STAR                         │
│                                                                 │
│ FEEDBACK RECORDED:                                             │
│ {                                                               │
│   routing_decision: {                                          │
│     agent_assigned: "Lisa",                                    │
│     recommended_agent: "Lisa",                                 │
│     match: true,  # ✓ Recommended agent was selected         │
│     confidence: 0.912,                                         │
│     alternatives: [{agent: "Marcus", score: 0.8735}]         │
│   },                                                            │
│   outcome: {                                                   │
│     resolved: true,                                            │
│     resolution_time_minutes: 6,                                │
│     customer_satisfaction: 5,                                  │
│     escalation_occurred: false,                                │
│     agent_csat: 4.9,                                           │
│     category_performance: "refund_resolution"                  │
│   },                                                            │
│   learning: {                                                  │
│     "Lisa's historical refund success: increased from 94% to" │
│     "94.1% (incremental improvement)",                        │
│     "Thompson Sampling bandit: Lisa's confidence interval"    │
│     "narrows: [0.875, 0.945] (becoming consistent winner)",   │
│     "Marcus's alternatives count: +1 (would have been OK but" │
│     "Lisa was better)",                                        │
│     "Next similar ticket → Lisa priority increases further"   │
│   }                                                             │
│ }                                                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## Diagram 3: Feedback Loop & Model Improvement

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ CONTINUOUS LEARNING & MODEL IMPROVEMENT FEEDBACK LOOP                       │
└─────────────────────────────────────────────────────────────────────────────┘

PRODUCTION SYSTEM
│
├─ Every agent action creates data point:
│  ├─ Accepted AI suggestion? (Yes/No) → Feedback on response quality
│  ├─ Routed where recommended? (Yes/No) → Feedback on routing accuracy
│  ├─ Resolved on first contact? (Yes/No) → Intent classification accuracy
│  ├─ Customer satisfaction rating (1-5) → Overall system effectiveness
│  ├─ Escalation occurred? (Yes/No) → Escalation prediction accuracy
│  └─ Timestamp, agent ID, customer ID → Full audit trail
│
↓

DATA COLLECTION PIPELINE
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│ Daily Aggregation (batch job at 2 AM UTC):                                 │
│  - Collect 50,000+ interactions from past day                              │
│  - Join with agent feedback (acceptance/rejection)                         │
│  - Join with customer feedback (CSAT, NPS)                                 │
│  - Aggregate into "labeled dataset"                                        │
│                                                                             │
│ Example labeled record:                                                    │
│ {                                                                           │
│   message_text: "My order hasn't arrived",                                │
│   detected_intent: "Order Status",      # What model predicted             │
│   true_intent: "Order Status",          # What actually correct (agent ok) │
│   detected_sentiment: "negative",       # What model predicted             │
│   true_sentiment: "negative",           # Correct (agent confirmed)        │
│   suggested_agent: "Sarah",             # Recommendation                   │
│   assigned_agent: "Sarah",              # What happened (match!)           │
│   resolved_first_contact: 1,            # Binary label                     │
│   resolution_time: 8,                   # Minutes                         │
│   customer_satisfaction: 5,              # 1-5 rating (implicit label)     │
│   escalated: 0                          # Binary label                     │
│ }                                                                           │
│                                                                             │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
QUALITY METRICS CALCULATION
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│ Intent Classification Accuracy:                                             │
│  - Correct: 48,500 / 50,000 = 97% (exceeds 92% target after fine-tuning) │
│  - Improved from: 92% baseline (production is doing better!)              │
│  - Error analysis: Top 3 confusion pairs                                   │
│    1. "Return Request" confused with "Refund Request" (3%)                │
│    2. "Complaint" misclassified as plain "Negative sentiment" (0.5%)      │
│    3. "Order Status" + "Escalation" combined intents (1%)                │
│                                                                             │
│ Sentiment Accuracy:                                                        │
│  - F1-score: 0.89 (improved from 0.87 baseline)                            │
│  - Precision: 0.91 (fewer false positives for "angry")                    │
│  - Recall: 0.87 (catching most negative sentiments)                       │
│                                                                             │
│ Routing Prediction:                                                        │
│  - Accuracy: 94.2% of routing suggestions were accepted/correct            │
│  - Confidence correlation: 0.93 (when confident, usually right)            │
│  - Worst-performing category: "Technical Support" (85% accuracy)           │
│                                                                             │
│ Escalation Prediction:                                                     │
│  - Precision: 78% (when we flag escalation, actually needs escalation)     │
│  - Recall: 72% (catching 72% of actual escalations needed)               │
│  - Action needed: More training data for edge cases                        │
│                                                                             │
│ Response Quality:                                                          │
│  - Acceptance rate: 78% agents accept suggested responses without edit     │
│  - CSAT when using suggestions: 4.6/5 (vs 4.2/5 without)                 │
│  - Tone appropriateness: 91% agent approval                               │
│                                                                             │
│ Business Impact:                                                            │
│  - Avg Handle Time: 9.2 min (vs 9.5 min target, exceeding goal)          │
│  - Customer Satisfaction: 4.58/5 (vs 4.3 baseline, +6% improvement)      │
│  - Agent Productivity: 18.3 tickets/day (vs 18.0 target, +1.7% over)     │
│                                                                             │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
ANOMALY DETECTION
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│ System detects degradation in:                                             │
│  ✓ Intent classification accuracy dropped to 89% (from 92%+)              │
│    → Flag for investigation                                                │
│  ✓ Routing to "John" has CSAT 2.1/5 (team average 4.5/5)                 │
│    → Performance alert, recommend training or reassignment                 │
│  ✓ Sentiment model false positive rate increased 12% (from 8%)           │
│    → Possible model drift, schedule retraining                             │
│  ✓ Response generation uses $5.2K/day in GPT-4 API calls (+15% vs avg)   │
│    → Cost spike alert, review if queries are getting longer              │
│                                                                             │
│ Alert severity levels:                                                     │
│  RED CRITICAL: Intent accuracy < 85%, escalation prediction < 60%         │
│  ORANGE HIGH: Routing errors > 10%, response CSAT < 4.0                     │
│  YELLOW MEDIUM: Slight trend degradation, investigate root cause              │
│  GREEN OK: All metrics within normal range                                    │
│                                                                             │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
RETRAINING DECISION
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│ Decision Tree:                                                              │
│                                                                             │
│ IF Intent accuracy < 88% OR                                                │
│    Routing accuracy < 90% OR                                               │
│    Response quality CSAT < 4.2                                             │
│ THEN:                                                                       │
│   1. Collect labeled dataset (10,000 new examples)                         │
│   2. Identify training set issues:                                         │
│      - Class imbalance (some intents rare)                                 │
│      - Mislabeled examples (agent marked wrong answer)                     │
│      - Domain drift (new issue types emerging)                             │
│   3. Fine-tune models:                                                     │
│      - Intent: Fine-tune RoBERTa on new 10K examples (8 hours wall time)  │
│      - Sentiment: Fine-tune DistilBERT (4 hours)                          │
│      - Routing: Retrain Thompson Sampling coefficients (30 minutes)        │
│   4. A/B test new models on 10% of traffic:                               │
│      - Route new messages to NEW intent model vs OLD                       │
│      - Monitor accuracy for 1-2 weeks                                      │
│      - If new > old by >1%, promote to 100% traffic                       │
│   5. If no improvement:roll back and investigate                          │
│                                                                             │
│ Status: Weekly retraining scheduled for continuous improvement             │
│                                                                             │
└────────────────────┬────────────────────────────────────────────────────────┘
                     │
                     ↓
MODEL DEPLOYMENT
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│ A/B Testing Framework:                                                     │
│                                                                             │
│ New Intent Model v2.1 vs Baseline v2.0:                                   │
│ ┌─────────────────────────────────────┐                                   │
│ │ 90% traffic → Model v2.0 (baseline) │ (Production)                      │
│ │ 10% traffic → Model v2.1 (candidate)│ (Testing)                         │
│ └─────────────────────────────────────┘                                   │
│                                                                             │
│ Metrics tracked over 500 interactions:                                     │
│ v2.0: Accuracy 92.1% | v2.1: Accuracy 93.8% CHECK (better!)                 │
│ v2.0: CSAT 4.58    | v2.1: CSAT 4.62 (slight improvement)                │
│                                                                             │
│ Promotion decision:                                                        │
│ v2.1 shows 1.7% accuracy improvement, statistical significance test       │
│ passes (p < 0.05). Schedule promotion to 100% traffic.                   │
│                                                                             │
│ Rollout plan:                                                              │
│ Day 1: 10% → 30% traffic                                                   │
│ Day 2: Monitor for errors/performance issues                              │
│ Day 3: 30% → 70% traffic (if no issues)                                   │
│ Day 4: 70% → 100% traffic (all users on v2.1)                            │
│ Rollback capability: If accuracy drops >0.5%, auto-revert to v2.0        │
│                                                                             │
│ Versioning:                                                                │
│ - Model v2.0 archived (kept for 90 days for rollback)                     │
│ - Model v2.1 becomes primary production version                           │
│ - Model v2.2 starts training in parallel for next cycle                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

This continuous loop ensures models stay accurate and systems improve over time with real-world data.
