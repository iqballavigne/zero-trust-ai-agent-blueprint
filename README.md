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
          "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}Z$"
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

* **Strict Temporal & Indexing Patterns:** Validates the `timestamp` property against a rigid ISO regular expression pattern (`^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}Z$`) to enforce strict temporal formatting at the database edge without relying on external environment plugins.
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
## 🌐 Semantic Graph Architecture

While the JSON Schema gateway handles structural enforcement at the ingestion boundary, the triage system streams validated incident states into a centralized semantic knowledge graph. This architecture transforms isolated support tickets into interlinked corporate intelligence entities, allowing cross-departmental operations teams to analyze macro-level system health and agent performance using graph-based semantic queries.

### 🧠 Ontological Architecture (TBox vs. ABox)

The triage ontology separates operational metadata from dynamic incident state declarations:
* **The TBox (Terminology Box):** Structures the organizational schema vocabulary. It defines the strict boundaries of classes (`ai:TriageAgent`, `ticket:IncidentTicket`) and maps structural constraints regarding how predicates connect data entities (e.g., establishing that a classification rating can only originate from an active agent resource).
* **The ABox (Assertion Box):** Asserts real-world transaction statements. It populates the graph with active data records synchronized directly from `valid-ticket.json`, establishing true graph relations between unique agent IDs, system logs, and security-escalation endpoints.

### 🎯 Graph Structural Equivalents to Schema Constraints

* **Conditional Logic Harmonization:** The conditional `if/then` structural requirements checked by the JSON Schema are represented semantically as explicit entity classifications in the graph database. When an incident manifests high urgency, it links directly to named infrastructure instances (`routing:devops`), ensuring graph trace-analytics match edge routing constraints.
* **Property Hallucination Defense:** By strictly defining property domains and ranges inside the semantic vocabulary block, the knowledge graph acts as an advanced secondary defense shield. Any unmapped or hallucinated key output by an AI model that slips past the edge gateway will violate the core graph topology, causing the database to reject the transaction graph update.

### 🕸️ The Semantic Triage Topology (`triage-ontology.ttl`)
```
@prefix rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:     <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:      <http://www.w3.org/2001/XMLSchema#> .
@prefix ai:       <http://api.sec-ops.internal/ontology/agent#> .
@prefix ticket:   <http://api.sec-ops.internal/ontology/ticket#> .
@prefix routing:  <http://api.sec-ops.internal/ontology/routing#> .
@prefix inst:     <http://api.sec-ops.internal/instances#> .

### 1. TBOX: ONTOLOGICAL FRAMEWORK (The Classes)
ai:TriageAgent rdf:type rdfs:Class ;
    rdfs:label "AI Operational Agent" ;
    rdfs:comment "An active LLM parsing engine validated to categorize inbound incident tickets." .

ticket:IncidentTicket rdf:type rdfs:Class ;
    rdfs:label "Corporate Incident Ticket" .

ticket:AgentAnalysis rdf:type rdfs:Class ;
    rdfs:label "Agent Cognitive Analytics Block" .

routing:EscalationTarget rdf:type rdfs:Class ;
    rdfs:label "Downstream Engineering Department" .

### 2. TBOX: RELATIONSHIPS & BOUNDARIES (The Properties)
# --- Object Properties (Entity to Entity links) ---
ai:processedBy rdf:type rdf:Property ;
    rdfs:domain ticket:IncidentTicket ;
    rdfs:range ai:TriageAgent .

ticket:hasAnalysis rdf:type rdf:Property ;
    rdfs:domain ticket:IncidentTicket ;
    rdfs:range ticket:AgentAnalysis .

ticket:assignedTo rdf:type rdf:Property ;
    rdfs:domain ticket:IncidentTicket ;
    rdfs:range routing:EscalationTarget .

# --- Datatype Properties (Entity to Literal Value links) ---
ai:agentId rdf:type rdf:Property ;
    rdfs:domain ai:TriageAgent ;
    rdfs:range xsd:string .

ticket:ticketId rdf:type rdf:Property ;
    rdfs:domain ticket:IncidentTicket ;
    rdfs:range xsd:string .

ticket:rawSummary rdf:type rdf:Property ;
    rdfs:domain ticket:IncidentTicket ;
    rdfs:range xsd:string .

ticket:sentiment rdf:type rdf:Property ;
    rdfs:domain ticket:AgentAnalysis ;
    rdfs:range xsd:string .

ticket:urgencyScore rdf:type rdf:Property ;
    rdfs:domain ticket:AgentAnalysis ;
    rdfs:range xsd:integer .

ticket:confidenceScore rdf:type rdf:Property ;
    rdfs:domain ticket:AgentAnalysis ;
    rdfs:range xsd:float .

ticket:priorityHandling rdf:type rdf:Property ;
    rdfs:domain ticket:AgentAnalysis ;
    rdfs:range xsd:boolean .

### 3. ABOX: LIVE INSTANCE DATA (Synchronized from valid-ticket.json)
# Agent Instance Definition
inst:Agent_1024 rdf:type ai:TriageAgent ;
    ai:agentId "AGT-1024"^^xsd:string .

# Incident Ticket State Mapping
inst:Ticket_INC-2026-8841 rdf:type ticket:IncidentTicket ;
    ticket:ticketId "INC-2026-8841"^^xsd:string ;
    ai:processedBy inst:Agent_1024 ;
    ticket:rawSummary "Database connection pool is totally exhausted. Core production systems are down..."^^xsd:string ;
    ticket:hasAnalysis inst:Analysis_INC-2026-8841 ;
    ticket:assignedTo inst:Dept_Devops , inst:Dept_Security .

# Cognitive Analysis Layer Mapping (Reflecting Schema Guardrail Outcomes)
inst:Analysis_INC-2026-8841 rdf:type ticket:AgentAnalysis ;
    ticket:sentiment "frustrated"^^xsd:string ;
    ticket:urgencyScore "5"^^xsd:integer ;
    ticket:confidenceScore "0.98"^^xsd:float ;
    ticket:priorityHandling "true"^^xsd:boolean .

# Escalation Infrastructure Target Definitions
inst:Dept_Devops rdf:type routing:EscalationTarget ;
    rdfs:label "DevOps Infrastructure Team" .

inst:Dept_Security rdf:type routing:EscalationTarget ;
    rdfs:label "Information Security Incident Response" .
```
---

## ⚙️ CI/CD Automation & Governance

Automated governance is mandatory when integrating non-deterministic AI outputs into core corporate environments. This repository implements an automated validation gateway using GitHub Actions. Operating inside a high-performance **Node.js 24 runtime environment**, the pipeline acts as an immutable quality gate, ensuring that any structural modifications to the schema model or test controls are automatically validated before being integrated into production.

### 🧠 Architectural Pipeline Mechanics

The automation framework executes three foundational engineering design patterns to protect data integrity:

* **High-Performance Path-Filtering:** The automation runner executes exclusively when targeted modifications are committed to the core parsing rulebook (`ai-agent-triage-schema.json`) or the control payloads (`valid-ticket.json`, `invalid-ticket.json`). This optimization eliminates unnecessary compute consumption during simple updates to documentation or semantic graphs.
* **Positive Boundary Affirmation:** Using the strict-mode **AJV CLI compiler**, the workflow validates the positive control file (`valid-ticket.json`). This proves that compliant agent outputs—containing regularized IDs, safe sentiment fields, and appropriate escalation vectors—successfully satisfy every schema layer rule.
* **Negative Inversion Testing Gate:** To guarantee the schema remains strict enough to stop prompt leakages or model drifts, the pipeline intentionally passes the compromised payload (`invalid-ticket.json`) through the validator. A custom bash logic loop monitors the compilation; if the malformed text accidentally passes the boundary rules, the runner actively overrides the success signal, registers a critical security error, and triggers a hard system halt (`exit 1`).

### 📄 Automation Pipeline Configuration (`validate.yml`)
```
name: AI Agent Triage Validation CI

on:
  push:
    paths:
      - 'ai-agent-triage-schema.json'
      - 'valid-ticket.json'
      - 'invalid-ticket.json'
  pull_request:
    paths:
      - 'ai-agent-triage-schema.json'
      - 'valid-ticket.json'
      - 'invalid-ticket.json'

jobs:
  ai-triage-validation:
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout Repository Code
        uses: actions/checkout@v6

      - name: 🟢 Set up Node.js Runtime
        uses: actions/setup-node@v6
        with:
          node-version: '24'

      - name: 🛠️ Install AJV Validator CLI
        run: npm install -g ajv-cli

      - name: 🛡️ Assert Valid Ticket State PASSES
        run: |
          echo "Validating 'valid-ticket.json' against the AI Agent Triage schema..."
          ajv validate -s ai-agent-triage-schema.json -d valid-ticket.json

      - name: 🧪 Assert Invalid Ticket State FAILS (Negative Testing)
        run: |
          echo "Validating 'invalid-ticket.json' to ensure hallucination guardrails work..."
          
          # We expect 'ajv validate' to FAIL (exit code > 0) on the bad file.
          # If it passes (exit code 0), our schema isn't strict enough and our CI pipeline must fail.
          
          if ajv validate -s ai-agent-triage-schema.json -d invalid-ticket.json; then
            echo "❌ CRITICAL FAILURE: The invalid payload (hallucinated keys) mistakenly PASSED schema validation!"
            exit 1
          else
            echo "✅ SUCCESS: The validation layer blocked the malformed AI output as expected."
          fi
```
