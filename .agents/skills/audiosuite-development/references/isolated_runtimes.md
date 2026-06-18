# Isolated Subprocess Runtimes

Some engines require fragile dependency versions (e.g. pinned old versions of `librosa` or `descript-audiotools` that force numpy downgrades). To prevent package version conflicts in the main process, these engines must run inside isolated subprocess runtimes.

---

## 1. Virtual Environment Setup

Isolated runtimes maintain separate python environments under the `runtimes/` or `utils/runtimes/` directories.

* **No-Deps Installations**:
  When installing wheels inside isolated environments, use the `--no-deps` pip flag if dependencies conflict with other system libraries:
  ```bash
  pip install descript-audiotools --no-deps
  ```

* **Dynamic MSVC Compiler Detection**:
  Engines utilizing `torch.compile` (like Qwen3-TTS) on Windows require a C++ compiler (`cl.exe`). The suite detects Visual Studio and injects MSVC paths into the worker's subprocess environment:
  ```python
  import os
  import subprocess
  
  def get_worker_env():
      env = os.environ.copy()
      # Auto-detect VS path and inject MSVC build toolchain to env["PATH"]
      # prevents "compiler missing" warnings inside isolated torch compilation
      return env
  ```

---

## 2. JSON-RPC IPC Protocol

Communication between the main ComfyUI process and the isolated worker happens over standard input/output (`stdin`/`stdout`) streams using a JSON-RPC payload schema.

### Message Stream Mapping
* **Request** (Client to Subprocess):
  ```json
  {"jsonrpc": "2.0", "method": "generate", "params": {"text": "hello", "seed": 42}, "id": 1}
  ```
* **Response** (Subprocess to Client):
  ```json
  {"jsonrpc": "2.0", "result": {"audio_path": "/tmp/output.wav"}, "id": 1}
  ```

### Handling Subprocess Lifecycles in Python:
```python
import subprocess
import json

class SubprocessWorker:
    def __init__(self, python_exe, script_path):
        self.process = subprocess.Popen(
            [python_exe, script_path],
            stdin=subprocess.PIPE,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True
        )
        
    def call_method(self, method, params):
        payload = json.dumps({"jsonrpc": "2.0", "method": method, "params": params, "id": 1})
        self.process.stdin.write(payload + "\n")
        self.process.stdin.flush()
        
        response_line = self.process.stdout.readline()
        return json.loads(response_line).get("result")
        
    def terminate(self):
        if self.process:
            self.process.terminate()
            self.process.wait()
```
