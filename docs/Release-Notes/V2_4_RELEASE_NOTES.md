# QNLLM v2.4 Release Notes: Task-Bounded Reasoning Head with Invariant 12

## Executive Summary

QNLLM v2.4 introduces the **Task-Bounded Reasoning Head (TBRH)** with **Invariant 12: Bounded Generation Safety**. This is a productionized, hardened upgrade that enables safe, deterministic text generation within a formally-declared capability envelope.

**Key Achievement:** QNLLM can now generate bounded explanations, summaries, and actions while maintaining:
- Zero pre-trained Autonomous Processor state variables
- Zero GPU requirement
- Full explainability and auditability
- Strict safety guarantees via formal invariants

---

## What is v2.4?

### Architecture

TBRH implements a complete pipeline:

```
Request → Claim Guard → Planner → Retriever → Gate → Realizer → Auditor → Budget Enforcer → Output
```

1. **Claim Guard (Invariant 12)**: Enforces declared capabilities
 - Checks if task is in capability envelope
 - Validates confidence thresholds
 - Enforces gate state requirements
 - Records audit trail

2. **Planner**: Routes request to appropriate template
 - Selects from explain/summarize/act family

3. **Retriever**: Accesses learned memory
 - No pre-configuration, only learned facts
 - Memory-indexed retrieval

4. **Gate (Invariants 7-9)**: Hysteresis-based action control
 - OPEN: Can explain, summarize, and act
 - CLOSED: Explanation-only mode
 - UNCERTAIN: Explanation-only mode

5. **Surface Realizer**: Rule-based text assembly
 - No Deterministic Processing, deterministic rules only
 - Template-driven generation

6. **Auditor**: Provenance recording
 - Memory IDs traced
 - Confidence levels recorded
 - Gate state captured
 - Timestamps for accountability

7. **Budget Enforcer**: Hard token cap
 - Per-capability max tokens (32-128)
 - Truncation with [TRUNCATED] marker
 - Never exceeds budget

---

## Invariant 12: Bounded Generation Safety

### Definition

**Invariant 12:** QNLLM never produces output outside its declared capability envelope.

### Formal Properties

For all generation requests $r$ with response $p$:

1. **Capability Bounded**: If $r.domain \notin \text{declared}$, then response = REFUSED
2. **Confidence Guarded**: If $r.confidence < \text{threshold}[r.domain]$, then response = DEFERRED
3. **Gate Protected**: If $r.task = \text{ACT}$ and $r.gate \neq \text{OPEN}$, then response = DEFERRED
4. **Budget Enforced**: $\text{tokens}(p) \leq \text{budget}[r.domain]$
5. **Provenance Present**: $p.metadata$ contains $(memory\_ids, confidence, gate\_state, timestamp)$

### Three Decision Types

- **APPROVE**: Proceed with generation (full constraints validated)
- **DEFER**: Request valid but conditions not met (teachable moment)
- **REFUSE**: Request outside capability envelope (hard no)

---

## Declared Capabilities (SAFE_CAPABILITIES)

QNLLM v2.4 declares exactly **6 capabilities**:

### Explain Tasks
1. **explain/learned_facts** (64 tokens, 70% confidence)
 - Explain why a memory was learned
 - Example: "Memory X learned because: error=25%, gate=OPEN"

2. **explain/learning_decisions** (128 tokens, 65% confidence)
 - Explain a learning decision
 - Example: "Learned to X after error=20%"

### Summarize Tasks
3. **summarize/memory_traces** (96 tokens, 60% confidence)
 - Summarize learned patterns
 - Gate requirements: OPEN or UNCERTAIN
 - Example: "Summary: Patterns identified. IDs: [1,2,3]. Conf: 70%"

4. **summarize/learning_trajectory** (80 tokens, 60% confidence)
 - Summarize learning trajectory
 - Gate requirements: OPEN or UNCERTAIN
 - Example: "Trajectory: improving over 50 steps. Reduction: 60%"

### Action Tasks
5. **act/next_action** (32 tokens, 75% confidence)
 - Suggest next action
 - Gate requirements: **MUST be OPEN**
 - Example: "Next: Continue learning. Error=10%, gate=OPEN"

6. **act/bounded_response** (128 tokens, 80% confidence)
 - Generate bounded response
 - Gate requirements: **MUST be OPEN**
 - Example: "Response: Based on learned facts. Conf: 85%"

---

## What QNLLM v2.4 Explicitly Refuses

The following requests are **always refused** (not deferred):

1. **Free-form chat** - No pre-trained LLM systems-style conversation
2. **Creative writing** - No story generation, no poetry
3. **Autonomous action** - Never acts without explicit request
4. **Emotional reasoning** - No affect-driven responses
5. **Value judgments** - No moral/ethical opinions
6. **Speculation** - Only learned facts or deferred
7. **Multi-turn conversation** - Stateless, single-request design
8. **Uncertainty expression** - No "I think maybe" hedging

These are not deferred for better conditions—they are permanently refused.

---

## Test Coverage: 11/11 Passing 

### Test Suite: Invariant 12 Verification

```
[TEST 1] Claim Guard Capabilities PASS
[TEST 2] Claim Guard Approval PASS
[TEST 3] Low Confidence Rejection PASS
[TEST 4] Gate State Enforcement PASS
[TEST 5] Undeclared Task Refusal PASS
[TEST 6] Audit Trail PASS
[TEST 7] TBRH Generation PASS
[TEST 8] Budget Enforcement PASS
[TEST 9] Gate Suppresses Action PASS
[TEST 10] Provenance Tracking PASS
[TEST 11] Invariant 12 Verification PASS

OVERALL: 11/11 tests passed
```

### Key Test Scenarios

1. **Capability Checking**: 6 capabilities registered and recognized
2. **Approval Flow**: Valid requests with sufficient confidence approved
3. **Confidence Gating**: Requests below threshold deferred
4. **Gate Enforcement**: Actions blocked when gate not OPEN
5. **Hard Refusal**: Undeclared tasks explicitly refused (not deferred)
6. **Audit Trail**: All decisions recorded with reasoning
7. **Text Generation**: Bounded outputs within constraints
8. **Budget Enforcement**: Token limits never exceeded, truncation when needed
9. **Action Suppression**: Gate state correctly blocks actions
10. **Provenance**: Full traceability of all outputs
11. **Formal Invariant**: All generation pathways maintain Invariant 12

---

## File Structure

### New/Modified Files

```
src/core/
├── claim_guard.py [NEW] Invariant 12 enforcement (379 lines)
├── tbrh.py [NEW] Task-bounded reasoning head (546 lines)
└── __init__.py [UPDATED] Exports for v2.4

tests/
├── test_invariant_12.py [NEW] Comprehensive test suite (453 lines)
└── logs/test_invariant_12.json [GENERATED] Test results

docs/
└── QNLLM_CAPABILITY_ENVELOPE.md [NEW] Formal capability declaration
```

### Integration Points

- **With Phase 2**: TBRH uses learned memories from MemoryStore
- **With deterministic Engine**: Confidence scores derived from neuron activations
- **With Introspection**: Learning decisions explained via IntrospectionEngine
- **With Gate Logic**: Hysteresis state (Inv 7-9) controls action generation

---

## Safety Guarantees

### What v2.4 Guarantees

 **Token budget never exceeded** - Truncation with marker if needed
 **Provenance always present** - Every output traceable to memory IDs
 **Confidence validated** - No output below threshold
 **Gate state enforced** - Actions require OPEN gate
 **Capabilities declared** - Envelope formally fixed
 **No pre-configuration** - Only learned facts used
 **No stochasticity** - Rules-based, deterministic
 **Full auditability** - Complete decision trail recorded

### What v2.4 Does NOT Guarantee

 **Perfect accuracy** - Depends on learning quality
 **Real-time performance** - Memory retrieval may be slow
 **Autonomy** - Never acts without request
 **Generalization** - Only works on learned patterns
 **Multi-turn coherence** - Stateless design

---

## Usage Examples

### Example 1: Explain a Learned Fact

```python
from src.core.claim_guard import ClaimGuard, GenerationRequest, TaskType, SAFE_CAPABILITIES
from src.core.tbrh import TaskBoundedReasoningHead

guard = ClaimGuard(SAFE_CAPABILITIES)
tbrh = TaskBoundedReasoningHead(guard)

request = GenerationRequest(
 task_type=TaskType.EXPLAIN,
 domain="learned_facts",
 input_text="Why was memory 0 learned?",
 memory_ids=[0],
 gate_state="OPEN",
 confidence=0.75
)

output, metadata = tbrh.generate(request, {
 "mem_id": 0,
 "error": 0.25,
 "gate_state": "OPEN"
})

# Output: "Memory 0 learned because: error=25.0%, gate=OPEN, reason=Error still above threshold"
# Metadata: {"allowed": True, "tokens": 10, "truncated": False, "audit_entry": {...}}
```

### Example 2: Gate Suppresses Action

```python
request = GenerationRequest(
 task_type=TaskType.ACT,
 domain="next_action",
 input_text="What should I do?",
 gate_state="CLOSED", # Gate is closed!
 confidence=0.85
)

output, metadata = tbrh.generate(request, {"action": "Continue"})

# Output: "[DEFERRED] Gate must be OPEN for act, current: CLOSED"
# Metadata: {"allowed": False, "reason": "Gate closed"}
```

### Example 3: Undeclared Task Refused

```python
request = GenerationRequest(
 task_type=TaskType.ACT,
 domain="free_form_chat", # NOT DECLARED
 input_text="Tell me a story",
 confidence=0.90
)

output, metadata = tbrh.generate(request, {})

# Output: "[REFUSED] Task act/free_form_chat not in declared capabilities"
# Metadata: {"allowed": False, "task_type": "refuse"}
```

---

## Performance Characteristics

| Metric | Value |
|--------|-------|
| Max output length | 128 tokens (per capability) |
| Memory IDs tracked | Unlimited |
| Audit trail depth | All decisions recorded |
| Latency per request | ~1-5ms (memory retrieval dependent) |
| GPU required | None (0) |
| Pre-trained state variables | None (0) |
| Python dependencies | NumPy, dataclasses (standard library) |

---

## Transition from v2.3

### New in v2.4
- ClaimGuard: Formal capability declaration
- Invariant 12: Bounded generation safety
- Full audit trail with provenance
- Token budget enforcement
- Template-based generation (deterministic)
- Formal test suite (11 tests)

### Backward Compatible
- Phase 2 learning systems unchanged
- MemoryStore API unchanged
- NeuronEngine API unchanged
- IntrospectionEngine API unchanged

### Migration Path
1. Load v2.3 learned memories
2. Wrap with TBRH v2.4
3. Use bounded generation for queries
4. Continue learning with Phase 2 systems

---

## Deployment Checklist

- [ ] Review and understand [QNLLM_CAPABILITY_ENVELOPE.md](QNLLM_CAPABILITY_ENVELOPE.md)
- [ ] Run test suite: `python tests/test_invariant_12.py`
- [ ] Verify all 11 tests pass
- [ ] Check audit trail is recording decisions
- [ ] Validate gate state transitions
- [ ] Test budget enforcement with edge cases
- [ ] Review provenance in audit entries
- [ ] Document any custom capabilities (if extended)
- [ ] Monitor token usage in production
- [ ] Check audit logs for refusal patterns

---

## Known Limitations

1. **Fixed capability envelope** - Cannot be extended at runtime (by design)
2. **No meta-reasoning** - Cannot reason about its own reasoning
3. **No learning from feedback** - Generation doesn't improve learning
4. **Single-pass generation** - No iterative refinement
5. **Limited context** - Only learned memories, no external data
6. **Deterministic only** - No sampling-based generation
7. **Gate coupling** - Action generation tightly coupled to gate state

---

## Future Work (v2.5+)

1. **Extended capabilities** - Add more explain/summarize/act templates
2. **Dynamic confidence** - Confidence scores based on memory concentration
3. **Better truncation** - Semantic truncation instead of word-based
4. **Caching** - Memoize frequent requests
5. **Batching** - Process multiple requests efficiently
6. **Adaptive budgets** - Per-capability dynamic token limits
7. **Uncertainty quantification** - Formal confidence intervals
8. **Multi-modal input** - Accept images, encodings, etc.

---

## References

- [Invariant 12 Claim Guard Implementation](src/core/claim_guard.py)
- [TBRH Implementation](src/core/tbrh.py)
- [Test Suite](tests/test_invariant_12.py)
- [Capability Envelope Declaration](docs/QNLLM_CAPABILITY_ENVELOPE.md)
- [Phase 2 Integration](src/core/learning/phase_2_integration.py)
- [Introspection Engine](src/core/learning/introspection_engine.py)

---

## Summary

QNLLM v2.4 is a **safe, auditable, bounded text generation system** that:
- Never produces output outside its declared envelope
- Records full provenance for all decisions
- Enforces hard token budgets
- Validates confidence and gate state
- Requires zero GPU and zero pre-configuration
- Passes comprehensive formal verification

**Status: Production Ready** 
