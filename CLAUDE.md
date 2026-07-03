# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin that delivers a personal writing voice skill. It contains:

- **`skills/ernies-voice/SKILL.md`**: The skill definition. This is the core artifact — it tells Claude how to write in the target person's voice when the skill is triggered.
- **`.claude-plugin/plugin.json`**: Plugin metadata (name, version, author, keywords).
- **`templates/personal-voice-extraction-process.md`**: The methodology document for building or updating a voice skill from a writing corpus (3-pass: Baseline → Alignment → Calibration).

The current skill file is for Ernie Oporto. The skill name in the frontmatter is `ernie-voice`.

## Key Concepts

**SKILL.md frontmatter**: The `name` and `description` fields control when Claude triggers the skill. The description should list all content types and trigger phrases so the skill activates reliably.

**3-Pass extraction process** (documented in `templates/personal-voice-extraction-process.md`):
1. **Pass 1 (Baseline)**: Analyze a writing corpus → extract voice patterns → build initial SKILL.md
2. **Pass 2 (Alignment)**: Human reviews draft, flags wrong/missing/overstated patterns → Claude revises
3. **Pass 3 (Calibration)**: Generate sample texts → human marks them up → Claude finalizes

**Corpus requirements**: 10+ documents, 2+ content types, plain text/markdown, no AI-generated content.

## Customizing for a New Person

To adapt this skill for someone else:

1. Update `plugin.json`: change `name`, `description`, `author`
2. Update `skills/ernies-voice/SKILL.md` frontmatter: change `name` and `description`
3. Rename `skills/ernies-voice/` to match the new name
4. Follow the 3-pass extraction process in `templates/` with the new person's corpus

## Installation

```bash
# Local plugin install
claude plugin add /path/to/ernies-voice/

# Or copy to plugins directory first
cp -r . ~/claude-plugins/ernies-voice/
claude plugin add ~/claude-plugins/ernies-voice/
```

## The SKILL.md Structure

The skill file is organized into these sections (order matters for Claude):
1. YAML frontmatter (trigger conditions)
2. LLM-ism ban lists (highest priority rule)
3. Inclusive language rules
4. Anti-performative-writing rules
5. Affirmative vs question-led writing
6. Core voice DNA and personality
7. Language-specific patterns (English, then French if bilingual)
8. Context-specific voice modes (Informational default vs Narrative)
9. Format-specific guidelines per content type
10. Explicit anti-patterns ("What X does NOT do")
11. Adaptation rules for the model
