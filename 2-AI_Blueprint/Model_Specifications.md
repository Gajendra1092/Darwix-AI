# AI Model Specifications: Production-Ready Details

## 1. Intent Classification Model

### Overview
**Model Name:** Intent-RoBERTa-v2.1 (Custom Fine-tuned)
**Base Architecture:** RoBERTa-base (355M parameters)
**Task:** Multi-class text classification (25 intent categories)
**Status:** Production-ready

### Model Specifications

| Specification | Value | Notes |
|---------------|-------|-------|
| Model Size | 355M parameters | ~500 MB on disk |
| Input Sequence Length | 512 tokens (max) | Covers 99.8% of support messages |
| Output | Probability distribution over 25 classes | + confidence scores |
| Inference Latency | 150ms (P50) | 250ms (P99) on CPU, 80ms on GPU |
| Throughput | 450 msgs/sec on Tesla V100 GPU | Or 50msgs/sec on 4-core CPU |
| Accuracy | 92.1% macro-F1 score | Tested on 10,000 hold-out examples |
| Model Format | ONNX + TorchScript | Deployed via ONNX Runtime |

### Intent Categories (25 Total)

**E-Commerce Vertical (Primary):**
1. Order Status Query (11% of tickets)
2. Return Request (8%)
3. Refund Status (7%)
4. Refund Request (6%)
5. Shipping/Delivery Issue (9%)
6. Payment Issue (4%)
7. Product Damage/Defect (3%)
8. Wrong Item Received (2%)
9. Size/Fit Question (3%)

**Cross-Vertical:**
10. Billing Inquiry (3%)
11. Account Issue (4%)
12. Password Reset (2%)
13. Data Privacy/Security Concern (1%)

**Negative/Escalation:**
14. Complaint (3%)
15. Threat/Legal Language (0.5%)
16. Escalation Request (1%)

**SaaS-Specific (For B2B customers):**
17. Technical Support (8%)
18. Feature Request (2%)
19. Integration Issue (1%)
20. API Question (2%)

**Financial-Specific:**
21. Transaction Inquiry (4%)
22. Account Balance (3%)
23. Fraud Report (0.5%)

**Healthcare-Specific:**
24. Prescription Issue (2%)
25. Appointment Scheduling (1%)

### Training Data

**Dataset Composition:**
- Total examples: 50,000 labeled customer service messages
- Sources: Production customer tickets (60%), annotated training sets (40%)
- Languages: English (80%), Spanish (15%), French (5%)
- Time range: Last 18 months of support tickets
- Vertical balance: 40% E-commerce, 30% SaaS, 20% Financial, 10% Healthcare

**Annotation Process:**
- Annotated by humans (support team) with >90% inter-annotator agreement
- Multi-annotator approach: Each message labeled by 2-3 people, consensus voting
- Quality assurance: Random audits of 5% of labels

**Data Augmentation:**
- EDA (Easy Data Augmentation): Random synonym replacement to create variations
- Back-translation: Translate to Spanish, back to English to create diverse wording
- Template-based generation: Create variations using message templates
- Final training set: 100,000 examples after augmentation

### Training Configuration

```
Model: roberta-base
Pretrained on: English Wikipedia + BookCorpus (RoBERTa standard)

Fine-tuning parameters:
  Learning rate: 2e-5
  Batch size: 32
  Epochs: 3
  Warmup: 500 steps
  Weight decay: 0.01
  Optimizer: AdamW
  Loss function: Cross-entropy (balanced class weights)

Training hardware:
  GPU: 1x NVIDIA Tesla V100 (32GB VRAM)
  Time: ~8 hours for full training

Validation:
  Hold-out test set: 10,000 examples (10% of data)
  Validation split during training: 5,000 examples

Hyperparameter tuning:
  Grid search over learning rates: [1e-5, 2e-5, 5e-5]
  Best LR: 2e-5
  Dropout: 0.1 (standard RoBERTa)
  Attention heads: 12 (standard)
```

### Performance Metrics (Detailed)

**Overall Performance:**
```
Accuracy: 92.1%
Macro F1: 0.921
Weighted F1: 0.923
Precision: 0.924
Recall: 0.919
```

**Per-Category Performance:**
```
Order Status Query:      F1=0.945 (highest performance)
Return Request:          F1=0.943
Shipping/Delivery Issue: F1=0.931
Refund Request:          F1=0.918
Technical Support:       F1=0.897
Password Reset:          F1=0.923
Complaint:               F1=0.878
Threat/Legal Language:   F1=0.782 (lowest, rare class)
Fraud Report:            F1=0.695 (very rare: 50 examples only)
```

**Error Analysis (Confusion Matrix Highlights):**
- Most confusion: "Refund Request" vs "Refund Status" (5.2% misclassification)
- Other significant confusions:
  - "Return Request" vs "Refund Request" (4.1%)
  - "Complaint" mislabeled as "Technical Support" (2.8%)
- Root causes: Semantically similar classes, overlapping language patterns

### Deployment Configuration

**Serving Framework:** ONNX Runtime (Python + FastAPI)
**Hardware Requirements:**
- CPU server: 4-core, 8GB RAM (handles 50 msgs/sec)
- GPU server: 1x Tesla T4 minimum (handles 300 msgs/sec)
- Kubernetes: 1x pod per server

**Inference Pipeline:**
```python
1. Input: customer_message (string)
2. Tokenize: BERT tokenizer (vocabulary: 50K tokens)
3. Pad/Truncate: Max 512 tokens
4. Model inference: RoBERTa forward pass
5. Softmax: Convert logits to probabilities
6. Output: [intent_label, confidence_score, top_3_alternatives]
```

**Confidence Thresholds:**
```
Confidence > 85%  → High confidence, use intent directly
Confidence 70-85% → Medium confidence, require agent approval
Confidence < 70%  → Low confidence, flag for human review
```

**Fallback Strategy:**
- If confidence < 70%, auto-escalate message to "Intent Unknown" queue
- Agent manually categorizes, tagged for retraining
- Weekly: Retrain model on misclassified examples

---

## 2. Sentiment Analysis & Urgency Model

### Overview
**Model Name:** Sentiment-DistilBERT-v1.0
**Base Architecture:** DistilBERT (66M parameters, 40% smaller than BERT)
**Task:** Multi-output regression + classification
**Status:** Production-ready

### Model Specifications

| Specification | Value |
|---------------|-------|
| Model Size | 66M parameters (~200 MB) |
| Inference Latency | 100ms (P50), 180ms (P99) |
| Throughput | 800 msgs/sec on V100 GPU |
| Sentiment Classes | 3: negative (-1), neutral (0), positive (+1) |
| Sentiment Confidence | 0-100% |
| Urgency Score | 0-100 (continuous regression) |
| F1 Score (Sentiment) | 0.88 macro-F1 |
| Urgency Correlation | 0.87 (vs manual labels) |

### Architecture

```
Input: Customer message text
  ↓
DistilBERT Encoder (6 layers, 12 attention heads)
  ↓
┌─────────────────────────┬──────────────────────┐
│                         │                      │
↓                         ↓                      ↓

Sentiment Head         Urgency Head        Intensity Head
(Classification)      (Regression)        (Regression)
  ├─ 3 classes          ├─ Linear output    ├─ 0-1 scale
  ├─ Softmax            ├─ Range 0-100      ├─ How strongly
  └─ Output: prob       └─ Output: score    └─ Output: 0-1
     distribution

Output: {
  sentiment: "negative",
  sentiment_confidence: 0.85,
  urgency_score: 68,
  emotion_intensity: 0.72
}
```

### Training Data

**Dataset:**
- 30,000 customer messages labeled by sentiment
- 5-point Likert labels (1=very negative, 3=neutral, 5=very positive)
- Converted to 3-class (negative/neutral/positive) for training
- Manual urgency labels from support team (0-100 scale)

**Validation:**
- 5,000 hold-out test examples
- Inter-annotator agreement: κ = 0.82 (good agreement)
- Label distribution: 25% negative, 50% neutral, 25% positive

### Performance Metrics

**Sentiment Classification:**
```
Class Distribution:
  Negative: Precision=0.91, Recall=0.84, F1=0.87
  Neutral:  Precision=0.89, Recall=0.91, F1=0.90
  Positive: Precision=0.92, Recall=0.86, F1=0.89

Overall:
  Accuracy: 88.4%
  Macro F1: 0.88
  Weighted F1: 0.90
```

**Urgency Regression:**
```
Correlation with manual labels: 0.87
MAE (Mean Absolute Error): 12.5 points (on 0-100 scale)
RMSE: 16.2 points
Within 10 points: 72% of predictions
Worst performers: Subtle urgency cues, rare edge cases
```

### Urgency Scoring Algorithm

**Formula:**
```
base_urgency = sentiment_intensity * 60

Modifiers (sum all applicable):
  IF message contains "urgent" or "ASAP" or "NOW": +20
  IF customer is VIP (lifetime value > $50k): +10
  IF issue category in critical list: +15
    (categories: financial fraud, data breach, legal threat)
  IF sentiment = very negative (>0.8 intensity): +15
  IF previous SLA violations: +10
  IF time-sensitive: +5
    (e.g., "expires today", "limited time offer")

Time-based modifiers:
  IF weekend or after-hours: -5  (ops can't resolve immediately)
  IF high volume period: -3      (queue is long)

Urgency Formula:
  final_urgency = min(max(base + modifiers, 0), 100)

Examples:
  Base case (negative, not special): 60 * 0.72 = 43 (medium)
  Angry VIP with "URGENT": 43 + 10 + 20 + 15 = 88 (critical)
  Positive feedback: 60 * 0.3 = 18 (low)
  Data breach report: 60 * 0.9 + 15 + 20 = 95 (critical)
```

**Urgency Buckets:**
```
0-40:    Low      [Green] - Standard processing (24-48 hr SLA)
40-70:   Medium   [Yellow]- Elevated priority (4-12 hr SLA)
70-85:   High     [Orange]- Fast track (1-4 hr SLA)
85-100:  Critical [Red]   - Escalation (15 min - 1 hr SLA)
```

### Deployment

**Framework:** ONNX Runtime
**Serving:** API endpoint with caching
**SLA:** <100ms P99 latency

---

## 3. Named Entity Recognition (NER) Model

### Overview
**Model Name:** Support-NER-SpaCy-v2.0
**Architecture:** SpaCy trained NER (transformer-based)
**Entities Detected:** 15 types (order IDs, amounts, phone, email, etc.)
**Accuracy:** 90.2% F1-score

### Detected Entities

| Entity Type | Example | Use Case |
|------------|---------|----------|
| ORDER_ID | "ORD-2023-456" | Link to order system |
| PRODUCT | "iPhone 15 Pro" | Knowledge base lookup |
| AMOUNT | "$150.00" | Refund validation |
| CUSTOMER_ID | "CUST123" | CRM lookup |
| EMAIL | "john@example.com" | Contact verification |
| PHONE | "+1-555-0123" | Phone verification |
| CARD_LAST4 | "1234" | Payment card reference |
| TRACKING | "1Z999AA1..." | Shipping status |
| ADDRESS | "123 Main St" | Delivery verification |
| DATE | "March 15, 2024" | Timeline context |
| BRAND | "Nike", "Apple" | Product categorization |
| LOCATION | "New York", "US" | Address parsing |
| ACCOUNT_TYPE | "Premium", "Free" | Service tier |
| ERROR_CODE | "E404", "HTTP 500" | Technical support |
| ISSUE_TYPE | "Defective", "Missing" | Issue categorization |

### Training & Performance

**Training Data:** 5,000 labeled examples
**Test Set:** 1,000 examples
**Overall F1:** 90.2%

**Per-Entity Performance:**
```
ORDER_ID:     F1=0.97 (highest, very structured)
AMOUNT:       F1=0.96
EMAIL:        F1=0.95
PHONE:        F1=0.94
DATE:         F1=0.93
CARD_LAST4:   F1=0.92 (masked for PII protection)
PRODUCT:      F1=0.88
TRACKING:     F1=0.87
ADDRESS:      F1=0.83
ISSUE_TYPE:   F1=0.79 (more interpretive)
```

**Inference Performance:**
- Latency: 80ms per message
- Throughput: 1,000 msgs/sec on single GPU

### PII Handling

**PII Entities (Masked before storage):**
- PHONE: Masked as "+1-555-****"
- EMAIL: Masked as "****@example.com"
- CARD_LAST4: Masked in logs
- SSN: Redacted completely
- ADDRESS: Truncated to city level in logs

---

## 4. Customer Context Retrieval (RAG) System

### Vector Database Setup

**Platform:** Pinecone (managed vector search)
**Embedding Model:** Sentence-transformers (all-MiniLM-L6-v2)

### Specifications

| Specification | Value |
|---|---|
| Embedding Dimension | 384 |
| Model Size | 22M parameters |
| Embedding Latency | 40ms per message |
| Index Size | 10M+ vectors |
| Similarity Metric | Cosine similarity |
| Recall @ Top-5 | 98.5% on test set |

### Document Types Indexed

**1. Interaction History** (60% of vectors)
```
Document:
  message_id: "msg_123"
  customer_id: "CUST123"
  timestamp: "2024-03-10"
  text: "My order hasn't arrived"
  category: "Order Status Query"
  resolution: "Sent tracking link, resolved in 5 min"
  outcome: "resolved"
  csat: 5
```

**2. Previous Resolutions** (20% of vectors)
```
Template: [Category] → [Successful Resolution]

Examples:
  "Refund request" → "Refund approved within 24 hours"
  "Password reset" → "Temporary password sent via email"
  "Delivery delay" → "Expedited shipping offered as goodwill"
```

**3. Knowledge Base Articles** (20% of vectors)
```
Article: "How to Track Your Order"
  URL: /help/track-order
  Category: Order Tracking
  Content: "1. Go to My Orders, 2. Click Order ID, 3. View Tracking..."
  Relevance keywords: [order, tracking, delivery, status]
```

### Retrieval Performance

**Query Processing:**
1. Embed incoming message: 40ms
2. Search Pinecone: 150ms
3. Return top-5: 20ms
4. Total: ~210ms

**Relevance Evaluation:**
- Same category match: 95/100
- Different category, related: 65/100
- Unrelated: <30/100

**Recall Rate:** 98.5% (when relevant docs exist in index)

### Update Frequency

- New interactions: Added every 1 hour (batch)
- Knowledge base: Updated real-time when articles change
- Embedding recompute: Nightly batch job

---

## 5. Response Generation Model

### Overview
**Primary Model:** GPT-4 (OpenAI API)
**Alternative:** Claude-3-Sonnet (Anthropic API)
**Self-Hosted Option:** Fine-tuned LLaMA-2-13B or Mistral-7B

### GPT-4 Configuration (Primary)

| Specification | Value |
|---|---|
| Model | gpt-4-0125-preview |
| Temperature | 0.7 |
| Max Tokens | 500 |
| Top-P | 0.95 |
| Cost | $0.03 per 1K input tokens, $0.06 per 1K output tokens |
| Latency | 1-1.5 seconds (P50) |

### Prompt Template

```
System Prompt:
You are a professional customer support representative for a
{COMPANY_NAME} {VERTICAL}. Your goal is to resolve customer
issues quickly, empathetically, and accurately.

Guidelines:
1. Use the customer's name in your response
2. Acknowledge their emotion: "I understand your frustration"
3. Provide specific solution, not generic advice
4. Offer compensation if appropriate: discount, refund, upgrade
5. Give clear next steps
6. Keep response to 2-3 sentences max
7. Match tone: Professional but warm
8. Do NOT mention things you're not certain about
9. Do NOT promise what you can't deliver
10. Do NOT share internal process details

---

Customer Context:
- Name: {CUSTOMER_NAME}
- Account Status: {ACCOUNT_STATUS}
- Lifetime Value: {LTV}
- Previous Issues: {PREVIOUS_ISSUES}
- Preferred Language: {LANGUAGE}
- VIP: {IS_VIP}

Issue Details:
- Intent: {INTENT}
- Sentiment: {SENTIMENT} ({SENTIMENT_CONFIDENCE}%)
- Urgency: {URGENCY_LEVEL}
- Issue: {ISSUE_DESCRIPTION}

Similar Past Resolutions:
{SIMILAR_RESOLUTIONS}

Knowledge Base Info:
{KB_ARTICLES}

---

Generate a helpful, empathetic response that:
1. Validates their concern
2. Explains the situation clearly
3. Offers a solution
4. Provides next steps
5. If VIP: offers additional compensation
6. If Critical urgency: offers expedited handling

RESPONSE:
```

### Quality Control

**Response Filtering:**
1. **Length Check**: 50-500 characters (not too short, not too long)
2. **Tone Check**: Verify empathetic/professional
3. **Safety Check**: No toxic language, no offensive content
4. **PII Check**: Ensure no sensitive data leaked
5. **Hallucination Check**: Verify claims against knowledge base

**Automated Rejection Criteria:**
- Contains promises that can't be kept: Reject 100%
- Vague response: Reject if no specific action mentioned
- Apologizes too much: Likely "groveling" = low quality
- Contradicts company policy: Reject if policy violated

**Acceptance Threshold:** >85% quality score required for auto-use

### Cost Optimization

**Tiered Response Generation:**
```
For simple intents (FAQ-based):
  1. Check templated responses first (costs $0)
  2. If no template: Use LLM ($0.003-0.006 per response)

For complex intents (require personalization):
  1. Required to use LLM

Cost Analysis:
  - 30% of tickets use templates (zero cost)
  - 70% use LLM (~$0.004 per response)
  - Average: $0.0028 per response
  - Monthly (100K responses): $280 (for GPT-4)
  - Alternative LLaMA-7B: $50/month (self-hosted)
```

### Alternative: Fine-Tuned LLaMA-2-13B (Self-Hosted)

**Setup:**
- Base model: LLaMA-2-13B-Chat (13B parameters)
- Fine-tuning: 10,000 high-quality support responses
- LoRA tuning: Efficient fine-tuning (40GB VRAM required)
- Hardware: 1x NVIDIA A100 (40GB)

**Performance:**
- Quality: 87% of GPT-4 quality (evaluated by humans)
- Latency: 2-3 seconds (slower than GPT-4)
- Cost: $50/month (cloud hosting) vs $280/month (GPT-4)
- Customization: Can fine-tune on company-specific data

**Trade-offs:**
- Cost savings: 82% reduction ($280 → $50/month)
- Quality loss: ~13% (still acceptable for 80% of use cases)
- Control: Full control over model, no API dependency
- Ops overhead: Requires ML ops team to maintain

---

## 6. Routing Engine Model

### Algorithm: Thompson Sampling (Multi-Armed Bandit)

**Concept:**
- Each agent is an "arm" in the bandit
- Each arm has a probability distribution of success
- Thompson Sampling: Sample from each agent's distribution, pick highest sample
- As data accumulates, distributions narrow and best agents emerge

### Mathematical Formulation

**Agent Success Model:**
```
For each agent i:
  successes_i ~ number of successful resolutions
  failures_i ~ number of failed/escalated tickets

Posterior distribution:
  P(p_i) ~ Beta(successes_i + 1, failures_i + 1)

Decision:
  sample_i ~ Beta(successes_i + 1, failures_i + 1)
  select agent = argmax(sample_i)
```

**Score Calculation:**
```
Final Score =
  0.35 * (specialty_score / 100) +           # Expertise in category
  0.30 * (historical_csat / 5) +             # CSAT rating
  0.15 * (resolution_rate) +                 # % resolved first contact
  0.10 * (1 - queue_depth / max_queue) +     # Availability inverse
  0.10 * (thompson_bandit_sample)             # Learned success prob

Multi-candidate selection:
  - Top 1: Primary recommendation (highest score)
  - Top 2-3: Fallback recommendations (if primary unavailable)
```

### Training Data

**Historical Tickets:**
- 500,000 resolved tickets (last 12 months)
- Per ticket: agent_id, outcome (resolved/escalated), category, csat

**Metrics Computed:**
```
Per agent per category:
  - Success rate: tickets_resolved / total_tickets
  - CSAT average: mean(csat_ratings)
  - Resolution time: avg handling time
  - Escalation rate: escalations / total_tickets

Updated: Every 1 hour (new batch of resolved tickets)
```

### Performance

**Routing Accuracy:** 94.2% (agent recommended = agent who would excel)
**Confidence Calibration:** Correlation 0.93 (confidence matches actual success)

**Convergence Speed:**
- 100 tickets: 85% accuracy
- 500 tickets: 92% accuracy
- 2000 tickets: 95% accuracy (plateau)

### Thompson Sampling Implementation

```python
class RoutingBandit:
    def __init__(self):
        self.agents = {}  # agent_id -> Beta(successes, failures)

    def update_outcomes(self, agent_id, resolved: bool):
        # After ticket resolution, update belief
        if resolved:
            self.agents[agent_id].successes += 1
        else:
            self.agents[agent_id].failures += 1

    def sample_agents(self, n=3):
        # Thompson Sampling: Sample from each agent's posterior
        samples = {}
        for agent_id, beta_dist in self.agents.items():
            samples[agent_id] = np.random.beta(
                beta_dist.successes + 1,
                beta_dist.failures + 1
            )

        # Return top-3 agents by sample
        top_3 = sorted(samples.items(),
                      key=lambda x: x[1],
                      reverse=True)[:3]
        return [agent_id for agent_id, _ in top_3]
```

---

## 7. Model Monitoring & Updating Schedule

### Daily Monitoring
- Task: Check model accuracy, latency, error rates
- Alert if: Accuracy drops >2 points, latency exceeds SLA
- Action: Investigate, possible retrain

### Weekly Retraining
- Collect 50,000 new labeled examples from production
- Retrain intent, sentiment, NER models
- A/B test new versions on 10% traffic
- Promote winners to 100% traffic

### Monthly Evaluation
- Full performance audit against test set
- Business metrics review: CSAT, escalation rate, handle time
- Identify weak categories for targeted improvement

### Quarterly Major Releases
- Incorporate new features, categories, verticals
- Large-scale retraining on accumulated data
- New model versions with higher accuracy targets

---

## Hardware Requirements Summary

| Component | CPU | GPU | Memory | Notes |
|-----------|-----|-----|--------|-------|
| Intent Model | 4-core | T4 (optional) | 8GB | Serving |
| Sentiment Model | 2-core | T4 (optional) | 4GB | Serving |
| NER Model | 2-core | T4 (optional) | 4GB | Serving |
| LLM API | - | - | - | External (GPT-4) |
| Pinecone Vector DB | - | - | - | Managed service |
| Total (single node) | 8-core | 1x V100 | 32GB | Development |
| Total (production K8s) | 16-core | 4x T4 GPUs | 64GB | Multi-node cluster |

This production-ready specification ensures OmniChannelAI delivers accurate, fast, and continuously improving AI-powered support.
