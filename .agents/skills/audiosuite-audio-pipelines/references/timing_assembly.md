# SRT & Timeline Assembly

SRT subtitle processors orchestrate generation timings. They must never call raw models directly or duplicate generation logic. Instead, they reuse the main TTS text processors and assemble outputs using the suite's shared timing systems.

---

## 1. SRT Processor Loop Pattern

SRT processors iterate over subtitle chunks, calling `processor.process_text()` for each cue. They must periodically check for interruption signals:

```python
import comfy.model_management as model_management

class MyEngineSRTProcessor:
    def generate_srt_speech(self, srt_content, ...):
        srt_segments = self.parse_srt(srt_content)
        audio_segments = []
        
        for i, segment in enumerate(srt_segments):
            # CRITICAL: Check for user interruption before starting segment
            if model_management.interrupt_processing:
                raise InterruptedError(f"SRT generation cancelled at cue {i+1}/{len(srt_segments)}")
                
            # Delegate to core text processor
            audio = self.text_processor.process_text(segment.text, ...)
            audio_segments.append((segment, audio))
            
        # Assemble timings
        return self.assemble_timeline(audio_segments)
```

---

## 2. Shared Timing & Assembly APIs

Never guess the naming of timing and assembly methods. Use the exact APIs imported from `utils/timing/`:

| Timing Strategy | Class & Method | Description |
|---|---|---|
| **stretch_to_fit** | `TimedAudioAssembler.assemble_timed_audio(...)` | Stretches segment duration to fit strict SRT cue window. |
| **pad_with_silence**| `AudioAssemblyEngine.assemble_with_overlaps(...)` | Pads gaps with silence and handles overlapping subtitle boundaries. |
| **concatenate** | `TimingEngine.calculate_concatenation_adjustments(...)` + `assemble_concatenation(...)` | Glues clips end-to-end and offsets timing labels. |
| **smart_natural** | `TimingEngine.calculate_smart_timing_adjustments(...)` + `assemble_smart_natural(...)` | Optimizes speaking speed dynamically around gap sizes. |

---

## 3. Reporting with ChunkCombiner

To output structured generation reports (duration, characters, segments timing breakdowns), use `ChunkCombiner` and `ChunkTimingHelper`:

```python
from utils.audio.chunk_combiner import ChunkCombiner
from utils.timing.reporting import ChunkTimingHelper

# 1. Combine segments and extract timing metadata
combined_audio, chunk_info = ChunkCombiner.combine_chunks(
    chunks=audio_segments,
    return_info=True
)

# 2. Enhance prompt report details
enhanced_info = ChunkTimingHelper.enhance_generation_info(
    base_info={"engine": "my_engine"},
    chunk_info=chunk_info
)
```
