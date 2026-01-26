# NLLM Chat Quick Start

## Running the Chat Interface

The NLLM chat interface is configured and ready to use with real NVIDIA processing.

### Basic Usage

```powershell
# Default: Uses Llama 3.1 405B (configured in .env)
& ".\.venv\Scripts\python.exe" ".\Mainsys\chat.py"
```

### Using Different Models

```powershell
# Nemotron 3 Nano (fast, efficient)
& ".\.venv\Scripts\python.exe" ".\Mainsys\chat.py" nvidia "nvidia/nemotron-3-nano-30b-a3b"

# GPT-OSS 120B (balanced)
& ".\.venv\Scripts\python.exe" ".\Mainsys\chat.py" nvidia "openai/gpt-oss-120b"

# Llama 3.1 405B (high quality, default)
& ".\.venv\Scripts\python.exe" ".\Mainsys\chat.py" nvidia "meta/llama-3.1-405b-instruct"
```

## API Keys Configuration

Your API keys are stored in `.env` (already configured):

- **NVIDIA_API_KEY**: Primary key (Llama 405B)
- **NVIDIA_API_KEY_NEMOTRON**: Nemotron model
- **NVIDIA_API_KEY_GPT_OSS**: GPT-OSS model
- **NVIDIA_API_KEY_LLAMA**: Llama model

The system automatically selects the correct key based on the model you choose.

## Chat Commands

Once inside the chat:

- Type your message to chat
- `/help` - Show available commands
- `/stats` - Show system statistics
- `/memories` - Show recent memories
- `/clear` - Clear conversation history
- `/exit` - Exit chat session

## Features

- **Memory System**: Stores and retrieves relevant past interactions
- **Quality Scoring**: Evaluates response quality
- **Safety Guardrails**: Content safety checks
- **Context Assembly**: Integrates memories with queries

## How It Works

1. **Memory Retrieval**: When you ask a question, the system searches for relevant past interactions
2. **Context Building**: Retrieved memories are combined with your current query
3. **NVIDIA processing**: Your query + context is sent to the selected NVIDIA model
4. **Quality Assessment**: Response is scored and checked for safety
5. **Memory Storage**: The interaction is stored for future retrieval

## Troubleshooting

**401 Authentication Error**
- Verify your API key is correctly set in `.env`
- Ensure you're using the Build API key from https://build.nvidia.com/

**Module Not Found**
- Activate the virtual environment: `& ".\.venv\Scripts\Activate.ps1"`
- Verify packages are installed: `pip list`

**No Response / Timeout**
- Check your internet connection
- Verify the NVIDIA API endpoint is reachable

## Example Session

```
You: What is memory plasticity?

NLLM: Memory plasticity refers to the ability of memories to strengthen or 
weaken over time based on experience and feedback. In this system, memories 
are adjusted through agreement-based scoring from multiple teacher models...

You: /stats

SYSTEM STATISTICS
Conversation turns: 1
Total memories stored: 9
Average response quality: 0.875
```

## Next Steps

- Try different models to compare quality and speed
- Use `/memories` to see what the system remembers
- Experiment with complex multi-turn conversations to see memory retrieval in action
