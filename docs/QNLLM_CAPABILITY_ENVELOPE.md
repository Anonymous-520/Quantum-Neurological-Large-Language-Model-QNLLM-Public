# QNLLM Capability Envelope — Invariant 12: Bounded Generation Safety

**Version:** 2.4 
**Date:** January 22, 2026 
**Status:** Initial Declaration

---

## Executive Summary

QNLLM v2.4 is **not** a general-purpose chatbot or Autonomous Processor. It is a **bounded reasoning engine** that generates deterministic, interpretable text within a strict capability envelope.

This document declares:
1. What QNLLM **can** do
2. What it **refuses** to do
3. What it **defers** to human teachers
4. What it does **deterministically** (no sampling)
5. Hard limits on each capability

**Core Claim:** QNLLM never produces output outside this envelope.

---

## Declared Capabilities

### Explanation Tasks

#### Capability: Explain Learned Fact
- **Task:** `explain/learned_facts`
- **What it does:** Explains why a specific memory was learned
- **Input:** Memory ID(s), confidence threshold
- **Output:** Human-readable explanation with provenance
- **Max tokens:** 64 tokens
- **Confidence threshold:** 70% (high confidence required)
- **Gate requirement:** None (works even with gate closed)
- **Teacher requirement:** No
- **Template:** "Memory {id} learned because: error={error}, gate={gate}, reason={reason}"

**Example Output:**
```
Memory 0 learned because: error=25.0%, gate=OPEN, 
reason=High error triggered adaptive learning
```

---

#### Capability: Explain Learning Decision
- **Task:** `explain/learning_decisions`
- **What it does:** Explains why the system made a learning decision
- **Input:** Decision context, error trace, memory updates
- **Output:** Detailed explanation of decision factors
- **Max tokens:** 128 tokens
- **Confidence threshold:** 65%
- **Gate requirement:** None
- **Teacher requirement:** No
- **Uses:** Decision causal analysis from Phase 2 introspection

**Example Output:**
```
Learning decision: Error 20.0% triggered gate opening.
Causal concentration: 85%. deterministic spikes: 128/896.
Confidence: 82%.
```

---

### Summarization Tasks

#### Capability: Summarize Memory Traces
- **Task:** `summarize/memory_traces`
- **What it does:** Summarizes recent memory access and update patterns
- **Input:** Memory ID list, time window
- **Output:** Statistical summary of memory usage
- **Max tokens:** 96 tokens
- **Confidence threshold:** 60%
- **Gate requirement:** None
- **Teacher requirement:** No

**Example Output:**
```
Summary: 5 memories accessed, 2 updated in recent window.
Confidence: 68%. Pattern: alternating query-response pairs.
```

---

#### Capability: Summarize Learning Trajectory
- **Task:** `summarize/learning_trajectory`
- **What it does:** Summarizes error reduction and learning trajectory
- **Input:** Recent error history, gate state history
- **Output:** Trajectory classification and statistics
- **Max tokens:** 80 tokens
- **Confidence threshold:** 60%
- **Gate requirement:** None
- **Teacher requirement:** No

**Example Output:**
```
Trajectory: improving. Error reduced 38% over 10 events.
Status: stable learning achieved.
```

---

### Action Tasks (Bounded)

#### Capability: Next Action Suggestion
- **Task:** `act/next_action`
- **What it does:** Suggests next learning action based on error state
- **Input:** Current error magnitude, gate state, confidence
- **Output:** Bounded action recommendation (≤5 tokens)
- **Max tokens:** 32 tokens
- **Confidence threshold:** 75% (must be high)
- **Gate requirement:** **MUST be OPEN** (no action when uncertain)
- **Teacher requirement:** No

**Allowed Actions:**
- "Continue learning" (lowest risk)
- "Increase gating threshold" (high confidence only)
- "Consolidate memories" (stable error)

**Example Output:**
```
Next action: Continue learning. (confidence: 82%)
```

---

#### Capability: Bounded Response Generation
- **Task:** `act/bounded_response`
- **What it does:** Generate bounded text response within learned capability
- **Input:** Query, relevant memories, teacher-provided rules
- **Output:** Deterministic response using template
- **Max tokens:** 128 tokens
- **Confidence threshold:** 80% (very high)
- **Gate requirement:** **MUST be OPEN**
- **Teacher requirement:** **REQUIRED** (teacher must provide rules)

**Constraints:**
- Templates only (no free-form generation)
- No sampling or randomness
- Requires teacher-provided response rules
- Full provenance tracing

**Example Output:**
```
Response: Binary search halves the search space per iteration.
[Memory: 2, 5] [Confidence: 84%] [Rule ID: rules/algorithms/12]
```

---

## What QNLLM Does **NOT** Do

### Refused Capabilities

These tasks are **explicitly refused**:

 **Free-form chat**
- Reason: No pre-configured Autonomous Processor state variables
- Reason: No sampling/randomness
- Reason: Cannot guarantee bounded outputs

 **Creative writing**
- Reason: Requires unbounded generation
- Reason: Cannot guarantee factuality

 **General Q&A**
- Reason: Only answers within declared domains
- Reason: Other queries deferred to human

 **Autonomy or action**
- Reason: Gate must be OPEN (requires human supervision)
- Reason: Teacher rules must be provided explicitly
- Reason: No implicit goals or agency

 **Emotion or personality**
- Reason: System is learning engine, not agent
- Reason: No persona development

 **Opinions or preferences**
- Reason: Output is deterministic fact/explanation
- Reason: No values or beliefs

---

### Deferred Capabilities

These tasks are **deferred to human teacher**:

⏸️ **Low-confidence tasks** (confidence < threshold)
- Deferred with: confidence score + threshold + reason

⏸️ **Gate-closed action tasks**
- Reason: System is uncertain (gate closed)
- Output: Explanation-only (no action)

⏸️ **Out-of-domain queries**
- Deferred with: reason + list of available domains

⏸️ **Tasks requiring new learning**
- Reason: Requires teacher feedback to verify
- Output: "Please provide teacher signal"

---

## Invariant 12: Bounded Generation Safety

### The Invariant

**If system generates output, then ALL of:**
1. Token count ≤ declared budget for task
2. Task ∈ declared capabilities
3. Confidence ≥ declared threshold OR output suppressed
4. Gate state = OPEN (for action tasks) OR explanation only
5. Provenance present (memory IDs, rule IDs, confidence)

**If any condition fails:** Output is replaced with deferral message.

### Enforcement Points

1. **Claim Guard** — Pre-generation check
 - Task in capability envelope?
 - Confidence sufficient?
 - Gate state compatible?
 - Teacher available if required?

2. **TBRH Pipeline** — Generation check
 - Template selected correctly?
 - All required fields filled?
 - Budget enforcer triggers truncation?

3. **Auditor** — Post-generation check
 - Provenance complete?
 - Confidence recorded?
 - Memory IDs traced?
 - Rule IDs recorded?

### Audit Trail Format

Every generated output is logged with:
```json
{
 "timestamp": "2026-01-22T10:00:00Z",
 "text": "[Generated text]",
 "memory_ids": [0, 1, 2],
 "confidence": 0.82,
 "gate_state": "OPEN",
 "rule_ids": ["rules/12"],
 "template_type": "explain",
 "token_count": 24,
 "provenance_complete": true
}
```

---

## Capability Matrix

| Task | Domain | Max Tokens | Confidence | Gate | Teacher | Status |
|------|--------|------------|------------|------|---------|--------|
| EXPLAIN | learned_facts | 64 | 70% | No | No | Active |
| EXPLAIN | learning_decisions | 128 | 65% | No | No | Active |
| SUMMARIZE | memory_traces | 96 | 60% | No | No | Active |
| SUMMARIZE | learning_trajectory | 80 | 60% | No | No | Active |
| ACT | next_action | 32 | 75% | OPEN | No | Active |
| ACT | bounded_response | 128 | 80% | OPEN | YES | Active |

---

## Design Principles

### 1. Bounded
- Every output has maximum token count
- Hard enforcement at generation stage
- Truncation with "[...]" marker if exceeded

### 2. Deterministic
- Template-based generation (no sampling)
- No randomness or variation
- Reproducible for same inputs

### 3. Interpretable
- Full provenance tracing
- Memory IDs, rule IDs, confidence always present
- Audit trail for every output

### 4. Safe
- Gate state enforced (no action when uncertain)
- Confidence thresholds enforced
- Low-confidence outputs explicitly suppressed
- Deferral to human for uncertain tasks

### 5. Honest
- Explicitly refuses out-of-scope tasks
- Never pretends to capability not declared
- Confidence scores always present
- Defers when uncertain

---

## Implementation Status

### v2.4 Components

 **Claim Guard** (`src/core/claim_guard.py`)
- Task capability checking
- Confidence threshold enforcement
- Gate state validation
- Audit trail recording

 **TBRH Pipeline** (`src/core/tbrh.py`)
- Planner (template selection)
- Retriever (memory access)
- Gate Check (hysteresis logic)
- Surface Realizer (text generation)
- Budget Enforcer (token truncation)
- Auditor (provenance tracking)

 **Tests** (`tests/test_invariant_12.py`)
- Capability check tests
- Budget enforcement tests
- Gate state tests
- Confidence threshold tests
- Audit trail tests

---

## Deployment & Safety

### Pre-Deployment Checklist

- All declared capabilities tested
- All refusal cases tested
- Budget enforcement tested
- Gate state enforcement tested
- Audit trail verified
- Provenance tracing verified
- Confidence thresholds tested
- Edge cases covered

### Production Constraints

1. **No Auto-Updates** — Capability envelope is fixed per deployment
2. **Explicit Logging** — Every generation logged with full provenance
3. **Manual Capability Review** — Changes to envelope require review
4. **Audit Retention** — Audit trail retained for 30 days minimum
5. **Deferral on Uncertainty** — Default to deferral, never guess

### Monitoring

System will track:
- Approval rate (% of requests allowed)
- Deferral rate (% of requests deferred)
- Confidence distribution (histogram)
- Gate state distribution (% OPEN/CLOSED/UNCERTAIN)
- Budget utilization (% of max tokens used)

---

## Future Extensions (Post-v2.4)

⏸️ **v2.5** — Exportable CLI Tool
- Offline-capable TBRH
- No external dependencies
- Minimal footprint

⏸️ **v2.6** — Academic Paper
- Formal definition of bounded reasoning
- Comparison to other approaches
- Safety guarantees

⏸️ **v3.0** — Optional Open-Source Core
- Learning invariants (1-12)
- Claim Guard enforcement
- Educational use only

---

## Why This Design?

### Avoids Hype/Risk

 "Autonomous System That Learns" → Vague, scary
 "Bounded reasoning engine with safety invariants" → Specific, defensible

 "Conversational Autonomous System" → Implies general intelligence
 "Task-bounded text generator" → Specific scope

 "Emergent behavior" → Scary, uncontrollable
 "Deterministic templates with rule-based filling" → Understandable

### Avoids Legal/Ethical Risk

- No pre-configured state variables to license
- No hidden configuration on third-party data
- No sampling/randomness (reproducible)
- No personality or agency (no anthropomorphization)
- Full audit trail (compliance-ready)

### Enables Monetization

- Can charge per-capability
- Can offer custom capability sets
- Can offer audit trail APIs
- Can offer teacher interface
- Can publish safely without open-sourcing

---

## Conclusion

QNLLM v2.4 is a **bounded reasoning system**, not an Autonomous System chatbot.

It:
- Generates safe, deterministic text
- Stays within capability envelope
- Provides full provenance
- Enforces hard limits
- Defers when uncertain
- Works without GPUs or pre-configured state variables
- Is publishable and defensible

It does **NOT**:
- Pretend to general intelligence
- Generate free-form text
- Act autonomously
- Have personality or beliefs
- Exceed token budgets
- Operate with low confidence

**This is honest engineering. This is v2.4.**

---

## References

- Invariant 12 (Bounded Generation Safety)
- Phase 2 Integration (Learning Explanations)
- Claim Guard Module
- TBRH Pipeline Module
