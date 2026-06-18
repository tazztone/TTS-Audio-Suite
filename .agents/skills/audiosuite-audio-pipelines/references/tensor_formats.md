# Audio Tensors & Shapes

ComfyUI custom nodes require a specific dictionary format and 3D tensor shape for audio inputs and outputs. Deviating from these requirements will raise runtime shape/indexing errors.

---

## 1. ComfyUI Audio Dictionary Format

All audio data flowing between nodes is represented as a Python dictionary containing the raw waveform tensor and its sample rate:

```python
audio_data = {
    "waveform": torch.Tensor,     # Must be 3D tensor [batch, channels, samples]
    "sample_rate": int            # e.g., 22050, 24000, 44100, 48000
}
```

* **Safely Extracting Waveform**:
  ```python
  # Always extract waveform and sample rate safely
  waveform = audio_data.get("waveform")
  sample_rate = audio_data.get("sample_rate")
  ```

---

## 2. The 3D Tensor Requirement (CRITICAL)

ComfyUI audio nodes expect the `waveform` tensor to be **3D** `[batch, channels, samples]`. Passing 1D or 2D tensors directly causes index exceptions in audio save widgets:
`IndexError: Dimension out of range (expected to be in range of [-1, 0], but got 1) in save_audio()`

### Shape Conversion Utilities

Always format raw waveforms into 3D tensors before returning them from node execution methods:

```python
import torch

def to_comfy_audio_tensor(waveform: torch.Tensor):
    # 1D: [samples] -> 3D: [1, 1, samples]
    if waveform.dim() == 1:
        return waveform.unsqueeze(0).unsqueeze(0)
        
    # 2D: [channels, samples] -> 3D: [1, channels, samples]
    elif waveform.dim() == 2:
        return waveform.unsqueeze(0)
        
    # 3D: already [batch, channels, samples]
    elif waveform.dim() == 3:
        return waveform
        
    else:
        raise ValueError(f"Unsupported waveform dimension: {waveform.dim()}")
```

---

## 3. Resampling Reference Audio

Inference engines require reference voices (audio clues) to match their native internal sample rate (e.g. 24000 Hz for Step Audio EditX or F5-TTS).
* **Pitfall**: Failing to resample external audio causes severe pitch shifts and speed anomalies.

```python
import torchaudio.functional as F

def preprocess_reference_audio(audio_dict, target_sr=24000):
    waveform = audio_dict["waveform"]
    source_sr = audio_dict["sample_rate"]
    
    if source_sr != target_sr:
        print(f"🔄 Resampling reference voice from {source_sr}Hz to {target_sr}Hz")
        waveform = F.resample(waveform, source_sr, target_sr)
        
    return {
        "waveform": to_comfy_audio_tensor(waveform),
        "sample_rate": target_sr
    }
```
