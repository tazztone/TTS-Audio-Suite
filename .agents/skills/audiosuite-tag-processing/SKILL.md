---
name: audiosuite-tag-processing
description: Guide for text tag parsing, character switching, segment parameter overrides, pause processing, and language default priorities in the TTS Audio Suite. Use this skill when modifying text preprocessors, resolving narrator voices, managing character alias mapping, or updating tag syntax tables.
---

# TTS Audio Suite Tag Processing & Parsing

This skill defines the standards for parsing markup tags, managing speaker/narrator changes, processing pause annotations, and handling parameter overrides inside text blocks.

## Quick Start
Before running text through an adapter or inference pipeline:
1. Parse character/narrator segments using `CharacterParser`.
2. Extract pause cues using `PauseTagProcessor.parse_pause_tags()`.
3. Check and apply inline parameter overrides (`|seed:X|temp:Y`) using segment mappings.

See [Inline Syntax Standards](references/syntax_standards.md) for tags format.

## Workflows

### 1. Tag Syntax Validation
- [ ] Confirm character tags are lowercase, cleaned of colons, and mapped via aliases: `[CharacterName]`.
- [ ] Extract pauses in milliseconds or seconds: `[pause:500ms]` or `[pause:2.5s]`.
- [ ] Group per-segment parameters inside character blocks using pipes: `[Alice|seed:42|temp:0.7]`.

See [Inline Syntax Standards](references/syntax_standards.md) for syntax rules and comparison tables.

### 2. Narrator Fallbacks & Alias Matching
- [ ] Map narrator fallbacks if a character is not found inside the voice folder directory.
- [ ] Parse default languages and speaker mappings from `#character_alias_map.txt`.
- [ ] Apply language mappings based on priority: explicit tag (`[de:Alice]`) $\rightarrow$ character default $\rightarrow$ global default.

See [Alias Mapping & Fallbacks](references/fallback_logic.md) for resolving character default languages and voice mapping arrays.
