# QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM - Complete Architecture Guide

**Status**: C++ Implementation Complete | **Version**: 1.0.0

---

## Table of Contents

1. [C++ System Architecture](#c-system-architecture)
2. [Multi-Teacher Learning (MTL-1) Specification](#multi-teacher-learning-mtl-1-specification)
3. [Real-Time Adaptation Framework](#real-time-adaptation-framework)

---

# C++ System Architecture (C++ Rewrite)

## Build and Binaries
```bash
cmake -S cpp -B cpp/build -DCMAKE_BUILD_TYPE=Release
cmake --build cpp/build --config Release
./cpp/build/Release/example_usage.exe # demo
./cpp/build/Release/test_nllm.exe # tests
```

## High-Level Flow (C++)
```
User Query

Teachers (mock/NIM) > MTL Loop > Feedback scores

 Memory Store

 Embedder
```

## Core Modules (C++)

- Config
 - [cpp/include/config/configs.hpp](cpp/include/config/configs.hpp): all runtime settings as structs (no YAML/JSON).
 - [cpp/src/config/config_loader.cpp](cpp/src/config/config_loader.cpp): CLI/file overrides.

- Teachers & MTL
 - [cpp/include/systems/teachers/mock_teacher.hpp](cpp/include/systems/teachers/mock_teacher.hpp): deterministic/random mock teachers.
 - [cpp/include/systems/teachers/nim_teacher.hpp](cpp/include/systems/teachers/nim_teacher.hpp): placeholder NIM API teacher.
 - [cpp/include/core/pipeline/mtl_loop.hpp](cpp/include/core/pipeline/mtl_loop.hpp): multi-teacher orchestration, fan-out/fan-in, feedback aggregation.

- Feedback
 - [cpp/include/systems/feedback/scorer.hpp](cpp/include/systems/feedback/scorer.hpp): output scoring (grammar, relevance, coherence, fluency) and disagreement scoring.

- Memory
 - [cpp/include/core/memory/store.hpp](cpp/include/core/memory/store.hpp): in-memory vector store with metadata and cosine retrieval.
 - [cpp/include/core/memory/embedder.hpp](cpp/include/core/memory/embedder.hpp): stub embedder producing fixed-size vectors.

- Utils
 - [cpp/include/utils/utils.hpp](cpp/include/utils/utils.hpp): strings, vectors, logging, timing.

## Execution Path (example_usage.exe)
1) Build config defaults in C++ (no external files).
2) Instantiate mock teachers from config teacher pool.
3) MTL loop queries teachers in parallel (std::async), aggregates responses.
4) Feedback scorer computes quality/disagreement metrics.
5) Memory store persists in-memory entries (future disk persistence stubbed).
6) Embedder generates vectors for demo text and batch operations.

## Data & Persistence
- Default paths are defined in config structs; no YAML/JSON required at runtime.
- Memory persistence/loading is stubbed for future C++ disk I/O.
- Logs: optional; JSON artifacts are no longer required.

## Key Parameters (from configs.hpp)
- Teacher pool: mock-teacher-1..4 enabled, timeout 30s, retries 2, parallel on.
- Memory: encoding dim 768, max_memories 10000, similarity_threshold 0.3, default_k 5.
- Generation: max_tokens 300, temperature 0.7, top_p 0.9, top_k 50.
- System: device auto, cache enabled, data/log dirs under `data/` and `logs/` (no YAML).

## Testing
- Binary tests: run `./cpp/build/Release/test_nllm.exe`.
- Demo run: `./cpp/build/Release/example_usage.exe`.

## Migration Notes
- All Python/YAML configs and scripts were removed; configuration now lives in C++ headers.
- Any references to `.py` pipelines, YAML configs, or JSON config files are obsolete.

## Retrieval Configuration

```
RETRIEVAL:
 similarity_threshold = 0.3
 top_k = 5
 confidence_weighting = optional (default: 1.0)

FEEDBACK MAPPING:
 quality 0.9 â†’ state variables 0.8 â†’ slower decay (reinforce)
 quality 0.5 â†’ state variables 1.2 â†’ normal decay
 quality 0.2 â†’ state variables 1.5 â†’ faster decay (punish)
```

## Data Flow Summary

```
Query â†’ encoding â†’ Retrieval â†’ Decay â†’ Assembly â†’ Autonomous Processor â†’ Response
 â†‘ â†“
 Feedback Loop (Quality Score) 
 â†“
 Memory Update (Plasticity)
```

---

# Multi-Teacher Learning (MTL-1) Specification

**Status:** Implementation Complete 
**Date:** January 13, 2026 
**Scope:** Consensus-driven feedback signals for v1.0 plasticity mechanism

## Overview

MTL-1 is a signal-generation layer that feeds outcome feedback to the frozen v1.0 plasticity mechanism. Instead of synthetic binary feedback (0.0 = poor, 1.0 = excellent), MTL-1 computes feedback by:

1. Querying a pool of independent teachers (pre-trained LLM systems, Claude, Gemini, DeepSeek)
2. Analyzing semantic agreement and confidence across responses
3. Scoring disagreement as a learning pressure signal
4. Mapping agreement/confidence to a quality score (input to v1.0's `feedback_weight` function)

**Key invariant:** Teachers never write to memory. They only generate signals. Memory plasticity remains orthogonal to generation.

## Architecture

### Data Flow

```
User Prompt
 â†“
NLLM v1.0 (memory retrieval)
 - Embed query
 - Retrieve top-k memories
 - Assemble context
 â†“
Generate Response
 â†“
Teacher Pool (parallel execution)
 - pre-trained LLM systems: {text, confidence, tokens}
 - Claude: {text, confidence, tokens}
 - Gemini: {text, confidence, tokens}
 - DeepSeek: {text, confidence, tokens}
 â†“
Disagreement Scorer
 - Semantic similarity (pairwise)
 - Confidence variance
 - Agreement threshold
 â†“
Quality Score Computation
 - agreement_score âˆˆ [0, 1]
 - confidence_score âˆˆ [0, 1]
 - combined_quality = f(agreement, confidence)
 â†“
Feedback Application (v1.0)
 - quality_score â†’ feedback_weight
 - decay_factor â† state variablesed_decay(quality_score)
 - memory[i].m_i â† updated decay state
```

## Disagreement Scoring

### Semantic Similarity

Computes pairwise cosine similarity between teacher responses:
- High similarity (> threshold) = agreement
- Low similarity = disagreement (learning pressure)
- Cosine metric: encodings use all-MiniLM-L6-v2 (768-dim)

### Agreement Score

```
agreement_score âˆˆ [0, 1]
 - agreement_score > 0.8 = "Strong agreement"
 - agreement_score > 0.5 = "Moderate agreement"
 - agreement_score â‰¤ 0.5 = "Significant disagreement"
```

### Confidence Spread

```
spread_score âˆˆ [0, 1] (0 = certain, 1 = highly uncertain)
 - spread < 0.2 = "Teachers confident and aligned"
 - spread < 0.5 = "Moderate confidence variance"
 - spread â‰¥ 0.5 = "High uncertainty"
```

### Quality Score Computation

```python
quality = agreement Ã— (1.0 - 0.5 Ã— spread)
```

Mapping to feedback:
- High agreement + low spread â†’ quality â‰ˆ 1.0 â†’ reinforce
- Low agreement + high spread â†’ quality â‰ˆ 0.0 â†’ punish
- Mixed signals â†’ quality â‰ˆ 0.5 â†’ neutral

## Integration with v1.0

```
Memory Update Flow:
1. Retrieve memories (v1.0)
2. Query teachers (MTL-1)
3. Apply feedback to v1.0
4. Persist memories (unchanged)
```

**Non-modification guarantee:** MTL-1 does not import or modify:
- Memory decay equations
- Retrieval mechanisms
- Storage layer

MTL-1 only generates quality scores and feeds them to existing feedback functions.

## Configuration (from config/mtl.yaml)

```yaml
teachers:
 enabled:
 - nvidia/nemotron-3-nano-30b-a3b
 - meta/llama-3.1-405b-instruct
 - openai/gpt-oss-120b
 timeout_seconds: 30
 max_retries: 2

api_keys:
 nvidia/nemotron-3-nano-30b-a3b: "<set via NVIDIA_API_KEY_NEMOTRON>"
 meta/llama-3.1-405b-instruct: "<set via NVIDIA_API_KEY_LLAMA>"
 openai/gpt-oss-120b: "<set via NVIDIA_API_KEY_GPT_OSS>"

disagreement:
 encoding_model: "sentence-deterministic processors/all-MiniLM-L6-v2"
 agreement_threshold: 0.7
 confidence_weight: 1.0

feedback:
 quality_low: 0.0
 quality_neutral: 0.5
 quality_high: 1.0
 apply_to_v1_0: true
```

## Testing

Unit tests validate:
- Identical responses â†’ high agreement
- Diverse responses â†’ low agreement
- High agreement + low spread â†’ reinforce
- Low agreement + high spread â†’ punish

Integration tests with mock teachers verify:
- Parallel teacher querying
- Feedback aggregation
- v1.0 integration

---

# Real-Time Adaptation Framework

## Concept

**Learning does not pause during use. Interaction *is* configuration.**

The NLLM adapts continuously during live user interaction, treating each exchange not as an isolated processing but as a learning event that influences future behavior.

## Core Principles

### 1. Interaction as Neurological Stimulus

Each user message triggers:
- Memory retrieval
- Contextual reasoning
- Quality evaluation
- Selective reinforcement or decay

### 2. Real-Time Behavioral Adjustment

The NLLM adjusts how it answers based on:
- User satisfaction signals (explicit or inferred)
- Repetition or correction patterns
- Conversational pacing and structure
- Concept reuse efficiency

Adaptation occurs **within the same session**, not only across sessions.

### 3. Live Memory Plasticity

User interaction directly affects:
- Which memories are reinforced
- Which representations weaken
- Which reasoning patterns persist

### 4. Continuous Judgment Loop

```
Input â†’ Reason â†’ Respond â†’ Evaluate â†’ Adapt â†’ Next Response
```

### 5. Emergent Conversational Competence

Over time, the system:
- Becomes more concise with the same user
- Reduces unnecessary explanations
- Aligns to the user's preferred reasoning style
- Anticipates informational needs

## Mathematical Formulation

### Signal Aggregation

$$\Sigma_t = \sum_{i=1}^{n} w_i \cdot v_i$$

where:
- $S_t = \{s_1, s_2, \ldots, s_n\}$ = interaction signals
- $w_i$ = signal state variables
- $v_i$ = valence âˆˆ [-1, 1]

### Behavioral Modulation

$$\theta_{t+1} = \theta_t + \alpha \cdot f(\Sigma_t)$$

where:
- $\theta_t$ = current parameter state
- $\alpha$ = adaptation rate (e.g., 0.05)
- $f(\Sigma_t)$ = bounded transformation (e.g., $\tanh(\Sigma_t)$)

### Memory Reinforcement Integration

$$\text{decay\_factor}_i = \text{base\_decay} \times (1 + \beta \cdot q_i)$$

where:
- $q_i \in [0, 1]$ = quality signal
- $\beta \in [0.3, 1.7]$ = bounded modulation range

## Interaction Signal Types

| Signal | Valence | Impact | Example |
|--------|---------|--------|---------|
| Confirmation | +1.0 | Reinforce memory | "Yes, that's correct" |
| Correction | -1.0 | Weaken memory | "No, that's wrong" |
| Repetition | +0.5 | Re-retrieve memory | Asking same question again |
| Expansion | +0.7 | Extend context window | "Tell me more" |
| Compression | -0.5 | Compress response | "That's too long" |

## Architectural Integration

### Connection to MTL

- **MTL background**: Asynchronous multi-teacher evaluation
- **Real-time adaptation**: Synchronous in-session learning
- Both share same plasticity substrate (memory decay modulation)

### Separation from Safety Systems

Adaptation operates **within safety bounds**:

```
User Input
 â†“
Safety Guardrails (ALWAYS ENFORCED)
 â†“
Retrieval + Reasoning
 â†“
Real-Time Adaptation Layer
 â†“
Response Generation
```

Adaptation cannot:
- Bypass safety filters
- Generate prohibited content
- Violate system constraints

## Implementation Properties

### 1. Statefulness with Reversibility

```python
session_state = {
 'signals': [], # History
 'param_deltas': {}, # Adjustments
 'memory_modulations': {} # Learning
}
```

State can be saved, reset, or inspected.

### 2. Bounded Adaptation

All parameter changes constrained:
- Temperature: [0.1, 1.5]
- Top-p: [0.5, 1.0]
- Max tokens: [50, 2000]

### 3. Signal Decay

$$w_i(t) = w_{i,0} \cdot e^{-\lambda \cdot (t - t_i)}$$

Prevents ancient feedback from dominating current behavior.

## Measurable Properties

### 1. Behavioral Convergence

Track parameter delta magnitude:
$$\text{Convergence}(t) = \frac{1}{k} \sum_{i=1}^{k} |\theta_i^{(t)} - \theta_i^{(0)}|$$

**Expected**: Large initial changes â†’ gradual stabilization

### 2. Memory Retrieval Shift

$$\text{Retrieval\_Shift}(t) = \text{KL}(P_t \| P_0)$$

**Expected**: Positive correlation with correction/confirmation count

### 3. Response Compression

$$\text{Compression}(t) = 1 - \frac{\text{len}(r_t)}{\text{len}(r_0)}$$

**Expected**: Gradual reduction as user needs learned

### 4. Signal-Response Causality

Controlled experiment:
1. Session A: provide positive signals
2. Session B: provide negative signals
3. Measure response divergence

**Expected**: Different signals â†’ different parameters â†’ different outputs

## Distinction from Existing Work

| Approach | Learning Time | Feedback | Scope |
|----------|---|---|---|
| Few-shot | This response | In prompt | Context window |
| RLHF | Offline | Reward model | Task level |
| Personalization | Offline | User profile | User level |
| **Real-time adaptation** | **During session** | **User signals** | **System behavior** |

## Validation Experiments

1. **Signal Causality Test**: Identical queries with opposite signals
2. **Memory Reinforcement Test**: Track retrieval probability shifts
3. **Session Stability Test**: 100-turn conversations with random signal injection
4. **Cross-Session Persistence Test**: Save/restore/validate adapted behavior

## Related Components

- **MTL Background Learning**: Asynchronous multi-teacher evaluation
- **Memory Decay Equations**: Mathematical substrate for plasticity
- **Feature Configuration**: Enable/disable control
- **RealTime Adapter Implementation**: Code realization

---

## Key Insight

> The NLLM does not wait for configuration to learn.
> It learns **because** it is interacting.

---

*Document Consolidated: January 15, 2026* 
*Status: Production Ready*
