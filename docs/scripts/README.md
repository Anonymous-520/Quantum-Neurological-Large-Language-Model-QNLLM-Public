# Scripts

**Interactive chat demos and utility scripts for QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM**

---

## Chat Scripts

### chat_ai.ps1 (Recommended)
**Advanced PowerShell chat with reasoning**

Features:
- Mathematical reasoning (evaluates expressions)
- Knowledge base matching (Deterministic Processing, Autonomous System, deterministic networks)
- Multi-region deterministic processing simulation (MTL-3)
- Quality/agreement/confidence scoring
- Session logging

Usage:
```powershell
powershell -ExecutionPolicy Bypass -File scripts/chat_ai.ps1
```

Commands:
- `/exit` - Exit chat
- `/help` - Show help
- `/stats` - Show statistics
- `/clear` - Clear screen

Example queries:
- "2 x 2 + 6" â†’ Mathematical reasoning
- "What is Deterministic Processing?" â†’ Knowledge lookup
- "Hello" â†’ Greeting response

### nllm_chat.ps1
**Brain-inspired chat system (v1.1)**

Features:
- deterministic architecture visualization
- Cortex region activation simulation
- Synaptic learning messages
- Session logging with timestamps

Usage:
```powershell
powershell -ExecutionPolicy Bypass -File scripts/nllm_chat.ps1
```

### nllm.ps1
**Original chat interface (v1.0)**

Features:
- Basic deterministic pathway simulation
- Simple pattern matching
- Lightweight implementation

Usage:
```powershell
powershell -ExecutionPolicy Bypass -File scripts/nllm.ps1
```

### chat_interactive.bat
**Batch file wrapper for C++ chat**

Features:
- Launches C++ example_usage.exe
- Conversation logging
- Keyword-based responses
- Windows command-line interface

Requirements:
- C++ build must be complete
- `cpp/build/Release/example_usage.exe` must exist

Usage:
```cmd
scripts\chat_interactive.bat
```

---

## Script Comparison

| Script | Language | Complexity | Features | Best For |
|--------|----------|------------|----------|----------|
| **chat_ai.ps1** | PowerShell | High | Math reasoning, rich responses | Demos, testing |
| **nllm_chat.ps1** | PowerShell | Medium | deterministic simulation | Educational |
| **nllm.ps1** | PowerShell | Low | Simple chat | Lightweight testing |
| **chat_interactive.bat** | Batch + C++ | Medium | C++ integration | Production testing |

---

## Output Logs

All chat scripts save logs to:
```
logs/autonomous_outputs/chat_session_YYYYMMDD_HHMMSS.log
```

Log format:
```
[HH:MM:SS] User: <message>
[HH:MM:SS] NLLM Response: <response>
Quality: X.XXX | Agreement: X.XXX | Confidence: X.XXX
```

---

## Notes

**These are demo scripts, not production systems**
- Pattern matching only (no real Autonomous Processor processing)
- Hardcoded responses for common queries
- Useful for:
 - Testing chat interface flow
 - Demonstrating MTL-3 architecture
 - Logging conversation patterns
 - Educational demos

**For production:**
- Use `test_reasoning_real_nim.py` with NVIDIA NIM
- See `docs/05-Deployment/DEPLOYMENT_GUIDE.md`

---

**Last Updated:** January 15, 2026 (v1.4 finalization)
