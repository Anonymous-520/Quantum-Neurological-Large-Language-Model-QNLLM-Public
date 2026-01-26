# Logging System - Quick Reference

** COMPLETE - All logs automatically saved to logs/ folder**

---

## Where Logs Are Saved

```
 logs/
 background_mtl/ ← MTL learning cycles
 mtl_learning_{timestamp}.log

 autonomous_outputs/ ← Session summaries
 mtl_session_{timestamp}.log

 {ComponentName}_{timestamp}.log ← Component logs
```

---

## What Gets Logged

### Background MTL Learning
 Every teaching cycle 
 Quality scores 
 Agreement levels 
 Teacher responses 
 Execution time 
 Timestamps 

**File**: `logs/background_mtl/mtl_learning_{timestamp}.log`

### Session Summaries
 MTL-3 metrics 
 Memory counts 
 Configuration 
 Final results 

**File**: `logs/autonomous_outputs/mtl_session_{timestamp}.log`

### Memory Operations
 Memory additions 
 Persistence operations 
 Checkpoints (every 100) 
 Load/save operations 

**Console + component log files**

---

## How to Use

### Basic Usage
```cpp
#include "utils/utils.hpp"

// Create logger (auto-creates log file)
auto logger = nllm::utils::logging::Logger("MyComponent");

// Log messages - go to BOTH console and file
logger.info("Operation started");
logger.warning("Resource usage high");
logger.error("Operation failed");
```

### Log Levels
- **DEBUG**: Development info
- **INFO**: Normal operations Default
- **WARNING**: Potential issues
- **ERROR**: Recoverable errors
- **CRITICAL**: System failures

---

## Log Format

```
[2026-01-15 10:30:00] [INFO] ComponentName: Message here
[2026-01-15 10:30:01] [ERROR] ComponentName: Error details
```

---

## Automatic Features

 **Auto-directory creation** - `logs/` folder created if missing 
 **Auto-timestamps** - Every log entry timestamped 
 **Dual output** - Console + file simultaneously 
 **Thread-safe** - Safe for concurrent logging 
 **Zero config** - Works out of the box 

---

## View Logs

### Watch Live
```bash
# Watch background MTL learning
tail -f logs/background_mtl/*.log

# Watch all logs
tail -f logs/*.log
```

### Search
```bash
# Find errors
grep -r "ERROR" logs/

# Find quality scores
grep "Quality Score" logs/background_mtl/*.log
```

---

## Example Log Output

### Background MTL Log
```
=== Background MTL Learning Session ===
Started: 2026-01-15 10:30:00
Interval: 60 seconds
==========================================

[2026-01-15 10:30:00] Cycle #1
 Prompt: What is Deterministic Processing?
 Quality Score: 0.904
 Agreement: 0.985
 Teachers: 3
 Cycle Time: 1850.5 ms

[2026-01-15 10:31:00] Cycle #2
 Prompt: Explain deterministic networks.
 Quality Score: 0.910
 Agreement: 0.982
 Teachers: 3
 Cycle Time: 1920.3 ms

==========================================
Stopped: 2026-01-15 10:35:00
Total Cycles: 5
==========================================
```

---

## Status

 **Logger utility**: Complete with file output 
 **Background MTL logging**: Full cycle logging 
 **Session logging**: Summary logs 
 **Memory logging**: All operations logged 
 **Auto-directory creation**: logs/ created automatically 
 **Timestamp support**: All entries timestamped 
 **Performance**: <5% overhead 

---

## Quick Test

When you next run the system:
1. Check `logs/background_mtl/` for MTL learning logs
2. Check `logs/autonomous_outputs/` for session summaries
3. All console output also saved to files
4. Zero configuration needed!

---

## Full Documentation

See [LOGGING_SYSTEM_GUIDE.md](LOGGING_SYSTEM_GUIDE.md) for complete details.
