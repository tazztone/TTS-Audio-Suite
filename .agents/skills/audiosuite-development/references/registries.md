# Component Registration & Integration

To integrate a new engine or node into the suite, you must wire it into three critical registry files. Failing to do so causes "unknown node class" errors or bypasses unified parameters like segment seeds.

---

## 1. Node Class Registration

Register your UI classes in [nodes.py](../../../nodes.py) so ComfyUI discovers and lists them in the browser canvas menus.

```python
# nodes.py
from nodes.engines.my_engine_node import MyEngineNode

# 1. Add class mappings
NODE_CLASS_MAPPINGS = {
    "MyEngineNode": MyEngineNode,
}

# 2. Add user-facing display names (use consistent emojis)
NODE_DISPLAY_NAME_MAPPINGS = {
    "MyEngineNode": "⚙️ My TTS Engine",
}
```

* **Best Practice**: Use conditional imports for nodes that require isolated environments, preventing registration failures if local requirements are missing.

---

## 2. Model Factory Registration

All engine loading must use `unified_model_interface.load_model()` to enable smart caching and prevent duplicate VRAM allocation.

Register your factory functions in [engine_registry.py](../../../utils/models/engine_registry.py):

```python
# utils/models/engine_registry.py

# 1. Define factory registration
def register_my_engine_factory():
    from utils.models.unified_model_interface import unified_model_interface
    
    def factory(config, device):
        from engines.my_engine.my_engine import MyEngine
        return MyEngine(config=config, device=device)
        
    unified_model_interface.register_factory("my_engine_name", factory)

# 2. Add to global initialization
def initialize_all_factories():
    register_f5tts_factory()
    register_chatterbox_factory()
    register_my_engine_factory() # Register here
```

---

## 3. Segment Parameter Registration

The suite supports inline parameters like `[seed:42]`. For a new engine to recognize segment overrides, register it in [segment_parameters.py](../../../utils/text/segment_parameters.py).

```python
# utils/text/segment_parameters.py

# Add your engine name to PARAMETER_ENGINES list
PARAMETER_ENGINES = [
    "f5tts",
    "chatterbox",
    "vibevoice",
    "indextts",
    "my_engine_name" # Add engine name here
]
```

* **Effect**: Allows the text preprocessor to parse inline parameters (e.g. `[seed:123]`) and pass them as dictionary arguments down to the adapter/processor.
