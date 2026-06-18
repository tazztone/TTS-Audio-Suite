# Voice Conversion & Source Separation

The suite features Retrieval-based Voice Conversion (RVC) and UVR5 source separation pipelines to transform voices and isolate vocals from multi-track audio.

---

## 1. RVC Voice Conversion Architecture

RVC nodes translate the pitch and timber characteristics of a source audio into a target voice model using pre-trained Hubert content encoders.

```
[ Source Audio ] ──> [ RVC Pitch Extractor ] ──> [ Hubert Content Encoder ]
                                                          │
                                                          ▼
                                                 [ RVC Inference Engine ] ──> [ Converted Audio ]
```

### Integration Handler
In the main `VoiceChangerNode`, delegate RVC execution using `_handle_rvc_conversion()` which manages formatting, model cache lookups, and parameter boundaries:

```python
def _handle_rvc_conversion(self, source_audio, rvc_model_name, hubert_model_name, **params):
    # 1. Fetch cached model instances
    rvc_engine = unified_model_interface.load_model({
        "engine_name": "rvc",
        "model_type": "voice_changer",
        "model_name": rvc_model_name,
        "hubert_name": hubert_model_name
    })
    
    # 2. Extract raw waveform and sample rate
    waveform = source_audio.get("waveform")
    sr = source_audio.get("sample_rate")
    
    # 3. Execute conversion
    converted_waveform, output_sr = rvc_engine.convert(
        waveform=waveform,
        sample_rate=sr,
        f0_method=params.get("f0_method", "rmvpe"),
        index_rate=params.get("index_rate", 0.75),
        protect_rate=params.get("protect_rate", 0.33)
    )
    
    return {"waveform": converted_waveform, "sample_rate": output_sr}
```

---

## 2. Pitch Extraction & Parameters

RVC conversion quality depends heavily on the pitch extraction algorithm:
* **Crepe / Mangio-Crepe**: Accurate but slow. Hop length settings balance performance.
* **RMVPE**: High-quality, fast, and robust across clean and noisy vocals.
* **FCPE**: Highly modern, specialized for speech conversion.

---

## 3. UVR5 Source Separation (Vocal Removal)

To isolate vocal tracks from background music before RVC conversion, use the UVR5 models:
* **Models**: HP5, MDX-NET, RoFormer.
* **Outputs**: Returns a dual dictionary output for `vocals` and `instrumentals`.
* **Caching**: Result caching must be implemented in the separation processor to speed up repeated runs during workflow iterations.

---

## 4. Mixing with Merge Audio Node

To recombine vocals and backing tracks, use `MergeAudioNode` mixing algorithms:
* `mean` / `median`: Simple overlays.
* `weighted`: Sets volume balance to prevent clipping.
* `crossfade`: Implements linear transitions between tracks.
