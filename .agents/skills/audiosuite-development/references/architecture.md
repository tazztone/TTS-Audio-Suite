# Codebase Layering & Architecture

The TTS Audio Suite uses a strict, unified modular layering pattern across all engines to isolate UI concerns, business logic, parameter mapping, and model runtimes.

## The Layered Modular Architecture

Every engine integration must be separated into the following distinct components:

```
[ ComfyUI Graph Canvas ]
           ↓
 1. Engine Configuration Node (UI Layer)
           ↓
 2. Unified Nodes (Thin Routing Layer)
           ↓
 3. Engine Processor (Orchestration & Text Processing)
           ↓
 4. Engine Adapter (Translation & Unified Interface)
           ↓
 5. Engine Wrapper / Subprocess (Inference Execution)
```

### 1. Engine Configuration Node (`nodes/engines/<engine>_engine_node.py`)
Provides the engine-specific UI widgets and inputs. Exposes **only native official parameters** to prevent UI clutter and false capabilities.

### 2. Unified Nodes (`nodes/unified/`)
The `Unified TTS Text` and `Unified SRT TTS` nodes are thin routing layers. They receive the unified prompt and delegate execution directly to the active engine's processor. 
* **Critical Rule**: Do not put engine-specific orchestration or execution logic into unified node files.

### 3. Engine Processor (`nodes/<engine>/<engine>_processor.py`)
Coordinates text preprocessing, pause tags, character/narrator switching, and text chunking/combination.
* **Critical Rule**: Caching, chunking, and loop-based progress feedback belong in the processor. Do not push chunking down to the adapter.

### 4. Engine Adapter (`engines/adapters/<engine>_adapter.py`)
Bridges the processor to the underlying engine implementation. Translates the standardized processor args into the exact shapes and parameters expected by the raw inference wrapper.

### 5. Engine Inference Wrapper (`engines/<engine>/<engine_name>.py`)
Initializes the model files, places them on CUDA/CPU, and runs the official inference scripts.

---

## Inference Execution Runtimes

Engines are run in one of two runtime environments depending on their package dependencies:

### A. Main Environment (Transformers 5)
Engines with standard dependencies that do not conflict with the core package environment run natively in the main process.

### B. Isolated Subprocess Runtimes (`utils/runtimes/`)
Fragile engine families with severe dependency conflicts (e.g. pinned old versions of numpy, librosa, or transformers) must be isolated.
* **Engines in Isolated Runtimes**: VibeVoice, Qwen3-TTS / ASR, Granite forced alignment, and Higgs Audio 2.
* **Routing Rule**: Runtime subprocess routing happens globally through `ModelLoadConfig.runtime_mode` and `runtime_profile`, not through ad-hoc subprocess spawning.
