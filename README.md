# 🛡️ AI Agent Triage Blueprint: Automated Incident Ticket Validation

An enterprise-grade data governance framework designed to validate structured JSON outputs from AI Agents parsing incoming corporate incident tickets. This architecture utilizes a strict schema validation layer to eliminate non-deterministic LLM behaviors, enforcing conditional routing logic and ticket compliance before payloads hit downstream orchestration systems.

---

## 📐 Project Overview & Core Architecture

The core runtime gateway converts unpredictable, open-ended LLM text parsing into deterministic, structurally sound payloads. This ensures that automated incident classification, alerting systems, and SLA calculations operate without risk of data corruption or prompt injection side effects.

```text
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

## 🧠 Core Engineering Challenges & Architectural Solutions

This blueprint provides elegant, deterministic answers to three classic structural problems found in unstructured-to-structured AI data pipelines:

### 1. The Hallucinating LLM Dilemma
* **The Problem:** LLMs processing unstructured text frequently hallucinate unmapped properties (such as injecting an unexpected `recommended_action` key) or output malformed fields that downstream databases cannot parse.
* **The Solution:** Implemented an uncompromising root-level and sub-object `"additionalProperties": false` constraint strategy. This turns the validation layer into a zero-trust sandbox; any unmapped key returned by an erratic model is immediately blocked at the gateway, forcing an explicit structural format.

### 2. State-Dependent Urgency Escalation
* **The Problem:** Enterprise tickets flagged with maximum severity require precise destination routing arrays (e.g., `devops`, `security`). Relying on the LLM to remember to include these arrays inside its generated string is unreliable, resulting in unrouted critical system failures.
* **The Solution:** Utilized sequential `if/then` conditional validation logic gates nested inside an object-level `allOf` compiler array. If the model computes an `urgency_score` literal of `5`, the validation engine dynamically overrides standard requirements and mandates an explicit, non-empty `escalation_target` string array.

### 3. Sentiment-Driven Priority Enforcement
* **The Problem:** Frustrated customers require immediate systemic priority overrides. Writing brittle application-level logic to post-process text sentiment and retroactively flip priority flags introduces unnecessary compute latency and tight architectural coupling.
* **The Solution:** Enforced an inline semantic guardrail. When the schema identifies a customer sentiment state of `"frustrated"`, it evaluates an active constraint requiring the immediate presence of a `priority_handling` flag, while simultaneously restricting its payload value strictly to a `const: true` boolean literal.

---

## 💾 Code Highlight: Multi-Rule Conditional Governance

The triage gateway enforces deterministic data constraints by evaluating the agent's structural JSON output against immutable logical rules at runtime. Rather than relying on post-parsing application scripts, the schema evaluates co-dependent fields instantly using a strict validation block.

```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
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

### Key Structural Mechanics

* **Strict Temporal & Indexing Patterns:** Validates the `ticket_id` property against an enterprise-wide regular expression layout (`^INC-2026-[0-9]{4}$`). This ensures rigid chronological sorting and cryptographic indexing constraints are maintained at the database edge.
* **Conditional Escalation Safeguards (Rule A):** Instantly hooks into the agent's calculations. If the model sets the `urgency_score` integer to `5`, the schema overrides optional constraints and mandates an `escalation_target` array to prevent high-severity incidents from stalling without engineering ownership.
* **SLA Protection Bounds (Rule B):** Evaluates the parsed data stream for volatile inputs. If the model identifies customer sentiment as `"frustrated"`, the validation engine forces the `priority_handling` flag to resolve to a literal `true` boolean, eliminating human or model oversight on high-visibility complaints.
* **Zero-Trust Surface Area Reduction:** Enforces an absolute `"additionalProperties": false` restriction at every structural tier. This immediately quarantines and rejects payloads containing unmapped keys, serving as a primary defense against LLM prompt leakages and structural hallucinations.

---

## 📊 System Validation Matrix

The triage engine maps incoming structured AI payloads against specific logic gates to guarantee that automated incident classification and escalation tasks run without data corruption.

| Test File | Target Coordinate | Input Payload State | Expected Outcome | Active Constraint Evaluated |
| :--- | :--- | :--- | :--- | :--- |
| `valid-ticket.json` | `metadata.agent_id` | `"AGT-1024"` | **🟢 PASS** | Matches regex sequence string template (`^AGT-[0-9]{4}$`). |
| `valid-ticket.json` | `analysis` | `sentiment: "frustrated"` + `urgency_score: 5` | **🟢 PASS** | Satisfies both conditional `if/then` constraints simultaneously by supplying all required sub-keys. |
| `invalid-ticket.json` | `analysis` | `urgency_score: 5` with missing `escalation_target` | **🔴 FAIL** | **Conditional Boundary Breach:** Triggered `if` statement for max urgency but failed the `then` constraint by omitting the routing array. |
| `invalid-ticket.json` | Root Coordinate | Injects unmapped `"recommended_action"` key | **🔴 FAIL** | **Schema Surface Area Breach:** AI hallucination blocked cleanly at the boundary gate by the strict `"additionalProperties": false` constraint. |
---
