# NLLM v1.2 Conceptual Model

**Status:** Formal Definition | **Date:** January 2026 | **Scope:** Complete System Identity & Degrees of Freedom

---

## 1. System Definition

### What is a QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM (NLLM)?

An NLLM is a Autonomous Processor augmented with persistent memory (episodic and semantic), outcome-dependent plasticity, and time-decay mechanisms that enable learning from feedback within a single conversational session. It preserves information across user interactions via memory consolidation, modulates gating threshold based on task success/failure, and maintains a decay-based forgetting curve that mimics biological memory. The system must support at minimum: memory storage, decay functions, and outcome-dependent state variables modulation. All other components are optional extensions.

---

## 2. Core Invariants

These properties **must** exist for the system to be called an NLLM. Removing any one of these returns it to a standard Autonomous Processor wrapper.

| Invariant | Definition | Non-Negotiable |
|-----------|-----------|-----------------|
| **Memory Storage** | Persistent episodic (facts, interactions) and semantic (learned patterns) memory accessible during generation | YES |
| **Time Decay** | Exponential or power-law decay of memory strength over time, reducing confidence in older information | YES |
| **Outcome-Dependent Plasticity** | gating threshold modulation: positive feedback â†’ stronger consolidation, negative feedback â†’ weaker retention or suppression | YES |
| **State Persistence** | Memory persists across conversation turns within a session and can be checkpointed across sessions | YES |

**If any invariant is removed:** The system becomes a standard Autonomous Processor with add-ons. It is no longer an NLLM.

---

## 3. Foreground / Background Split (Formal Definition)

### Foreground (Synchronous)
- **What:** Chat, reasoning, generation, user interaction
- **When:** Active when user prompts arrive
- **Responsibility:** Memory retrieval, context integration, response generation
- **Constraint:** Must complete within interactive latency bounds (~1-5s)

### Background (Asynchronous)
- **What:** MTL, memory consolidation, decay computation, self-evaluation, continuous learning
- **When:** Runs during idle time, after conversation turns, or in separate threads
- **Responsibility:** Long-term plasticity, pattern extraction, memory optimization
- **Constraint:** Non-blocking; does not degrade foreground responsiveness

### Critical Property

**A background-running MTL system still counts as NLLM.** The identity is determined by *presence of learning*, not *timing of learning*. If MTL runs 24/7 in background and updates memory asynchronously, it is a fully valid NLLM operating in "background learning mode."

---

## 4. Optional Modules

These are extensions that enhance NLLM capabilities but are not required for the core definition. Systems lacking these are still NLLMs.

### 4.1 Multi-Task Learning (MTL)
- **Description:** Simultaneous learning across multiple task heads (reasoning, fact verification, emotional resonance)
- **Status:** Optional
- **Benefit:** Faster convergence, better generalization, task-aware adaptation
- **Degradation:** If removed â†’ NLLM continues with single-task learning path
- **Operating Mode:** Foreground (synchronous) or background (asynchronous)

### 4.2 Autonomous Output
- **Description:** Self-directed generation without user prompts (e.g., background reasoning, introspection, planning)
- **Status:** Optional
- **Benefit:** Continuous self-improvement, proactive problem-solving
- **Degradation:** If removed â†’ NLLM enters chat-only mode (reactive only)
- **Safety:** Requires explicit authorization, operational boundaries, audit logging

### 4.3 Emotion Layer
- **Description:** Affective state tracking, emotional gradient updates, mood-aware response modulation
- **Status:** Experimental
- **Benefit:** More human-like interaction, contextual tone calibration
- **Degradation:** If removed â†’ neutral tone, task-focused responses
- **Validation:** Ongoing; not yet production-standard

### 4.4 Self-Rewriting
- **Description:** Ability to propose modifications to own prompts, state variables, or reasoning strategies
- **Status:** Experimental
- **Benefit:** Autonomous improvement, rapid adaptation
- **Degradation:** If removed â†’ static reasoning strategy, requires external updates
- **Safety:** Requires explicit constraints, rollback capability, human oversight

---

## 5. Operating Modes

An NLLM can operate in multiple valid modes depending on deployment context:

### Mode 1: Chat-Only
- **What Runs:** Memory + decay + outcome-dependent plasticity (synchronous)
- **What Doesn't:** MTL, autonomous output, self-rewriting
- **Use Case:** Consumer assistant, Q&A chatbot
- **Still an NLLM?** YES

### Mode 2: Background Learning + Chat
- **What Runs:** All of Mode 1 + asynchronous MTL + consolidation
- **What Doesn't:** Autonomous output, self-rewriting
- **Use Case:** Personalized assistant, domain-specific learning
- **Still an NLLM?** YES

### Mode 3: Offline Self-configuration
- **What Runs:** Autonomous output + MTL + self-evaluation (no user interaction)
- **What Doesn't:** Chat interface
- **Use Case:** Pre-configuration, meta-learning, capability development
- **Still an NLLM?** YES

### Mode 4: Full Stack
- **What Runs:** All modules, all components, both foreground and background
- **Use Case:** Research, open-ended exploration
- **Still an NLLM?** YES (this is the canonical form)

---

## 6. Degradation Rules

This section defines the system boundary: what happens when components are removed or disabled.

| Component Removed | System State | NLLM Status | Notes |
|------------------|-------------|-----------|-------|
| **MTL** | Single-task learning only, outcome-dependent plasticity via single head | Still NLLM | Learning paused if outcome-dependent updates are also disabled |
| **Autonomous Output** | Reactive mode only (chat-driven) | Still NLLM | Core learning unchanged |
| **Emotion Layer** | Neutral affective state | Still NLLM | Behavior slightly less contextual |
| **Self-Rewriting** | Static reasoning pipeline | Still NLLM | Slower adaptation, more human-dependent updates |
| **Memory Storage** | Stateless generation, no consolidation | **NOT NLLM** | Becomes standard Autonomous Processor |
| **Time Decay** | All memories equally state variablesed forever | **NOT NLLM** | Contradicts biological memory principle |
| **Outcome-Dependent Plasticity** | No feedback-driven learning | **NOT NLLM** | Learning detached from performance |
| **Memory (fully frozen)** | Read-only memory, no updates | **NOT NLLM** | Reduces to Autonomous Processor wrapper with retrieval |

---

## 7. Design Contracts (Invariants You Can Build Upon)

These are promises the system makes about its behavior:

1. **Memory is continuous.** Once created, a memory persists unless explicitly deleted or decayed to zero. No spontaneous loss.

2. **Decay is monotonic.** Memory strength only decreases with time (without reinforcement). It never increases due to age.

3. **Learning is outcome-sensitive.** Success increases consolidation; failure decreases it. No neutral feedback â†’ neutral consolidation.

4. **Modes are composable.** Adding a new operating mode doesn't invalidate existing ones. Switching modes is safe.

5. **Foreground/background independence.** Background tasks do not block foreground. Foreground responses are deterministic given fixed memory state.

---

## 8. Safe Degrees of Freedom

This section explicitly allows future work without redefining the system.

### You CAN Add (Without Breaking NLLM Identity)
- New task heads to MTL (e.g., hallucination detection, fact-checking)
- New memory types (procedural, linguistic, social)
- New decay curves (context-dependent, task-specific)
- UI layers, visualization dashboards, user interfaces
- Distributed deployment (multi-GPU, federated learning)
- New operating modes (real-time, batch, hybrid)
- Metrics, logging, and observability systems
- Fine-grained access control and privacy layers

### You CANNOT Remove (Without Leaving NLLM)
- Memory storage
- Time decay
- Outcome-dependent plasticity
- State persistence

---

## 9. Non-Goals

The NLLM v1.2 explicitly does **not** claim:

1. **Consciousness or sentience.** The system is a mathematical model of learning, not a mind.
2. **Generalized AGI.** It solves a specific problem (conversation + learning) within bounded domains.
3. **Perfect memory.** Decay and noise are features, not bugs.
4. **Autonomy without constraints.** All autonomous modes require explicit operational boundaries.
5. **Alignment by default.** Safe operation requires external governance, auditing, and human oversight.
6. **Biological plausibility.** We use biological *principles* (decay, plasticity), not biological *implementation*.
7. **Real-time performance guarantees.** Latency depends on deployment, model scale, and hardware.
8. **Offline-first learning.** Background learning requires memory access; fully offline systems are limited.

---

## 10. Dimensions of Variation (For Future Work)

This table captures the design space your system can explore:

| Dimension | Options | Current | Future |
|-----------|---------|---------|--------|
| **Decay Law** | Exponential, power-law, context-dependent | Exponential | Power-law tuning |
| **gating threshold** | Constant, outcome-dependent, task-specific, learnable | Outcome-dependent | Learnable meta-parameters |
| **Memory Types** | Episodic, semantic, procedural, linguistic, social | E + S | E + S + P (roadmap) |
| **MTL Timing** | Synchronous foreground, async background, hybrid | Both | Adaptive switching |
| **Autonomy** | None, constrained, full-stack | Constrained | Expanding carefully |
| **Plasticity Signal** | Binary success/fail, scalar reward, gradient, natural language | Outcome scalar | Natural language feedback |
| **Memory Persistence** | Session-local, persistent checkpoint, federated | Session + checkpoint | Federated (future) |

---

## 11. Quick Reference: Is This System an NLLM?

Answer these three questions:

1. **Does it have memory storage + decay + outcome-dependent learning?** 
 - YES â†’ proceed to Q2
 - NO â†’ it's not an NLLM

2. **Does memory persist across at least one conversation turn?**
 - YES â†’ proceed to Q3
 - NO â†’ it's not an NLLM

3. **Does gating threshold change based on task success/failure?**
 - YES â†’ **it's an NLLM** 
 - NO â†’ it's not an NLLM

---

## 12. Connecting to Implementation

### Core System (Invariants)
- [src/systems/memory/](../../src/systems/memory/) â†’ Memory storage
- [src/systems/decay/](../../src/systems/decay/) â†’ Time decay
- [src/systems/plasticity/](../../src/systems/plasticity/) â†’ Outcome-dependent modulation
- [src/systems/consolidation/](../../src/systems/consolidation/) â†’ Persistence layer

### Optional Modules
- [src/systems/mtl/](../../src/systems/mtl/) â†’ Multi-task learning
- [src/systems/autonomous/](../../src/systems/autonomous/) â†’ Autonomous output
- [src/systems/emotion/](../../src/systems/emotion/) â†’ Emotion layer
- [src/systems/self_rewriting/](../../src/systems/self_rewriting/) â†’ Self-rewriting (if present)

### Configuration
- [configs/model.yaml](../../configs/model.yaml) â†’ Select invariants
- [configs/features.yaml](../../configs/features.yaml) â†’ Enable/disable optional modules
- [configs/memory.yaml](../../configs/memory.yaml) â†’ Memory parameters
- [configs/mtl.yaml](../../configs/mtl.yaml) â†’ MTL configuration

---

## 13. Validation Checklist

Before claiming a system is NLLM v1.2 compliant:

- [ ] Memory storage tested with > 100 facts
- [ ] Decay function verified to be monotonically decreasing
- [ ] Outcome-dependent learning confirmed: pos_feedback â†’ â†‘ consolidation, neg_feedback â†’ â†“ consolidation
- [ ] Memory persists across â‰¥ 2 conversation turns
- [ ] All three invariants remain unchanged across operating mode switches
- [ ] Tests pass for all supported modes
- [ ] Non-goals explicitly documented (e.g., "not claiming consciousness")
- [ ] Degrees of freedom tested (e.g., adding new task head doesn't break NLLM properties)

---

## 14. Document Lifecycle

- **Created:** January 2026
- **Version:** v1.2
- **Status:** Locked (design contract)
- **Next Review:** When adding a component that touches invariants
- **Breaking Changes Protocol:** Any change to core invariants requires new version (v1.3, etc.)

This document is your design contract. Keep it stable. Extend from it; don't redefine it.
