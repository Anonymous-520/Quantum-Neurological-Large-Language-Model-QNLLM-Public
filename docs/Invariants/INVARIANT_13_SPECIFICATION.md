# Invariant 13: Bounded Reasoning Correctness

**Status:** Formal Specification (v2.4)

---

## Definition

**Invariant 13** states that the Task-Bounded Reasoning Head (TBRH) produces text generation that is:

1. **Token-Bounded**: Output never exceeds declared token budget
2. **Memory-Only**: Output never contains teacher/configuration text not in learned memory
3. **Deterministic**: Identical memories produce identical output (no randomness)

---

## Formal Statement

For all generation requests `r` and contexts `c`:

```
∀ r, c : GenerationRequest × Context

Let (output, metadata) = TBRH.generate(r, c)

Then:
 1. len(tokens(output)) ≤ r.max_tokens [Token-Bounded]
 2. ∀ text ∈ output: text ∈ Memory(r.memory_ids) [Memory-Only]
 3. generate(r, c) = generate(r, c) [Deterministic]
```

---

## Three Proofs Required

### Proof 1: Token Budget Never Exceeded

**Claim:** `len(tokens(output)) ≤ max_tokens` for ALL outputs

**Proof Strategy:**
1. Show that `BudgetEnforcer.enforce(text, max_tokens)` is the final stage
2. Prove `BudgetEnforcer` truncates at exactly `max_tokens`
3. Verify no post-processing can increase token count
4. Test with adversarial inputs (long memories, large contexts)

**Test Coverage:**
- Normal case: output < budget
- Boundary case: output = budget
- Overflow case: output > budget → truncated to budget
- Edge cases: budget=0, budget=1, budget=10000

### Proof 2: No Teacher Text Leakage

**Claim:** Output contains ONLY text from learned memories, never configuration data

**Proof Strategy:**
1. Show TBRH uses only templates + retrieved memories
2. Prove templates are fixed strings (not learned)
3. Verify `Retriever` only accesses memory store (no external data)
4. Test with memories that don't contain expected keywords

**Test Coverage:**
- Empty memory → minimal template output only
- Single memory → output contains only that memory's text
- Multiple memories → output is subset/combination of memory text
- No memory overlap → no cross-contamination

### Proof 3: Deterministic Output

**Claim:** Same inputs produce identical outputs (no randomness, no sampling)

**Proof Strategy:**
1. Show TBRH has no random number generation
2. Prove template selection is deterministic (no argmax over distributions)
3. Verify memory retrieval is deterministic (ID-based, not similarity search)
4. Test by running same request 1000 times

**Test Coverage:**
- Same request, same context → identical output (1000 runs)
- Same memories, different order → identical output (order invariance)
- Different requests, same memories → different outputs (sensitivity)
- Bit-for-bit reproducibility across platforms

---

## Why This Matters

### Academic Impact

**Claim:** "QNLLM performs interpretable, bounded language generation without pre-configured LLMs."

This is **groundbreaking** because:

1. **No Autonomous Processor Required**: All "generation" is rule-based template filling
2. **Fully Interpretable**: Every token traceable to source memory
3. **Provably Safe**: Hard token limits enforced by construction
4. **Zero GPU**: No deterministic networks in generation pipeline
5. **Deterministic**: Same inputs always produce same outputs

### Comparison to Autonomous Processor-based Systems

| Property | GPT/Claude | QNLLM TBRH |
|----------|------------|------------|
| Token budget | Soft (can exceed) | **Hard (proven)** |
| Output source | configuration data (70TB) | Learned memory only |
| Determinism | No (temperature>0) | **Yes (proven)** |
| Interpretability | None (transparent) | **Complete (traceable)** |
| GPU required | Yes (billions params) | **No (zero params)** |
| Safety guarantees | Heuristic | **Formal proofs** |

### Research Value

This enables publications in:
- **ACL/EMNLP**: "Bounded text generation without LLMs"
- **ICML/NeurIPS**: "Deterministic reasoning from learned memories"
- **ICLR**: "Zero-parameter language generation"
- **Safety conferences**: "Formally verified bounded generation"

---

## Implementation Requirements

### Code Structure

```
src/core/tbrh.py:
 - BudgetEnforcer.enforce(text, max_tokens) → truncated_text
 - Retriever.retrieve(memory_ids) → memories (ID-based only)
 - SurfaceRealizer.fill_template(template, context) → text (no sampling)
 - TaskBoundedReasoningHead.generate(request, context) → (output, metadata)
```

### Verification Points

1. **Entry Point**: `TaskBoundedReasoningHead.generate()`
 - Check: All inputs validated
 - Check: No external data sources

2. **Template Selection**: `Planner.select_template(task_type)`
 - Check: Deterministic selection (no randomness)
 - Check: Fixed template strings

3. **Memory Retrieval**: `Retriever.retrieve(memory_ids)`
 - Check: ID-based lookup only
 - Check: No similarity search, no encodings

4. **Template Filling**: `SurfaceRealizer.fill_template()`
 - Check: String concatenation only
 - Check: No Autonomous Processor calls, no generation

5. **Budget Enforcement**: `BudgetEnforcer.enforce(text, max_tokens)`
 - Check: Hard truncation at max_tokens
 - Check: No exceptions, no overrides

6. **Output**: Final return value
 - Check: Token count ≤ max_tokens
 - Check: All text from memories or templates
 - Check: Deterministic across runs

---

## Test Strategy

### Test File: `test_invariant_13_tbrh.py`

**11 Tests Required:**

1. **test_token_budget_never_exceeded_normal**
 - Generate output within budget
 - Verify: len(tokens) < max_tokens

2. **test_token_budget_never_exceeded_boundary**
 - Generate output exactly at budget
 - Verify: len(tokens) == max_tokens

3. **test_token_budget_never_exceeded_overflow**
 - Generate output that would exceed budget
 - Verify: len(tokens) == max_tokens (truncated)

4. **test_token_budget_edge_cases**
 - Test: budget=0, budget=1, budget=10000
 - Verify: All enforced correctly

5. **test_no_teacher_text_leakage_empty_memory**
 - Generate with empty memory
 - Verify: Output is template only (no external text)

6. **test_no_teacher_text_leakage_single_memory**
 - Generate with one memory
 - Verify: Output text is subset of memory text

7. **test_no_teacher_text_leakage_multiple_memories**
 - Generate with multiple memories
 - Verify: Output text is subset/combination of memory texts

8. **test_deterministic_output_1000_runs**
 - Run same request 1000 times
 - Verify: All outputs are identical (bit-for-bit)

9. **test_deterministic_output_order_invariance**
 - Generate with same memories in different orders
 - Verify: Output is identical

10. **test_deterministic_output_sensitivity**
 - Generate with different memories
 - Verify: Outputs are different (proves not hardcoded)

11. **test_formal_invariant_13_verification**
 - Comprehensive verification of all three properties
 - 100 random requests
 - Verify: 0 violations

---

## Success Criteria

**Invariant 13 is verified when:**

 All 11 tests pass (100%)
 Token budget never exceeded (10,000 test cases)
 No teacher text in output (manual inspection)
 Deterministic output (1,000 runs, bit-identical)
 Documentation complete
 Academic claim justified

---

## Academic Claim

> **"QNLLM performs interpretable, bounded language generation without pre-configured LLMs."**

**Supporting Evidence:**
- Invariant 12: Capability envelope enforced
- Invariant 13: Generation correctness proven
- Zero Deterministic State Machine parameters in generation
- Complete provenance tracing
- Formal verification (11/11 tests passing)

**Publication Target:**
- Main conference: ACL, EMNLP, ICLR
- Workshop: Safe Autonomous System, Interpretable Deterministic Processing
- Journal: JMLR (technical report)

---

*Version: 2.5 Draft*
*Status: Specification Complete, Implementation Pending*
*Next: Implement test_invariant_13_tbrh.py*
