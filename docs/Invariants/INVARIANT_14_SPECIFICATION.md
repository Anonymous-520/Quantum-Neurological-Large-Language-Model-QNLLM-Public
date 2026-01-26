# Invariant 14: Bounded Task Resumption

**Version:** 2.5 (PATH 3)
**Status:** Formal Specification
**Date:** 2026-01-22

---

## Definition

**Invariant 14** states that the QNLLM system can:

1. **Maintain Long-Horizon Context**: Store incomplete tasks with full state
2. **Resume Tasks After Interruption**: Pick up where it left off with bounded drift
3. **Forget Abandoned Goals**: Clean up interrupted tasks with forgetting schedule

**Core Statement:**
```
∀ task t, interruption event i:

 If task t interrupted:
 1. Save complete state (context, progress, confidence)
 2. Allow resumption at later time
 3. Drift(resumed_task) ≤ MAX_DRIFT
 4. After N interruptions, forget task
```

---

## Three Core Capabilities

### 1. Task Queue Memory

**Purpose:** Store incomplete tasks with full provenance

**Data Structure:**
```
TaskQueueEntry:
 - task_id: UUID
 - goal: str (what's the objective?)
 - state: dict (current progress)
 - memories_involved: List[int] (which memories used)
 - confidence: float (0-1)
 - created_at: timestamp
 - last_resumed_at: timestamp
 - interruption_count: int
 - metadata: dict (audit trail)
```

**Guarantees:**
- All tasks recoverable (complete state saved)
- Provenance preserved (audit trail)
- Bounded storage (max N concurrent tasks)
- FIFO ordering (first interrupted, first resumed)

---

### 2. Goal Resumption Logic

**Purpose:** Resume interrupted tasks with bounded drift

**Resumption States:**
```
PENDING → ACTIVE → INTERRUPTED → RESUMED → COMPLETED
 ↓ ↓
 FAILED MAX_DRIFT_EXCEEDED
```

**Bounded Drift Formula:**
```
For resumed task r at time T_resume (originally at T_original):

 time_elapsed = T_resume - T_original

 context_drift = sum of confidence decrements
 = interruption_count × 0.05 (5% per interruption)

 Constraint: context_drift ≤ 0.30 (30% max)

 If drift > 30%:
 → Mark for forgetting (next cycle)
 → Return INTERRUPTED status (human decision)
```

**Resumption Guarantee:**
```
∀ resumed_task:
 confidence(resumed_task_t) ≥ confidence(original) - drift_amount
```

---

### 3. Forgetting of Abandoned Goals

**Purpose:** Clean up interrupted tasks (memory hygiene)

**Forgetting Schedule:**
```
Interruption Count → Action:
 1st interruption → Store for 1 hour (short-term buffer)
 2nd interruption → Mark for extended storage (24 hours)
 3rd interruption → Alert user: "Goal being forgotten"
 4th interruption → Remove from active queue (archive)
 5th+ interruption → Purge from system (forgotten)
```

**Forgetting Constraints:**
- Tasks never silently deleted (user notified)
- Archived tasks recoverable for 7 days
- Provenance preserved in audit log
- User can pin tasks (prevent forgetting)

---

## Use Cases

### Use Case 1: Research Assistant

**Scenario:** User doing literature review, interrupted frequently

```
Session 1:
 Goal: "Summarize Deterministic State Machine papers on optimization"
 Task active for 15 minutes
 → User needs to check something else
 → System INTERRUPTS task (saves state)

Session 2 (2 hours later):
 User resumes: "Continue research on optimization"
 → System recovers state
 → Confidence: 0.85 → 0.80 (5% drift from 1 interruption)
 → Continues from exactly where it left off

Success: Long-horizon task completed across multiple sessions
```

### Use Case 2: Offline Agent

**Scenario:** System deployed without internet, battery-constrained

```
Mission:
 Goal: "Download and process sensor data daily"
 Run on raspberry pi (256MB RAM)
 Interrupted by: power loss, network outage, full storage

Behavior:
 1. Save task state before power loss
 2. Resume on reboot (within drift tolerance)
 3. Handle interruptions gracefully
 4. Forget low-priority goals to free memory

Success: System continues learning despite resource constraints
```

### Use Case 3: Collaborative Autonomous System

**Scenario:** Human + Autonomous System working together on complex project

```
Project: "Analyze customer feedback dataset"
Handoff pattern:
 - Autonomous System: Analyzes subset 1 (interrupted: human review needed)
 - Human: Provides feedback and corrections
 - Autonomous System: Resumes with updated context
 - Repeat for subsets 2, 3, 4...

Success: Continuous goal despite frequent human-Autonomous System handoffs
```

---

## Implementation Requirements

### Component 1: TaskQueueMemory Class

```python
class TaskQueueMemory:
 """Manages long-horizon task continuity."""

 def __init__(self, max_tasks: int = 100):
 self.task_queue: Dict[str, TaskQueueEntry] = {}
 self.max_tasks = max_tasks

 def create_task(self, goal: str, context: dict) -> str:
 """Create new task. Returns task_id."""

 def interrupt_task(self, task_id: str) -> bool:
 """Interrupt task, save state. Returns success."""

 def resume_task(self, task_id: str) -> (bool, dict):
 """Resume task. Returns (success, state_dict)."""

 def forget_task(self, task_id: str) -> bool:
 """Forget abandoned task. Returns success."""

 def get_resumable_tasks(self) -> List[str]:
 """Get list of tasks that can be resumed."""

 def get_task_state(self, task_id: str) -> dict:
 """Get current state of task."""
```

### Component 2: GoalTracker Class

```python
class GoalTracker:
 """Tracks goal progress and resumption."""

 def __init__(self, max_drift: float = 0.30):
 self.max_drift = max_drift
 self.active_goals: Dict[str, Goal] = {}

 def start_goal(self, goal: str) -> str:
 """Start tracking goal. Returns goal_id."""

 def interrupt_goal(self, goal_id: str) -> float:
 """Interrupt goal. Returns resulting drift."""

 def resume_goal(self, goal_id: str) -> (bool, float):
 """Resume goal. Returns (allowed, new_drift)."""

 def check_drift(self, goal_id: str) -> float:
 """Check current drift amount."""

 def mark_complete(self, goal_id: str) -> bool:
 """Mark goal as complete."""

 def abandon_goal(self, goal_id: str) -> bool:
 """Forget goal (stop tracking)."""
```

### Component 3: Bounded Drift Verifier

```python
class DriftVerifier:
 """Verifies resumed tasks don't exceed drift bounds."""

 def calculate_drift(self, task: TaskQueueEntry) -> float:
 """Calculate total drift for task."""

 def is_resumable(self, task: TaskQueueEntry) -> bool:
 """Check if task can be safely resumed."""

 def get_resumption_penalty(self, task: TaskQueueEntry) -> float:
 """Get confidence penalty for resumption."""
```

---

## Formal Guarantees

### Guarantee 1: Task State Recovery

**Claim:** All interrupted tasks can be recovered with complete state

**Proof:**
- All state saved to TaskQueueEntry (deterministic)
- State includes: goal, context, memories, confidence, metadata
- Audit trail tracks all state changes

**Test:** 
- Interrupt task at random point
- Resume task
- Verify state identical to before interruption

---

### Guarantee 2: Bounded Drift

**Claim:** Resumed tasks never exceed 30% drift from original

**Formula:**
```
max_drift = interruption_count × 0.05
bound = 0.30 (30%)

If max_drift > 0.30:
 → Cannot resume (user must start fresh)
```

**Test:**
- Create task, interrupt 1 time → 5% drift (resumable)
- Create task, interrupt 6 times → 30% drift (can resume boundary)
- Create task, interrupt 7 times → 35% drift (NOT resumable)

---

### Guarantee 3: Graceful Forgetting

**Claim:** Abandoned goals never deleted silently

**Pattern:**
```
Interruption 1: "Task interrupted" (store)
Interruption 2: "Task still available" (extended store)
Interruption 3: "Warning: goal being forgotten soon"
Interruption 4: "Goal archived (7-day recovery window)"
Interruption 5: "Goal forgotten (permanent)"
```

**Test:**
- Interrupt task 5+ times
- Verify: user warned at step 3, task archived at step 4, forgotten at step 5
- Verify: audit trail preserved

---

## Test Strategy (8 Tests)

### Test 1: Task Queue Creation
- Create 10 tasks
- Verify all stored with unique IDs
- Verify max_tasks limit respected

### Test 2: State Preservation
- Create task with context
- Interrupt it
- Resume it
- Verify state identical (deterministic)

### Test 3: Bounded Drift (Resumable)
- Interrupt task 6 times (30% drift)
- Verify: resumable (at boundary)

### Test 4: Bounded Drift (Non-Resumable)
- Interrupt task 7 times (35% drift)
- Verify: NOT resumable (exceeds bound)

### Test 5: Task Forgetting Schedule
- Interrupt task 5 times
- Verify: warnings at interruptions 3, 4
- Verify: archived at interruption 4
- Verify: forgotten at interruption 5

### Test 6: Multiple Concurrent Tasks
- Create 5 concurrent tasks
- Interrupt all randomly
- Resume each
- Verify: all states correct, no cross-contamination

### Test 7: Human-in-Loop Handoff
- Autonomous System starts task
- Interruption (human review)
- Human updates context
- Autonomous System resumes with new context
- Verify: continuity with human feedback

### Test 8: Formal Invariant 14 Verification
- 20 random task scenarios
- All interruptions handled correctly
- All drift guarantees met
- All resumptions deterministic
- Verify: 0 violations

---

## Success Criteria

 Invariant 14 is verified when:
- All 8 tests pass (100%)
- Task state perfectly recoverable
- Drift never exceeds 30%
- Forgetting follows schedule (never silent)
- Audit trail complete for all operations
- Works with offline deployment (no internet required)

---

## Academic Impact

### Publication Angle

**Title:** *"Bounded Long-Horizon Reasoning: Task Resumption with Guaranteed Drift"*

**Key Claims:**
1. **First bounded resumption system** - Drift has hard limit (30%)
2. **Explicit forgetting schedule** - No silent data loss
3. **Offline-compatible** - Works without internet
4. **Edge-deployable** - Runs on Raspberry Pi / IoT devices

### Conference Targets
- **ICLR** - Learning theory angle
- **ICML** - Multi-task learning
- **JMLR** - Theoretical guarantees
- **Edge Autonomous System Workshop** - Resource-constrained deployment

---

*Version: 2.5 Draft*
*Status: Specification Complete*
*Next: Implement TaskQueueMemory, GoalTracker, test_invariant_14_autonomy.py*
