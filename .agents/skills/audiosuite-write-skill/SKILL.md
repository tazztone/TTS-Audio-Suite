---
name: audiosuite-write-skill
description: Guide for creating new workspace-specific agent skills for the TTS-Audio-Suite. Use this skill when the user requests a new skill, wants to document repository patterns, or asks to format/reorganize existing skills.
---

# Audiosuite Skills Writer

This meta-skill guides AI developer agents through the process of writing clean, localized, and predictable developer skills for this repository, adhering to the `audiosuite` prefix and structure conventions.

## Quick Start
Before creating a new agent skill:
1. Confirm it covers a specific TTS-Audio-Suite capability that is not already covered.
2. Prefix the skill directory and frontmatter name with `audiosuite-`.
3. Plan to split the skill: keep `SKILL.md` under 100 lines and use `references/` for detailed guides.

See [Writing Skills Guidelines](references/guidelines.md) for trigger blocks and layout rules.

## Workflows

### 1. Planning & Scoping
- [ ] Determine the triggers and branches. Do not create a new skill if the concept can be collapsed into an existing skill.
- [ ] Review `docs/` and identify which documentation files will be consolidated into the new skill.
- [ ] Define the information hierarchy (the sequence steps vs. demand reference guides).

### 2. Drafting the Skill Folder
- [ ] Initialize the folder under `.agents/skills/audiosuite-<name>/`.
- [ ] Create `SKILL.md` with:
  * Frontmatter block specifying the name and triggers (in third person).
  * Main checklist / workflow steps (strictly under 100 lines).
  * Context pointers to the detailed guides under `references/`.
- [ ] Create detailed reference guides under `references/` (e.g. `references/api_details.md`).

### 3. Registration & Documentation
- [ ] Add the new skill to the workspace skills using:
  ```bash
  npx skills add .agents/skills/audiosuite-<name>
  ```
- [ ] Update the central [.agents/skills/README.md](../README.md) catalog list, mapping out descriptions, references, and a **Consolidates** list of the source `docs/` files.
