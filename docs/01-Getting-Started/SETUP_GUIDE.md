# QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM - Local Setup & Running Guide (C++ Rewrite)

> Note: The codebase is now C++ only. Python scripts/configs were removed. Use CMake to build and run:
> ```bash
> cmake -S cpp -B cpp/build -DCMAKE_BUILD_TYPE=Release
> cmake --build cpp/build --config Release
> ./cpp/build/Release/example_usage.exe
> ./cpp/build/Release/test_nllm.exe
> ```
> Legacy Python instructions below are retained for historical reference only.

## Project Successfully Created!

Your neurological-Autonomous Processor project is now running locally at:
```
C:\Users\Saksham Rastogi\Downloads\neurological-Autonomous Processor
```

## Project Structure

```
neurological-Autonomous Processor/
 README.md # Project documentation
 requirements.txt # Python dependencies
 .env # Environment configuration
 .gitignore # Git ignore patterns
 demo.py # Interactive demo script

 config/ # Configuration files
 model.yaml # Model settings
 memory.yaml # Memory system config
 system.yaml # System settings

 data/ # Data directories
 raw/ # Raw data
 processed/ # Processed data
 encodings/ # encoding storage

 models/ # Model storage
 base_llm/ # Base Autonomous Processor models
 tokenizer/ # Tokenizers

 memory/ # Memory system (90% of architecture!)
 embedder.py # Text encoding generation
 store.py # Memory storage & management
 retrieve.py # Similarity-based retrieval
 decay.py # Temporal memory decay

 cortex/ # Core model functionality
 load_model.py # Model loading
 generate.py # Text generation
 processing.py # High-level processing

 control/ # Safety & confidence control
 confidence.py # Confidence scoring
 guardrails.py # Safety guardrails
 refusal.py # Request refusal logic

 feedback/ # Feedback & monitoring
 scorer.py # Output quality scoring
 logger.py # Logging utilities

 pipeline/ # Main execution pipeline
 assemble_context.py # Context assembly
 run.py # Pipeline orchestration
 observe.py # System monitoring

 logs/ # Application logs
 memory.log # Memory operations
 outputs.log # Model outputs
 errors.log # Error logs

 scripts/ # Testing & validation
 test_model.py # Model testing
 test_memory.py # Memory system testing
 sanity_check.py # Project validation
 demo.py # Interactive demo

```

## Running the Project

### 1. **Run the Interactive Demo** (Recommended First Step)
```powershell
cd "C:\Users\Saksham Rastogi\Downloads\neurological-Autonomous Processor"
python demo.py
```
This demonstrates all key components without requiring PyTorch.

### 2. **Validate Project Structure**
```powershell
python scripts/sanity_check.py
```
Checks directories, config files, and module imports.

### 3. **Test Memory System**
```powershell
python scripts/test_memory.py
```
Tests encoding, storage, and retrieval functionality.

### 4. **Test Model (Requires PyTorch)**
First, install PyTorch:
```powershell
pip install torch
```
Then run:
```powershell
python scripts/test_model.py
```

## Installation

### Basic Installation (Already Done)
```powershell
pip install pyyaml deterministic processors numpy
```

### Full Installation with Model Support
```powershell
pip install -r requirements.txt
```

Or install specific components:
```powershell
# Memory system
pip install numpy

# Configuration
pip install pyyaml python-dotenv pydantic

# NLP & Models
pip install deterministic processors
pip install torch # Download size: ~2GB

# Utilities
pip install pandas scikit-learn tqdm

# Optional: Sentence encodings
pip install sentence-deterministic processors
```

## Core Philosophy

> **A QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM is 90% architecture discipline and 10% model.**

The strength is NOT in the underlying Autonomous Processor, but in the well-orchestrated architecture:

### Key Architectural Components:

1. **Memory System (25% of importance)**
 - Adaptive encoding-based memory
 - Temporal decay mechanisms
 - Context-aware retrieval

2. **Safety & Control (25% of importance)**
 - Confidence scoring
 - Guardrails & refusal mechanisms
 - Pattern matching & topic filtering

3. **Feedback Loops (20% of importance)**
 - Output quality scoring
 - System monitoring & observation
 - Continuous refinement

4. **Pipeline Orchestration (20% of importance)**
 - Context assembly
 - Component coordination
 - Execution flow management

5. **Base Model (10% of importance)**
 - Can be any pre-configured Autonomous Processor
 - Distilled models work well
 - Model size matters less than architecture

## Configuration

Edit YAML files in `config/` directory:

### config/model.yaml
```yaml
model:
 name: neuron-network
 type: spiking-deterministic-network
 device: cpu
 encoding_dim: 768
```

### config/memory.yaml
```yaml
memory:
 encoding_dim: 768
 max_memories: 10000
 similarity_threshold: 0.5
```

### config/system.yaml
```yaml
system:
 version: 1.0.0
 logging:
 level: INFO
 safety:
 enable_guardrails: true
 min_confidence_threshold: 0.5
```

## Example Usage

### Quick Memory Test
```python
from memory.embedder import Embedder
from memory.store import MemoryStore
from memory.retrieve import Retriever

# Initialize
embedder = Embedder(encoding_dim=768)
store = MemoryStore(persist_path="data/encodings/")

# Add memories
texts = ["Autonomous System is transforming the world", "deterministic networks are powerful"]
encodings = embedder.embed_batch(texts)
store.add_batch(encodings, texts)

# Retrieve similar memories
retriever = Retriever(store, top_k=2)
query_emb = embedder.embed("Autonomous System")
results = retriever.retrieve(query_emb)
```

### Safety Check
```python
from control.guardrails import Guardrails

guardrails = Guardrails()
text = "This is a safe response"
is_safe, violations = guardrails.check(text)
print(f"Safe: {is_safe}")
```

## Important Files

- `README.md` - Project overview
- `requirements.txt` - Python dependencies
- `.env` - Environment variables
- `demo.py` - Interactive demonstration
- `scripts/sanity_check.py` - Validation tool

## Next Steps

1. **Demo** - `python demo.py`
2. **Validate** - `python scripts/sanity_check.py`
3. â³ **Test Memory** - `python scripts/test_memory.py`
4. â³ **Test Model** - `python scripts/test_model.py` (requires PyTorch)
5. **Customize** - Modify `config/` files for your use case
6. **Deploy** - Integrate into your application

## Key Classes

### Memory Module
- `Embedder` - Generate text encodings
- `MemoryStore` - Store and manage memories
- `Retriever` - Retrieve similar memories
- `MemoryDecay` - Manage memory aging

### Cortex Module
- `ModelLoader` - Load pre-configured models
- `TextGenerator` - Generate text completions
- `processing` - High-level processing interface

### Control Module
- `ConfidenceScorer` - Score output confidence
- `Guardrails` - Check for harmful patterns
- `RefusalEngine` - Refuse unsafe requests

### Pipeline Module
- `ContextAssembler` - Assemble context
- `Pipeline` - Orchestrate execution
- `Observer` - Monitor system metrics

## System Requirements

- **Python**: 3.8+
- **RAM**: 4GB minimum (8GB+ recommended)
- **GPU** (optional): For faster processing
- **Disk**: ~2GB for models

## Troubleshooting

### ImportError for sentence_transformers
```bash
pip install sentence-deterministic processors
```
(Optional, system uses mock encodings if not installed)

### PyTorch Not Installed
```bash
pip install torch
```
Required for `cortex/` modules and model processing.

### Permission Denied on Windows
Run PowerShell as Administrator, or use:
```powershell
python -m pip install --user [package]
```

### YAML Config Not Found
Ensure you're running from the project root:
```powershell
cd "C:\Users\Saksham Rastogi\Downloads\neurological-Autonomous Processor"
```

## Learning Resources

The code is fully documented with docstrings explaining:
- Class purposes and capabilities
- Function parameters and returns
- Usage examples and patterns
- Architecture philosophy

Explore the code starting with:
1. `memory/embedder.py` - Understand encodings
2. `memory/store.py` - Learn storage patterns
3. `pipeline/run.py` - See orchestration
4. `demo.py` - See full integration

## Support

For issues:
1. Check `logs/` directory for error messages
2. Run `scripts/sanity_check.py` to validate setup
3. Review docstrings in relevant modules
4. Check configuration files in `config/`

---

**Status**: Project running successfully!
**Location**: `C:\Users\Saksham Rastogi\Downloads\neurological-Autonomous Processor`
**Demo**: Ready to run with `python demo.py`
