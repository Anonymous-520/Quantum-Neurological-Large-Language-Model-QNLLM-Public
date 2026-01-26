# Memory Persistence System

**Date**: January 15, 2026 
**System**: QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM (NLLM) - Memory & Data Storage

---

## Overview

All memory, encodings, and learning data are automatically persisted to the `data/` folder structure. This ensures that learning progress is preserved across sessions and the system maintains continuity.

---

## Directory Structure

```
data/
 encodings/ # Memory store persistence
 memory_0.txt # Individual memory files
 memory_1.txt
 memory_2.txt
 ...
 memory_index.txt # Index with metadata

 processed/ # Processed learning data
 [future use]

 raw/ # Raw configuration data
 [future use]
```

---

## Memory File Format

Each memory is saved in a structured text format:

```
ID: 0
TEXT: First memory content here
encoding_SIZE: 768
encoding: 0.123,0.456,0.789,...
METADATA_COUNT: 3
META_timestamp: 1736899200
META_access_count: 5
META_confidence: 0.85
```

---

## Auto-Persistence Features

### 1. Real-Time Persistence

Every memory is automatically saved to disk when added:

```cpp
int memory_id = memory_store.add_memory(encoding, text, metadata);
// Automatically persisted to data/encodings/memory_{id}.txt
```

### 2. Checkpoint Summaries

Every 100 memories, a checkpoint is logged:

```
[MemoryStore] Checkpoint: 100 memories persisted
[MemoryStore] Checkpoint: 200 memories persisted
...
```

### 3. Index File

The system maintains `memory_index.txt` with summary information:

```
TOTAL_MEMORIES: 250
NEXT_ID: 250
TIMESTAMP: 1736899200
```

---

## Session Logging

All MTL-3 sessions are logged to `logs/autonomous_outputs/`:

```
logs/
 autonomous_outputs/
 mtl_session_1736899200.log
 mtl_session_1736902800.log
 ...
```

### Log Content

```
=== MTL-3 Session Log ===
Timestamp: 1736899200
Teachers: Nemotron, Llama, GPT-OSS
Memories Saved: 250
Quality Score: 0.904
Agreement Level: 0.985
Confidence: 0.823
Background Learning: Enabled (60s interval)
Persist Path: data/encodings/
```

---

## Usage Examples

### Save All Memories Manually

```cpp
auto memory_store = nllm::core::memory::MemoryStore("data/encodings/");

// Add memories...
memory_store.add_memory(encoding, text);

// Manually save all (also happens automatically)
int saved = memory_store.save_all_memories();
std::cout << "Saved " << saved << " memories" << std::endl;
```

### Load Memories from Disk

```cpp
auto memory_store = nllm::core::memory::MemoryStore("data/encodings/");

// Load previous session
int loaded = memory_store.load_all_memories();
std::cout << "Loaded " << loaded << " memories from disk" << std::endl;

// Continue adding to existing memories
memory_store.add_memory(new_embedding, new_text);
```

### Custom Persist Path

```cpp
// Use different storage location
auto memory_store = nllm::core::memory::MemoryStore("data/processed/custom/");
```

---

## Persistence Guarantees

### Automatic Persistence

- **Every memory is saved immediately** after addition
- No need for manual save calls in most cases
- Background thread-safe operations

### Atomic Operations

- Each memory file is written atomically
- Partial writes are avoided
- Index updated after successful saves

### Error Handling

```
[MemoryStore] Error: Could not open file for writing: data/encodings/memory_5.txt
[MemoryStore] Warning: Could not load memory 5: File not found
```

- Errors are logged but don't crash the system
- Failed saves/loads are skipped gracefully
- System continues with available data

---

## Performance Considerations

### Write Performance

| Operation | Time | Notes |
|-----------|------|-------|
| Single memory add | ~2ms | Includes disk write |
| Batch 100 memories | ~200ms | Parallelizable |
| Full save (1000 memories) | ~2s | Rarely needed |

### Read Performance

| Operation | Time | Notes |
|-----------|------|-------|
| Load single memory | ~1ms | From disk |
| Load 1000 memories | ~1s | Startup time |
| Index read | <1ms | Very fast |

### Storage Requirements

- **Per memory**: ~1-5 KB (depending on encoding size)
- **1000 memories**: ~1-5 MB
- **10,000 memories**: ~10-50 MB
- **100,000 memories**: ~100-500 MB

All storage sizes are manageable for modern systems.

---

## Background MTL Integration

The background MTL learner automatically persists learning feedback:

```cpp
auto background_learner = std::make_shared<nllm::core::pipeline::BackgroundMTLLearner>(
 mtl_loop, 60, true); // persist_feedback = true
```

### What Gets Saved

1. **Teacher Responses**: All 3 teacher outputs
2. **Quality Scores**: Consensus quality measurements
3. **Agreement Levels**: Multi-teacher agreement data
4. **Confidence Values**: Teacher confidence scores
5. **Timestamps**: When each learning cycle occurred

---

## Data Recovery

### Scenario 1: System Restart

```cpp
// On startup, automatically load previous session
auto memory_store = nllm::core::memory::MemoryStore("data/encodings/");
memory_store.load_all_memories();

// System continues with previous knowledge
```

### Scenario 2: Partial Failure

If some memory files are corrupted:

```
[MemoryStore] Warning: Could not load memory 42: Parse error
[MemoryStore] Loaded 999/1000 memories
```

System continues with 999 valid memories.

### Scenario 3: Full Data Loss

If `data/encodings/` is empty:

```
[MemoryStore] No index found at data/encodings/memory_index.txt. Starting fresh.
[MemoryStore] Initialized with persist path: data/encodings/
```

System starts with a clean slate.

---

## Best Practices

### DO

- **Use default persist path** (`data/encodings/`) for consistency
- **Let auto-persistence handle saves** in most cases
- **Check logs** for persistence errors
- **Backup `data/` folder** periodically
- **Monitor storage space** for large memory stores

### DON'T

- Don't manually delete memory files while system is running
- Don't modify memory files directly (use API)
- Don't disable auto-persistence unless necessary
- Don't ignore persistence errors in production

---

## Configuration Options

### Persist Path

```cpp
// Default: data/encodings/
auto store1 = nllm::core::memory::MemoryStore();

// Custom path
auto store2 = nllm::core::memory::MemoryStore("custom/path/");

// Separate stores for different purposes
auto train_store = nllm::core::memory::MemoryStore("data/configuration/");
auto eval_store = nllm::core::memory::MemoryStore("data/evaluation/");
```

### Background Persistence

```cpp
// Enable background persistence (default: true)
auto learner = BackgroundMTLLearner(mtl_loop, 60, true);

// Disable for testing only
auto test_learner = BackgroundMTLLearner(mtl_loop, 60, false);
```

---

## Monitoring & Debugging

### Check Persistence Status

```bash
# Count memory files
ls data/encodings/memory_*.txt | wc -l

# View index
cat data/encodings/memory_index.txt

# Check latest memories
ls -lt data/encodings/memory_*.txt | head -5

# View session logs
ls -lt logs/autonomous_outputs/*.log | head -5
```

### View Memory Contents

```bash
# View specific memory
cat data/encodings/memory_0.txt

# Search for specific text
grep "TEXT:" data/encodings/memory_*.txt | head -10
```

### Disk Space Usage

```bash
# Check data folder size
du -sh data/

# Check encodings specifically
du -sh data/encodings/

# Check logs
du -sh logs/autonomous_outputs/
```

---

## Troubleshooting

### Issue: "Could not create directory"

**Cause**: Insufficient permissions

**Solution**:
```bash
# Ensure write permissions
chmod -R 755 data/
mkdir -p data/encodings/
```

### Issue: "Could not open file for writing"

**Cause**: Path doesn't exist or disk full

**Solution**:
```bash
# Check disk space
df -h

# Verify directory exists
mkdir -p data/encodings/
```

### Issue: "Could not load memory from disk"

**Cause**: Corrupted file or wrong format

**Solution**:
- Check file with `cat data/encodings/memory_{id}.txt`
- Remove corrupted file if necessary
- System will continue with remaining memories

---

## Future Enhancements

### Planned Features

1. **Compression**: Gzip compression for older memories
2. **Archival**: Move old memories to archive storage
3. **Cloud Sync**: Optional cloud backup integration
4. **Encryption**: Encrypted memory storage option
5. **Indexing**: SQLite index for faster searches
6. **Versioning**: Memory version control for rollback

---

## Summary

 **All memory automatically saved to `data/encodings/`** 
 **Session logs in `logs/autonomous_outputs/`** 
 **Thread-safe, atomic operations** 
 **Graceful error handling** 
 **Fast performance (<2ms per memory)** 
 **Efficient storage (1-5 KB per memory)**

The system ensures **continuous learning persistence** with **zero data loss** under normal operation.

---

**Related Documentation**:
- [MTL-3 Implementation](MTL3_BACKGROUND_IMPLEMENTATION.md)
- [Testing Guide](Test%20&%20All%20Type%20Report/TESTING_GUIDE.md)
- [Architecture](Architecture/ARCHITECTURE_COMPLETE.md)
