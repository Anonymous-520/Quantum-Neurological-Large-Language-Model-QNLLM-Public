# QNLLM v2.2 - Teaching Package

## Table of Contents

1. [Tutorial Series](#tutorial-series)
2. [Workshop Materials](#workshop-materials)
3. [Course Syllabus](#course-syllabus)
4. [Student Exercises](#student-exercises)
5. [Video Scripts](#video-scripts)

---

## Tutorial Series

### Beginner Track (3 hours)

#### Tutorial 1: Introduction to QNLLM (30 min)
**Prerequisites:** Basic Python, NumPy 
**Learning Objectives:**
- Understand the 9 QNLLM invariants
- Learn the difference between gated learning and gradient accumulation
- Run your first QNLLM example

**Hands-On Lab:**
```python
# tutorial_1_intro.py
from scripts.qnllm_unified_api import QNLLMFactory

# Create your first QNLLM instance
print("=== Tutorial 1: Introduction to QNLLM ===\n")

# Step 1: Create a CodeLearn variant
qnllm = QNLLMFactory.create("codelearn", input_dim=50, output_dim=5)
print("Created CodeLearn variant")

# Step 2: Train on a simple example
error, gate_open = qnllm.train([0.1]*50, "function")
print(f"configuration: error={error:.3f}, gate_open={gate_open}")

# Step 3: Query the model
prediction = qnllm.query([0.1]*50)
print(f"Prediction: {prediction}")

# Step 4: Check invariants
print(f"\nInvariants Check:")
print(f"1. Gate duty cycle: {qnllm.gate_open_count/qnllm.step:.2%}")
print(f"2. Memory size: {len(qnllm.memory)}")
print(f"3. Total steps: {qnllm.step}")
```

**Key Concepts:**
- Adaptive gate (θ_high=0.65, θ_low=0.45)
- Error-driven updates (only learns when uncertain)
- Memory consolidation (fast → medium → slow)

**Quiz:**
1. What are the 9 QNLLM invariants?
2. Why does the gate have hysteresis (θ_high ≠ θ_low)?
3. What happens when uncertainty < θ_low?

---

#### Tutorial 2: Understanding the Four Variants (45 min)
**Prerequisites:** Tutorial 1 
**Learning Objectives:**
- Compare CodeLearn, Strategy, Multimodal, MultiAgent
- Choose the right variant for your task
- Understand complexity trade-offs

**Hands-On Lab:**
```python
# tutorial_2_variants.py
from scripts.qnllm_unified_api import QNLLMFactory

print("=== Tutorial 2: Four Variants Comparison ===\n")

# Create all 4 variants
variants = {
 "codelearn": QNLLMFactory.create("codelearn"),
 "strategy": QNLLMFactory.create("strategy"),
 "multimodal": QNLLMFactory.create("multimodal"),
 "multiagent": QNLLMFactory.create("multiagent")
}

# Train on same example
example = [0.1]*50
label = "test"

for name, qnllm in variants.items():
 error, gate = qnllm.train(example, label)
 print(f"{name:12s}: error={error:.3f}, gate={gate}")

# Compare capabilities
print("\nCapability Comparison:")
comparison = QNLLMFactory.compare_variants()
for variant, info in comparison.items():
 print(f" {variant}: {info['use_cases']}")
```

**Decision Tree:**
```
Your Task?
├─ Code analysis → CodeLearn (O(n))
├─ Strategic planning → Strategy (O(n·s))
├─ Mixed domains → Multimodal (O(d·n))
└─ Distributed learning → MultiAgent (O(m·d))
```

**Quiz:**
1. Which variant handles multi-domain data?
2. What's the complexity of MultiAgent with 5 agents and 3 domains?
3. When should you use Strategy over CodeLearn?

---

#### Tutorial 3: configuration and Querying (45 min)
**Prerequisites:** Tutorial 1-2 
**Learning Objectives:**
- Train QNLLM on custom datasets
- Understand gate dynamics
- Interpret query results

**Hands-On Lab:**
```python
# tutorial_3_training.py
from scripts.qnllm_unified_api import QNLLMFactory
import numpy as np

print("=== Tutorial 3: configuration and Querying ===\n")

# Create instance
qnllm = QNLLMFactory.create("codelearn")

# configuration dataset
train_data = [
 ([0.1]*50, "function"),
 ([0.2]*50, "loop"),
 ([0.3]*50, "class"),
 ([0.4]*50, "import"),
 ([0.5]*50, "if_else")
]

# Train with gate tracking
gate_history = []
for i, (x, y) in enumerate(train_data):
 error, gate = qnllm.train(x, y)
 gate_history.append(gate)
 print(f"Step {i+1}: {y:8s} → error={error:.3f}, gate={'OPEN' if gate else 'CLOSED'}")

# Query on new examples
test_data = [
 [0.12]*50, # close to "function"
 [0.25]*50, # close to "loop"
]

print("\nQuery Results:")
for i, x in enumerate(test_data):
 pred = qnllm.query(x)
 print(f"Test {i+1}: predicted '{pred}'")

# Gate statistics
print(f"\nGate Statistics:")
print(f" Duty cycle: {sum(gate_history)/len(gate_history):.2%}")
print(f" Total opens: {sum(gate_history)}/{len(gate_history)}")
```

**Key Insights:**
- Gate opens when uncertainty > θ_high (0.65)
- Gate closes when uncertainty < θ_low (0.45)
- Hysteresis prevents oscillation

**Exercise:**
Modify the code to track error over time and plot the learning curve.

---

#### Tutorial 4: Memory Consolidation (45 min)
**Prerequisites:** Tutorial 1-3 
**Learning Objectives:**
- Understand multi-timescale memory
- Observe consolidation process
- Measure forgetting

**Hands-On Lab:**
```python
# tutorial_4_memory.py
from scripts.qnllm_unified_api import QNLLMFactory

print("=== Tutorial 4: Memory Consolidation ===\n")

qnllm = QNLLMFactory.create("codelearn")

# Train 10 examples
for i in range(10):
 qnllm.train([i*0.1]*50, f"example_{i}")

# Check memory tiers
print("Memory Distribution:")
print(f" Fast tier (max 20): {len(qnllm.memory_fast)}")
print(f" Medium tier (max 100): {len(qnllm.memory_medium)}")
print(f" Slow tier (max 500): {len(qnllm.memory_slow)}")

# Trigger consolidation
qnllm.consolidate_memory()

print("\nAfter Consolidation:")
print(f" Fast tier: {len(qnllm.memory_fast)}")
print(f" Medium tier: {len(qnllm.memory_medium)}")
print(f" Slow tier: {len(qnllm.memory_slow)}")
```

**Quiz:**
1. What triggers memory consolidation?
2. How many memory tiers does QNLLM have?
3. What happens to old memories?

---

#### Tutorial 5: API Server Deployment (15 min)
**Prerequisites:** Tutorial 1-4, Docker installed 
**Learning Objectives:**
- Deploy QNLLM as REST API
- Send HTTP requests
- Monitor server health

**Hands-On Lab:**
```bash
# Start server
cd deployment
docker-compose up -d

# Check health
curl http://localhost:8000/health

# Create instance
curl -X POST http://localhost:8000/create \
 -H "Content-Type: application/json" \
 -d '{"variant":"codelearn"}'

# Train
curl -X POST http://localhost:8000/train \
 -H "Content-Type: application/json" \
 -d '{"instance_id":"ID","data":[{"input":[0.1],"label":"test"}]}'

# Query
curl -X POST http://localhost:8000/query \
 -H "Content-Type: application/json" \
 -d '{"instance_id":"ID","input":[0.1]}'
```

**Production Checklist:**
- [ ] HTTPS enabled
- [ ] Authentication configured
- [ ] Rate limiting active
- [ ] Monitoring dashboard
- [ ] Backup strategy

---

### Intermediate Track (4 hours)

#### Tutorial 6: Hybrid Systems (60 min)
**Prerequisites:** Beginner track 
**Learning Objectives:**
- Combine Multimodal + MultiAgent
- Implement cross-domain learning
- Analyze consensus emergence

**Hands-On Lab:**
```python
# tutorial_6_hybrid.py
from scripts.qnllm_unified_api import QNLLMFactory

print("=== Tutorial 6: Hybrid System ===\n")

# Create 3 agents
agents = [
 QNLLMFactory.create("multimodal", output_dim=10),
 QNLLMFactory.create("multimodal", output_dim=10),
 QNLLMFactory.create("multimodal", output_dim=10)
]

# Train on 3 domains
domains = ["code", "strategy", "language"]

for domain in domains:
 for i in range(10):
 example = [i*0.1]*50
 label = f"{domain}_{i}"

 # Each agent learns
 for agent in agents:
 agent.train(example, label)

# Check consensus
test_example = [0.25]*50
predictions = [agent.query(test_example) for agent in agents]

print("Agent Predictions:")
for i, pred in enumerate(predictions):
 print(f" Agent {i+1}: {pred}")

# Calculate agreement
agreement = len(set(predictions)) == 1
print(f"\nConsensus: {'YES' if agreement else 'NO'}")
```

**Exercise:**
Implement state variablesed voting (state variables by agent confidence).

---

#### Tutorial 7: Real-World Benchmarks (45 min)
**Prerequisites:** Tutorials 1-6 
**Learning Objectives:**
- Benchmark QNLLM on real tasks
- Measure accuracy, latency, memory
- Compare with baseline methods

**Hands-On Lab:**
Run the benchmark suite and analyze results.
```bash
python scripts/benchmark_real_world.py
```

**Metrics:**
- Accuracy (exact match, keyword match)
- Decision quality (reasoning depth)
- Integration quality (domains covered)
- Consensus rate (high agreement)

---

#### Tutorial 8: Research Extensions (60 min)
**Prerequisites:** Tutorials 1-7 
**Learning Objectives:**
- Add task routings
- Implement hierarchical consolidation
- Design curriculum schedules

**Hands-On Lab:**
```python
# tutorial_8_extensions.py
from scripts.research_extensions import (
 QNLLMWithAttention,
 QNLLMWithHierarchy,
 QNLLMWithCurriculum
)

# Attention
qnllm_attn = QNLLMWithAttention()
qnllm_attn.train_with_attention([...], "example")

# Hierarchy
qnllm_hier = QNLLMWithHierarchy()
qnllm_hier.consolidate_hierarchical()

# Curriculum
qnllm_curr = QNLLMWithCurriculum()
easy, medium, hard = [...], [...], [...]
qnllm_curr.train_with_curriculum([easy, medium, hard])
```

**Warning:** These are experimental features (not production-ready).

---

#### Tutorial 9: Production Optimization (45 min)
**Prerequisites:** Tutorials 1-8 
**Learning Objectives:**
- Profile memory usage
- Optimize latency
- Scale horizontally

**Hands-On Lab:**
```bash
# Profile performance
python scripts/performance_profile.py

# Load test
python deployment/load_test.py

# Scale
docker-compose up -d --scale qnllm-api=5
```

---

### Advanced Track (3 hours)

#### Tutorial 10: Custom Variants (90 min)
**Prerequisites:** Intermediate track 
**Learning Objectives:**
- Design new QNLLM variants
- Implement custom routing logic
- Validate invariants

**Assignment:**
Create a `QNLLMTimeSeries` variant for forecasting.

---

#### Tutorial 11: Distributed configuration (45 min)
**Prerequisites:** Tutorials 1-10 
**Learning Objectives:**
- Split configuration across multiple nodes
- Aggregate gradients
- Handle failures

---

#### Tutorial 12: Advanced Integration (45 min)
**Prerequisites:** Tutorials 1-11 
**Learning Objectives:**
- Integrate with existing Deterministic Processing pipelines
- Export/import models
- Version control for QNLLM

---

## Workshop Materials

### Half-Day Workshop (4 hours)

**Agenda:**
- 09:00-09:30: Introduction to QNLLM (slides)
- 09:30-10:30: Tutorial 1-2 (hands-on)
- 10:30-10:45: Break
- 10:45-11:45: Tutorial 3-4 (hands-on)
- 11:45-12:30: Tutorial 6 (hybrid systems)
- 12:30-13:00: Q&A and wrap-up

**Materials Needed:**
- Laptops with Python 3.8+
- Docker installed (for Tutorial 5)
- GitHub repository cloned
- Slides (see `workshops/slides.pdf`)

---

### Full-Day Workshop (8 hours)

**Agenda:**
- 09:00-12:00: Beginner track (Tutorials 1-5)
- 12:00-13:00: Lunch
- 13:00-16:00: Intermediate track (Tutorials 6-9)
- 16:00-16:15: Break
- 16:15-17:00: Advanced demos (Tutorial 10-12)
- 17:00-18:00: Capstone project

**Capstone Project:**
Build a custom QNLLM variant for your domain (code, finance, robotics, etc.)

---

## Course Syllabus

### CS 599: Advanced Learning Systems with QNLLM (Graduate Level)

**Instructor:** (TBD) 
**Credits:** 3 
**Prerequisites:** Deterministic Processing (CS 229), Linear Algebra (MATH 51)

**Course Description:**
This course covers the theory and practice of Quantum-Neurological Learning Models (QNLLM), a new paradigm for measurable, gated learning. Students will learn the 9 invariants, implement specialized variants, and deploy production systems.

**Learning Outcomes:**
1. Explain the difference between gated learning and gradient accumulation
2. Implement all 4 QNLLM variants from scratch
3. Design custom variants for specific domains
4. Deploy QNLLM as production API
5. Conduct original research extending QNLLM

**Grading:**
- Homework (40%): 4 assignments
- Midterm (20%): Theory exam
- Final Project (30%): Original research
- Participation (10%): Class discussions

**Schedule (10 weeks):**

**Week 1:** Introduction to QNLLM
- Reading: QNLLM_MASTER.md, NEURON_DEFINITION.md
- Homework 1: Implement basic QNLLM

**Week 2:** The Nine Invariants
- Reading: QNLLM_FOR_ARXIV.md (Sections 3-4)
- Lab: Validate all invariants

**Week 3:** CodeLearn and Strategy Variants
- Reading: QNLLM_VARIANTS_GUIDE.md
- Homework 2: Implement CodeLearn from scratch

**Week 4:** Multimodal and MultiAgent Variants
- Reading: demo_qnllm_multimodal.py, demo_qnllm_multiagent.py
- Lab: Build hybrid system

**Week 5:** Midterm Exam
- Topics: Invariants, variants, complexity analysis

**Week 6:** Real-World Applications
- Reading: benchmark_real_world.py
- Homework 3: Benchmark on custom dataset

**Week 7:** Production Deployment
- Reading: DEPLOYMENT_GUIDE.md
- Lab: Deploy API server with Docker

**Week 8:** Research Extensions
- Reading: research_extensions.py
- Homework 4: Implement task routing

**Week 9:** Custom Variants
- Guest lecture: Industry applications
- Project: Design domain-specific variant

**Week 10:** Final Presentations
- Student projects (15 min each)
- Peer review and feedback

---

## Student Exercises

### Exercise 1: Gate Dynamics (Beginner)
**Objective:** Understand hysteresis-based gating

**Task:**
1. Create a QNLLM instance
2. Train on 20 examples
3. Plot gate state (open/closed) over time
4. Calculate gate duty cycle

**Expected Output:**
- Gate duty cycle: 60-80%
- Gate opens when error > θ_high
- Gate closes when error < θ_low

**Solution:**
```python
# exercise_1_solution.py
import matplotlib.pyplot as plt
from scripts.qnllm_unified_api import QNLLMFactory

qnllm = QNLLMFactory.create("codelearn")

gate_history = []
for i in range(20):
 _, gate = qnllm.train([i*0.05]*50, f"ex_{i}")
 gate_history.append(1 if gate else 0)

plt.plot(gate_history)
plt.xlabel("Step")
plt.ylabel("Gate State")
plt.title("Gate Dynamics")
plt.show()

print(f"Duty cycle: {sum(gate_history)/len(gate_history):.2%}")
```

---

### Exercise 2: Variant Selection (Intermediate)
**Objective:** Choose the right variant for a task

**Task:**
You have 4 datasets:
1. Python code snippets (1000 examples)
2. Chess games (500 positions)
3. Medical images + text reports (200 pairs)
4. Distributed sensor readings (10 sensors, 1000 readings each)

Which QNLLM variant would you use for each? Justify.

**Solution:**
1. **CodeLearn** - Syntax-aware routing for code
2. **Strategy** - Multi-level decisions for chess
3. **Multimodal** - Cross-domain (image + text)
4. **MultiAgent** - Distributed consensus across sensors

---

### Exercise 3: Memory Consolidation (Advanced)
**Objective:** Implement custom consolidation strategy

**Task:**
Modify `consolidate_memory()` to use importance-state variablesed consolidation (keep high-importance memories longer).

**Starter Code:**
```python
def consolidate_memory_weighted(self):
 # Assign importance scores
 for mem in self.memory_fast:
 mem['importance'] = ... # Your code here

 # Sort by importance
 sorted_mem = sorted(self.memory_fast, key=lambda m: m['importance'], reverse=True)

 # Move to medium tier (keep top 50%)
 self.memory_medium.extend(sorted_mem[:len(sorted_mem)//2])
 self.memory_fast = sorted_mem[len(sorted_mem)//2:]
```

---

## Video Scripts

### Video 1: "What is QNLLM?" (5 min)

**Script:**

[INTRO]
"Hi, I'm [Name], and today I'm going to show you QNLLM - Quantum-Neurological Learning Models.

QNLLM is a new approach to Deterministic Processing that proves learning is a measurable gated process, not just gradient accumulation.

[PROBLEM]
Traditional deterministic networks update state variables on every example, even when they're confident. This leads to:
- Catastrophic forgetting
- Overconfidence
- Unexplainable decisions

[SOLUTION]
QNLLM uses an adaptive gate that only learns when uncertain. When uncertainty exceeds θ_high (0.65), the gate opens. When uncertainty drops below θ_low (0.45), the gate closes.

[DEMO]
Let me show you a quick example... [screen recording]

[CODE]
```python
qnllm = QNLLMFactory.create("codelearn")
qnllm.train([0.1]*50, "function")
prediction = qnllm.query([0.1]*50)
```

[INVARIANTS]
QNLLM satisfies 9 invariants - measurable properties that all instances must have:
1. Gated learning
2. Error-driven plasticity
3. Multi-timescale memory
... (continue list)

[OUTRO]
Want to learn more? Check out our GitHub repository and full documentation. Thanks for watching!"

---

### Video 2: "Four QNLLM Variants Explained" (8 min)
(Script similar structure...)

---

### Video 3: "Deploying QNLLM in Production" (10 min)
(Script similar structure...)

---

**Version:** 2.2 
**Status:** Ready for Teaching 
**Last Updated:** January 19, 2026
