# QNLLM Neuron Definition (Formal & Non-Negotiable)

**Version:** 2.0-quantum 
**Date:** 2026-01-18 
**Status:** Locked Definition

---

## 1. Formal Neuron Definition

### Unified Neuron Model

A neuron in QNLLM is a **stateful computational unit** with three components:

$$\text{Neuron} = \{\text{State}, \text{Update Rule}, \text{Learning Influence}\}$$

#### 1.1 State (Classical)

Each neuron maintains:

- **Membrane Potential** $V_m(t) \in \mathbb{R}$ (scalar, mV)
 - Range: $[-\infty, +\infty]$ (practically $[-100, +50]$ mV)
 - Initialized: $V_m(0) = -70$ mV (rest potential)
 - Decays: $V_m(t+1) = 0.99 \cdot V_m(t)$ (leak conductance)

- **Synaptic state variables** $\mathbf{w} \in \mathbb{R}^{D}$ (vector, unitless)
 - Dimension: $D = 768$ (input encoding dimension)
 - Initialized: $\mathbf{w} \sim \mathcal{N}(0, 0.01^2)$
 - Updated: via Hebbian learning

- **Firing State** $s(t) \in \{0, 1\}$ (binary spike)
 - Refractory period: 2 timesteps post-spike

#### 1.2 State (Quantum)

In hybrid quantum-classical mode, each neuron is paired with:

- **Quantum Superposition** $|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$ per neuron
 - $\alpha, \beta \in \mathbb{C}$, normalized: $|\alpha|^2 + |\beta|^2 = 1$
 - 8 qubits per quantum neuron (default, configurable)
 - Implements superposition, entanglement, quantum gates

#### 1.3 Update Rule (Classical)

**Integrate-and-Fire Dynamics:**

$$V_m(t+1) = 0.99 \cdot \left[ V_m(t) + \langle \mathbf{w}, \mathbf{x}(t) \rangle \right]$$

where:
- $\mathbf{x}(t) \in \mathbb{R}^{768}$ is the input vector
- $\langle \cdot, \cdot \rangle$ is the inner product (synaptic integration)
- $0.99$ is the leak coefficient

**Spike Generation:**

$$s(t) = \begin{cases} 1 & \text{if } V_m(t) > \theta = -55\text{ mV} \\ 0 & \text{otherwise} \end{cases}$$

**Post-Spike Reset:**

If $s(t) = 1$:
$$V_m(t+1) \leftarrow -70\text{ mV (rest potential)}$$

#### 1.4 Learning Influence (state variables Update)

**Hebbian Learning Rule:**

$$\mathbf{w}(t+1) = \mathbf{w}(t) + \eta \cdot e(t) \cdot \mathbb{1}[\text{spike}]$$

where:
- $\eta = 0.01$ (gating threshold)
- $e(t) \in [-0.1, +0.1]$ (error signal from MTL feedback or supervised learning)
- Learning occurs **only post-spike** (Hebbian principle)

---

## 2. Type Classification

### 2.1 Classical Neuron

- **State:** $(V_m, \mathbf{w})$ only
- **Operations:** receive_input(), fire(), reset(), learn()
- **Overhead:** ~32 bytes (float $V_m$) + 768 × 8 bytes (state variables) = **6.2 KB/neuron**

### 2.2 Quantum Neuron

- **State:** $(V_m, \mathbf{w}, \{\alpha_i, \beta_i\}_{i=1}^{8})$
- **Operations:** classical ops + apply_hadamard(), apply_cnot(), apply_phase_gate(), quantum_process()
- **Overhead:** 6.2 KB + 8 × 16 bytes (2 complex per qubit) = **6.3 KB/neuron**

### 2.3 Sparse Neuron (Brain-Scale)

- **State:** $(V_m, \text{sparse\_weights}, \text{active\_indices})$
- **Operations:** same as classical, but with sparse matrix multiplication
- **Sparsity:** 0.1% (1,000 active state variables per ~100M-neuron layer)
- **Overhead:** ~40 bytes (sparse index + references)

---

## 3. Formal Addressability

### Neuron Identity

Each neuron has a **unique global ID**:

$$\text{NeuronID} \in [0, N-1], \quad N \in \{896, 10^9, 10^{11}\}$$

Mapping to storage (see **NEURON_CAPACITY_ANALYSIS.md**):

- **Classical standard scale:** $N = 896$, direct array indexing
- **Brain-scale (100B):** $N = 100 \times 10^9$, hierarchical virtual indexing
- **Quantum-scale:** $N = 100 \times 10^9$, sparse address space

---

## 4. Non-Negotiable Constraints

1. **Every neuron must have a unique ID** (addressable)
2. **Every neuron must have an update rule** (deterministic dynamics)
3. **Every neuron must influence learning** (Hebbian or error-driven)
4. **No neuron is implicit or collapsed** (all are explicit in accounting)
5. **Spikes are events, not states** (binary, transient, measurable)

---

## 5. Scope Limitation

This definition applies to:
- Classical spiking neurons (NeuronEngine)
- Quantum-enhanced neurons (QuantumNeuron)
- Sparse neuron layers (BrainScaleLayer)

This definition **does not** apply to:
- deterministic processor attention heads (not neurons, different abstraction)
- RNN hidden states (different update rule)
- Autonomous Processor tokens (different semantics)

---

**Signature:** NEURON_DEFINITION v2.0-quantum
