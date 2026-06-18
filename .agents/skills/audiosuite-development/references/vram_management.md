# VRAM & Model Lifecycles

In ComfyUI, models are offloaded to CPU dynamically to make room for other nodes. Your engine wrapper must support CPU offloading and VRAM clearing without leaking memory or throwing device mismatch exceptions.

---

## 1. Implement `.to(device)` on Engine Classes

Every engine class must implement a `.to()` method to recursively move its neural networks between CPU and CUDA devices.

```python
def to(self, device):
    """
    Move all model sub-modules to the target device.
    Required for ComfyUI's CPU offload & VRAM clearing system.
    """
    self.device = device
    
    if self._model is not None:
        # 1. Simple single model offload
        if hasattr(self._model, 'to'):
            try:
                self._model = self._model.to(device)
            except ValueError as e:
                # Catch BitsAndBytes quantized (int4/int8) model limitations
                if "is not supported for" in str(e) or "8-bit" in str(e):
                    print(f"⚠️ Quantized model components cannot be moved to CPU via .to()")
                else:
                    raise e
                    
        # 2. Multi-component offload
        for component_name in ['s3gen', 't3', 'voice_encoder', 'tokenizer']:
            if hasattr(self._model, component_name):
                comp = getattr(self._model, component_name)
                if hasattr(comp, 'to'):
                    setattr(self._model, component_name, comp.to(device))
                    
    return self
```

---

## 2. Check and Reload Before Generation

Models are often offloaded silently to CPU. Verify the device of the model parameters before starting generation:

```python
def generate(self, text, reference_audio, **kwargs):
    target_device = "cuda" if torch.cuda.is_available() else "cpu"
    
    if self._model is not None:
        # Check first parameter's device
        if hasattr(self._model, 'parameters'):
            try:
                first_param = next(self._model.parameters())
                current_device = str(first_param.device)
                
                # If offloaded to CPU, reload to CUDA
                if current_device != target_device:
                    print(f"🔄 Reloading from {current_device} to {target_device}")
                    self.to(target_device)
            except StopIteration:
                pass
```

---

## 3. Explicit VRAM Teardown / Unloading

To completely free memory when switching engines or clicking "Clear VRAM", define an explicit unload method. Do NOT rely on python's `__del__` destructors as they can cause premature unloads.

```python
def unload_model(self):
    """
    Drop references, clear caches, and invoke garbage collection.
    """
    if hasattr(self, '_model'):
        self._model = None
        
    import gc
    import torch
    
    gc.collect()
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        torch.cuda.ipc_collect()
```
