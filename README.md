# AI Agent Triage Blueprint: Automated Incident Ticket Validation

An enterprise-grade data governance framework designed to validate structured JSON outputs from AI Agents parsing incoming corporate incident tickets. This architecture utilizes a strict schema validation layer to eliminate non-deterministic LLM behaviors, enforcing conditional routing logic and ticket compliance before payloads hit downstream orchestration systems.

---

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
---

## 🛠️ Architectural Challenges & Design Choices

* **The Challenge:** LLMs processing unstructured support tickets frequently output unpredictable properties (such as hallucinating a `recommended_action` key), fail to apply consistent urgency scores, or omit critical routing escalations when handling highly frustrated customers.
* **The Naive Approach:** Writing complex application-layer parsing logic to cross-reference customer sentiment against priority fields. This introduces technical debt, slows down execution speeds, and risks breaking down when fields are omitted entirely.
* **The Architectural Solution:** Shifting state-dependent constraints directly into the validation layer using JSON Schema `allOf` and conditional `if/then` statements. The schema dynamically alters field requirements based on data properties at runtime, ensuring complete system safety.

---

## 💾 Code Highlight: Multi-Rule Conditional Governance

This schema configuration demonstrates how the triage gateway dynamically enforces downstream routing parameters based on real-time agent evaluations using an immutable logical structure:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "AIAgentTriageBlueprint",
  "description": "Validates structured output from AI Agent parsing incoming incident tickets.",
  "type": "object",
  "required": ["metadata", "analysis", "ticket_details"],
  "additionalProperties": false,
  "properties": {
    "metadata": {
      "type": "object",
      "required": ["agent_id", "timestamp", "schema_version"],
      "additionalProperties": false,
      "properties": {
        "agent_id": {
          "type": "string",
          "pattern": "^AGT-[0-9]{4}$"
        },
        "timestamp": {
          "type": "string",
          "format": "date-time"
        },
        "schema_version": {
          "type": "string",
          "const": "1.0.0"
        }
      }
    },
    "ticket_details": {
      "type": "object",
      "required": ["ticket_id", "raw_summary"],
      "additionalProperties": false,
      "properties": {
        "ticket_id": {
          "type": "string",
          "pattern": "^INC-2026-[0-9]{4}$"
        },
        "raw_summary": {
          "type": "string",
          "maxLength": 500
        }
      }
    },
    "analysis": {
      "type": "object",
      "required": ["sentiment", "urgency_score", "confidence_score"],
      "additionalProperties": false,
      "properties": {
        "sentiment": {
          "type": "string",
          "enum": ["positive", "neutral", "negative", "frustrated"]
        },
        "urgency_score": {
          "type": "integer",
          "minimum": 1,
          "maximum": 5
        },
        "confidence_score": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 1.0
        },
        "priority_handling": {
          "type": "boolean"
        },
        "escalation_target": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["devops", "security", "legal", "customer-success"]
          },
          "minItems": 1,
          "uniqueItems": true
        }
      },
      "allOf": [
        {
          "dependentComment": "Rule A: If urgency_score is 5, escalation_target is required.",
          "if": {
            "properties": {
              "urgency_score": { "const": 5 }
            }
          },
          "then": {
            "required": ["escalation_target"]
          }
        },
        {
          "dependentComment": "Rule B: If sentiment is frustrated, priority_handling must be explicitly true.",
          "if": {
            "properties": {
              "sentiment": { "const": "frustrated" }
                }
          },
          "then": {
            "properties": {
              "priority_handling": { "const": true }
            },
            "required": ["priority_handling"]
          }
        }
      ]
    }
  }
}
```
---

### Key Structural Mechanics:
1. **Strict Temporal Patterns:** Validates `ticket_id` structures strictly against an enterprise-wide `^INC-2026-[0-9]{4}$` regular expression layout, ensuring chronological indexing constraints are maintained.
2. **Conditional Escalation Safeguards (Rule A):** If the LLM sets `urgency_score` to 5, the schema automatically mandates an `escalation_target` array to prevent critical incidents from stalling without team ownership.
3. **SLA Protection Bounds (Rule B):** If the LLM identifies a customer status as `frustrated`, the schema forces `priority_handling` to evaluate to a literal `true`, eliminating human/AI oversight on high-visibility complaints.
4. **Surface Area Reduction:** Enforces `additionalProperties: false` at every tier, immediately rejecting payloads containing unapproved or hallucinated model components.

---

## 🛡️ Validation & Verification Matrix

The validation gateway acts as a hard boundary, ensuring data pipelines execute with verified system payloads:

| Payload Scenario | Structural State | System Result | Reason |
| :--- | :--- | :--- | :--- |
| **Valid Critical Ingestion** | `urgency_score: 5`, `sentiment: "frustrated"`, `priority_handling: true`, `escalation_target: ["devops", "security"]`, `ticket_id: "INC-2026-8841"` | **PASS (200 OK)** | Fully complies with root validation, regular expression blocks, and both conditional rules simultaneously. |
| **Malformed Identifier** | `ticket_id: "INC-2026-841"` | **FAIL (400 Bad Request)** | Fails compliance pattern check; regular expressions require exactly four trailing digits. |
| **Critical Escalation Failure** | `urgency_score: 5`, `escalation_target` omitted | **FAIL (400 Bad Request)** | Blocked by conditional Rule A; high-priority tickets must include routing targets. |
| **Hallucinated Property** | Contains root-level `"recommended_action"` key | **FAIL (400 Bad Request)** | Caught by strict `additionalProperties: false` layout restrictions. |
