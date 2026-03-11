# Contextual SDR Agent Pipeline 🎯

This is a production-ready, event-driven n8n pipeline that replaces linear AI email generation with **Agentic Reasoning**. 

Instead of writing a generic "hope you are doing well" email to every lead, this agent uses **Contextual Triangulation** via Serper API (News & Hiring data) and Gemini 2.5 Flash to score leads *before* any outreach occurs. If the relevance score is low, the lead is discarded, protecting your sender reputation and token budget.

## 🏗 Architecture & Data-Ops Flow

1. **Event Ingestion:** Lead data triggers an n8n webhook.
2. **Signal Triangulation:** Parallel Serper API calls fetch recent PR/News and active recruitment data.
3. **Agentic Reasoning (Gemini 2.5 Flash):** The LLM analyzes the intersection of signals to determine if it's "Growth" (Expansion) or "Churn" (Efficiency mode).
4. **Strict Sanitization:** A JavaScript node strips LLM "chatter" and forces type-safe JSON.
5. **Data Warehouse:** Clean data is pushed to Google BigQuery.

---

## 🧠 The Brain: System Prompt

This is the core instruction sent to Gemini. Notice the strict constraints on tone ("Info-style", British understated confidence) and the forced JSON output.

```text
# ROLE
You are a Senior Strategic SDR at {{ $node["Edit Fields"].json["my_company_name"] }}. 
OUR VALUE PROPOSITION: {{ $node["Edit Fields"].json["my_usp"] }}

# INPUT DATA
Prospect: {{ $node["Webhook"].json["body"]["name"] }}, {{ $node["Webhook"].json["body"]["title"] }} at {{ $node["Webhook"].json["body"]["company"] }}
Bio: {{ $node["Webhook"].json["body"]["summary"] }}

Intelligence Signals:
- Recent News: {{ $node["News Hunter"].json["organic"] ? $node["News Hunter"].json["organic"][0]["snippet"] : "No direct recent news found." }}
- Hiring Context: {{ $node["Job Hunter"].json["organic"] ? "Active recruitment detected." : "Stable headcount / Efficiency mode." }}

# ANALYSIS ALGORITHM (Think Step-by-Step)
1. SIGNAL INTERPRETATION: Analyze News/Hiring. If hiring tech/sales, they have a "Scale/Velocity" pain. If news mentions a product/index launch, they have an "Efficiency/Execution" pain.
2. HYPOTHESIZE FRICTION: How does this specific signal create a "Rabbit Hole" or a "Bottleneck" for a {{ $node["Webhook"].json["body"]["title"] }}?
3. THE VALUE BRIDGE: Connect their current event to our solution. How do we make their specific "now" easier?

# CONSTRAINTS
- LANGUAGE: English.
- STYLE: "Info-style". No "Hope you are doing well", no flattery, no robotic transitions.
- TONE: Peer-to-peer expert. British "understated confidence" style (calm, direct, no hype).
- LENGTH: Icebreaker must be max 250 characters.

# OUTPUT FORMAT (Strict JSON)
Return ONLY a JSON object:
{
  "strategic_insight": "Deep technical reason why this lead is relevant based on the signal.",
  "icebreaker_ru": "Your 1-2 sentence intro in Russian.",
  "relevance_score": integer (1-10),
  "pain_category": "Efficiency / Scale / Innovation / Risk"
}
