---
name: audiosuite-development
description: Guide for developing custom nodes, adapters, processors, and configuring model downloader YAML parameters in the TTS Audio Suite. Use this skill when modifying repository directory layout, registering nodes/parameters, configuring downloads, or debugging VRAM offloading/lifecycles.
---

# TTS Audio Suite Development Standards

This skill defines the directory structures, registration protocols, VRAM management conventions, and model downloading rules for developer agents working on the TTS Audio Suite.

## Quick Start
Before creating or modifying any Python code in this suite:
1. Identify the layer to change (UI Node vs. Processor vs. Adapter).
2. Register the component in `nodes.py` and `engine_registry.py`.
3. Check and implement deviceoffloading to CPU/VRAM clearing properly.

See [REST & Subprocess Architecture](references/architecture.md) for how nodes delegate to runtime environments.

## Workflows

### 1. Codebase Layering & Architecture
- [ ] Implement UI definitions strictly in `nodes/engines/` or `nodes/unified/`.
- [ ] Keep unified nodes thin (delegation only); place orchestration logic in the engine processor.
- [ ] Build the adapter layer to translate abstract method arguments into engine-specific parameters.

See [Codebase Architecture](references/architecture.md) for details on layers, workers, and runtimes.
See [Isolated Subprocess Runtimes](references/isolated_runtimes.md) for configuring worker virtual environments and JSON-RPC IPC streams.

### 2. Component Registration
- [ ] Add new node classes to `NODE_CLASS_MAPPINGS` inside [nodes.py](../../../nodes.py).
- [ ] Register new engine adapters in [engine_registry.py](../../../utils/models/engine_registry.py) and `initialize_all_factories()`.
- [ ] Update [segment_parameters.py](../../../utils/text/segment_parameters.py) to enable custom segment parameters support (e.g. `[seed:X]`).

See [Component Registration](references/registries.md) for step-by-step registry guides.

### 3. VRAM offloading & Lifecycle
- [ ] Implement a `.to(device)` method inside the underlying engine class.
- [ ] Integrate device checks before executing model generation paths.
- [ ] Add explicit VRAM unloads, garbage collection, and CUDA empty caches during clear/teardown commands.

See [VRAM & Model Lifecycles](references/vram_management.md) for memory offloading code patterns.
See [Stateless Inference Wrappers](references/stateless_wrappers.md) for breaking CUDA Graph tracking and implementing thread-safe execution.

### 4. Downloader Configurations
- [ ] Prevent any auto-downloads to user caches; map weight targets strictly under `ComfyUI/models/TTS/`.
- [ ] Define the downloading configuration mappings in YAML format.

See [Model Downloader & Layouts](references/model_downloader.md) for model YAML structure templates and download helpers.
