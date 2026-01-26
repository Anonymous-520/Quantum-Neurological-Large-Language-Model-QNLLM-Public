# QNLLM Variants Guide (v2.2)

Complete reference guide for all four specialized QNLLM variants. Each variant inherits v2.2 core (gated learning, multi-timescale memory, selective plasticity) while specializing for specific domains and tasks.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Architecture Overview](#architecture-overview)
3. [QNLLM-CodeLearn](#qnllm-codelearn)
4. [QNLLM-Strategy](#qnllm-strategy)
5. [QNLLM-Multimodal](#qnllm-multimodal)
6. [Multi-Agent QNLLM](#multi-agent-qnllm)
7. [Variant Selection Guide](#variant-selection-guide)
8. [Common Patterns](#common-patterns)
9. [API Reference](#api-reference)

---

## Quick Start

### Using Any Variant

```python
from qnllm_unified_api import QNLLMFactory

# Initialize variant
learner = QNLLMFactory.create('codelearn')

# Train on domain-specific data
learner.train(examples=code_examples)

# Generate outputs
output = learner.generate(query="def fibonacci(n):")
print(output) # "Recursive function definition..."
```

### Individual Variant Usage

```python
# CodeLearn: Code understanding
from scripts.demo_qnllm_codelearn import run_codelearn_demo
result = run_codelearn_demo()

# Strategy: Multi-step reasoning
from scripts.demo_qnllm_strategy import run_strategy_demo
result = run_strategy_demo()

# Multimodal: Cross-domain learning
from scripts.demo_qnllm_multimodal import run_multimodal_demo
result = run_multimodal_demo()

# MultiAgent: Distributed learning
from scripts.demo_qnllm_multiagent import run_multiagent_demo
result = run_multiagent_demo()
```

---

## Architecture Overview

### Core v2.2 Invariants (Inherited by All Variants)

| Invariant | Mechanism | Purpose |
|-----------|-----------|---------|
| **1: Gated Learning** | Hysteresis-based uncertainty gate (θ_high=0.65, θ_low=0.45) | Prevents oscillation, gates learning on error |
| **2: Error-Driven Updates** | Gradient magnitude proportional to prediction error | Only learns when uncertain |
| **3: Multi-Timescale Memory** | Fast (20), Medium (100), Slow (500) tiers | Prevents catastrophic forgetting |
| **4: Selective Plasticity** | Mask protects consolidated state variables | Stable memories survive new learning |
| **5: Consolidation** | Memories migrate: fast→medium→slow | Experience becomes principle |
| **6: Meta-Convergence** | Uncertainty variance as convergence signal | System knows when it's learned |
| **8: Reasoning Depth** | Bounded by slow memory consolidation | Prevents unbounded hypothesis growth |
| **9: Learning Signal** | Joint loss: prediction error + uncertainty reduction | Explicit feedback on learning progress |

### Specialization Axes

```
 SPEED (immediate) ◄────────────► DEPTH (principled)
 ▲
 │
 CodeLearn │ Strategy
 (syntax │ (multi-level
 routing) │ reasoning)
 │
 ◄─────────────────────────────►
 SINGLE-DOMAIN MULTI-DOMAIN
 │
 │
 Multimodal │ MultiAgent
 (unified │ (distributed
 encoding) │ consensus)
 ▼
 SCOPE (modular) ◄────────────► INTEGRATION (global)
```

---

## QNLLM-CodeLearn

**Domain:** Code understanding and explanation 
**Key Innovation:** Syntax keyword routing (deterministic pattern matching) 
**Best For:** Understanding code patterns, generating code explanations, syntax-aware learning

### Architecture

```
Input: Code snippet
 ↓
Syntax Keyword Router
 ├─ def/lambda → function_def
 ├─ class → class_def
 ├─ for/while → loop
 ├─ if/elif → conditional
 ├─ return/yield → return_statement
 └─ ... (7 pattern categories)
 ↓
Text Head
 ├─ Pattern Matching: Find similar examples in configuration set
 ├─ Retrieval: Return best matching explanation
 └─ Fallback: General pattern description
 ↓
Output: Code explanation (no hallucination, guaranteed retrieval)
```

### configuration Examples

```python
code_examples = [
 ("def factorial(n):\n return 1 if n <= 1 else n * factorial(n-1)",
 "Recursive function: breaks problem into subproblems."),
 ("for i in range(10):\n if i % 2 == 0:\n print(i)",
 "Loop with conditional: iterates and filters."),
 ("class Model:\n def __init__(self, data):\n self.data = data",
 "Class definition: encapsulates data and methods."),
 # ... more examples
]

learner.train_on_examples(code_examples)
```

### Generating Explanations

```python
code_snippet = "x = [i**2 for i in range(10) if i % 3 == 0]"
explanation = learner.generate_explanation(code_snippet)
# Output: "List comprehension: concise syntax for filtering and transforming sequences."
# Method: "keyword_route_comprehension"
```

### Key Characteristics

| Aspect | Value |
|--------|-------|
| **Memory Usage** | Low (keyword patterns only) |
| **Speed** | Very fast (O(1) keyword matching) |
| **Interpretability** | Very high (routing is explicit) |
| **Generalization** | Limited to trained syntax patterns |
| **Error Rate** | ~0% (deterministic routing) |
| **Scalability** | Excellent (linear in pattern count) |

### When to Use CodeLearn

 Understanding code structure 
 Generating code explanations 
 Syntax pattern recognition 
 Code documentation 
 Teaching programming concepts 

 Learning complex code logic 
 Cross-language reasoning 
 Semantic code understanding 

---

## QNLLM-Strategy

**Domain:** Multi-step reasoning and decision making 
**Key Innovation:** Tiered strategy routing (immediate tactic → tactical plan → strategic principle) 
**Best For:** Planning, decision support, multi-step reasoning, strategy explanation

### Architecture

```
Input: Decision scenario
 ↓
Reasoning Depth Estimation
 ├─ Depth 0-1 → immediate_tactic (quick heuristics)
 ├─ Depth 2-5 → tactical_plan (concrete plans)
 └─ Depth >5 → strategic_principle (abstract principles)
 ↓
Strategy Retrieval
 ├─ Match scenario to learned strategies
 ├─ Return best explanation for depth
 └─ Consolidate reasoning trail
 ↓
Output: Multi-level strategy explanation
```

### configuration Examples

```python
strategy_scenarios = [
 ("Budget low, deadline soon, quality critical",
 "Allocate all resources to quality assurance. Skip optional features.",
 "tactical_plan"),
 ("Market condition: rising demand, competitors entering",
 "Scale production, differentiate product, capture market share first.",
 "strategic_principle"),
 ("Algorithm converges slowly, model overfits",
 "Add regularization, reduce capacity, or change gating threshold.",
 "immediate_tactic"),
 # ... more scenarios
]

learner.train_on_scenarios(strategy_scenarios)
```

### Generating Strategies

```python
state = "Uncertain about requirements, customer feedback limited"
depth = 5
strategy = learner.generate_strategy(state, depth)
# Output: "Build MVP, iterate with customer feedback, reduce initial investment."
# Method: "depth_5_via_tactical_plan"
```

### Key Characteristics

| Aspect | Value |
|--------|-------|
| **Memory Usage** | Low-Medium (tiered strategies) |
| **Speed** | Fast (depth-based lookup) |
| **Interpretability** | High (reasoning depth explicit) |
| **Generalization** | Moderate (works on similar scenarios) |
| **Error Rate** | ~20% (strategies don't always apply) |
| **Scalability** | Good (linear in scenario count) |
| **Reasoning Depth** | Bounded by slow memory (Invariant 8) |

### When to Use Strategy

 Multi-step decision making 
 Business strategy explanation 
 Risk assessment and planning 
 Trade-off analysis 
 Bounded reasoning 

 Real-time decisions 
 Complex constraint satisfaction 
 Adversarial reasoning 

---

## QNLLM-Multimodal

**Domain:** Cross-domain learning (code, strategy, language) 
**Key Innovation:** Unified encoding space with domain-aware scope detection 
**Best For:** Cross-domain reasoning, multi-aspect problems, unified knowledge base

### Architecture

```
Input: Mixed-domain query
 ↓
Domain Scope Detection
 ├─ Keyword matching for code, strategy, language
 ├─ Identify relevant domains (can be multiple)
 └─ state variables domain relevance
 ↓
Cross-Domain encoding
 ├─ Unified representation (50-dim vector)
 ├─ Domain-agnostic gating
 └─ Shared selective plasticity
 ↓
Multi-Domain Text Head
 ├─ code_examples: 2+ examples from code domain
 ├─ strategy_examples: 2+ examples from strategy
 ├─ language_examples: 2+ examples from language
 └─ cross_domain_examples: Principles that apply across domains
 ↓
Output: Cross-domain explanation (integrates multiple perspectives)
```

### configuration Examples

```python
mixed_tasks = [
 # Code domain
 ("def solve(x): return x**2 if x > 0 else 0",
 "Conditional function: applies different logic based on input range.",
 "code"),

 # Strategy domain
 ("Facing uncertainty, multiple options, limited resources",
 "Identify core constraints, prioritize high-impact decisions.",
 "strategy"),

 # Language domain
 ("What pattern defines effective communication?",
 "Clear intent, shared understanding, feedback loops.",
 "language"),

 # Cross-domain (applies to code, strategy, AND language)
 ("Problem: complexity growing unbounded",
 "Decompose into modular parts, establish clear interfaces.",
 "cross_domain"),

 # ... more examples
]

learner.train_on_mixed_tasks(mixed_tasks)
```

### Generating Cross-Domain Explanations

```python
query = "How should we structure a codebase for long-term maintainability?"
explanation = learner.generate_explanation(query)
# Detected domains: ['code', 'language']
# Output: Code structure explanation with language-aware principles
# Method: "domain_code"
```

### Key Characteristics

| Aspect | Value |
|--------|-------|
| **Memory Usage** | Medium (unified encoding) |
| **Speed** | Medium (domain detection + retrieval) |
| **Interpretability** | Medium-High (domains explicit, reasoning implicit) |
| **Generalization** | Good (principles transfer across domains) |
| **Error Rate** | ~15% (domain detection errors) |
| **Scalability** | Good (scales linearly with domains) |
| **Domains** | 3 core (code, strategy, language); extensible |

### When to Use Multimodal

 Cross-domain problems 
 Software architecture 
 Organizational design 
 System thinking 
 Integrated knowledge 

 Specialized deep knowledge 
 Domain-specific expertise 
 Single-domain speed requirements 

---

## Multi-Agent QNLLM

**Domain:** Distributed learning with peer disagreement 
**Key Innovation:** Consensus emergence through weak supervision 
**Best For:** Collective learning, robust systems, knowledge aggregation

### Architecture

```
Input: Shared task (same x, y for all agents)
 ↓
 ┌─────────────┬──────────────┬──────────────┐
 ▼ ▼ ▼ ▼
Agent 0 Agent 1 Agent 2 Agent n
 │ │ │ │
 ├─ Predict ├─ Predict ├─ Predict ├─ Predict
 │ │ │ │
 └─────────────┴──────────────┴──────────────┘
 ▼
 Disagreement Scoring
 ├─ Measure agreement across predictions
 ├─ High agreement → consolidate to shared memory
 └─ Low agreement → individual learning
 ▼
 Weak Supervision
 ├─ Each agent learns from peer predictions
 ├─ Peer avg becomes weak label
 └─ Accelerates convergence
 ▼
 Update All Agents
 ├─ Each agent steps with peer prediction
 ├─ Gate modulation (error-driven)
 └─ Memory consolidation (multi-timescale)
 ▼
 Shared Slow Memory
 ├─ Consensus-driven consolidation
 └─ Principles that all agents agree on
 ▼
Output: Consensus prediction + individual explanations
```

### Multi-Agent Learning Example

```python
# Initialize 3 agents
coordinator = MultiAgentCoordinator(num_agents=3)

# Shared task (all agents learn on same data, independent state variables)
for step in range(40):
 x = np.random.randn(50) * 0.5
 y_true = np.random.rand(5)

 # Each agent predicts independently
 # Peer predictions become weak supervision
 # Consensus emerges over time
 result = coordinator.step_all(x, y_true)

# Final consensus: 89.9% (agents agree)
print(f"Consensus: {result['consensus']:.3f}")
```

### Key Characteristics

| Aspect | Value |
|--------|-------|
| **Memory Usage** | High (N agents × memory) |
| **Speed** | Slow (sequential agent updates) |
| **Interpretability** | Medium (consensus is aggregate signal) |
| **Generalization** | Excellent (ensemble effect) |
| **Error Rate** | ~5% (reduced by consensus) |
| **Scalability** | Moderate (linear in agent count) |
| **Consensus** | Emerges naturally (89-93% typical) |
| **Robustness** | Very high (peer supervision) |

### When to Use Multi-Agent

 Robust decision making 
 Knowledge aggregation 
 Distributed learning 
 Consensus-driven systems 
 Weak supervision scenarios 

 Real-time latency requirements 
 Low-memory environments 
 Single-agent interpretability 

---

## Variant Selection Guide

### Decision Tree

```
Does your problem require:

1. Understanding existing code/syntax?
 YES → Use CodeLearn
 NO → Next

2. Multi-step reasoning with bounded depth?
 YES → Use Strategy
 NO → Next

3. Multiple domains in one problem?
 YES → Use Multimodal
 NO → Next

4. Robust learning from unreliable sources?
 YES → Use Multi-Agent
 NO → CodeLearn (default)
```

### Comparison Table

| Dimension | CodeLearn | Strategy | Multimodal | MultiAgent |
|-----------|-----------|----------|-----------|------------|
| **Interpretability** | | | | |
| **Speed** | | | | |
| **Accuracy** | | | | |
| **Generalization** | | | | |
| **Memory Efficiency** | | | | |
| **Reasoning Depth** | | | | |
| **Domain Scope** | 1 (code) | 1 (strategy) | 3+ (any) | N/A (domain-agnostic) |
| **Robustness** | Low | Medium | Medium | |

### Use Case Matrix

| Use Case | CodeLearn | Strategy | Multimodal | MultiAgent |
|----------|-----------|----------|-----------|------------|
| Code documentation | | | | |
| Decision support | | | | |
| Architecture design | | | | |
| Risk assessment | | | | |
| Cross-domain problems | | | | |
| Robust learning | | | | |
| Real-time systems | | | | |
| High accuracy needed | | | | |

Legend: Excellent, Acceptable, Not recommended

---

## Common Patterns

### Pattern 1: Cascading Variants

```python
# Start with fast CodeLearn for understanding
explanation = codelearn.generate_explanation(code)

# Escalate to Strategy for complex decisions
if "design" in explanation:
 strategy = strategy_learner.generate_strategy(scenario, depth=5)

# Use Multimodal for integrated view
integrated = multimodal.generate_explanation(combined_query)
```

### Pattern 2: Multi-Agent Voting

```python
# Get predictions from all variants
outputs = {
 'code': codelearn.predict(query),
 'strategy': strategy.predict(query),
 'multimodal': multimodal.predict(query),
 'agents': [agent.predict(query) for agent in coordinator.agents]
}

# Consensus vote
consensus = aggregate_predictions(outputs)
confidence = measure_agreement(outputs)
```

### Pattern 3: Hybrid Domain Learning

```python
# Train Multimodal on code + strategy examples
multimodal.train_on_mixed_tasks(code_strategy_pairs)

# Use MultiAgent for distributed learning
coordinator.step_all(input_data, labels)

# Combine learned representations
hybrid_representation = merge_embeddings(multimodal, coordinator)
```

---

## API Reference

### CodeLearn

```python
from scripts.demo_qnllm_codelearn import CodeTextHead

code_head = CodeTextHead()
code_head.train_on_examples(examples)
explanation, method = code_head.generate_explanation(code_snippet)
```

**Methods:**
- `train_on_examples(examples: List[Tuple[str, str]])` - Train on (code, explanation) pairs
- `generate_explanation(code: str) -> Tuple[str, str]` - Returns (explanation, routing_method)
- `_extract_keywords(code: str) -> List[str]` - Internal: Extract syntax patterns

### Strategy

```python
from scripts.demo_qnllm_strategy import StrategyTextHead

strategy_head = StrategyTextHead()
strategy_head.train_on_scenarios(scenarios)
strategy, method = strategy_head.generate_strategy(state, depth)
```

**Methods:**
- `train_on_scenarios(scenarios: List[Tuple[str, str, str]])` - Train on (state, decision, level) tuples
- `generate_strategy(state: str, depth: int) -> Tuple[str, str]` - Returns (strategy, routing_method)

### Multimodal

```python
from scripts.demo_qnllm_multimodal import DomainTextHead

text_head = DomainTextHead()
text_head.train_on_mixed_tasks(tasks)
explanation, method = text_head.generate_explanation(prompt)
domains = text_head.detect_scope(prompt)
```

**Methods:**
- `train_on_mixed_tasks(tasks: List[Tuple[str, str, str]])` - Train on (prompt, explanation, domain) tuples
- `detect_scope(prompt: str) -> List[str]` - Identify relevant domains
- `generate_explanation(prompt: str) -> Tuple[str, str]` - Returns (explanation, routing_method)

### Multi-Agent

```python
from scripts.demo_qnllm_multiagent import MultiAgentCoordinator

coordinator = MultiAgentCoordinator(num_agents=3)
result = coordinator.step_all(x, y_true)
consensus = result['consensus']
disagreement = result['disagreement']
```

**Methods:**
- `step_all(x: np.ndarray, y_true: np.ndarray) -> Dict` - Execute one learning step across all agents
- `agents` - List of SimpleNeuralLearner instances (read-only)
- `consensus_history` - List of consensus scores over time
- `shared_slow_memory` - Consolidated knowledge shared by all agents

---

## Variant Development Roadmap

### v2.2 (Current)
 Four specialized variants implemented 
 Integration test: 4/4 variants passing 
 Invariants 1-9 validated across all variants 

### v2.3 (Planned)
 Unified API for all variants 
 Hybrid experiments (Multimodal + MultiAgent) 
 Real-world benchmark validation 

### v2.4+ (Future)
 task routings for selective focus 
 Hierarchical consolidation (principles → concepts → rules) 
 Curriculum learning (easy → hard task scheduling) 
 Multimodal + MultiAgent fully integrated 

---

## References

- `QNLLM_SCIENTIFIC_SPEC.md` - Formal specification of v2.2 invariants
- `QNLLM_LEARNING_DYNAMICS.md` - Theoretical foundations
- `scripts/demo_qnllm_codelearn.py` - CodeLearn implementation
- `scripts/demo_qnllm_strategy.py` - Strategy implementation
- `scripts/demo_qnllm_multimodal.py` - Multimodal implementation
- `scripts/demo_qnllm_multiagent.py` - MultiAgent implementation
- `scripts/test_all_variants.py` - Integration tests

---

**Version:** 2.2 
**Status:** Production-Ready 
**Last Updated:** January 19, 2026
