# NVIDIA NIM Setup for NLLM

## What is NIM?

**NVIDIA processing Microservices** - Run LLMs locally with zero cloud cost.

## Quick Start (Docker)

### 1. Get NGC API Key

```bash
# Get free key from: https://ngc.nvidia.com/
export NGC_API_KEY="your-key-here"
```

### 2. Run NIM Container

```bash
docker run -d \
 --gpus all \
 --shm-size=16GB \
 -e NGC_API_KEY=$NGC_API_KEY \
 -v ~/nim-cache:/opt/nim/.cache \
 -p 8000:8000 \
 nvcr.io/nim/meta/llama-3.1-8b-instruct:latest
```

### 3. Test NLLM Chat

```bash
python chat.py nim
```

## Available Models

- `meta/llama-3.1-8b-instruct` (recommended, 8GB VRAM)
- `meta/llama-3.1-70b-instruct` (powerful, 40GB+ VRAM)
- `mistralai/mixtral-8x7b-instruct-v0.1`

## PowerShell Commands

```powershell
# Set NGC key
$env:NGC_API_KEY="your-key"

# Run NIM
docker run -d --gpus all --shm-size=16GB -e NGC_API_KEY=$env:NGC_API_KEY -v $HOME/nim-cache:/opt/nim/.cache -p 8000:8000 nvcr.io/nim/meta/llama-3.1-8b-instruct:latest

# Start chat
python chat.py nim
```

## Custom NIM Endpoint

```bash
python chat.py nim --base-url http://your-nim-server:8000/v1
```

## Advantages

 **Zero API cost** (runs locally) 
 **Full privacy** (data never leaves your machine) 
 **Fast processing** (local GPU) 
 **Same architecture** (OpenAI-compatible) 
 **No rate limits**

## System Requirements

- NVIDIA GPU (RTX 3090, A100, etc.)
- Docker with GPU support
- 16GB+ VRAM (for 8B models)
- NGC account (free)

## Troubleshooting

**"NIM engine not running"**
→ Start NIM container first

**Connection refused**
→ Check port: `docker ps`

**Out of memory**
→ Use smaller model or add `--max-batch-size=1`

## Next Steps

1. Start NIM container
2. Test with: `python chat.py nim`
3. Chat: "2+2", "what is a tree?", etc.
4. Add human feedback loop
5. Watch learning happen!
