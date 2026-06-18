# TTS Audio Suite - Repository Test Reference

This document provides repository-specific test layout information for the TTS Audio Suite. 

For the underlying custom node testing framework and architecture (isolation strategies, `conftest.py` mocking, parent `pytest.ini` interference, etc.), please refer to the standard [comfyui-node-testing](file:///home/tazztone/Applications/Data/Packages/ComfyTTS/custom_nodes/TTS-Audio-Suite/.agents/skills/comfyui-node-testing/SKILL.md) skill.

## Running Tests

Execute tests from the custom node root directory:

```bash
# Run unit tests only
python3 tests/run_tests.py -m unit

# Run integration tests (automatically manages ComfyUI server lifecycle)
python3 tests/run_tests.py -m integration

# Run all tests
python3 tests/run_tests.py --all
```

---

## Test Discovery & Inventory

To prevent documentation from going out of date, you can list and discover tests dynamically using the test runner:

```bash
# Collect and list all available tests without running them
python3 tests/run_tests.py --collect-only

# List only unit tests
python3 tests/run_tests.py -m unit --collect-only

# Show all defined pytest markers
python3 tests/run_tests.py --markers
```

### Directory Structure

- **[tests/unit/](file:///home/tazztone/Applications/Data/Packages/ComfyTTS/custom_nodes/TTS-Audio-Suite/tests/unit/)**: Standalone unit tests checking calculations, text preprocessing (like pause tagging and SRT parsing), and audio utility functions.
- **[tests/integration/](file:///home/tazztone/Applications/Data/Packages/ComfyTTS/custom_nodes/TTS-Audio-Suite/tests/integration/)**: Tests checking node registry status, workflow loaders, and end-to-end audio output validation using serialized ComfyUI pipeline configurations.

---

## Test Fixtures (`tests/fixtures/workflows/`)

Integration tests run against serialized workflow JSON configurations located under `tests/fixtures/workflows/`:

| File | Purpose |
|------|---------|
| `test_chatterbox_e2e.json` | E2E ChatterBox generation pipeline validation |
| `test_cosyvoice_e2e.json` | E2E CosyVoice3 generation pipeline validation |
| `test_chatterbox_engine.json` | ChatterBox engine node setup parsing |
| `test_cosyvoice_engine.json` | CosyVoice3 engine node setup parsing |
| `test_f5tts_engine.json` | F5-TTS engine node setup parsing |
| `test_indextts_engine.json` | IndexTTS engine node setup parsing |
| `test_moss_tts_engine.json` | MOSS-TTS engine node setup parsing |

---

## Specific Markers

Apart from standard markers, the following repo-specific markers are supported:

- `-m slow`: E2E generation tests (requires pre-downloaded model checkpoints)
- `-m cosyvoice`: CosyVoice3-specific integration runs
