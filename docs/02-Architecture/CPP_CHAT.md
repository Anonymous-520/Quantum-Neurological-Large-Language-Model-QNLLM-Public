# NLLM Interactive Chat

**Chat with your QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM powered by MTL-3 (Nemotron + Llama + GPT-OSS)**

---

## Quick Start

### Option 1: Run Directly (if already built)
```bash
cd cpp/build/Release
./chat.exe
```

### Option 2: Use Batch File (Windows)
```bash
cd cpp
run_chat.bat
```

### Option 3: Build and Run (PowerShell)
```powershell
cd cpp
.\build_chat.ps1
```

---

## Features

 **Interactive Chat Interface** - Natural conversation with NLLM 
 **MTL-3 Powered** - 3 specialized teachers (Nemotron, Llama, GPT-OSS) 
 **Real-time Metrics** - Quality, agreement, and confidence scores 
 **Memory Integration** - Stores conversation in memory for context 
 **Conversation History** - Auto-saved to `logs/autonomous_outputs/` 
 **Commands** - Built-in commands for control

---

## Chat Commands

| Command | Description |
|---------|-------------|
| `/help` | Show available commands |
| `/stats` | Display system statistics |
| `/clear` | Clear the screen |
| `/exit` or `/quit` | Exit chat (saves memories) |

---

## Example Session

```

 QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM - Interactive Chat 
 MTL-3 System 

 Teachers: Nemotron + Llama + GPT-OSS 

 Initializing MTL-3 System...

 Nemotron teacher initialized
 Llama teacher initialized
 GPT-OSS teacher initialized
 MTL Loop created (agreement threshold: 0.7)
 Memory store initialized
 Embedder initialized (768 dimensions)

 System ready! Type your message or /help for commands.

You: What is Deterministic Processing?

NLLM: Deterministic Processing is a subset of Autonomous System that enables 
computers to learn from data and improve their performance over time without 
being explicitly programmed for each task...

 [Quality: 0.904 | Agreement: 0.985 | Confidence: 0.823 | Teachers: 3/3]

You: Can you explain deterministic networks?

NLLM: deterministic networks are computational models inspired by biological deterministic 
networks in the human brain. They consist of interconnected nodes (neurons) 
organized in layers...

 [Quality: 0.910 | Agreement: 0.982 | Confidence: 0.831 | Teachers: 3/3]

You: /stats

 System Statistics:
 Messages: 2
 Memories: 2
 Teachers: 3
 Agreement Threshold: 0.7

You: /exit

 Goodbye! Saving conversation...
 Memories saved to data/encodings/
 Chat history in logs/autonomous_outputs/

Thank you for chatting with NLLM! 
```

---

## What Gets Saved

### Conversation History
**Location**: `logs/autonomous_outputs/chat_history_[day].log`

```
[2026-01-15 10:30:00]
USER: What is Deterministic Processing?
NLLM: Deterministic Processing is a subset of Autonomous System...
 Quality: 0.904 | Agreement: 0.985 | Confidence: 0.823
---

[2026-01-15 10:31:00]
USER: Can you explain deterministic networks?
NLLM: deterministic networks are computational models...
 Quality: 0.910 | Agreement: 0.982 | Confidence: 0.831
---
```

### Memory Store
**Location**: `data/encodings/`

Each question-answer pair is stored with:
- encoding vector (768 dimensions)
- User question text
- Response snippet
- Quality score
- Timestamp

---

## Understanding Metrics

### Quality Score (0-1)
- **0.90+**: Excellent response quality
- **0.80-0.90**: Good response
- **0.70-0.80**: Acceptable response
- **< 0.70**: May need improvement

### Agreement Level (0-1)
- **0.95+**: Very high teacher consensus
- **0.85-0.95**: Good agreement
- **0.70-0.85**: Moderate agreement
- **< 0.70**: Low consensus (disagreement)

### Confidence Score (0-1)
- **0.80+**: High confidence
- **0.70-0.80**: Moderate confidence
- **0.60-0.70**: Lower confidence
- **< 0.60**: Low confidence

### Teachers
Shows how many of the 3 teachers provided responses (should be 3/3)

---

## Building from Source

### Prerequisites
- Visual Studio 2022 with C++ tools
- CMake 3.15+
- C++17 compiler

### Build Steps

#### Method 1: CMake + MSBuild
```bash
cd cpp
cmake -S . -B build -G "Visual Studio 17 2022"
cmake --build build --config Release --target chat
```

#### Method 2: Visual Studio Solution
```bash
cd cpp/build
# Open neurological-Autonomous Processor-cpp.sln in Visual Studio
# Build the 'chat' project in Release mode
```

#### Method 3: Direct MSBuild
```bash
cd cpp/build
"C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" ^
 neurological-Autonomous Processor-cpp.sln /t:chat /p:Configuration=Release
```

### Run
```bash
cd cpp/build/Release
./chat.exe
```

---

## Configuration

### Teacher API Keys
The chat uses the 3 NIM teachers configured with API keys in `chat.cpp`:

```cpp
// Nemotron
"<set via NVIDIA_API_KEY_NEMOTRON>"

// Llama
"<set via NVIDIA_API_KEY_LLAMA>"

// GPT-OSS
"<set via NVIDIA_API_KEY_GPT_OSS>"
```

### Agreement Threshold
Default: **0.7** (70% agreement required for consensus)

Can be adjusted in `chat.cpp`:
```cpp
auto mtl_loop = std::make_shared<nllm::core::pipeline::MTLLoop>(teachers, 0.7f);
```

---

## Troubleshooting

### Issue: "Teachers not responding"
- Check API keys are valid
- Verify internet connection
- Check NIM API status

### Issue: "Build failed"
- Ensure Visual Studio 2022 is installed
- Verify C++17 compiler support
- Check CMake version (3.15+)

### Issue: "Memories not saving"
- Check `data/encodings/` exists and is writable
- Verify disk space available

### Issue: "Log files not created"
- Check `logs/autonomous_outputs/` directory permissions
- Ensure sufficient disk space

---

## Tips for Best Results

### Ask Clear Questions
 Good: "What is Deterministic Processing and how does it work?"
 Unclear: "Deterministic Processing?"

### Provide Context
 Good: "Can you explain deterministic networks in the context of image recognition?"
 Limited: "deterministic nets"

### Use Commands
- Check `/stats` regularly to monitor system
- Use `/clear` to refresh screen for readability
- Always `/exit` properly to save memories

### Monitor Metrics
- High quality (>0.9) + high agreement (>0.95) = excellent response
- Low agreement (<0.7) = teachers disagree, may need clarification
- Watch teacher count - should always be 3/3

---

## Architecture

```
User Input
 â†“
MTL Loop (3 Teachers in Parallel)
 Nemotron â†’ Response A
 Llama â†’ Response B
 GPT-OSS â†’ Response C
 â†“
Disagreement Scoring
 Semantic Similarity
 Agreement Measurement
 Confidence Calculation
 â†“
Best Response Selection
 â†“
Quality Scoring
 Grammar Check
 Relevance Analysis
 Coherence Evaluation
 Fluency Assessment
 â†“
Memory Storage
 Embed Question (768-dim)
 Store with Metadata
 Save to data/encodings/
 â†“
Conversation Log
 Save to logs/autonomous_outputs/
 â†“
Display to User
```

---

## Performance

- **Response Time**: 2-5 seconds per query (3 teachers in parallel)
- **Memory Usage**: ~50-100 MB
- **Disk Usage**: ~1-5 KB per conversation turn
- **Throughput**: ~10-20 messages per minute

---

## Future Enhancements

- [ ] Streaming responses (word-by-word)
- [ ] Multi-turn context (conversation memory)
- [ ] Response regeneration option
- [ ] Export conversation to file
- [ ] Voice input/output
- [ ] Web interface
- [ ] Custom teacher selection
- [ ] Response quality feedback

---

## Support

For issues or questions:
1. Check logs in `logs/autonomous_outputs/`
2. Review metrics for low quality/agreement
3. Verify API keys are valid
4. Ensure teachers are responding (3/3)

---

## License

See [LICENSE.md](../docs/Security%20&%20CC%20Realeted/LICENSE.md)

---

**Enjoy chatting with your NLLM! **
