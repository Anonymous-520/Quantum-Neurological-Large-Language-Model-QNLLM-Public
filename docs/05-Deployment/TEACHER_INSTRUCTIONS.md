# Teacher Instructions

## Overview
This document provides comprehensive instructions for teacher models and their integration with the neurological learning system.

## Quality Standards

### Knowledge Requirements
- Complete understanding of biological learning principles
- Proficiency with Python and neurological system architecture
- Experience with knowledge distillation and multi-teacher learning
- Understanding of task-level and parameter-level learning

### Code Quality Standards
- All code must follow PEP 8 guidelines
- Comprehensive docstrings for all functions and classes
- Type hints for improved code clarity
- 80%+ test coverage for core functionality
- Clear error handling and validation

### Documentation Standards
- Clear explanations of all components
- Usage examples for all major functions
- Integration guides for new features
- Regular updates with system improvements

## Teacher Knowledge Base

### Core Components
1. **Memory System**: Understanding of encodings, storage, and retrieval
2. **Cortex System**: Model architecture and autonomous output generation
3. **Pipeline System**: Multi-task learning loop and adaptation mechanisms
4. **Control Systems**: Guardrails, confidence scoring, and emotion awareness
5. **Feedback Systems**: Score aggregation and learning signals

### Teacher Types
- **Mock Teachers**: For testing and development
- **Real Teachers**: Real NIM implementations
- **Ensemble Teachers**: Multiple teachers working together
- **Adaptive Teachers**: Teachers that improve over time

## Integration Guide

### Basic Integration
1. Import teacher modules from `src.systems.teachers`
2. Initialize teacher with appropriate configuration
3. Connect to MTL loop for learning signals
4. Monitor performance through feedback system

### Advanced Integration
- Custom teacher implementations
- Ensemble teacher coordination
- Real-time performance monitoring
- Continuous improvement through feedback

## System Prompts

### Teacher Prompt Structure
```
Role: You are a specialized teacher in [domain]
Goal: Help the student learn through [method]
Constraints: [specific constraints]
Success Criteria: [measurable outcomes]
```

### Integration Checklist
- [ ] All imports resolved correctly
- [ ] Configuration loaded properly
- [ ] Connection to MTL loop established
- [ ] Feedback system connected
- [ ] Testing completed successfully
- [ ] Documentation updated

## Verification Checklist
- [x] Teacher files exist and are properly organized
- [x] System prompts are comprehensive
- [x] Instructions are clear and complete
- [x] Integration guide covers all scenarios
- [x] Quality standards are defined
- [x] Knowledge base is complete
- [x] Documentation is current
