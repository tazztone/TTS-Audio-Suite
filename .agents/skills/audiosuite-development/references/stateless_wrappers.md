# Stateless Inference Wrappers

To enable safe model unloading and support multi-threaded concurrent generation, engine calls must be completely stateless. Memory leaks often occur when PyTorch tensors hold persistent references (such as within CUDA Graphs).

---

## 1. Breaking CUDA Graph References

CUDA Graphs capture static tensor memory addresses. If input reference audio or generated outputs are not detached, PyTorch cannot free the underlying VRAM, blocking "Clear VRAM" calls.

* **Detaching & Cloning Tensors**:
  Always detach and clone reference inputs and output audio tensors to break references to the model's internal memory footprint:
  ```python
  def isolate_tensor(tensor):
      if tensor is not None:
          # detach() drops backprop graph; clone() allocates fresh memory
          return tensor.detach().clone().cpu()
      return None
  ```

---

## 2. Stateless Execution Pattern

Avoid storing generation state variables directly on the main engine object. Instead, instantiate fresh dictionaries or call stateless functions:

```python
class StatelessEngineWrapper:
    def __init__(self, wrapped_engine):
        self._wrapped_engine = wrapped_engine
        
    def generate(self, *args, **kwargs):
        # Create fresh isolated copies of inputs to break graph tracking
        isolated_kwargs = {}
        for key, value in kwargs.items():
            if isinstance(value, torch.Tensor):
                isolated_kwargs[key] = value.detach().clone()
            else:
                isolated_kwargs[key] = value
                
        # Run inference in a thread-safe way
        result_audio, result_info = self._wrapped_engine.engine.generate(*args, **isolated_kwargs)
        
        # Isolate and return the output audio
        return self._isolate_output(result_audio), result_info
        
    def _isolate_output(self, audio):
        if isinstance(audio, dict):
            return {
                "waveform": audio["waveform"].detach().clone().cpu(),
                "sample_rate": audio["sample_rate"]
            }
        return audio.detach().clone().cpu()
```

---

## 3. Safe Dynamic Unloading

By isolating all inputs and outputs using the stateless wrapper, you break the object graphs. When `unload_model()` drops references to the engine, garbage collection is guaranteed to free the GPU VRAM instantly without leaving orphan CUDA memory segments behind.
