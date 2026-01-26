# QNLLM v2.3-dev - Research Extensions Implementation

## Overview

This document tracks the promotion of experimental features from `scripts/research_extensions.py` to production-ready modules for v2.3.

**Status:** In Development 
**Target Release:** Q2 2026 
**Branch:** `v2.3-dev`

---

## Three Core Extensions

### 1. task routings Implemented (Experimental)

**Current State (v2.2):**
- Implemented in `research_extensions.py`
- Selective focus on important examples
- Attention state variables based on uncertainty
- Max attention: 0.318 on highest-uncertainty examples

**Production Requirements for v2.3:**
- [ ] Move to `src/attention/qnllm_attention.py`
- [ ] Add configurable attention strategies (dot-product, scaled, multi-head)
- [ ] Implement attention masking (prevent looking at future examples)
- [ ] Add attention visualization tools
- [ ] Benchmark attention overhead (<5% latency increase)
- [ ] Write unit tests (90% coverage)
- [ ] Validate on all 4 variants
- [ ] Document API in `docs/attention.md`

**API Design (v2.3):**
```python
from src.attention import QNLLMWithAttention

# Create with attention
qnllm = QNLLMWithAttention(
 variant="codelearn",
 attention_type="scaled", # dot-product, scaled, multi-head
 num_heads=4, # for multi-head
 attention_dropout=0.1
)

# Train with attention
state variables = qnllm.train_with_attention(input_data, label)

# Query attention
prediction, attn_weights = qnllm.query_with_attention(input_data)

# Visualize attention
qnllm.plot_attention(save_path="attention_heatmap.png")
```

**Validation Criteria:**
- Attention improves accuracy by 5-10%
- Latency increase <5%
- Attention state variables sum to 1.0
- All 4 variants support attention

---

### 2. Hierarchical Consolidation ⏳ Partially Implemented

**Current State (v2.2):**
- Implemented in `research_extensions.py`
- Three tiers: Facts (raw examples) → Concepts (patterns) → Rules (abstractions)
- Current results: 4 facts stored, 0 concepts formed, 0 rules extracted (depth 1/3)
- Concept formation logic incomplete

**Production Requirements for v2.3:**
- [ ] Fix concept formation (currently 0 concepts from 4 facts)
- [ ] Implement rule extraction (abstraction layer)
- [ ] Add hierarchical query (check rules → concepts → facts)
- [ ] Support incremental consolidation (online learning)
- [ ] Benchmark memory savings (expect 30-50% reduction)
- [ ] Validate on real-world hierarchical data (taxonomies, ontologies)
- [ ] Write tests for each tier
- [ ] Document consolidation triggers

**API Design (v2.3):**
```python
from src.hierarchy import QNLLMWithHierarchy

# Create with hierarchy
qnllm = QNLLMWithHierarchy(
 variant="multimodal",
 hierarchy_depth=3, # facts → concepts → rules
 consolidation_threshold=10, # consolidate after 10 facts
 abstraction_method="clustering" # clustering, rule-mining, encoding
)

# Train (facts stored in tier 1)
qnllm.train(input_data, label)

# Trigger consolidation (facts → concepts)
concepts_formed = qnllm.consolidate_hierarchical()
print(f"Formed {concepts_formed} concepts")

# Query (searches rules → concepts → facts)
prediction, tier = qnllm.query_hierarchical(input_data)
print(f"Matched at tier: {tier}") # 1=facts, 2=concepts, 3=rules

# Export hierarchy
hierarchy = qnllm.export_hierarchy()
# {'facts': 20, 'concepts': 5, 'rules': 2}
```

**Validation Criteria:**
- Concept formation rate: >50% (5+ concepts from 10 facts)
- Rule extraction rate: >20% (1+ rules from 5 concepts)
- Query speedup: 2-3x faster (check rules before facts)
- Memory savings: 30-50% (consolidated representation)

**Blockers:**
- Concept formation logic needs revision (currently extracts 0 concepts)
- Rule extraction not implemented yet

---

### 3. Curriculum Learning Implemented (Experimental)

**Current State (v2.2):**
- Implemented in `research_extensions.py`
- Progressive difficulty: easy → medium → hard
- Performance trajectory: 70% → 92% over 7 steps
- Advances to next stage at 92% accuracy threshold
- Bug fixed: Function signature now accepts `stage` parameter

**Production Requirements for v2.3:**
- [ ] Move to `src/curriculum/qnllm_curriculum.py`
- [ ] Add adaptive difficulty (auto-adjust based on performance)
- [ ] Support custom curriculum schedules (manual, linear, exponential)
- [ ] Implement curriculum metrics (pacing, mastery, frustration)
- [ ] Benchmark on standard curriculum datasets (CUB-200, CIFAR-10)
- [ ] Validate on all 4 variants
- [ ] Write tests for scheduling logic
- [ ] Document curriculum design patterns

**API Design (v2.3):**
```python
from src.curriculum import QNLLMWithCurriculum

# Create with curriculum
qnllm = QNLLMWithCurriculum(
 variant="strategy",
 schedule="adaptive", # manual, linear, exponential, adaptive
 mastery_threshold=0.9, # advance when accuracy > 90%
 frustration_limit=5 # retry after 5 consecutive failures
)

# Train with curriculum (auto-schedules difficulty)
easy_data = [...]
medium_data = [...]
hard_data = [...]

curriculum = [easy_data, medium_data, hard_data]
history = qnllm.train_with_curriculum(curriculum)

# Check curriculum progress
progress = qnllm.get_curriculum_progress()
# {'stage': 2, 'accuracy': 0.92, 'next_stage_in': 3}

# Manually schedule next batch
next_batch = qnllm.schedule_next_batch(performance=0.85)
```

**Validation Criteria:**
- Curriculum improves final accuracy by 10-15%
- Reduces configuration time by 20-30%
- Adaptive schedule outperforms fixed schedule
- All 4 variants benefit from curriculum

---

## Integration Plan

### Phase 1: Attention (Weeks 1-4)
1. **Week 1:** Move to `src/attention/`, refactor API
2. **Week 2:** Add multi-head attention, masking
3. **Week 3:** Benchmark on all variants, optimize
4. **Week 4:** Write tests, documentation, merge to v2.3-dev

### Phase 2: Curriculum (Weeks 5-7)
1. **Week 5:** Move to `src/curriculum/`, add adaptive scheduling
2. **Week 6:** Benchmark on standard datasets, validate variants
3. **Week 7:** Write tests, documentation, merge to v2.3-dev

### Phase 3: Hierarchy (Weeks 8-12)
1. **Week 8-9:** Fix concept formation logic (major blocker)
2. **Week 10:** Implement rule extraction
3. **Week 11:** Add hierarchical query, benchmark
4. **Week 12:** Write tests, documentation, merge to v2.3-dev

### Phase 4: Integration Testing (Weeks 13-14)
1. **Week 13:** Test all 3 extensions together (attention + curriculum + hierarchy)
2. **Week 14:** Performance regression testing, finalize v2.3

---

## Extension Combinations

### Attention + Curriculum
**Use Case:** Focus on hard examples in later curriculum stages

```python
qnllm = QNLLMWithAttention(
 attention_type="scaled",
 curriculum="adaptive"
)

# Train with curriculum
# Attention automatically focuses on hard examples in later stages
qnllm.train_with_curriculum_and_attention(curriculum_data)
```

**Expected Benefit:** 15-20% accuracy improvement over curriculum alone

---

### Hierarchy + Attention
**Use Case:** Attend to important concepts/rules during consolidation

```python
qnllm = QNLLMWithHierarchy(
 hierarchy_depth=3,
 attention_type="multi-head"
)

# Consolidate with attention
# High-attention facts become concepts
concepts = qnllm.consolidate_with_attention()
```

**Expected Benefit:** Better concept quality (70% purity vs. 50% baseline)

---

### All Three Extensions
**Use Case:** Complete adaptive learning pipeline

```python
qnllm = QNLLMComplete(
 attention_type="multi-head",
 hierarchy_depth=3,
 curriculum="adaptive"
)

# Train with full pipeline
# 1. Curriculum schedules examples (easy → hard)
# 2. Attention focuses on important examples
# 3. Hierarchy consolidates learned patterns
qnllm.train_complete(data)
```

**Expected Benefit:** 25-30% accuracy improvement over baseline

---

## Benchmarks

### Attention Benchmarks (Target v2.3)
| Variant | Baseline Accuracy | With Attention | Latency Overhead |
|--------------|-------------------|----------------|------------------|
| CodeLearn | 21.7% | 28-30% | +3-5% |
| Strategy | 60% | 68-72% | +3-5% |
| Multimodal | 60% | 68-72% | +4-6% |
| MultiAgent | 89.7% consensus | 92-94% | +5-7% |

### Curriculum Benchmarks (Target v2.3)
| Dataset | Baseline Accuracy | With Curriculum | configuration Time |
|--------------|-------------------|-----------------|----------------|
| Code Tasks | 21.7% | 35-40% | -20-25% |
| Strategy | 60% | 70-75% | -15-20% |
| Multimodal | 60% | 70-75% | -20-25% |
| MultiAgent | 89.7% | 92-94% | -10-15% |

### Hierarchy Benchmarks (Target v2.3)
| Variant | Memory Usage | Query Speedup | Concept Quality |
|--------------|--------------|---------------|-----------------|
| CodeLearn | -30-40% | 2-3x | 65-75% purity |
| Strategy | -25-35% | 2-2.5x | 60-70% purity |
| Multimodal | -35-45% | 2.5-3x | 70-80% purity |
| MultiAgent | -30-40% | 2-3x | 65-75% purity |

---

## Testing Strategy

### Unit Tests (Per Extension)
```bash
# Attention
pytest tests/test_attention.py -v
# 15 tests: dot-product, scaled, multi-head, masking, visualization

# Curriculum
pytest tests/test_curriculum.py -v
# 12 tests: manual, linear, exponential, adaptive, mastery, frustration

# Hierarchy
pytest tests/test_hierarchy.py -v
# 18 tests: fact storage, concept formation, rule extraction, query routing
```

### Integration Tests (All Extensions)
```bash
# Combined extensions
pytest tests/test_extensions_integration.py -v
# 10 tests: attention+curriculum, hierarchy+attention, all three, variants
```

### Regression Tests
```bash
# Ensure v2.3 doesn't break v2.2 behavior
pytest tests/test_regression_v2.2_to_v2.3.py -v
# 20 tests: all v2.2 demos still work, invariants satisfied, performance maintained
```

---

## Documentation

### New Docs for v2.3
- **docs/extensions/attention.md** - task routings guide
- **docs/extensions/curriculum.md** - Curriculum learning guide
- **docs/extensions/hierarchy.md** - Hierarchical consolidation guide
- **docs/extensions/EXTENSIONS_GUIDE.md** - Comprehensive reference

### Updated Docs
- **QNLLM_VARIANTS_GUIDE.md** - Add extension usage per variant
- **QNLLM_COMPARISON_MATRIX.md** - Add extension comparison rows
- **TEACHING_PACKAGE.md** - Add tutorials 13-15 (extensions)

---

## Release Criteria for v2.3

### Must-Have
- [x] task routings (production-ready)
- [x] Curriculum learning (production-ready)
- [ ] Hierarchical consolidation (production-ready) ← **BLOCKER**
- [ ] All 3 extensions work on all 4 variants
- [ ] 90%+ test coverage
- [ ] Performance benchmarks meet targets
- [ ] Documentation complete

### Nice-to-Have
- [ ] Multi-head attention visualization
- [ ] Curriculum design templates
- [ ] Hierarchy export/import
- [ ] Extension profiling tools

### Won't Have (Defer to v2.4)
- Distributed configuration
- Streaming data support
- Real-time adaptation
- Edge device deployment

---

## Migration Guide (v2.2 → v2.3)

### Code Changes
```python
# v2.2 (no extensions)
from scripts.qnllm_unified_api import QNLLMFactory
qnllm = QNLLMFactory.create("codelearn")
qnllm.train(x, y)

# v2.3 (with extensions)
from src.extensions import QNLLMComplete
qnllm = QNLLMComplete(
 variant="codelearn",
 attention="scaled",
 curriculum="adaptive",
 hierarchy_depth=3
)
qnllm.train_complete(x, y)
```

### Breaking Changes
- `research_extensions.py` deprecated (use `src/extensions/` instead)
- `train_with_attention()` → `train()` with `use_attention=True`
- `consolidate_hierarchical()` → automatic consolidation every N steps

---

**Version:** 2.3-dev 
**Status:** In Development 
**Target Release:** Q2 2026 
**Blocker:** Hierarchical concept formation needs major revision
