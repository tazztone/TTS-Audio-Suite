# Inline Syntax Standards

To support multi-speaker, parameters, and style controls, the suite uses a standardized bracket-based tagging syntax.

---

## 1. Character & Narrator Tags
* **Format**: `[CharacterName]Text content...`
* **Aliases**: Map custom user names to actual voice filenames using `#character_alias_map.txt`.
* **Cleaning**: Character tags are case-insensitive. Trailing punctuation like colons is automatically stripped (`[Alice:]` $\rightarrow$ `alice`).

---

## 2. Per-Segment Parameter Overrides
* **Format**: `[CharacterName|param:val|param:val]` (order-independent)
* **Pipes**: Multiple parameters are separated by pipes (`|`).
* **Universal Parameters**:
  * `seed` (int, `0-4294967295`)
  * `temperature` (alias: `temp`, float, `0.1-2.0`)
* **Engine-Specific Parameters**:

| Engine | Parameter | Alias | Range | Description |
|---|---|---|---|---|
| **ChatterBox** | `cfg` | `cfg_weight` | `0.0-1.0` | Speaker guidance strength |
| | `exaggeration` | `exag` | `0.0-2.0` | Emotion exaggeration |
| **F5-TTS** | `cfg` | — | `0.0-20.0` | CFG strength |
| | `speed` | — | `0.5-2.0` | Speed multiplier |
| **VibeVoice** | `cfg` | — | `0.0-20.0` | CFG strength |
| | `top_p` | `topp` | `0.0-1.0` | Nucleus sampling |
| | `inference_steps` | `steps` | `1-100` | Inference steps |
| **IndexTTS-2** | `cfg` | — | `0.0-20.0` | CFG strength |
| | `top_p` | `topp` | `0.0-1.0` | Nucleus sampling |
| | `top_k` | `topk` | `1-100` | Top-K sampling |

---

## 3. Pause Tags
* **Format**: `[pause:2]`, `[pause:1.5s]`, `[pause:500ms]`
* **Parser API**: Always extract pauses using `PauseTagProcessor.parse_pause_tags(text)` which parses brackets and returns list of `("text", str)` or `("pause", duration_seconds)` tuples.

---

## 4. Special Inline Engine Tags

Some advanced engines support special inline emotional or sound effect tokens:

### Step Audio EditX
* **Format**: `<Laughter>`, `<emotion:happy>`, `<style:whisper:2>`, `<restore:1@2>`

### Higgs Audio v3
* **Format**: `<|emotion:amusement|>`, `<|style:whispering|>`, `<|prosody:pause|>`

### CosyVoice3
* **Format**: `<breath>`, `<laughing>text here</laughing>`
