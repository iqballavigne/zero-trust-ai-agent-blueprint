# AI Agent Triage Blueprint: Automated Incident Ticket Validation

An enterprise-grade data governance framework designed to validate structured JSON outputs from AI Agents parsing incoming corporate incident tickets. This architecture utilizes a strict schema validation layer to eliminate non-deterministic LLM behaviors, enforcing conditional routing logic and ticket compliance before payloads hit downstream orchestration systems.
## 📐 System Data Flow

```
   [ Raw Customer Ticket ] 
              │
              ▼
     [ AI Triage Agent ]
              │
              ▼
┌──────────────────────────────────────┐
│    AIAgentTriageBlueprint Gateway    │
│  ──────────────────────────────      │
│  • Enforces Regex ID Patterns        │
│  • Evaluates Multi-Rule Conditional  │◀── [ Immutable Schema Rules ]
│    Logical Blocks (allOf)            │
│  • Restricts Hallucinated Properties │
└──────────────────────────────────────┘
              │
     ┌────────┴──────────────┐
     │                       │
     ▼                       ▼
[ VALID PAYLOAD ]       [ INVALID PAYLOAD ]
     │                       │
     ▼                       ▼
[ PagerDuty / Jira API ] [ Auto-Retry / Prompt Correction ]
```
## 🛠️ Architectural Challenges & Design Choices

* **The Challenge:** LLMs processing unstructured support tickets frequently output unpredictable properties (such as hallucinating a `recommended_action` key), fail to apply consistent urgency scores, or omit critical routing escalations when handling highly frustrated customers.
* **The Naive Approach:** Writing complex application-layer parsing logic to cross-reference customer sentiment against priority fields. This introduces technical debt, slows down execution speeds, and risks breaking down when fields are omitted entirely.
* **The Architectural Solution:** Shifting state-dependent constraints directly into the validation layer using JSON Schema `allOf` and conditional `if/then` statements. The schema dynamically alters field requirements based on data properties at runtime, ensuring complete system safety.
## 💾 Code Highlight: Multi-Rule Conditional Governance

This schema configuration demonstrates how the triage gateway dynamically enforces downstream routing parameters based on real-time agent evaluations using an immutable logical structure:

```json
{
  "$schema": "[https://json-schema.org/draft/2020-12/schema](https://json-schema.org/draft/2020-12/schema)",
  "title": "AIAgentTriageBlueprint",
  "type": "object",
  "properties": {
    "analysis": {
      "type": "object",
      "properties": {
        "sentiment": { "type": "string", "enum": ["positive", "neutral", "negative", "frustrated"] },
        "urgency_score": { "type": "integer", "minimum": 1, "maximum": 5 },
        "priority_handling": { "type": "boolean" },
        "escalation_target": { "type": "array", "items": { "type": "string" } }
      },
      "allOf": [
        {
          "dependentComment": "Rule A: If urgency_score is 5, escalation_target is required.",
          "if": { "properties": { "urgency_score": { "const": 5 } } },
          "then": { "required": ["escalation_target"] }
        },
        {
          "dependentComment": "Rule B: If sentiment is frustrated, priority_handling must be explicitly true.",
          "if": { "properties": { "sentiment": { "const": "frustrated" } } },
          "then": {
            "properties": { "priority_handling": { "const": true } },
            "required": ["priority_handling"]
          }
        }
      ]
    }
  }
}
```
