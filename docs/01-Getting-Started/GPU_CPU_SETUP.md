ex# GPU/CPU Configuration Guide

## Current Status

Your QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM is now configured with automatic hardware detection:

- **Configuration**: `device: auto` (GPU if available, else CPU)
- **Current Hardware**: CPU (16 threads)
- **GPU Status**: Not detected (PyTorch CPU-only version installed)

## How It Works

The system automatically detects and uses the best available hardware:

1. **Auto Mode** (Recommended): Set `device: auto` in config
 - Uses GPU if CUDA is available
 - Falls back to CPU if no GPU detected

2. **Manual Mode**: Specify `device: cuda` or `device: cpu`
 - Forces specific hardware
 - Warns if CUDA requested but unavailable

## Configuration Files

### Model Config: `config/model.yaml`
```yaml
model:
 device: auto # auto, cuda, or cpu
```

### System Config: `config/system.yaml`
```yaml
hardware:
 device: auto
 cuda_device_id: 0 # Which GPU to use (if multiple)
 enable_mixed_precision: false # Faster GPU processing
 cpu_threads: 4 # CPU thread count
```

## Enabling GPU Support

Your current PyTorch installation is CPU-only. To enable GPU:

### 1. Check GPU Availability
```bash
nvidia-smi
```

### 2. Install CUDA Toolkit
Download from: https://developer.nvidia.com/cuda-downloads

### 3. Install PyTorch with CUDA
```bash
# For CUDA 11.8
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# For CUDA 12.1
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

### 4. Verify Installation
```bash
python check_hardware.py
```

## Using Different Devices

### Quick Check
```bash
python check_hardware.py
```

### Change Device Temporarily (in code)
```python
from cortex.load_model import ModelLoader

# Force GPU
loader = ModelLoader(device="cuda")

# Force CPU
loader = ModelLoader(device="cpu")

# Auto-detect (default)
loader = ModelLoader(device="auto")
```

### Change Device Permanently (in config)
Edit `config/model.yaml`:
```yaml
model:
 device: cuda # or cpu, or auto
```

## Performance Tips

### CPU Optimization
- Increase `cpu_threads` in `config/system.yaml`
- Use smaller batch sizes for faster response
- Consider quantization for memory efficiency

### GPU Optimization 
- Enable `enable_mixed_precision: true` for faster processing
- Increase batch size to fully utilize GPU
- Use larger models (GPU has more memory)
- Monitor GPU memory with `nvidia-smi`

## Multi-GPU Support

If you have multiple GPUs:

1. Set GPU device in `config/system.yaml`:
```yaml
hardware:
 cuda_device_id: 0 # Use first GPU
```

2. Or specify in code:
```python
loader = ModelLoader(device="cuda:0") # First GPU
loader = ModelLoader(device="cuda:1") # Second GPU
```

## Troubleshooting

### "CUDA out of memory"
- Reduce batch size in `config/system.yaml`
- Reduce `max_new_tokens` in generation
- Enable gradient checkpointing
- Clear cache: `torch.cuda.empty_cache()`

### "CUDA not available"
- Check GPU with `nvidia-smi`
- Verify CUDA installation
- Reinstall PyTorch with CUDA support
- Check driver compatibility

### Slow CPU Performance
- Increase `cpu_threads`
- Use quantized models
- Reduce max_length and max_new_tokens
- Consider batch processing

## Current Configuration Summary

 **Auto-detection**: Enabled 
 **CPU Access**: Enabled (16 threads) 
 **GPU Access**: Not available (install CUDA + PyTorch with GPU support) 
 **Fallback**: CPU will be used automatically 

Your NLLM will work immediately with CPU and automatically switch to GPU once you install CUDA support!
