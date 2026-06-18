# Model Downloader & Path Layouts

To maintain an organized workspace, the suite enforces structured folder placements and uses a centralized downloader script.

---

## 1. Folder Placements & Layouts

All models, model variants, and auxiliary checkpoints must be downloaded to ComfyUI's standard model tree, never to default user cache directories (such as `~/.cache/huggingface`).

| Model Category | Folder Target Path |
|---|---|
| **TTS Engines** | `ComfyUI/models/TTS/<engine_name>/` |
| **Voice Changer** | `ComfyUI/models/RVC/` |
| **ASR (Whisper)** | `ComfyUI/models/whisper/` |
| **Vocal Separator** | `ComfyUI/models/vocal_remover/` |

* **Critical Rule**: Always check if files exist at these target paths before attempting to load or initialize them.

---

## 2. Using `UnifiedDownloader`

Download logic must use the `UnifiedDownloader` utility. It accepts target lists mapped as lists of dictionaries with `"remote"` and `"local"` keys:

```python
from utils.downloads.unified_downloader import UnifiedDownloader

def download_model_files(model_name):
    # Remote = filename on HF; Local = target path relative to models/TTS/
    files_to_download = [
        {"remote": "model.safetensors", "local": "model.safetensors"},
        {"remote": "config.json", "local": "config.json"},
    ]
    
    downloader = UnifiedDownloader(
        repo_id="HuggingFaceUser/RepoName",
        files=files_to_download,
        local_dir="TTS/my_engine_folder"
    )
    downloader.download()
```

* **Verify HF Structure**: Always run `curl` or inspect the repository on HuggingFace first. Do not guess filenames, and check if multi-part files exist (e.g. `model.safetensors-00001-of-00002`).

---

## 3. Registering Download metadata (YAML)

All download URLs and metadata must be documented in the suite's YAML files:
* Engine weights: `docs/Dev reports/tts_audio_suite_engines.yaml`
* Helper / post-processing weights: `docs/Dev reports/tts_audio_suite_aux_models.yaml`

Once updated, regenerate markdown tables for documentation:
```bash
# Regenerate main README comparison tables
python3 scripts/generate_engine_tables.py --readme

# Regenerate auxiliary docs
python3 scripts/generate_aux_model_docs.py
```
