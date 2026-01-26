# TBRH (Task-Bounded Reasoning Head) Specification v2.3

## Executive Summary

**TBRH** is a deterministic, auditable language generation system that enables QNLLM to express internal state without external Autonomous Processor state variables. All generation is:

- **Bounded**: Hard cap of 64 tokens per response
- **Rule-based**: Template-driven, no deterministic sampling
- **Auditable**: Complete trace of every decision
- **Cited**: Every claim links to memory ID
- **Gate-aware**: Respects learning gate state

## Architecture

```
User Input (Task)
 ↓
[Planner] → Plan with sections and templates
 ↓
[Budgeter] → Token budget allocation (respects gate)
 ↓
[Realizer] → Template-based text generation with slots
 ↓
[Auditor] → 6-check invariant verification
 ↓
Output (Text + Citations + Audit Log)
```

## Task Types (Whitelist Only)

TBRH supports **exactly 3 task types**. No free-form generation.

### 1. EXPLAIN
**Purpose**: Describe why system took an action.

**Parameters**:
- `action` (required): What the system did (e.g., "reopened learning")
- `reason` (required): Why it did it (e.g., "hysteresis threshold crossed")

**Output Template**:
```
System {action}.
Reason: {reason}
Continuing with updated state.
```

**Token Budget**: 45 tokens max

**Example**:
```python
from src.systems.tbrh import TBRH

tbrh = TBRH(max_tokens=64)
result = tbrh.generate(
 task="explain",
 gate_state="open",
 memory_ids=[4521, 4522],
 confidence=0.92,
 task_params={
 "action": "reopened learning",
 "reason": "3 consecutive high-disagreement cycles"
 }
)
print(result.text)
# Output: "System reopened learning. Reason: 3 consecutive high-disagreement cycles. Continuing with updated state."
```

### 2. RECALL
**Purpose**: Report previously learned facts.

**Parameters**:
- `facts` (required): What was learned (e.g., "User prefers brief responses")
- `quality_score` (required): Confidence (0-100)

**Output Template**:
```
Recalling: {facts}
Quality: {quality_score}/100
```

**Token Budget**: 30 tokens max

**Example**:
```python
result = tbrh.generate(
 task="recall",
 gate_state="open",
 memory_ids=[1234, 1235],
 confidence=0.87,
 task_params={
 "facts": "User prefers concise technical explanations",
 "quality_score": "87"
 }
)
# Output: "Recalling: User prefers concise technical explanations\nQuality: 87/100"
```

### 3. SUMMARIZE
**Purpose**: Compress recent context.

**Parameters**:
- `summary` (required): Compressed summary (e.g., "3 conversation turns, all queries answered")
- `omitted` (required): What was left out (e.g., "Intermediate reasoning steps")

**Output Template**:
```
{summary}
Omitted: {omitted}
```

**Token Budget**: 45 tokens max

**Example**:
```python
result = tbrh.generate(
 task="summarize",
 gate_state="open",
 memory_ids=[5001, 5002, 5003],
 confidence=0.75,
 task_params={
 "summary": "3 conversation turns, all technical questions answered successfully",
 "omitted": "Intermediate reasoning steps and backtracking"
 }
)
```

## Gate States

### OPEN
- **Meaning**: Learning is active, system can generate freely
- **Budget**: Full allocation (up to max)
- **Use case**: Normal operation, system reasoning visible

### CLOSED
- **Meaning**: Learning is disabled, system generation prohibited
- **Budget**: 0 tokens (no output)
- **Use case**: System freeze, diagnostic mode

### UNCERTAIN
- **Meaning**: Gate state unclear or transitioning
- **Budget**: 50% of allocation
- **Use case**: Conservative mode during state transitions

## Invariants (Always Enforced)

### 1. Token Budget (Hard Cap)
- **Maximum**: 64 tokens per response
- **Enforcement**: Budgeter rejects generation if exceeded
- **Never violated**: Hard boundary in code

### 2. Gate Respect
- If `gate_state == "closed"`: Output is empty (return None)
- If `gate_state == "uncertain"`: Budget reduced to 50%
- If `gate_state == "open"`: Full budget available
- **Audit check**: `gate_state_consistent`

### 3. Provenance (All Claims Cited)
- Every factual claim must cite ≥1 memory ID
- Citations include: memory_id, confidence, timestamp
- No claim without source
- **Audit check**: `claims_cited`

### 4. Confidence Threshold
- Minimum confidence: 0.3 (default)
- Below threshold: Generation rejected or minimal
- **Audit check**: `confidence_threshold`

### 5. No External Autonomous Processor
- TBRH generates without calling external models
- No openai, hugging face, or third-party Autonomous Processor calls
- Template-based slot-filling only
- **Audit check**: `no_external_llm` (always passes)

### 6. Memory ID Validity
- All cited memory IDs must be non-negative integers
- All cited IDs must exist in system memory
- **Audit check**: `memory_ids_valid`

## Audit Verification

Every TBRH generation is audited with 6 checks. Output includes:

```python
result.audit_status # "pass", "warn", or "fail"
result.audit_log # List of check results
```

**Audit Checks**:
1. `tokens_within_budget` - Output ≤ max tokens
2. `gate_state_consistent` - Gate respected
3. `memory_ids_valid` - All IDs are valid
4. `confidence_threshold` - Confidence ≥ 0.3
5. `no_external_llm` - No Autonomous Processor calls made
6. `claims_cited` - All claims have citations

**Example Audit Result**:
```json
{
 "audit_status": "pass",
 "tokens_used": 18,
 "checks": [
 {"name": "tokens_within_budget", "status": "pass"},
 {"name": "gate_state_consistent", "status": "pass"},
 {"name": "memory_ids_valid", "status": "pass"},
 {"name": "confidence_threshold", "status": "pass"},
 {"name": "no_external_llm", "status": "pass"},
 {"name": "claims_cited", "status": "pass"}
 ]
}
```

## API Reference

### TBRH.generate()

```python
result = tbrh.generate(
 task: str, # "explain", "recall", or "summarize"
 gate_state: str, # "open", "closed", or "uncertain"
 gate_reason: str = None, # Why gate is in this state
 hysteresis_stable: bool = True, # Gate stability indicator
 memory_ids: list[int], # Source memory IDs [1001, 2002, ...]
 confidence: float, # 0.0-1.0 confidence score
 task_params: dict = None, # Task-specific parameters
)

# Returns: TBRHResult or None
# TBRHResult contains:
# .text: str # Generated response
# .tokens_used: int # Token count
# .citations: list[dict] # [{"memory_id": 1001, "confidence": 0.9}, ...]
# .audit_status: str # "pass", "warn", or "fail"
# .full_audit_log: dict # Complete audit trace
```

## Implementation Examples

### Example 1: Explain System Adaptation
```python
from src.systems.tbrh import TBRH

tbrh = TBRH(max_tokens=64)

result = tbrh.generate(
 task="explain",
 gate_state="open",
 hysteresis_stable=True,
 memory_ids=[4521, 4522, 4523],
 confidence=0.92,
 task_params={
 "action": "adapted gating threshold",
 "reason": "convergence pattern shifted"
 }
)

if result and result.audit_status == "pass":
 print(result.text)
 print(f"Cited {len(result.citations)} memories")
 print(f"Used {result.tokens_used}/64 tokens")
```

### Example 2: Recall With Quality Score
```python
result = tbrh.generate(
 task="recall",
 gate_state="open",
 memory_ids=[1234, 1235, 1236],
 confidence=0.87,
 task_params={
 "facts": "User prefers step-by-step explanations",
 "quality_score": "87"
 }
)

if result:
 print(result.text)
 # Recalling: User prefers step-by-step explanations
 # Quality: 87/100
```

### Example 3: Gate Closed (No Output)
```python
result = tbrh.generate(
 task="explain",
 gate_state="closed",
 memory_ids=[1001],
 confidence=0.5,
 task_params={"action": "paused", "reason": "learning disabled"}
)

if result is None:
 print("Gate is closed - no output generated")
```

## Integration Points

### 1. In Reasoning Engine (src/core/cortex/reasoning.py)

```python
from src.systems.tbrh import TBRH

# In ReasoningEngine class:
def generate_bounded(self, task, gate_state, memory_ids, confidence, task_params):
 """Generate bounded, auditable response using TBRH."""
 tbrh = TBRH(max_tokens=64)
 result = tbrh.generate(
 task=task,
 gate_state=gate_state,
 memory_ids=memory_ids,
 confidence=confidence,
 task_params=task_params,
 )
 return result.text if result else None
```

### 2. In Chat System (Mainsys/unified_chat.py)

```python
# Add TBRH option for system explanations
if should_use_bounded_generation:
 text, audit = engine.generate_bounded(
 task="explain",
 gate_state=current_gate_state,
 memory_ids=relevant_memories,
 confidence=system_confidence
 )
```

### 3. As Standalone Module

```python
from src.systems.tbrh import TBRH

# Direct use
tbrh = TBRH()
result = tbrh.generate(
 task="summarize",
 gate_state="open",
 memory_ids=[...],
 confidence=0.8,
 task_params={}
)
```

## Testing

TBRH includes comprehensive test suite (`tests/test_tbrh.py`) with 4 critical tests:

1. **test_tbrh_gate_closed()**: Gate closed → no output
2. **test_tbrh_budget_respected()**: Output ≤ 64 tokens
3. **test_tbrh_provenance()**: Every claim cites memory ID
4. **test_tbrh_confidence_threshold()**: Confidence gates output

Run tests:
```bash
cd /path/to/QNLLM
python -m tests.test_tbrh
```

Expected output:
```
================================================================================
TBRH TEST SUITE - 4 Critical Invariant Tests
================================================================================

 TEST 1 PASSED: Gate closed → no output
 TEST 2 PASSED: Budget respected (18 ≤ 64 tokens)
 TEST 3 PASSED: Provenance verified (3 citations with valid memory IDs)
 TEST 4 PASSED: Confidence threshold respected

================================================================================
RESULTS: 4 passed, 0 failed out of 4 tests
================================================================================
```

## Design Philosophy

### Why TBRH?

1. **Honest**: System expresses internal state without pretending to reason
2. **Bounded**: Hard limits prevent runaway generation
3. **Auditable**: Every output can be traced to source and checked
4. **No new dependencies**: Uses templates, not external models
5. **Interpretable**: Users understand exactly how output was generated

### Why Templates?

- No deterministic sampling → deterministic output
- No token accounting → no learning from generation
- Consistent across runs → repeatable behavior
- Easy to audit → clear decision points

### Why Gate-Aware?

- Respects learning_gate setting
- Prevents generation during system freeze
- Enables conservative mode (UNCERTAIN)
- Integrates with existing control systems

## Version History

- **v2.3** (Current): TBRH implementation
 - 5 components: Planner, Budgeter, Realizer, Auditor, Orchestrator
 - 1,140 lines of production code
 - 4 test suite with 100% pass rate
 - Integrated into reasoning engine
 - Hard cap: 64 tokens
 - Task whitelist: EXPLAIN, RECALL, SUMMARIZE
 - 6-check audit system

- **v2.2**: Pre-TBRH (NGC API integration)
- **v2.1**: Initial QNLLM with MTL teachers
- **v2.0**: Neuron system baseline

## Future Extensions

Possible extensions (v2.4+) without breaking invariants:

1. Additional task types: QUESTION, REFLECTION (new whitelist entries)
2. Custom templates: User-defined patterns (still audited)
3. Extended budget: Task-specific allocations (never exceeds 64)
4. Confidence modulation: Adjust template verbosity
5. Multi-language support: Translated templates

## Compliance

TBRH is designed to support publication and defensible claims:

- No unauthorized external API calls
- No pre-configured Autonomous Processor state variables usage
- Complete audit trail for reproducibility
- Deterministic generation (same input → same output)
- All claims cite sources
- Test coverage for all invariants

## Questions?

For implementation questions, see:
- Architecture details: [TBRH_ARCHITECTURE.md](TBRH_ARCHITECTURE.md)
- Integration guide: [Integration in reasoning.py](src/core/cortex/reasoning.py#L30)
- Test examples: [tests/test_tbrh.py](tests/test_tbrh.py)
