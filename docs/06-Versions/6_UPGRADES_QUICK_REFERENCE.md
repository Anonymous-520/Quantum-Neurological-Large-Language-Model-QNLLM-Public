# 6 Upgrades: Quick Reference

## At a Glance

| # | Name | File | Lines | Purpose |
|---|------|------|-------|---------|
| 1 | Virtual Neurons | `virtual_neurons.py` | 600 | 100B addresses, O(1) lookup, implicit zeros |
| 2 | Event-Driven | `event_driven.py` | 500 | Spike queue, O(active) processing, skip silent |
| 3 | Hierarchical | `hierarchical_learning.py` | 600 | Region→Assembly→Neuron, sparse state variables updates |
| 4 | Learning Gating | `hierarchical_learning.py` | 400 | Conditional learning, error/disagreement/novelty signals |
| 5 | Reasoning Control | `reasoning_control_enforce.py` | 550 | Activation masking, depth budgets, separation of concerns |
| 6 | Hypothesis Mgmt | `hypothesis_management.py` | 600 | Superposition/collapse, Bayesian updates, entropy |

---

## When to Use Each Upgrade

### Upgrade 1: Virtual Neurons
```python
from src.core.cortex.virtual_neurons import VirtualNeuronStore

# Create a 100B-neuron addressable space with 10k max active
store = VirtualNeuronStore(
 max_virtual_neurons=100_000_000_000,
 max_active=10_000
)

# Lazy instantiate when needed
neuron = store.get_or_create(neuron_id=42)

# Memory: ~40 bytes per active neuron, not per virtual
```

**Use when:**
- You need brain-scale addressability
- Only a small fraction are active
- You can't afford dense allocation

---

### Upgrade 2: Event-Driven Execution
```python
from src.core.cortex.event_driven import EventDrivenEngine

engine = EventDrivenEngine(num_neurons=1_000_000)

# Process only spikes, skip silent neurons
spike_events = {
 "neuron_100": 1.0,
 "neuron_542": 0.8,
 # ... only 1000 spikes, not 1M neurons
}

results = engine.process_events(spike_events) # O(active), not O(N)
```

**Use when:**
- Most neurons are silent
- Spike processing is the bottleneck
- You need O(active) complexity

---

### Upgrade 3: Hierarchical Groups
```python
from src.core.cortex.hierarchical_learning import HierarchicalNeuralSystem

system = HierarchicalNeuralSystem()

# Regions are automatically created:
# - Sensory: 100k neurons, 100 assemblies
# - Association: 200k neurons, 200 assemblies
# - Integration: 100k neurons, 100 assemblies
# - Motor: 50k neurons, 50 assemblies

# Forward pass activates region, tracks assembly activity
result = system.forward(inputs=input_vector, region_id=0)
print(result["stats"]) # Per-assembly statistics
```

**Use when:**
- You need organized neuron groups
- You want assembly-level learning
- You track activity per group

---

### Upgrade 4: Learning Gating
```python
from src.core.cortex.hierarchical_learning import LearningGate, LearningGateController

# Create gating controller
gate = LearningGateController(mode=LearningGate.ADAPTIVE)

# Check if learning should happen
enabled, signal = gate.should_enable_learning(
 prediction_error=0.5,
 mtl_disagreement=0.2,
 novelty_score=0.1
)

if enabled:
 learning_rate = 0.01 * signal # Modulate by signal strength
 # Update state variables
else:
 # Don't learn
 pass
```

**Use when:**
- You want conditional plasticity
- You have error/disagreement/novelty signals
- You want to prevent catastrophic drift

**Gating Modes:**
- `ALWAYS`: Always learn (default behavior)
- `PREDICTION_ERROR`: Learn only if |error| > threshold
- `DISAGREEMENT`: Learn only if teachers disagree
- `NOVELTY`: Learn only if input is novel
- `ADAPTIVE`: Combine all signals (recommended)

---

### Upgrade 5: Reasoning-as-Control
```python
from src.core.cortex.reasoning_control_enforce import ReasoningEnforcer

enforcer = ReasoningEnforcer(num_assemblies=100)

# Get reasoning decision
controlled_spikes, info = enforcer.enforce_control_flow(
 query="Is this a cat?",
 spike_mask=original_spikes,
 importance=0.8, # How important is this decision?
 urgency=0.5 # How fast must we decide?
)

# Key insight: reasoning ONLY modifies activation
# state variables are NEVER modified by reasoning
# Control is applied as mask, not state variables update
```

**Control Rules:**
- **High importance** → Activate many assemblies, deep processing
- **High urgency** → Activate few assemblies, shallow processing
- **Normal** → Moderate activation

**Use when:**
- You want reasoning to control flow, not neurons
- You need activation masking
- You want to enforce separation of concerns

---

### Upgrade 6: Hypothesis Management
```python
from src.core.quantum.hypothesis_management import CognitiveQuantumSystem

system = CognitiveQuantumSystem()

# Create hypothesis space (superposition)
space_id = system.perceive_and_hypothesize({
 "description": "Is this animal a cat or dog?",
 "possible_interpretations": ["cat", "dog", "fox", "other"]
})

# Process evidence (Bayesian-style update)
system.process_evidence(space_id, {
 "supports": [
 {"hypothesis_id": 0, "strength": 0.8}, # cat
 ],
 "contradicts": [
 {"hypothesis_id": 1, "strength": 0.6}, # dog
 ]
})

# Make decision (collapse superposition)
decision = system.make_decision(space_id, confidence=0.9)
print(f"Decision: {decision['chosen_hypothesis']}") # "cat"
print(f"Probability: {decision['probability']:.2%}") # 92%
```

**Use when:**
- You have multiple competing explanations
- You need Bayesian-style belief updates
- You want principled decision-making
- You need to track uncertainty

---

## Common Patterns

### Pattern 1: Complete Forward Pass
```python
from src.core.cortex.virtual_neurons import VirtualNeuronStore
from src.core.cortex.event_driven import EventDrivenEngine
from src.core.cortex.hierarchical_learning import HierarchicalNeuralSystem

# Initialize
store = VirtualNeuronStore()
engine = EventDrivenEngine(num_neurons=10_000_000)
system = HierarchicalNeuralSystem()

# Generate spikes in sensory region
for i in range(1000):
 neuron_id = np.random.randint(0, 100_000)
 engine.process_events({f"neuron_{neuron_id}": 1.0})

# Track assembly activity
result = system.forward(inputs=input_data, region_id=0)
```

### Pattern 2: Conditional Learning
```python
from src.core.cortex.hierarchical_learning import LearningGate, HierarchicalNeuralSystem

system = HierarchicalNeuralSystem(mode=LearningGate.ADAPTIVE)

# Forward pass
output = model.forward(input_data)
prediction_error = compute_error(output, target)

# Check gating
enabled, signal = system.learning_gate.should_enable_learning(
 prediction_error=prediction_error,
 mtl_disagreement=mtl_score,
 novelty_score=novelty
)

# Conditional state variables update
if enabled:
 learning_rate = 0.01 * signal
 optimizer.step(learning_rate)
```

### Pattern 3: Reasoning-Driven Computation
```python
from src.core.cortex.reasoning_control_enforce import ReasoningEnforcer

enforcer = ReasoningEnforcer(num_assemblies=100)

# Reasoning decides what to do
controlled_spikes, info = enforcer.enforce_control_flow(
 query=task_description,
 spike_mask=current_spikes,
 importance=task_importance,
 urgency=time_pressure
)

# Now compute with only active assemblies
result = deterministic_network(controlled_spikes)
```

### Pattern 4: Hypothesis-Based Reasoning
```python
from src.core.quantum.hypothesis_management import CognitiveQuantumSystem

system = CognitiveQuantumSystem()

# Create multiple interpretations
space = system.perceive_and_hypothesize({
 "description": input_query,
 "possible_interpretations": possible_answers
})

# Gather evidence from different sources
for evidence_source in evidence_sources:
 evidence = gather_evidence(evidence_source, space)
 system.process_evidence(space, evidence)

# Make final decision
decision = system.make_decision(space, confidence=confidence_level)
return decision["chosen_hypothesis"]
```

---

## Debugging Checklist

### Memory Issues?
- [ ] Check `VirtualNeuronStore.active_neurons` size (should be ~1% of virtual)
- [ ] Monitor `psutil.Process().memory_info().rss`
- [ ] Verify `max_active` setting in VirtualNeuronStore
- [ ] Profile with `tracemalloc`

### Speed Issues?
- [ ] Verify spike count (should be sparse, not dense)
- [ ] Check `EventDrivenEngine` queue length
- [ ] Profile event processing time
- [ ] Ensure `skip_silent_update()` is being called

### Learning Issues?
- [ ] Check learning gate statistics with `gate.get_stats()`
- [ ] Verify error/disagreement/novelty signals
- [ ] Monitor `state variables_modulation` per assembly
- [ ] Check if state variables are actually updating

### Reasoning Issues?
- [ ] Verify separation of concerns (no state variables updates in reasoning)
- [ ] Check `validation_passed` flag in control results
- [ ] Monitor active assembly fraction
- [ ] Verify depth budget allocation

### Hypothesis Issues?
- [ ] Check entropy of hypothesis space (should decrease over time)
- [ ] Verify Bayesian updates (probabilities should sum to 1)
- [ ] Monitor dominant hypothesis probability
- [ ] Ensure collapse is working

---

## Configuration Reference

### Virtual Neuron Store
```python
VirtualNeuronStore(
 max_virtual_neurons=100_000_000_000, # Addressable space
 max_active=100_000 # Physical limit
)
```

### Event-Driven Engine
```python
EventDrivenEngine(
 num_neurons=10_000_000,
 input_dim=768
)
```

### Hierarchical System
```python
HierarchicalNeuralSystem(
 input_dim=768,
 mode=LearningGate.ADAPTIVE # or ALWAYS, PREDICTION_ERROR, etc.
)

# Built-in regions:
# - Sensory: 100k neurons
# - Association: 200k neurons
# - Integration: 100k neurons
# - Motor: 50k neurons
```

### Learning Gate
```python
LearningGateController(
 mode=LearningGate.ADAPTIVE,
 error_threshold=0.1,
 disagreement_threshold=0.3,
 novelty_threshold=0.5
)
```

### Reasoning Enforcer
```python
ReasoningEnforcer(
 num_assemblies=100
)

# Decision rules:
# importance > 0.8: Deep processing (100% depth)
# urgency > 0.8: Fast processing (30% depth)
# else: Normal (60% depth)
```

### Hypothesis Manager
```python
HypothesisManager(
 max_concurrent_spaces=10
)

# Bayesian update multiplier:
# P(H) *= (1 + strength) if supports
# P(H) *= (1 - strength) if contradicts
```

---

## Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Virtual neurons | 100B | Fully addressable |
| Active neurons | 0.01% of virtual | ~100k for 10M |
| Memory per neuron | ~40 bytes | Only active neurons |
| Forward pass time | O(active) | Not O(N) |
| Learning gate | <5% overhead | Just a threshold check |
| Hypothesis management | <10ms per update | Usually <100 hypotheses |

---

## Testing Commands

```bash
# Run full integration test
python tests/test_integration_6upgrades.py

# Test individual upgrades
python -c "from src.core.cortex.virtual_neurons import VirtualNeuronStore; s = VirtualNeuronStore(); print(' Upgrade 1')"
python -c "from src.core.cortex.event_driven import EventDrivenEngine; e = EventDrivenEngine(1000); print(' Upgrade 2')"
python -c "from src.core.cortex.hierarchical_learning import HierarchicalNeuralSystem; h = HierarchicalNeuralSystem(); print(' Upgrade 3')"
python -c "from src.core.cortex.hierarchical_learning import LearningGateController; l = LearningGateController(); print(' Upgrade 4')"
python -c "from src.core.cortex.reasoning_control_enforce import ReasoningEnforcer; r = ReasoningEnforcer(); print(' Upgrade 5')"
python -c "from src.core.quantum.hypothesis_management import CognitiveQuantumSystem; c = CognitiveQuantumSystem(); print(' Upgrade 6')"

# Memory profiling
python -c "
import psutil
from src.core.cortex.virtual_neurons import VirtualNeuronStore
store = VirtualNeuronStore(max_virtual_neurons=10_000_000)
for i in range(100_000):
 store.get_or_create(i)
p = psutil.Process()
print(f'Memory: {p.memory_info().rss / 1e9:.2f} GB')
"
```

---

## Troubleshooting

**Q: Getting OOM errors?**
A: Reduce `max_active` in VirtualNeuronStore or verify you're using Upgrades 1-2.

**Q: Reasoning is modifying state variables?**
A: Use `ReasoningEnforcer.validate_separation()` to catch violations.

**Q: Learning never happens?**
A: Check learning gate statistics with `gate.get_stats()`, verify signal strength.

**Q: Hypothesis probabilities not updating?**
A: Verify `_normalize_probabilities()` is called after evidence, check evidence strength > 0.

**Q: Spike processing is slow?**
A: Profile with `%timeit` on event processing, ensure spike count is truly sparse.

---

## Next: Integration with Existing Code

The 6 upgrades are modular and can be integrated gradually:

1. **Replace neuron instantiation** with VirtualNeuronStore
2. **Replace forward loops** with EventDrivenEngine
3. **Wrap neuron groups** with HierarchicalNeuralSystem
4. **Add learning gate** to state variables update step
5. **Wrap reasoning layer** with ReasoningEnforcer
6. **Replace quantum layer** with HypothesisManager

No need to refactor all at once. Each upgrade adds value independently.
