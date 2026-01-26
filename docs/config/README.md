# Configuration Files

This directory contains environment configuration for the Neurological-Autonomous Processor system.

## Files

### `.env`
Contains sensitive API keys and configuration. **NOT committed to repository.**

Required variables:
```bash
NVIDIA_API_KEY=your_key_here
NVIDIA_BASE_URL=https://integrate.api.nvidia.com/v1
OPENAI_API_KEY=your_openai_key_here # Optional, for OpenAI teachers
```

### `.env.example`
Template file showing required environment variables. Safe to commit.

## Setup

1. Copy `.env.example` to `.env`:
 ```bash
 cp config/.env.example config/.env
 ```

2. Edit `.env` and add your API keys

3. Verify configuration:
 ```python
 from pathlib import Path
 from dotenv import load_dotenv
 import os

 load_dotenv(Path(__file__).parent.parent / "config" / ".env")
 print("NVIDIA API configured:", bool(os.getenv("NVIDIA_API_KEY")))
 ```

## Security

- `.env` is in `.gitignore` - never commit API keys
- Use environment-specific `.env` files for development/production
- Rotate keys regularly
- Review `.env.example` when adding new configuration variables

## See Also

- [docs/01-Getting-Started/NIM_SETUP.md](../docs/01-Getting-Started/NIM_SETUP.md) - NVIDIA NIM setup guide
- [docs/01-Getting-Started/SETUP_GUIDE.md](../docs/01-Getting-Started/SETUP_GUIDE.md) - Full setup instructions
