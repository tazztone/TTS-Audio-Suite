# Agent Skills — TTS Audio Suite

This directory contains custom workspace skills that guide AI developer agents (such as Antigravity, Cline, and Codex) when modifying or extending the TTS Audio Suite codebase.

---

## 🛠️ Workspace Skills Index

The repository-specific skills are prefixed with `audiosuite-` or named for key processes to distinguish them from general ComfyUI node skills.

### 1. [new-engine-integration](audiosuite-new-engine/SKILL.md)
* **Description**: Guide for implementing a new TTS, Voice Changer, or ASR engine.
* **Scope**: Phase-by-phase implementation sequence (research $\rightarrow$ scope $\rightarrow$ adapter $\rightarrow$ processor $\rightarrow$ SRT), parity checklists, and PR review requirements.
* **Consolidates**:
  * [docs/New Engines Guides/README.md](../../docs/New%20Engines%20Guides/README.md) (Fully)
  * [docs/New Engines Guides/01_FIRST_ASK_THE_LLM_TO_RESEARCH_THE_OFFICIAL_MODEL.md](../../docs/New%20Engines%20Guides/01_FIRST_ASK_THE_LLM_TO_RESEARCH_THE_OFFICIAL_MODEL.md) (Fully)
  * [docs/New Engines Guides/02_CHECK_EXISTING_COMFYUI_IMPLEMENTATIONS.md](../../docs/New%20Engines%20Guides/02_CHECK_EXISTING_COMFYUI_IMPLEMENTATIONS.md) (Fully)
  * [docs/New Engines Guides/03_DECIDE_ENGINE_SCOPE.md](../../docs/New%20Engines%20Guides/03_DECIDE_ENGINE_SCOPE.md) (Fully)
  * [docs/New Engines Guides/NEW_ENGINE_IMPLEMENTATION_GUIDE.md](../../docs/New%20Engines%20Guides/NEW_ENGINE_IMPLEMENTATION_GUIDE.md) (Fully)
  * [docs/New Engines Guides/fails_to_avoid_TTS_Engine_Implementation.md](../../docs/New%20Engines%20Guides/fails_to_avoid_TTS_Engine_Implementation.md) (Fully)
  * [docs/New Engines Guides/06_USER_PROMPTS_TO_COPY_PASTE.md](../../docs/New%20Engines%20Guides/06_USER_PROMPTS_TO_COPY_PASTE.md) (Fully)
  * [docs/New Engines Guides/05_REQUIRED_PARITY_CHECKLIST.md](../../docs/New%20Engines%20Guides/05_REQUIRED_PARITY_CHECKLIST.md) (Fully)
  * [docs/New Engines Guides/07_PR_REVIEW_CHECKLIST.md](../../docs/New%20Engines%20Guides/07_PR_REVIEW_CHECKLIST.md) (Fully)
* **References**:
  * [references/user_prompts.md](audiosuite-new-engine/references/user_prompts.md) — 8-prompt sequence to guide LLMs through model research and plan execution.
  * [references/technical_guide.md](audiosuite-new-engine/references/technical_guide.md) — Inference templates, `.to(device)` offload hooks, and SRT timing assembly calls.
  * [references/fails.md](audiosuite-new-engine/references/fails.md) — Chronological lessons learned and pitfall checklists.

### 2. [audiosuite-development](audiosuite-development/SKILL.md)
* **Description**: Core codebase layout, modular architecture, registries, and VRAM lifecycles.
* **Scope**: Architectural standards (UI Nodes $\rightarrow$ Adapters $\rightarrow$ Processors $\rightarrow$ Wrappers), isolated subprocess runtime routing, registering class mappings, and memory offload hooks.
* **Consolidates**:
  * [PROJECT_INDEX.md](../../PROJECT_INDEX.md) (Fully)
  * [docs/BUMP_SCRIPT_INSTRUCTIONS.md](../../docs/BUMP_SCRIPT_INSTRUCTIONS.md) (Fully)
  * [docs/MODEL_LAYOUTS.md](../../docs/MODEL_LAYOUTS.md) (Fully)
  * [docs/MODEL_DOWNLOAD_SOURCES.md](../../docs/MODEL_DOWNLOAD_SOURCES.md) (Fully)
  * [docs/AUX_MODEL_LAYOUTS.md](../../docs/AUX_MODEL_LAYOUTS.md) (Fully)
  * [docs/AUX_MODEL_SOURCES.md](../../docs/AUX_MODEL_SOURCES.md) (Fully)
  * [docs/Dev reports/VERSION_UPDATE_GUIDE.md](../../docs/Dev%20reports/VERSION_UPDATE_GUIDE.md) (Fully)
  * [docs/Dev reports/VERSION_3.1_RELEASE_GUIDE.md](../../docs/Dev%20reports/VERSION_3.1_RELEASE_GUIDE.md) (Fully)
  * [docs/Dev reports/ISOLATED_RUNTIMES_PLAN.md](../../docs/Dev%20reports/ISOLATED_RUNTIMES_PLAN.md) (Fully)
  * [docs/Dev reports/DEPENDENCY_MANAGEMENT_GUIDE.md](../../docs/Dev%20reports/DEPENDENCY_MANAGEMENT_GUIDE.md) (Fully)
  * [docs/Dev reports/STATELESS_WRAPPER_IMPLEMENTATION_PLAN.md](../../docs/Dev%20reports/STATELESS_WRAPPER_IMPLEMENTATION_PLAN.md) (Fully)
  * [docs/Dev reports/REFACTORING_PLAN.md](../../docs/Dev%20reports/REFACTORING_PLAN.md) (Partially)
  * [docs/Dev reports/MODEL_LOADING_ANALYSIS.md](../../docs/Dev%20reports/MODEL_LOADING_ANALYSIS.md) (Partially)
  * [docs/Dev reports/MODEL_LOADING_REFACTOR_REPORT.md](../../docs/Dev%20reports/MODEL_LOADING_REFACTOR_REPORT.md) (Partially)
  * [docs/qwen3_tts_optimizations.md](../../docs/qwen3_tts_optimizations.md) (Partially)
* **References**:
  * [references/architecture.md](audiosuite-development/references/architecture.md) — Directory structures, execution flow, and subprocess worker environments.
  * [references/registries.md](audiosuite-development/references/registries.md) — Registry guides for `nodes.py`, `engine_registry.py`, and `segment_parameters.py`.
  * [references/vram_management.md](audiosuite-development/references/vram_management.md) — CPU/GPU offload, quantized model VRAM teardowns, and garbage collection.
  * [references/model_downloader.md](audiosuite-development/references/model_downloader.md) — Model paths layout under `models/TTS/` and `UnifiedDownloader` implementations.
  * [references/isolated_runtimes.md](audiosuite-development/references/isolated_runtimes.md) — Isolated python venv setups, dependencies install, JSON-RPC IPC streams, and compiler variables injection.
  * [references/stateless_wrappers.md](audiosuite-development/references/stateless_wrappers.md) — Breaking CUDA Graph tracing, memory references tracking, detaching tensors, and dynamic unloading.

### 3. [audiosuite-audio-pipelines](audiosuite-audio-pipelines/SKILL.md)
* **Description**: Audio dictionary standards, 3D shapes, resamplers, timeline assembly, and vocal removal/conversions.
* **Scope**: Formatting output tensors, mapping timeline assembly methods, and pre-processing reference audio files.
* **Consolidates**:
  * [docs/VOCAL_REMOVAL_GUIDE.md](../../docs/VOCAL_REMOVAL_GUIDE.md) (Fully)
  * [docs/TEXT_CHUNKING_GUIDE.md](../../docs/TEXT_CHUNKING_GUIDE.md) (Fully)
  * [docs/RVC/RVC_INTEGRATION_SUMMARY.md](../../docs/RVC/RVC_INTEGRATION_SUMMARY.md) (Fully)
  * [docs/RVC/ComfyUI-RVC-Integration-Analysis-CORRECTED.md](../../docs/RVC/ComfyUI-RVC-Integration-Analysis-CORRECTED.md) (Fully)
  * [docs/RVC/ComfyUI-RVC-Analysis-CORRECTED.md](../../docs/RVC/ComfyUI-RVC-Analysis-CORRECTED.md) (Fully)
  * [docs/RVC/RVC_FEATURE_COMPARISON.md](../../docs/RVC/RVC_FEATURE_COMPARISON.md) (Fully)
  * [docs/Dev reports/SRT_IMPLEMENTATION.md](../../docs/Dev%20reports/SRT_IMPLEMENTATION.md) (Fully)
  * [docs/🌊_Audio_Wave_Analyzer-Complete_User_Guide.md](../../docs/🌊_Audio_Wave_Analyzer-Complete_User_Guide.md) (Partially)
  * [docs/Dev reports/STATELESS_WRAPPER_IMPLEMENTATION_PLAN.md](../../docs/Dev%20reports/STATELESS_WRAPPER_IMPLEMENTATION_PLAN.md) (Partially)
* **References**:
  * [references/tensor_formats.md](audiosuite-audio-pipelines/references/tensor_formats.md) — 3D tensor formatting `[batch, channels, samples]` and functional resampling loops.
  * [references/timing_assembly.md](audiosuite-audio-pipelines/references/timing_assembly.md) — Timing strategies mapping (stretch, overlap padding, smart natural) and `ChunkCombiner` reports.
  * [references/voice_conversion.md](audiosuite-audio-pipelines/references/voice_conversion.md) — UVR5 vocal separation nodes, RVC pitch extraction algorithms (Crepe, RMVPE), and backing track mixing.

### 4. [audiosuite-tag-processing](audiosuite-tag-processing/SKILL.md)
* **Description**: Markup tags parsing, speaker transitions, segment overrides, and lookup fallbacks.
* **Scope**: Standardizing pipe-separated overrides, pause duration processors, and priority language lookups.
* **Consolidates**:
  * [docs/CHARACTER_SWITCHING_GUIDE.md](../../docs/CHARACTER_SWITCHING_GUIDE.md) (Fully)
  * [docs/PARAMETER_SWITCHING_GUIDE.md](../../docs/PARAMETER_SWITCHING_GUIDE.md) (Fully)
  * [docs/COSYVOICE3_TAGS_GUIDE.md](../../docs/COSYVOICE3_TAGS_GUIDE.md) (Fully)
  * [docs/HIGGS_AUDIO_V3_INLINE_TAGS.md](../../docs/HIGGS_AUDIO_V3_INLINE_TAGS.md) (Fully)
  * [docs/IndexTTS2_Emotion_Control_Guide.md](../../docs/IndexTTS2_Emotion_Control_Guide.md) (Fully)
  * [docs/INLINE_EDIT_TAGS_USER_GUIDE.md](../../docs/INLINE_EDIT_TAGS_USER_GUIDE.md) (Fully)
  * [docs/CHATTERBOX_V2_SPECIAL_TOKENS.md](../../docs/CHATTERBOX_V2_SPECIAL_TOKENS.md) (Fully)
  * [docs/MULTILINE_TTS_TAG_EDITOR_GUIDE.md](../../docs/MULTILINE_TTS_TAG_EDITOR_GUIDE.md) (Fully)
  * [docs/Dev reports/PAUSE_TAGS_IMPLEMENTATION_REPORT.md](../../docs/Dev%20reports/PAUSE_TAGS_IMPLEMENTATION_REPORT.md) (Fully)
  * [docs/ADVANCED_TTS_TAG_EDITOR_NODE.md](../../docs/ADVANCED_TTS_TAG_EDITOR_NODE.md) (Partially)
* **References**:
  * [references/syntax_standards.md](audiosuite-tag-processing/references/syntax_standards.md) — Bracket parameters comparison tables, pause markup, and engine-specific inline style tags.
  * [references/fallback_logic.md](audiosuite-tag-processing/references/fallback_logic.md) — Narrator warning fallbacks, tiered language resolution, and `#character_alias_map.txt` configuration patterns.

### 5. [audiosuite-write-skill](audiosuite-write-skill/SKILL.md)
* **Description**: Guide for writing new workspace-specific agent skills for the TTS-Audio-Suite.
* **Scope**: Enforcing naming conventions, progressive references structures, and updating index listings.
* **Consolidates**:
  * [writing-great-skills/SKILL.md](file:///home/tazztone/.gemini/config/plugins/custom-skills/skills/writing-great-skills/SKILL.md) (Adapted)

---

## 🎛️ CLI Usage

Use the global skills CLI to manage or list workspace skills:

```bash
# List all workspace and global skills
npx skills list

# Register a new skill folder
npx skills add <folder-path>

# Remove a registered skill folder
npx skills remove <folder-path>
```

### Upgrading & Installing ComfyUI Node Skills

For external node skills (like `comfyui-node-testing` from `tazztone/comfyui-custom-node-skills`), you can install and upgrade them dynamically:

```bash
# Install a specific skill
npx skills add tazztone/comfyui-custom-node-skills --skill comfyui-node-testing

# Upgrade a specific skill
npx skills@latest upgrade tazztone/comfyui-custom-node-skills --skill comfyui-node-testing

# Upgrade all skills from the repository at once
npx skills@latest upgrade tazztone/comfyui-custom-node-skills
```
