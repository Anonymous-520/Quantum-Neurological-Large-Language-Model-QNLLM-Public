# TBRH (Task-Bounded Reasoning Head) v2.3 Documentation

This folder contains complete documentation for TBRH v2.3, QNLLM's deterministic, auditable language generation system.

## Contents

### Core Documentation
- **[TBRH_SPECIFICATION.md](TBRH_SPECIFICATION.md)** - Complete technical specification
 - All 3 task types with examples
 - API reference
 - Integration guide

- **[TBRH_IMPLEMENTATION_COMPLETE.md](TBRH_IMPLEMENTATION_COMPLETE.md)** - Implementation details
 - Phase-by-phase breakdown
 - Code metrics
 - Architecture overview

- **[TBRH_DOCUMENTATION_INDEX.md](TBRH_DOCUMENTATION_INDEX.md)** - Quick reference
 - Navigation guide
 - File locations
 - API summary

### Release Documentation
- **[v2.3_STATUS.md](v2.3_STATUS.md)** - Build status and features
- **[v2.3_COMPLETION_SUMMARY.md](v2.3_COMPLETION_SUMMARY.md)** - Executive summary
- **[VERIFICATION_REPORT_v2.3.md](VERIFICATION_REPORT_v2.3.md)** - QA verification
- **[RELEASE_NOTES_v2.3.md](RELEASE_NOTES_v2.3.md)** - Release overview

## Quick Start

### For Developers
Start with [TBRH_SPECIFICATION.md](TBRH_SPECIFICATION.md) for complete API and examples.

### For Integration
See [TBRH_IMPLEMENTATION_COMPLETE.md](TBRH_IMPLEMENTATION_COMPLETE.md) Phase 3 section.

### For Verification
Review [VERIFICATION_REPORT_v2.3.md](VERIFICATION_REPORT_v2.3.md) for QA results.

## Key Features

- **Bounded**: Hard 64-token cap (immutable)
- **Auditable**: 6-check verification on every output
- **Cited**: All claims link to memory IDs
- **Gate-aware**: Respects learning_gate state
- **Deterministic**: Same input = same output
- **Zero dependencies**: Template-based only

## Test Results

**4/4 tests passing (100%)**
- Gate closed → no output
- Budget respected (≤64 tokens)
- Provenance verified (all claims cited)
- Confidence threshold respected

## Location

- Source code: `src/systems/tbrh/`
- Tests: `tests/test_tbrh.py`
- Documentation: `docs/10-TBRH-v2.3/`

## Related

- Main specification: `docs/06-Versions/QNLLM_V2_SPEC.md`
- Learning laws: `docs/02-Architecture/LEARNING_LAWS_V2_2.md`
- Release notes: `docs/06-Versions/RELEASE_NOTES_v2.3.md`

---

**Version**: v2.3 
**Status**: PRODUCTION READY 
**Quality**: (5/5)
