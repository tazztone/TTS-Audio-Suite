---
name: new-engine-integration
description: Guide for implementing a new TTS, Voice Changer, or ASR engine in TTS Audio Suite (ComfyUI). Use this skill whenever the user wants to add a new engine, integrate a new TTS model, add voice conversion support, add ASR support, implement a new ComfyUI node for audio generation, or is asking about architecture, scope, parity, PR review, or implementation order for TTS Audio Suite engines. Also triggers for questions about how existing engines (Qwen3-TTS, Step Audio EditX, VibeVoice, etc.) are structured.
---

# New Engine Integration — TTS Audio Suite

This skill covers the complete process of adding a new engine (TTS, Voice Changer, ASR, or special audio model) to TTS Audio Suite.

## Quick Start
Before writing code, research the model and produce a capability report:
1. Read the official model implementation (this is the source of truth).
2. Produce a scope document defining native parameters, architecture, and parity.
3. Review references like Qwen3-TTS (`engines/adapters/qwen3_tts_adapter.py`).

See [Copy-Paste Prompts](references/user_prompts.md) for the 8-prompt sequence to guide LLMs through research, scoping, planning, and implementation.

## Workflows

### Phase 0 to 3: Planning & Scoping
- [ ] **Phase 0 (Init)**: Tell the LLM not to code yet. Check [Copy-Paste Prompts](references/user_prompts.md#prompt-1---official-model-research).
- [ ] **Phase 1 (Research)**: Build a capability report (native params, sample rate, licensing, file layout).
- [ ] **Phase 2 (Compare)**: Check existing ComfyUI integrations to learn gotchas.
- [ ] **Phase 3 (Scope)**: Document exact scope (TTS, SRT, voice/language switching, VRAM offloading).

### Phase 4 to 5: Implementation
Follow the strict implementation order to integrate the model correctly:
- [ ] **Step 1**: Register unified model factory (`register_<engine>_factory()`).
- [ ] **Step 2**: Build the engine wrapper (implement `.to()` and device checks for Clear VRAM).
- [ ] **Step 3**: Implement the adapter, TTS processor, and configuration node UI.
- [ ] **Step 4**: Build the SRT processor (reuses TTS generation, does NOT duplicate logic).
- [ ] **Step 5**: Integrate unified caching, pause tags, character/language systems, and interrupt checks.

See [Deep Technical Reference](references/technical_guide.md) for templates, code patterns, and assembly API calls.

### Phase 6 to 7: Parity & Review
Ensure the implementation conforms to suite standards:
- [ ] **Phase 6 (Parity)**: Run the mandatory parity checklist (VRAM unloading, caching, 3D tensor shape).
- [ ] **Phase 7 (Review)**: Perform PR review verification.

See [Known Traps to Avoid](references/fails.md) for pitfalls (e.g., `__del__` destructor VRAM leaks, 2D vs 3D audio tensors).