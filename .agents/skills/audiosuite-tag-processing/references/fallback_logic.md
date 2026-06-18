# Fallback Logic & Alias Mappings

To ensure a seamless user experience, the suite uses a robust voice fallback system and a tiered language priority hierarchy.

---

## 1. Character Voice Fallbacks

When a speaker is defined in the text (e.g. `[Alice]`) but no corresponding audio file exists in `voices_examples/` or `models/voices/`:
1. Log a warning to console: `⚠️ Using main voice for character 'Alice' (not found in voice folders)`.
2. Fall back automatically to the narrator / main reference voice.
3. **Never raise a crash** or stop prompt execution.

---

## 2. Language Priority Hierarchy

When resolving which language model or token encoder to load, evaluate parameters in this strict priority order:

```
[ Explicit Language Tag ]  (e.g., [de:Alice])
          │
          ▼
[ Character Default Language ]  (defined in #character_alias_map.txt)
          │
          ▼
[ Global Node Default Language ]  (selected in ComfyUI Node Dropdown)
```

### Example Behavior:
* `[de:Alice]` $\rightarrow$ Alice speaks in German (explicit override).
* `[Alice]` $\rightarrow$ Alice uses German if she has a default `de` mapped in the alias file.
* If no mapping exists $\rightarrow$ Alice uses the global node setting (e.g., English `en`).

---

## 3. Character Alias Configuration (`#character_alias_map.txt`)

Users organize friendly speaker names and map default languages inside `#character_alias_map.txt` located in `voices_examples/` or `models/voices/`.

### Supported Formats:
The parser recognizes both equal-sign and tab-separated formats, ignoring comments (`#`) and empty lines:

```
# Format 1: Tab-Separated (Name, Voice-Folder-Name, Default-Language)
Alice        female_01      de
Bob          male_01        fr

# Format 2: Equal-Sign (Name = Voice-Folder-Name, Default-Language)
Cowboy = clint_eastwood_enhanced, en
Shopkeeper = male_02
```

### Parsing implementation:
When reading segments, locate character aliases using `get_character_mapping()` which returns `(voice_filename, language_code)`. If it returns `(None, None)`, ensure you fall back gracefully to the narrator.
