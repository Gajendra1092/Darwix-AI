# Wireframes & User Experience Design: OmniChannelAI

## UI/UX Principles

**Core Design Philosophy:**
- Fast: Agent can see everything in <2 seconds
- Clear: Recommendations are obvious, not buried
- Actionable: One-click actions for common tasks
- Trustworthy: Confidence scores & transparency throughout
- Accessible: Mobile-responsive, keyboard shortcuts for power users

---

## 1. AGENT DASHBOARD - Main Interface

```
┌────────────────────────────────────────────────────────────────────────────────┐
│  OmniChannelAI    [Icon: =] Menu     [Search]     [Settings] [Sarah] │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  LEFT SIDEBAR (260px)                │ MAIN CONTENT AREA                      │
│  ┌──────────────────────────────┐    │                                        │
│  │ STATUS                       │    │  TICKET QUEUE & DETAILS                │
│  │ ┌──────────────────────────┐ │    │  ┌───────────────────────────────┐    │
│  │ │ Available    [Toggle On] │ │    │  │ QUEUE: 7 tickets waiting      │    │
│  │ │ (4h 32min available)     │ │    │  │ Avg Wait: 8 min | Peak: 12min│    │
│  │ └──────────────────────────┘ │    │  │                               │    │
│  │                              │    │  │ [RED] 1 Critical | [ORANGE] 3 High    │
│  │ TODAY STATS                  │    │  │ [YELLOW] 3 Normal  │ [GREEN] 0 Low      │
│  │ ┌──────────────────────────┐ │    │  │                               │    │
│  │ │ 12 resolved              │ │    │  │ By Channel: WhatsApp 3 •     │    │
│  │ │ [STAR] 4.7/5 CSAT            │ │    │  │ Email 2 • Chat 1 • Twitter 1 │    │
│  │ │ [CLOCK] 9 min AHT             │ │    │  └───────────────────────────────┘    │
│  │ └──────────────────────────┘ │    │                                        │
│  │                              │    │  ACTIVE TICKETS (Click to expand)      │
│  │ INBOX                       │    │  ┌───────────────────────────────┐    │
│  │ ┌──────────────────────────┐ │    │  │[#1] John Smith      3 min [CLOCK]  │
│  │ │ [PHONE] WhatsApp    3       │ │    │  │     URGENT [RED] Order Status  │
│  │ │ [EMAIL] Email       2       │ │    │  │     Sentiment: Frustrated 94%│
│  │ │ [CHAT] Chat        1       │ │    │  │     History: 2 prev contacts│
│  │ │ [BIRD] Twitter     1       │ │    │  │                               │
│  │ │ [MOBILE] SMS        0        │ │    │  │     [LIGHTBULB] AI RECOMMENDATION     │
│  │ └──────────────────────────┘ │    │  │     Route: Marcus (87%)      │
│  │                              │    │  │     Response: [Approve] [Edit]│
│  │ SHORTCUTS                    │    │  │                               │
│  │ ┌──────────────────────────┐ │    │  │─────────────────────────────│
│  │ │ [REFRESH] Refresh (F5)        │ │    │  │[#2] Maria Rodriguez  8 min  │
│  │ │ [STAR] Favorites (Ctrl+F)  │ │    │  │     Normal [YELLOW] Refund Status │
│  │ │ [CHART] My Reports          │ │    │  │     Suggested route: Jane   │
│  │ │ [QUESTION] Help                 │ │    │  │     [Route] [Take]          │
│  │ └──────────────────────────┘ │    │  │                               │
│  └──────────────────────────────┘    │  │─────────────────────────────│
│                                      │  │[#3] Alex Johnson   12 min   │
│                                      │  │     Low [GREEN] Product Info    │
│                                      │  │     Auto-recommend: [Reply] │
│                                      │  │                               │
│                                      │  └───────────────────────────────┘    │
│                                      │                                        │
└────────────────────────────────────────────────────────────────────────────────┘
```

**Key Features:**
- Status indicator (green = available, yellow = handling, red = offline)
- Real-time queue statistics
- Inbox by channel (visual priority)
- Quick access to main tickets
- One-click routing/response actions

---

## 2. EXPANDED TICKET DETAIL VIEW

```
┌────────────────────────────────────────────────────────────────────────────┐
│ TICKET #ORD-2023-456                                                       │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CUSTOMER CONTEXT (Left Panel - 30%)                                        │
│ ┌──────────────────────────────────┐                                      │
│ │ [PERSON] John Smith (CUST123)          │                                      │
│ │ Status: Standard Customer        │                                      │
│ │ Value: $8,500 lifetime           │                                      │
│ │ VIP: No                          │                                      │
│ │                                  │                                      │
│ │ ENGAGEMENT METRICS               │                                      │
│ │ • Contacts (30d): 2              │                                      │
│ │ • Contacts (all-time): 12        │                                      │
│ │ • CSAT (avg): 4.1/5              │                                      │
│ │ • Last contact: 10 days ago      │                                      │
│ │ • Churn risk: Low (0.12)         │                                      │
│ │                                  │                                      │
│ │ ISSUE HISTORY                    │                                      │
│ │ ┌──────────────────────────────┐ │                                      │
│ │ │ Mar 10: Order Status Query   │ │                                      │
│ │ │ → Resolved in 10 min [CHECK]       │                                      │
│ │ │                              │ │                                      │
│ │ │ Feb 28: Refund Status       │ │                                      │
│ │ │ → Resolved in 5 min [CHECK]        │                                      │
│ │ └──────────────────────────────┘ │                                      │
│ │ [View Full Profile]              │                                      │
│ └──────────────────────────────────┘                                      │
│                                                                             │
│ CONVERSATION TIMELINE (Center/Right Panel - 70%)                           │
│ ┌──────────────────────────────────────────────────────────────────────┐  │
│ │ UNIFIED CONVERSATION HISTORY                                         │  │
│ │ [🕐 TIME] [Channel] [SENDER]: MESSAGE                               │  │
│ │                                                                      │  │
│ │ 09:15 AM - 📱 WhatsApp                                              │  │
│ │ John: "My order hasn't arrived yet"                                 │  │
│ │       [Order Detection: ORD-2023-456]                               │  │
│ │                                                                      │  │
│ │ 09:20 AM - 📧 Email                                                 │  │
│ │ John: "Order #ORD-2023-456 still not shipped"                       │  │
│ │       [Same thread - message consolidated]                          │  │
│ │                                                                      │  │
│ │ 09:30 AM - 💬 Chat                                                   │  │
│ │ John: "This is unacceptable, I'm leaving!"                          │  │
│ │       [Sentiment escalation detected: 94% frustrated]               │  │
│ │                                                                      │  │
│ │ ══════════════════════════════════════════════════════════════════ │  │
│ │                      AI ANALYSIS & RECOMMENDATION                    │  │
│ │ ══════════════════════════════════════════════════════════════════ │  │
│ │                                                                      │  │
│ │ INTENT ANALYSIS                     SENTIMENT ANALYSIS              │  │
│ │ ┌────────────────────────┐         ┌────────────────────────┐     │  │
│ │ │ Primary: Order Status  │         │ Sentiment: NEGATIVE    │     │  │
│ │ │ Confidence: 96%        │         │ Confidence: 85%        │     │  │
│ │ │ Alternatives:          │         │ Intensity: 0.72        │     │  │
│ │ │ - Shipment (0.03)      │         │                        │     │  │
│ │ │ - Complaint (0.01)     │         │ Urgency Score: 68/100  │     │  │
│ │ └────────────────────────┘         │ Level: MEDIUM          │     │  │
│ │                                    └────────────────────────┘     │  │
│ │                                                                      │  │
│ │ RECOMMENDED ACTION                                                   │  │
│ │ ┌──────────────────────────────────────────────────────────────┐   │  │
│ │ │ Route to: 👤 Marcus Chen (Shipping Specialist)              │   │  │
│ │ │ Confidence: 87%                                              │   │  │
│ │ │ Available: Now (0 min wait)                                  │   │  │
│ │ │ His CSAT: 4.8/5 | Success Rate: 92% with orders            │   │  │
│ │ │                                                              │   │  │
│ │ │ Why Marcus? "92% success with order queries, currently     │   │  │
│ │ │  available, and VIP communication skills"                  │   │  │
│ │ │                                                              │   │  │
│ │ │ [Route to Marcus ✓] [Choose Different] [Take Myself]        │   │  │
│ │ └──────────────────────────────────────────────────────────────┘   │  │
│ │                                                                      │  │
│ │ SUGGESTED RESPONSE (Tap to accept & send)                           │  │
│ │ ┌──────────────────────────────────────────────────────────────┐   │  │
│ │ │ I sincerely apologize that your order ORD-2023-456 hasn't   │   │  │
│ │ │ arrived on time. Let me investigate immediately—your order  │   │  │
│ │ │ was approved to ship and should have been delivered by now. │   │  │
│ │ │ I'm connecting you with Marcus, our shipping expert, who    │   │  │
│ │ │ will track it down and get it to you ASAP. He'll contact    │   │  │
│ │ │ you within 5 minutes with an update.                        │   │  │
│ │ │                                                              │   │  │
│ │ │ Suggested by: GPT-4 | Confidence: 91% | Tone: Empathetic   │   │  │
│ │ │                                                              │   │  │
│ │ │ [✅ Accept & Send] [✏️ Edit First] [❌ Reject] [🐛 Report]   │   │  │
│ │ └──────────────────────────────────────────────────────────────┘   │  │
│ │                                                                      │  │
│ │ RELATED HELP ARTICLES                                               │  │
│ │ 📖 "How to Track Your Order" (Relevance: 93%)                       │  │
│ │ 📖 "Shipping & Delivery FAQ" (Relevance: 87%)                       │  │
│ │ 📖 "Order Status Definitions" (Relevance: 82%)                      │  │
│ │                                                                      │  │
│ └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│ QUICK ACTIONS                                                           │
│ [✅ Mark Resolved] [🔄 Suggest Follow-up] [📤 Route/Escalate]        │
│ [📌 Flag for Review] [⏰ Set Reminder]                                │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

**Key Elements:**
- Customer context (left sidebar at a glance)
- Unified conversation timeline (consolidates messages from all channels)
- AI recommendation prominently displayed
- Confidence scores for transparency
- One-button actions (Accept, Edit, Route)
- Related articles for quick knowledge
- Quick action buttons at bottom

---

## 3. MANAGER ANALYTICS DASHBOARD

```
┌────────────────────────────────────────────────────────────────────────────┐
│  Analytics Dashboard     [CHART Team Performance] [TARGET Quality] [PACKAGE Volume]   │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DATE RANGE: [Mar 1-15, 2024] ▼     COMPARE: [vs Previous Period] ▼  │
│                                                                             │
│ KEY METRICS (Top Row)                                                       │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│ │CHART Avg CSAT   │ │CLOCK Avg AHT    │ │CHECK FCR Rate   │ │SEND Escalation│      │
│ │ 4.5/5 ↗ +3%  │ │ 9.2 min ↘ -1%│ │ 68% ↗ +4%   │ │ 8% ↗ +1%   │      │
│ │(vs 4.4)      │ │(vs 9.3)      │ │(vs 65%)     │ │(vs 8%)     │      │
│ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                                             │
│ TEAM PERFORMANCE                              QUALITY MATRIX              │
│ ┌─────────────────────────────┐              ┌──────────────────────┐    │
│ │ Agent      │ CSAT │ AHT  │  FCR         │ Agent │ Intent Acc │ Route│    │
│ │──────────  │───── │──────┤─────         │─────────────────────┤     │    │
│ │PERSON Sarah    │ 4.8STAR │ 8m   │ 78% STAR STAR    │PERSON Sarah│ 96% CHECK     │ 95% │    │
│ │PERSON Marcus   │ 4.7STAR │ 9m   │ 82% STAR STAR    │PERSON Marcus│ 93% CHECK     │ 92% │    │
│ │PERSON Jane     │ 4.2STAR │ 11m  │ 65% STAR     │PERSON Jane│ 89% WARN     │ 88% │    │
│ │PERSON Alex     │ 4.1STAR │ 12m  │ 62% STAR     │PERSON Alex│ 85% WARN     │ 82% │    │
│ │PERSON Lisa     │ 4.9STAR │ 9m   │ 81% STAR STAR    │PERSON Lisa│ 97% CHECK      │ 96% │    │
│ │  Team      │ 4.5STAR │ 10m  │ 74% STAR STAR    │ Team │ 92% CHECK      │ 91% │    │
│ └─────────────────────────────┘              └──────────────────────┘    │
│                                                                             │
│ ISSUE CATEGORY ANALYSIS              CORRELATION: AI Impact              │
│ ┌──────────────────────────────┐    ┌──────────────────────────────┐   │
│ │ Category      │ Volume │ Esc% │    │ When AI suggestion used:     │   │
│ │────────────────────────────── │    │ • +12% faster resolution     │   │
│ │ Order Status  │  1,234 │ 4%   │    │ • +8% higher CSAT           │   │
│ │ Refund Status │    892 │ 8%   │    │ • -15% escalation rate      │   │
│ │ Return Req    │    654 │ 12%  │    │                             │   │
│ │ Technical     │    432 │ 18%  │    │ When AI routing used:       │   │
│ │ Billing       │    298 │ 6%   │    │ • +18% correct first route  │   │
│ │ Other         │    156 │ 15%  │    │ • +6% faster resolution     │   │
│ └──────────────────────────────┘    └──────────────────────────────┘   │
│                                                                             │
│ TREND CHARTS                                                                │
│ ┌──────────────────────────────┐  ┌──────────────────────────────┐      │
│ │ CSAT Trend (7 days)          │  │ Escalation Trend (7 days)    │      │
│ │                              │  │                              │      │
│ │ 5★ ╱                         │  │ 12% ╱╲                       │      │
│ │ 4★ ║╱  (↗ improvement)       │  │ 10% ║  ╲ (↘ improving)      │      │
│ │ 3★ ║    Avg: 4.5★            │  │  8% ║   ╲ Avg: 8%           │      │
│ │    ║                         │  │     ║                       │      │
│ │ Days Mon Tue Wed Thu Fri Sat │  │ Days Mon Tue Wed Thu Fri Sat│      │
│ └──────────────────────────────┘  └──────────────────────────────┘      │
│                                                                             │
│ INSIGHTS & RECOMMENDATIONS                                                 │
│ TARGET Sarah & Marcus: Top performers. Consider them for new team leads.     │
│ WARN Jane & Alex: Below team average on quality. Recommend training.        │
│ BULB Technical category: Highest escalation (18%). Review routing rules.   │
│ CHECK AI suggestions: 12% impact on resolution speed. Strong ROI.           │
│                                                                             │
│ [Export Report] [Schedule Training] [Set Goals] [View Details]            │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

**Key Features:**
- Key metrics at top (KPIs at a glance)
- Agent performance matrix
- Quality/routing accuracy grid
- Issue category analysis
- Trend charts with insights
- AI impact measurements
- Actionable recommendations

---

## 4. CUSTOMER PORTAL - Unified Conversation View

```
┌────────────────────────────────────────────────────────────────────────────┐
│  Welcome Back, John Smith! GREETING                                              │
│  Your Support Dashboard                                                    │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ YOUR ACTIVE ISSUES                                                          │
│ ┌────────────────────────────────────────────────────────────────────────┐ │
│ │ PACKAGE Order #ORD-2023-456                                                │ │
│ │ └─ Status: Shipped CHECK (Last updated: 2 hours ago)                     │ │
│ │    Expected Delivery: Mar 15, 2024                                     │ │
│ │    Assigned to: Marcus Chen (Shipping Specialist)                     │ │
│ │    [Track Order] [Chat with agent] [View full conversation]           │ │
│ │                                                                        │ │
│ │ CHAT Your Conversation with Us                                          │ │
│ │ ├─ 09:15 AM (WhatsApp): "My order hasn't arrived yet"               │ │
│ │ ├─ 09:20 AM (Email): "Order #ORD-2023-456 still not shipped"        │ │
│ │ ├─ 09:30 AM (Chat): "This is unacceptable, I'm leaving!"            │ │
│ │ └─ 10:05 AM (Marcus): "Apologies for the delay. Your order is       │ │
│ │                        shipping today via FedEx. Tracking: 1Z99..."  │ │
│ │                                                                        │ │
│ │ STAR Your Satisfaction: 5/5                                             │ │
│ │    "Thank you for excellent support!"                                 │ │
│ │                                                                        │ │
│ │ [Close Issue] [Send Follow-up] [More Options]                         │ │
│ └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│ RECENTLY RESOLVED                                                           │
│ ┌────────────────────────────────────────────────────────────────────────┐ │
│ │ TOOL Password Reset (Resolved Feb 28, 2024)                            │ │
│ │    Handled by: Sarah Chen | Resolution time: 5 minutes               │ │
│ │    Your rating: STAR STAR STAR STAR STAR "Quick and helpful!"                         │ │
│ │    [View conversation] [Rate again]                                  │ │
│ │                                                                        │ │
│ │ CARD Refund Status (Resolved Feb 20, 2024)                             │ │
│ │    Handled by: Marcus Chen | Resolution time: 3 minutes              │ │
│ │    Your rating: STAR STAR STAR STAR "Good, but could've been faster"             │ │
│ │    [View conversation]                                               │ │
│ └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│ START NEW CONVERSATION                                                      │
│ ┌────────────────────────────────────────────────────────────────────────┐ │
│ │ Choose your preferred channel:                                         │ │
│ │                                                                        │ │
│ │  [PHONE WhatsApp] [EMAIL Email] [CHAT Chat] [MICROPHONE Call] [BIRD Twitter]           │ │
│ │                                                                        │ │
│ │  Subject: [__________________________]                               │ │
│ │  Message: [_________________________]                                │ │
│ │           [_________________________]                                │ │
│ │                                                                        │ │
│ │  [Send] [Clear]                                                      │ │
│ │                                                                        │ │
│ │ BULB TIP: Remember, you don't need to repeat yourself! We'll have     │ │
│ │    your full conversation history regardless of channel.             │ │
│ └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│ SELF-SERVICE OPTIONS (Get instant answers)                                │
│ ┌────────────────────┐ ┌────────────────────┐ ┌────────────────────┐    │
│ │ BOOK Order Tracking  │ │ KEY Reset Password  │ │ QUESTION FAQ Search      │    │
│ │                    │ │                    │ │                    │    │
│ │ Track your orders  │ │ Secure password    │ │ Common questions   │    │
│ │ and shipments      │ │ reset flow         │ │ answered           │    │
│ │ [Track Now]        │ │ [Reset Now]        │ │ [Search]           │    │
│ └────────────────────┘ └────────────────────┘ └────────────────────┘    │
│                                                                             │
│ ACCOUNT SETTINGS                                                            │
│ [PENCIL Edit Profile] [PHONE Notification Preferences] [LOCK Security Settings]   │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

**Key Features:**
- Active issue tracking with status
- Unified conversation view (all channels together)
- Satisfaction rating from customer
- Recently resolved issues
- New conversation starter (omnichannel)
- Self-service shortcuts
- Account management options

---

## 5. ESCALATION & ROUTING DECISION INTERFACE

```
┌────────────────────────────────────────────────────────────────────────────┐
│ TRIGGERED: Escalation Decision Engine                                     │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ RED HIGH URGENCY TICKET - Requires Immediate Attention                      │
│                                                                             │
│ CUSTOMER MESSAGE:                                                           │
│ From: Marcus Todd (VIP - $500K lifetime value)                             │
│ Channel: WhatsApp                                                           │
│ Message: "I need my refund RIGHT NOW. This is completely unacceptable!     │
│          If I don't see it by 5 PM, I'm disputing this with my bank!"     │
│                                                                             │
│ ════════════════════════════════════════════════════════════════════════  │
│                         AI ANALYSIS RESULTS                                │
│ ════════════════════════════════════════════════════════════════════════  │
│                                                                             │
│ ESCALATION TRIGGER ANALYSIS                                                │
│ ✅ Intent Detected: Refund Status Query                                    │
│    Confidence: 98% (very clear)                                            │
│                                                                             │
│ ⚠️ THREAT LANGUAGE DETECTED: "dispute with my bank"                       │
│    Action: ESCALATION RECOMMENDED                                          │
│                                                                             │
│ ⚠️ URGENCY: CRITICAL (95/100)                                              │
│    - Contains: "RIGHT NOW" (urgency keyword)                               │
│    - Contains: "unacceptable" (negative intensity)                         │
│    - Threat indicator: "disputing with bank" (regulatory risk)            │
│    - Customer status: VIP (high value, preserve relationship)              │
│                                                                             │
│ 🔴 RECOMMENDATION: IMMEDIATE ESCALATION                                    │
│                                                                             │
│ ════════════════════════════════════════════════════════════════════════  │
│                      ESCALATION PATHWAY                                    │
│ ════════════════════════════════════════════════════════════════════════  │
│                                                                             │
│ This ticket is being routed to:                                            │
│                                                                             │
│ LEVEL 1: Manager Review (Sarah's Manager)                                 │
│ ├─ Time Expectation: <5 minutes response                                  │
│ ├─ Authority: Can approve immediate refund                                │
│ ├─ Action: Review refund eligibility, approve compensation                │
│ └─ Status: Routed to queue "manager_escalations"                          │
│                                                                             │
│ LEVEL 2 (if needed): Dispute Prevention Team                              │
│ ├─ Time Expectation: <15 minutes                                          │
│ ├─ Authority: Contact customer proactively, resolve before chargeback     │
│ └─ Tools: Advanced compensation options, account review                   │
│                                                                             │
│ ════════════════════════════════════════════════════════════════════════  │
│                   RECOMMENDED IMMEDIATE ACTION                             │
│ ════════════════════════════════════════════════════════════════════════  │
│                                                                             │
│ SUGGESTED RESPONSE (for manager approval):                                 │
│ ┌────────────────────────────────────────────────────────────────────┐   │
│ │ Marcus, I sincerely apologize for this frustration. I'm           │   │
│ │ immediately escalating your refund to my manager for priority     │   │
│ │ processing. You should hear from us within 5 minutes with a       │   │
│ │ resolution. Thank you for your patience.                          │   │
│ │                                                                    │   │
│ │ [✅ Manager: Approve & Send] [✏️ Edit] [❌ Deny]                   │   │
│ └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│ CONTEXT FOR MANAGER:                                                       │
│ • Customer is VIP ($500K lifetime value, churn risk: HIGH)                │
│ • Refund status: Approved but delayed in processing                       │
│ • Chargeback risk: HIGH (30% probability within 24 hours)                 │
│ • Recommended action: Approve express refund + $50 credit                 │
│ • Approval authority: YES (within manager limit)                          │
│                                                                             │
│ NEXT STEPS:                                                                │
│ [🔔 Notify Manager] [⏰ Set Timer (5 min SLA)] [📞 Call Customer]        │
│ [📧 Loop in Finance] [📋 Create Case Note]                              │
│                                                                             │
│ DO NOT SEND STANDARD RESPONSE - Escalation requires personal touch        │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

**Key Features:**
- Clear escalation trigger reasons
- Escalation pathway with timing
- Recommended response template
- Context for decision maker
- Next step actions
- Time-based SLA tracking

---

## 6. MOBILE AGENT INTERFACE (Sales Rep at Trade Show)

```
┌──────────────────────────────────────────┐
│  OmniChannelAI Mobile       [≡] [👤]     │
├──────────────────────────────────────────┤
│                                          │
│  QUICK STATUS                           │
│  ┌──────────────────────────────────┐  │
│  │ Available ✓ (2h 30min)            │  │
│  │ Queue: 3 tickets                  │  │
│  │ Avg Wait: 5 min                   │  │
│  └──────────────────────────────────┘  │
│                                          │
│  NEW TICKET 🔔                          │
│  ┌──────────────────────────────────┐  │
│  │ Sarah Johnson                     │  │
│  │ "Where's my refund?"              │  │
│  │ 🔴 URGENT | Refund Status        │  │
│  │ 2 min ago on WhatsApp             │  │
│  │                                  │  │
│  │ [Take Ticket] [Skip] [Route]      │  │
│  └──────────────────────────────────┘  │
│                                          │
│  SUGGESTION: "Approve refund within    │
│  24 hours. $200 pending approval"       │
│  [Accept] [Edit] [Reject]               │
│                                          │
│  ACTIVE TICKETS                         │
│  ┌──────────────────────────────────┐  │
│  │ Mike T. - 3 min                  │  │
│  │ "Tech not working"               │  │
│  │ [View] → [Response] [Route]      │  │
│  │                                  │  │
│  │ Lisa G. - 8 min                  │  │
│  │ "When ships?" (Order Q)          │  │
│  │ [View] → [Response] [Route]      │  │
│  │                                  │  │
│  │ Alex P. - 12 min                 │  │
│  │ "Product info needed"            │  │
│  │ [View] → [Response] [Route]      │  │
│  └──────────────────────────────────┘  │
│                                          │
│  [Menu] [Settings] [Help]               │
│                                          │
└──────────────────────────────────────────┘
```

**Key Features:**
- Compact, mobile-optimized layout
- Large touch targets
- Minimal scrolling
- Quick actions visible
- Notification support
- Works offline (offline-first design)

---

## 7. USER FLOW: Resolution Journey

```
CUSTOMER INITIATES CONTACT
        ↓
        ├─ Via WhatsApp?  → Twilio normalizer
        ├─ Via Email?     → SMTP normalizer
        └─ Via Chat?      → Widget normalizer
        ↓
    [Kafka Queue]
        ↓
    NLP Pipeline
        ├─ Intent: "Order Status"
        ├─ Sentiment: "Frustrated"
        └─ Urgency: 68/100
        ↓
    Context Retrieval (RAG)
        ├─ Customer profile loaded
        ├─ Similar past resolutions found
        └─ KB articles matched
        ↓
    Recommendation Engine
        ├─ Route: Marcus (87% confidence)
        ├─ Response: [AI generated]
        └─ Escalation: No
        ↓
    AGENT DASHBOARD
        ├─ Sees ticket with full context
        ├─ Accepts AI suggestion
        └─ Clicks "Send"
        ↓
    MESSAGE SENT TO CUSTOMER
        ├─ Via same channel (WhatsApp)
        ├─ With personalization
        └─ Full context preserved
        ↓
    CUSTOMER RECEIVES & RESPONDS
        ├─ Positive response? → Marked resolved
        ├─ More questions?   → Loop back to Agent
        └─ Escalation needed? → Manager takes over
        ↓
    FEEDBACK COLLECTION
        ├─ CSAT rating: "How satisfied? [⭐⭐⭐⭐⭐]"
        ├─ Agent feedback: "Accept AI? [✓ Yes / ✗ No]"
        └─ Comments: "What could we improve?"
        ↓
    LEARNING LOOP
        ├─ Feedback stored in training set
        ├─ Models retrained weekly
        ├─ A/B tested next version
        └─ Winner promoted to production
```

---

## Color & Visual Hierarchy Guide

**Status Indicators:**
- 🔴 Red: Critical, urgent escalation needed
- 🟠 Orange: High priority, needs attention
- 🟡 Yellow: Medium priority, standard processing
- 🟢 Green: Low priority, can wait
- 🔵 Blue: Informational, neutral

**Confidence Indicators:**
- 90-100%: Very High ✓ (green checkmark)
- 75-90%: High ⚠️ (orange checkmark)
- 60-75%: Medium ⚠️ (caution symbol)
- <60%: Low ✗ (red X, requires review)

**Typography:**
- Headers: Bold, large (24px+)
- Subheaders: Semi-bold (16-18px)
- Body: Regular (14-16px)
- Captions: Light (12-14px), gray

**Spacing:**
- Generous padding (16px minimum)
- Clear section separation
- Whitespace for legibility
- Mobile-first responsive design

---

## Accessibility Features

✓ High contrast ratios (WCAG AA minimum)
✓ Keyboard navigation (Tab, Arrow, Enter)
✓ Screen reader support (ARIA labels)
✓ Mobile touch targets (44px minimum)
✓ Color-blind safe palette
✓ No flashing/distracting elements

This comprehensive wireframe set ensures OmniChannelAI is intuitive, fast, and user-friendly for all roles.
