# Writing Skills Guidelines

Adhere to the following formatting, structure, and trigger guidelines when writing or updating agent skills for this repository.

---

## 1. Directory Structure

Each skill must follow a consistent modular layout:
```text
.agents/skills/audiosuite-<name>/
├── SKILL.md                 # Main orchestration & checklist (strictly < 100 lines)
└── references/              # Detailed execution/reference guides
    ├── guidelines.md
    └── ...
```

---

## 2. Main SKILL.md Format

Every `SKILL.md` must start with a YAML frontmatter block and keep the main body lean:

```markdown
---
name: audiosuite-<name>
description: Guide for <high-level summary>. Use this skill when the user wants to <trigger 1>, needs to <trigger 2>, or asks to <trigger 3>.
---

# Title of Skill

Brief description of what this skill orchestrates.

## Quick Start
Provide immediate constraints, checklist preconditions, or key context pointers.

## Workflows
Include sequential markdown checklist steps with explicit completion criteria.
```

---

## 3. Writing Effective Triggers

The frontmatter `description` determines when the agent invokes the skill:
- **Third-Person**: Use third-person active verbs ("Use this skill when...").
- **Rich Triggers**: Mention specific APIs, files, patterns, or terminology (e.g. `Unified ASR`, `SRT timeline assembly`, `VRAM offloading`) to hook the model's intent.
- **One Trigger Per Branch**: Avoid redundant synonyms. Keep the triggers distinct.

---

## 4. Progressive Disclosure

Keep the primary checklist short so the agent does not lose focus on the happy path.
- **Context Pointers**: Link directly to reference guides: `[Link Text](references/filename.md)`.
- **References Directory**: Put code details, API payloads, configurations, and complex architectures inside the `references/` subfolder.
- **Link Formatting**: Use relative file links. Never surround the link text with backticks.
  * **Correct**: `[guidelines](references/guidelines.md)`
  * **Incorrect**: `[`guidelines`](references/guidelines.md)`
