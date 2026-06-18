---
name: audiosuite-audio-pipelines
description: Guide for handling audio tensors, sample rates, SRT timeline assembly, vocal separation, and RVC voice conversion inside the TTS Audio Suite. Use this skill when modifying audio processor outputs, manipulating audio shapes, merging waveforms, or adjusting subtitle timings.
---

# TTS Audio Suite Audio Pipelines & Processing

This skill defines the standards for working with audio tensors, performing resampling, timing/subtitle assembly, vocal removal, and voice conversion inside the suite.

## Quick Start
Before returning audio data from any node or processor:
1. Ensure the output is formatted as a standard ComfyUI audio dictionary.
2. Confirm the waveform tensor has a **3D shape** `[batch, channels, samples]`.
3. Check and apply resamplers if input audio sample rates differ from the engine's native rate.

See [Audio Tensors & Shapes](references/tensor_formats.md) for shape conversions.

## Workflows

### 1. Audio Formats & Resampling
- [ ] Confirm input/output audio matches: `{"waveform": Tensor, "sample_rate": int}`.
- [ ] Shape-convert waveforms to 3D: unsqueeze 1D (`[samples]`) to `[1, 1, samples]`, and 2D (`[channels, samples]`) to `[1, channels, samples]`.
- [ ] Resample reference audios to the target engine's sample rate (e.g. 24000 Hz) before processing.

See [Audio Tensors & Shapes](references/tensor_formats.md) for resampler code patterns.

### 2. SRT Timing & Subtitle Assembly
- [ ] Avoid raw model calls in SRT processors; reuse text generation and assemble timings using the timeline helpers.
- [ ] Implement `TimedAudioAssembler.assemble_timed_audio()` for stretch-to-fit timings.
- [ ] Check for interruption signals inside loops (`model_management.interrupt_processing`) to allow users to cancel execution safely.

See [SRT & Timeline Assembly](references/timing_assembly.md) for timeline APIs and interrupt checkers.

### 3. Voice Changer (RVC) & Vocal Removal (UVR5)
- [ ] Load and cache RVC/Hubert weights through the unified adapter interface.
- [ ] Implement UVR5-based source separation (HP5, MDX-NET, RoFormer) to isolate clean voice streams.
- [ ] Use `MergeAudioNode` mix algorithms (median, overlay, weighted) to assemble background music.

See [Voice Conversion & Separation](references/voice_conversion.md) for RVC configurations and UVR5 templates.
