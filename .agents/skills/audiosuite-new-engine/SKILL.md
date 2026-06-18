---
name: new-engine-integration
description: Guide for implementing a new TTS, Voice Changer, or ASR engine in TTS Audio Suite (ComfyUI). Use this skill whenever the user wants to add a new engine, integrate a new TTS model, add voice conversion support, add ASR support, implement a new ComfyUI node for audio generation, or is asking about architecture, scope, parity, PR review, or implementation order for TTS Audio Suite engines. Also triggers for questions about how existing engines (Qwen3-TTS, Step Audio EditX, VibeVoice, etc.) are structured.
---

# New Engine Integration — TTS Audio Suite

This skill covers the complete process of adding a new engine (TTS, Voice Changer, ASR, or special audio model) to TTS Audio Suite. Follow each phase in order. **Do not write code before the research and scope phases are complete.**

---

## The Golden Rule

The **official model implementation is the source of truth.** Not this suite, not third-party ComfyUI packs, not demo scripts. Every feature exposed in the UI must trace back to a native official capability. Non-native enhancements need explicit maintainer discussion first.

---

## Phase 0 — Tell The LLM Not To Code Yet

Before anything, send this to the LLM:

```
Do not write code yet.

I want to add a new engine to TTS Audio Suite. First, read these files:

- PROJECT_INDEX.md
- docs/New Engines Guides/README.md
- docs/New Engines Guides/01_FIRST_ASK_THE_LLM_TO_RESEARCH_THE_OFFICIAL_MODEL.md
- docs/New Engines Guides/02_CHECK_EXISTING_COMFYUI_IMPLEMENTATIONS.md
- docs/New Engines Guides/03_DECIDE_ENGINE_SCOPE.md
- docs/New Engines Guides/NEW_ENGINE_IMPLEMENTATION_GUIDE.md
- docs/New Engines Guides/fails_to_avoid_TTS_Engine_Implementation.md

Then research the official model implementation and produce a capability report.
```

---

## Phase 1 — Research The Official Model

Ask the LLM to produce a **capability report** before implementation. The report must answer:

1. What task does this model support? (TTS means both Unified TTS Text **and** Unified SRT TTS)
2. What are the native official parameters?
3. What features are real native capabilities vs artificial additions?
4. What model files are required and where do they come from?
5. What voice/reference input mode does it use? (ref-audio only / ref-audio + transcript / none)
6. Sample rate, channel layout, audio tensor format?
7. Dependencies and version conflicts?
8. License restrictions?
9. Does it need a special node beyond the unified nodes?
10. Is it short-form or long-form? If long-form, what is the native segmentation strategy?

**The LLM must inspect:** official README, model card, inference examples, CLI/demo scripts, Python package metadata, requirements files, download instructions, license files, HuggingFace/ModelScope file layout, and known open issues.

**Do not start implementation until this report is complete.**

---

## Phase 2 — Check Existing ComfyUI Implementations

Search for existing ComfyUI implementations and clone them to:
```
IgnoredForGitHubDocs/For_reference/[ENGINE_NAME]/
```

Use them to learn: model loading patterns, dependency gotchas, expected model files, UI patterns users expect, and edge cases already hit.

**Do not copy blindly.** Compare every feature back to the official implementation. Never copy:
- Fake parameters not in the official model
- Silent downloads to random cache directories
- Engine logic stuffed into ComfyUI node UI code
- Dependency pins that break other engines
- UI controls that imply unsupported capabilities (fake speed, fake language dropdown, fake emotion)

---

## Phase 3 — Decide Engine Scope

Write a **scope document** before coding. A good scope looks like:

```
Engine scope:
- Supports TTS: yes, with both Unified TTS Text and Unified SRT TTS
- SRT approach: generate each subtitle independently, assemble timing with AudioAssemblyEngine
- Supports Voice Changer: no
- Supports ASR: no
- Needs special node: yes — official model has native voice design from text description

Native parameters to expose: seed, temperature, top_p, language
Do NOT expose: speed (not a native feature)

Primary reference: Qwen3-TTS (TTS/SRT architecture)
Runtime plan: Main Environment only

Manual tests:
- TTS Text basic generation
- TTS Text with character tags
- SRT generation with interrupt test
- Clear VRAM then regenerate
```

**Node types available:** TTS (Unified TTS Text + Unified SRT TTS, mandatory pair), Voice Changer, ASR Transcribe, special feature node. Add a special node only when the official model has a capability that does not fit the unified nodes.

If the scope is vague, do not code yet.

---

## Phase 4 — Reference Engines

| Reference | Use for |
|---|---|
| **Qwen3-TTS** (primary) | Full TTS/SRT architecture, character handling, pause tags, cache integration, interrupt handling, progress feedback, unified model loading |
| **Step Audio EditX** (secondary) | Wrapper/model lifecycle patterns, special post-processing, model loading edge cases |
| **VibeVoice** (long-form) | Duration-aware/minute-aware segmentation for long-form engines |

Files to read from Qwen3-TTS: `engines/adapters/qwen3_tts_adapter.py`, `nodes/qwen3_tts/qwen3_tts_processor.py`, `nodes/qwen3_tts/qwen3_tts_srt_processor.py`, `nodes/engines/qwen3_tts_engine_node.py`.

---

## Phase 5 — Implementation Order

Follow this order strictly:

1. Confirm scope
2. Verify model download layout (check actual HuggingFace repo with curl/API — never assume file naming)
3. Add/update downloader using `UnifiedDownloader` — ALL models go to `ComfyUI/models/TTS/`, never to random cache dirs
4. Build the engine wrapper around official inference (add `.to()` method + device checking — **CRITICAL for Clear VRAM**)
5. Register unified model factory (`register_<engine>_factory()` + add to `initialize_all_factories()`)
6. Build the adapter
7. Build the main TTS processor
8. Build the SRT processor (calls the main processor — does NOT duplicate generation logic)
9. Add the engine configuration node UI (native parameters only)
10. Wire node and adapter registration in `nodes.py`, `engines/adapters/__init__.py`, `unified_model_interface.py`, `engine_registry.py`
11. Add segment parameter support in `utils/text/segment_parameters.py`
12. Add generated audio cache
13. Add interrupt checks inside long loops (`comfy.model_management.interrupt_processing`)
14. Add progress feedback for long generation
15. Update docs/YAML metadata (`docs/Dev reports/tts_audio_suite_engines.yaml`)
16. Run manual ComfyUI tests
17. Run the required parity checklist (see Phase 6)

### Architecture rule — keep unified nodes thin

Do NOT put engine orchestration into `nodes/unified/tts_text_node.py` or `tts_srt_node.py`. Engine logic belongs in processors and adapters.

### SRT reuses TTS generation

The SRT processor is the timing/orchestration layer only. It must:
- Call the main TTS processor (`processor.process_text(...)`) for each subtitle's text
- Reuse the same adapter, cache, character handling, pause tags, parameter switching, and model lifecycle
- Assemble audio with the suite timing systems (`AudioAssemblyEngine`, `TimedAudioAssembler`, `TimingEngine`)

The SRT processor must NOT call the raw model directly or reimplement any generation logic.

### Chunking rule

Prefer the suite's shared chunk combiner. Do not add engine-local chunk-silence controls or duplicate chunk-combination UI unless the official model has a clearly different native long-form mechanism.

---

## Phase 6 — Required Parity Checklist

Paste this to the LLM before review. Every item must be "yes" or the PR is not ready.

**Architecture**
- [ ] Processor used instead of putting engine logic in unified nodes
- [ ] Unified nodes are thin routing/delegation only

**Model loading and lifecycle**
- [ ] `unified_model_interface.load_model()` used in all model-loading paths
- [ ] Clear VRAM / unload / reload works
- [ ] No `__del__` destructor that auto-unloads models after generation
- [ ] Model variants don't accumulate VRAM unnecessarily

**Downloads and dependencies**
- [ ] Real HuggingFace/ModelScope file layout verified
- [ ] Models download to `ComfyUI/models/TTS/` — no silent cache downloads
- [ ] Dependency conflicts documented; runtime isolation decision made and documented

**Audio format**
- [ ] Native sample rate verified; output is valid ComfyUI audio dict/tensor `[batch, channels, samples]`
- [ ] No pitch/speed bugs from wrong sample-rate metadata

**Generated audio cache**
- [ ] Cache added; cache key includes text, voice identity, model variant, seed, all behavior-affecting params
- [ ] Cache hit confirmed by running same input twice

**Text features**
- [ ] Character tags wired; narrator fallback works
- [ ] Pause tags wired
- [ ] Parameter switching wired for real native parameters
- [ ] No fake unsupported parameters in UI
- [ ] Suite chunking/chunk-combination controls reused — no duplicate engine-local chunk UI

**SRT features**
- [ ] SRT processor implemented for every TTS engine
- [ ] Interrupt checks in long SRT loops
- [ ] Correct shared timing/assembly API used (no guessed helper names)
- [ ] Character switching works in SRT

**UX**
- [ ] Progress feedback for long generation
- [ ] Console observability parity (input-text echo, live it/s/progress)
- [ ] Tooltips honest about limitations, license, VRAM, expected text length

**Docs and metadata**
- [ ] YAML-backed engine metadata updated
- [ ] Model paths and download sources documented

**Manual tests**
- [ ] TTS Text basic generation ✓
- [ ] TTS Text with character tags ✓
- [ ] TTS Text with pause tags ✓
- [ ] TTS Text with parameter switching ✓
- [ ] SRT generation ✓
- [ ] SRT interrupt/cancel ✓
- [ ] Clear VRAM then regenerate ✓
- [ ] Cache hit on repeat generation ✓

---

## Phase 7 — PR Review Checklist

A PR is not ready just because it generates audio.

**Required PR evidence:** official capability report, ComfyUI reference notes, selected scope, native parameters implemented, intentionally skipped features with reasons, any non-native enhancements with maintainer approval, model download paths, manual test results, known limitations.

**Architecture review:** unified nodes are thin; TTS Text processor + SRT processor both exist for every TTS engine; SRT missing = explicit maintainer exception required.

**Model lifecycle review:** all loading via `unified_model_interface.load_model()`; Clear VRAM works; no `__del__`; quantized models handled (bitsandbytes int4/int8 cannot use `.to()`); no VRAM accumulation.

**Feature review:** UI exposes native params only; language dropdown absent if model is English-only; character/narrator/pause/parameter-switching all pass.

**SRT review:** interrupt works; timing modes use shared assembly systems; no unnecessary restrictions; character switching + pause tags don't corrupt timing.

**Audio review:** output tensor is 3D `[batch, channels, samples]`; sample rate correct; reference audio resampled to native rate.

**Do not merge** until the implementation passes the parity checklist or gaps are explicitly accepted by the maintainer.

---

## Copy-Paste Prompts for Users

See `references/user_prompts.md` for the full 8-prompt sequence to guide an LLM through the process (research → ComfyUI refs → scope → plan → implement phase 1 → parity review → fake-param check → PR summary).

## Deep Technical Reference

See `references/technical_guide.md` for:
- Full file structure template
- Clear VRAM `.to()` implementation pattern (with wrapper reload logic)
- Interrupt handling patterns with code examples
- Unified systems integration (cache, character switching, language mapper, pause tags)
- Generation info / ChunkCombiner + ChunkTimingHelper pattern
- SRT shared API real call patterns (never guess helper names)
- ComfyUI audio tensor shape requirements

## Known Traps to Avoid

See `references/fails.md` for the full list of documented failures. Key ones:
- Never use `__del__` destructors — causes model unload after generation
- Quantized (bitsandbytes) models cannot be moved with `.to()` — wrap in try-except
- SRT processors must use `importlib.util.spec_from_file_location()` for cross-module imports
- Never guess SRT assembly API names — open a working SRT processor and copy the exact call pattern
- ComfyUI audio must be 3D `[batch, channels, samples]`, never 2D
- Add new engine to `PARAMETER_ENGINES` in `utils/text/segment_parameters.py` for `[seed:X]` tag support
- Language UI must use normalized display names, not raw backend codes