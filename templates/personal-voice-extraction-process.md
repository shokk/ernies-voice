# Personal Voice Extraction Process for Claude Code

A portable, self-contained process for building a personal voice skill from a corpus of your
own writing samples. Run this document on any machine with Claude Code installed.

## Prerequisites

- **Claude Code**: Anthropic's CLI tool for Claude. Install from https://claude.ai/claude-code
- **SKILL.md**: A markdown file with YAML frontmatter that defines a Claude Code skill. Claude
  reads the SKILL.md to understand how to behave when the skill is triggered.
- **Plugin**: A directory containing a `plugin.json` (metadata) and one or more skills. Claude
  Code discovers and loads plugins from configured directories.
- **This document**: Copy it to the target machine. All prompts reference this file on disk,
  so Claude Code can read the skeletons and templates it contains.

## What This Produces

By the end of this process, you will have:

1. **A personal voice skill** (`your-voice/SKILL.md`): defines your writing patterns, usable
   by Claude Code for content generation in your voice.
2. **A plugin package** (`plugin.json` + skill directory): installable as a Claude Code plugin.

Optionally, if you write in two languages:

3. **Bilingual voice coverage**: your voice patterns in both your primary and secondary
   language, integrated into a single SKILL.md.

## Methodology: 3-Pass Extraction

| Pass | What happens | Who does work | Output |
|------|-------------|---------------|--------|
| **Pass 1: Baseline** | Claude analyzes corpus, extracts voice patterns, builds initial SKILL.md | Claude (2 prompts) | Draft SKILL.md, analysis report |
| **Pass 2: Alignment** | You review the draft, flag inaccuracies, add missing patterns. Claude revises. | You + Claude (1 prompt) | Revised SKILL.md |
| **Pass 3: Calibration** | Claude generates sample texts, you mark up what's wrong, Claude incorporates feedback. | You + Claude (2 prompts) | Final SKILL.md |

Each pass has a verification checklist. Do not proceed to the next pass until all items pass.

---

## Part 1: Project Setup

### 1.1 Directory Structure

Create this structure on the target machine:

```
voice-project/
├── corpus/                          # Your writing samples, organized by content type
│   ├── blog/
│   │   ├── post-about-topic.md
│   │   └── another-post.md
│   ├── slack/
│   │   └── thread-about-migration.md
│   ├── docs/
│   │   └── architecture-decision-record.md
│   ├── email/
│   │   └── client-update.md
│   └── reddit/
│       └── forum-replies.md
├── analysis/                        # Generated during Pass 1-2 (don't create manually)
├── output/                          # Final skill files (generated during Pass 1-3)
└── notes.md                         # Your observations (used in Pass 2 and 3)
```

### 1.2 Corpus Preparation Rules

Follow these rules when collecting writing samples:

| Rule | Requirement | Why |
|------|------------|-----|
| **Minimum samples** | 10+ documents | Statistical significance for pattern extraction |
| **Content type diversity** | 2+ content types | Separates personal style from platform conventions |
| **Language coverage** | Include samples in both languages if you write bilingually | Needed for bilingual voice patterns |
| **Recency** | Prefer samples from the last 2 years | Writing styles evolve |
| **Authenticity** | Only include text you actually wrote | AI-generated content in the corpus poisons the extraction |
| **Length variety** | Mix short (Slack) and long (blog) samples | Voice manifests differently at different lengths |
| **Format** | Plain text or markdown, one document per file | Consistent parsing |
| **Filenames** | Descriptive: `migration-announcement-slack.md`, not `doc1.md` | Context helps during analysis |
| **Confidentiality** | Redact sensitive info (client names, credentials) | The corpus is processed locally but keep it clean |

**How to collect samples:**

- **Slack/Teams**: Export messages or copy-paste threads. Include enough context (the full
  thread, not just one reply).
- **Blog posts**: Copy the markdown source or the rendered text.
- **Docs**: Export from Confluence/Notion/Google Docs as markdown.
- **Email**: Copy-paste the body (strip headers, signatures unless they're custom).
- **Reddit/forums**: Copy-paste your comments with enough surrounding context.

### 1.3 Pre-flight Checklist

Before starting Pass 1, verify:

- [ ] Corpus has 10+ documents total
- [ ] At least 2 content types are represented
- [ ] Files are plain text or markdown (no .docx, .pdf, .html)
- [ ] No AI-generated content in the corpus (check if unsure)
- [ ] Sensitive information is redacted
- [ ] If bilingual: samples exist in both languages

---

## Part 2: Pass 1 - Baseline Extraction

This pass runs two prompts sequentially. Each prompt is self-contained: copy-paste it
into Claude Code as-is. Do not modify the prompts unless noted.

### Prompt 1A: Corpus Analysis + Pattern Extraction

Copy this entire block and paste it into Claude Code:

````
I need you to analyze a writing corpus to extract personal voice patterns.

**Step 1: Inventory the corpus.**
Scan the `corpus/` directory. For each content type subdirectory:
- Count the number of files
- Note the languages used
- Flag if there are fewer than 10 total files (warn but continue)
- Note the content types found

Write the inventory to `analysis/corpus-inventory.md`.

**Step 2: Read and analyze all corpus files.**
Read ALL files in the corpus and extract patterns across these 8 dimensions:

A. **Sentence patterns**: average length, variation, use of short punchy sentences vs long
   explanatory ones, parenthetical asides, how you handle lists.

B. **Opening patterns**: how you start blog posts, Slack messages, emails, docs. First
   sentence structure. Do you use greetings? Jump straight in? Context-set first?

C. **Vocabulary fingerprint**: recurring words/phrases, technical term handling (do you
   explain jargon or assume knowledge?), informal language, code-switching between languages.

D. **Structural patterns**: how you organize information (chronological, importance-first,
   problem-solution), use of headers, bullet points, numbered lists, visual aids.

E. **Tone markers**: formality level per content type, humor usage, directness level,
   how you handle disagreement, how you express uncertainty.

F. **Formatting habits**: bold usage, caps usage, emoji usage, punctuation quirks
   (Oxford comma? French spacing? Em dashes or colons?), code block usage.

G. **Language-specific patterns** (if bilingual): do you code-switch? Which words stay
   in the other language? Does formality differ between languages? Greeting/closing
   conventions per language.

H. **LLM-ism presence**: flag any patterns that look like typical AI-generated text
   (these might indicate AI-assisted writing in the corpus, or might be false positives
   from common business language). Note but don't filter yet.

**Step 3: Platform filtering.**
For each identified pattern, classify it:
- **VOICE**: appears across multiple content types, or deviates from the platform norm
  for that content type. This is a genuine personal voice pattern.
- **PLATFORM**: only appears in one content type and matches that platform's standard
  conventions (e.g., short messages in Slack, formal greetings in email). Not your voice,
  just the platform's convention.
- **BORDERLINE**: appears mostly in one content type but seems like a deliberate choice
  rather than a convention (e.g., unusually formal Slack messages, emoji in technical docs).

Keep VOICE patterns. Flag BORDERLINE patterns for human review in Pass 2.
Filter out PLATFORM patterns (but document them for transparency).

**Step 4: Build a customized LLM-ism ban list.**
Start from this research-backed base list of overrepresented LLM words (Kobak et al. 2025,
Liang et al. 2024):

Adjectives: delve/delving, actionable, bespoke, bustling, captivating, commendable,
comprehensive (as filler), cutting-edge, daunting, ever-evolving, game-changing,
groundbreaking, holistic, impactful, innovative, insightful, intricate, invaluable,
meticulous, multifaceted, notable, noteworthy, nuanced, paramount, pivotal, profound,
remarkable, revolutionary, robust (outside technical), seamless, state-of-the-art,
substantial, tailored, thought-provoking, transformative, unparalleled, unwavering,
vibrant, whimsical

Nouns: beacon, blueprint (metaphorical), ecosystem (metaphorical), enigma, frontier,
implications (vague), interplay, intricacies, journey (metaphorical), kaleidoscope,
landscape (metaphorical), linchpin, metropolis, milestone (metaphorical), paradigm,
paradigm shift, plethora, quest, realm, resonance, roadmap (metaphorical), synergy,
tapestry, testament, touchpoint, treasure trove, value proposition

Verbs: align (metaphorical), amplify, bolster, champion, craft (as "craft a solution"),
delve, elevate, embark, empower, encompass, enhance, enrich, facilitate, foster,
harness, leverage, navigate (metaphorical), pioneer, resonate, safeguard, shed light,
showcase, spearhead, streamline (outside technical), supercharge, tailor, transcend,
underscore, uncover, unleash, unlock, unpack, unravel, unveil, utilize (just say "use")

Adverbs: accordingly, arguably, consequently, effectively (filler), essentially (filler),
furthermore, hence, holistically, meticulously, moreover, namely, nonetheless,
notwithstanding, seamlessly, significantly (filler), strategically, subsequently,
substantially, thereby, therefore (overused), thus, ultimately (overused), undeniably,
undoubtedly

Then: scan the corpus for any of these words that you ACTUALLY use genuinely (e.g.,
"robust" in a technical context about system resilience). Remove those from the ban list
and note the exception with context. Add any personal LLM-isms found in the corpus
(words that appear suspiciously often and feel AI-generated).

Also build a banned phrases list covering:
- Scene-setting openers ("In today's fast-paced world...")
- Importance-flagging filler ("It's important to note...")
- Grandiose action phrases ("Navigate the complexities of...")
- Structural gimmicks (compulsive triads, machine-gunned short sentences)
- Sycophantic openers ("Great question!")
- Formulaic closings ("I hope this helps!")

If you write in a second language, build a parallel ban list for that language.

**Step 5: Write the voice profile.**
Write to `analysis/voice-profile.md`:
- Corpus statistics (total files, per content type, per language)
- The full voice pattern analysis organized by the 8 dimensions
- VOICE / PLATFORM / BORDERLINE classification for each pattern
- The customized LLM-ism ban list with exceptions noted
- Summary: "Your voice is characterized by..." (3-5 sentences)
- Any warnings (low sample counts, suspected AI content, missing content types)
````

**After running Prompt 1A**: Read `analysis/voice-profile.md`. Verify:
- The pattern classifications make sense
- Platform filtering didn't remove patterns that are genuine personal choices
- The ban list has appropriate exceptions for words you genuinely use
- The summary captures how you actually write

### Prompt 1B: Build SKILL.md + Plugin Structure

Copy this entire block and paste it into Claude Code:

````
I need you to build the initial personal voice SKILL.md and plugin structure from the
analysis results.

**Step 1: Read the inputs.**
Read `analysis/voice-profile.md` and all files in `corpus/` (for reference).

**Step 2: Create the output directory structure.**
```
output/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── your-voice/
│       └── SKILL.md
└── README.md
```

**Step 3: Write plugin.json.**
Use this template:

```json
{
  "name": "your-voice",
  "version": "0.1.0",
  "description": "Apply [Your Name]'s writing voice and style for content generation.",
  "author": {
    "name": "[Your Name]"
  },
  "license": "MIT",
  "keywords": ["writing", "style", "voice"]
}
```

Replace `[Your Name]` with the name found in the corpus (from email signatures,
blog post bylines, etc.). If not found, use a placeholder and note it for the user to fill in.

**Step 4: Write the personal voice SKILL.md.**
Read the file `personal-voice-extraction-process.md` in this project directory, specifically
Part 5 ("SKILL.md Skeleton"). Use that skeleton structure as your template. For each section
marked [DERIVED], populate it from the voice profile analysis. For sections marked [STANDARD],
use the provided template text.

Key rules for writing the SKILL.md:
- Every pattern must cite specific corpus evidence (e.g., "seen in 8/12 blog posts")
- Platform-filtered patterns are excluded
- The LLM-ism ban list goes in full, with your specific exceptions noted
- Format-specific guidelines are organized by the content types found in the corpus
- If bilingual: include a secondary language section. If secondary language samples exist,
  derive patterns from them. If no samples exist but you want bilingual support, write
  "INFERRED" patterns based on primary language voice + standard conventions for the
  secondary language, and mark them with <!-- INFERRED --> for validation in Pass 3.

**Step 5: Write README.md.**
Brief README explaining what the plugin does and how to install it.

Write all files to `output/`.
````

**After running Prompt 1B**: Verify the output directory contains all expected files.
Read `output/skills/your-voice/SKILL.md` end-to-end.

### Pass 1 Verification Criteria

Do not proceed to Pass 2 until ALL items pass:

| # | Criterion | How to check | Pass? |
|---|-----------|-------------|-------|
| 1 | SKILL.md has a frontmatter block with name and description | Read first 10 lines | |
| 2 | SKILL.md has an LLM-ism ban list section | Search for "ban" or "banned" | |
| 3 | Patterns cite corpus evidence (e.g., "seen in 8/12 posts") | Scan the document | |
| 4 | Platform-filtered patterns are excluded from voice rules | Cross-ref with analysis | |
| 5 | Format-specific guidelines exist for each content type in the corpus | Check section headers | |
| 6 | If bilingual: secondary language section exists (derived or INFERRED) | Check section headers | |
| 7 | No dangling references to other files (SKILL.md is self-contained) | Read through fully | |
| 8 | plugin.json is valid JSON with all fields | Parse it | |
| 9 | Directory structure matches the specification | `find output/ -type f` | |

---

## Part 3: Pass 2 - Alignment

### Before You Start Pass 2

This pass requires YOUR input. Before running the prompt, do the following:

**2.1 Review the draft SKILL.md** (`output/skills/your-voice/SKILL.md`):
- Read it section by section
- Does it capture how you actually write?
- Are there patterns you recognize that are missing?
- Are there patterns listed that feel wrong or overstated?

**2.2 Write your notes** in `notes.md` in the project root. Use this template:

```markdown
# Pass 2 Review Notes

## WRONG: patterns that are incorrect
- [Pattern name]: [What's wrong and what it should be]
- Example: "Uses em dashes for clarification": No, I use colons. Never em dashes.

## OVERSTATED: patterns that are exaggerated
- [Pattern name]: [What the real frequency/usage is]
- Example: "Heavy use of ALL CAPS": Only once or twice per text, not every paragraph.

## MISSING: patterns not captured
- [Describe the pattern]: [Give an example from your writing]
- Example: I always write affirmatively ("we realized X") not through rhetorical
  question setups ("we asked ourselves: what are we getting?").

## NEEDS_NUANCE: patterns that need context
- [Pattern name]: [What context to add]
- Example: "Direct openings": Yes for Slack, but blog posts usually have a one-line
  context-setter first.

## Additional corpus samples (optional)
If you realized samples are missing, add them to corpus/ and list them here:
- Added: corpus/reddit/cycling-forum-replies.md
- Added: corpus/chat/slack-team-messages.md

## Secondary language feedback (if applicable)
- [Any corrections to secondary language patterns]
- [INFERRED patterns that are wrong]
```

**2.3 If you added new corpus files**, make sure they follow the corpus preparation rules
from Part 1.

### Prompt 2A: Gap Analysis + Revision

Copy this entire block and paste it into Claude Code:

````
I need you to revise the personal voice SKILL.md based on my review feedback and optional
new corpus samples.

**Step 1: Read the inputs.**
Read all of these:
- `notes.md` (review notes from Pass 2 preparation)
- `output/skills/your-voice/SKILL.md` (current draft)
- `analysis/voice-profile.md`
- Any new files added to `corpus/` since Pass 1 (check file modification dates or
  look for files mentioned in notes.md)

**Step 2: Process feedback.**
For each item in notes.md:
- WRONG: verify against corpus. If the feedback is correct, remove or replace the pattern.
  If the corpus genuinely supports the pattern, explain why in a comment but defer to the
  author's judgment: you know your own writing better than the corpus shows.
- OVERSTATED: downgrade the pattern description to match actual frequency.
- MISSING: verify by scanning the corpus for evidence. Add the pattern with corpus citations
  and examples.
- NEEDS_NUANCE: update the pattern description to include the context-dependent behavior.
- SECONDARY LANGUAGE FEEDBACK: incorporate corrections. If INFERRED patterns were wrong,
  fix them. If new samples were added, re-analyze the secondary language patterns.

**Step 3: Process new corpus files (if any).**
If new files were added to corpus/:
- Analyze them using the same framework as Prompt 1A
- Update the voice profile
- Check if any patterns change classification (VOICE/PLATFORM/BORDERLINE)

**Step 4: Gap analysis.**
Compare the revised SKILL.md against the full corpus one more time:
- Are there patterns in the corpus that the SKILL.md still doesn't capture?
- Are there contradictions within the SKILL.md?
- If bilingual: is the secondary language section consistent with the primary
  (same voice, different language)?

**Step 5: Write the revised SKILL.md.**
Overwrite `output/skills/your-voice/SKILL.md` with the revised version.
Mark all changes with inline comments: `<!-- PASS2: [what changed and why] -->`.
These comments will be removed after Pass 3.

Also write `analysis/pass2-changelog.md` documenting all changes made.

**Step 6: Self-check.**
Re-read the revised SKILL.md and verify:
- No LLM-isms in the SKILL.md text itself
- All patterns cite corpus evidence
- No dangling references
- If bilingual: secondary language section is present and consistent
- Format-specific guidelines cover all content types
````

**After running Prompt 2A**: Read `analysis/pass2-changelog.md` to see what changed.
Read the revised SKILL.md and verify the changes match your feedback.

### Pass 2 Verification Criteria

| # | Criterion | How to check | Pass? |
|---|-----------|-------------|-------|
| 1 | All WRONG items from notes.md are addressed | Cross-ref with changelog | |
| 2 | All OVERSTATED items are downgraded | Cross-ref with changelog | |
| 3 | All MISSING patterns are added (if corpus-supported) | Cross-ref with changelog | |
| 4 | All NEEDS_NUANCE items have updated descriptions | Cross-ref with changelog | |
| 5 | Secondary language corrections incorporated (if applicable) | Check language sections | |
| 6 | New corpus files (if any) are reflected in the analysis | Check changelog | |
| 7 | PASS2 comments mark every change | Search for `<!-- PASS2` | |
| 8 | No new LLM-isms introduced in the revision | Read through | |

---

## Part 4: Pass 3 - Calibration

### Prompt 3A: Generate Calibration Samples

Copy this entire block and paste it into Claude Code:

````
I need you to generate calibration test samples using the personal voice skill to verify
it produces accurate output.

**Step 1: Read the inputs.**
Read `output/skills/your-voice/SKILL.md` (the Pass 2 revised version).

**Step 2: Identify the content types and languages in the corpus.**
Look at the corpus directory structure and the SKILL.md format-specific guidelines.
Note which content types have dedicated format modes and which languages are covered.

**Step 3: Generate calibration samples in the primary language.**
Generate 5 samples in the primary language. Each sample should:
- Represent a different content type found in the corpus
- Be realistic (something you might actually write)
- Be long enough to exhibit voice patterns (minimum 150 words each)

Choose 5 from this list, prioritizing content types that are in the corpus:

1. **Blog post opening** (first 3 paragraphs): about a technical project or decision.
   Tests: opening style, technical explanation, tone.

2. **Slack/Teams announcement**: telling a team about a planned change.
   Tests: directness, formatting, action items.

3. **Technical documentation excerpt**: a section explaining a design decision.
   Tests: structure, argumentation, technical vocabulary.

4. **Professional email**: updating someone on project progress or a delay.
   Tests: professional tone, clarity, register.

5. **Forum/Reddit comment**: responding to someone's technical question.
   Tests: casual register, helpfulness, personality.

6. **Incident post-mortem summary**: what went wrong and recommendations.
   Tests: factual reporting, technical detail.

7. **Chat/DM message**: quick back-and-forth about a work topic.
   Tests: casual register, message structure.

**Step 4: Generate secondary language samples (if bilingual).**
If the SKILL.md has a secondary language section, generate 2-3 samples:
- One casual (Slack/chat style)
- One professional (email/doc style)
- One that demonstrates code-switching patterns (if applicable)

**Step 5: Format the output.**
Write all samples to `analysis/calibration-samples.md` with clear headers and
separators between samples. Each sample should be preceded by:
- Sample number and type
- Target content type and language
- Which voice patterns this sample is designed to test

Do NOT include any meta-commentary within the samples themselves. They should read
as if you actually wrote them.
````

**After running Prompt 3A**: Read `analysis/calibration-samples.md`. Now comes the
most important step: YOUR feedback.

### User Markup: How to Give Feedback on Calibration Samples

For each sample, mark it up using this system. Write your feedback in `notes.md`
(append to the file, don't overwrite Pass 2 notes).

```markdown
# Pass 3 Calibration Feedback

## Sample 1: [type]
**Overall**: [GOOD / CLOSE / OFF]
**Specific issues:**
- Line "...exact quote...": [TOO_FORMAL / TOO_CASUAL / WRONG_WORD / LLM_ISM / NOT_ME]
  Correction: "...how I'd actually write this..."
- Line "...exact quote...": [MISSING_PATTERN] I always do X here but the sample doesn't.
**What's RIGHT about this sample:**
- [Note what feels authentic so Claude knows to keep it]

## Sample 2: [type]
**Overall**: [GOOD / CLOSE / OFF]
...

[Repeat for all samples]

## General calibration notes
- [Overall observations across all samples]
- [Patterns that are consistently wrong]
- [Patterns that are consistently right]
```

**Feedback labels explained:**

| Label | Meaning | Maps to SKILL.md section |
|-------|---------|--------------------------|
| `GOOD` | Sounds like I wrote it | No changes needed |
| `CLOSE` | Almost right, few tweaks | Minor adjustments |
| `OFF` | Doesn't sound like me | Major revision needed |
| `TOO_FORMAL` | Register is too high | Format-specific modes |
| `TOO_CASUAL` | Register is too low | Format-specific modes |
| `WRONG_WORD` | Vocabulary miss | Vocabulary patterns or ban list |
| `LLM_ISM` | AI-generated sounding | Ban lists |
| `NOT_ME` | Correct info, wrong voice | Core voice patterns |
| `MISSING_PATTERN` | Pattern should appear but doesn't | Add to relevant section |

### Prompt 3B: Incorporate Feedback + Finalize

Copy this entire block and paste it into Claude Code:

````
I need you to incorporate calibration feedback into the personal voice SKILL.md and
finalize it.

**Step 1: Read the inputs.**
Read all of these:
- `notes.md` (Pass 3 calibration feedback, appended after Pass 2 notes)
- `output/skills/your-voice/SKILL.md` (current version)
- `analysis/calibration-samples.md` (the samples that were evaluated)

**Step 2: Analyze the calibration feedback.**
For each sample's feedback:
- GOOD samples: identify what the SKILL.md got right (reinforce these patterns)
- CLOSE samples: trace each issue back to a specific SKILL.md section and fix it
- OFF samples: this indicates a systemic issue. Identify which SKILL.md sections are
  responsible and significantly revise them

Group the issues by root cause:
- Missing pattern? Add it.
- Wrong pattern? Fix it.
- Priority issue (right pattern, wrong emphasis)? Adjust the description.
- Format-specific issue (right pattern, wrong context)? Add context qualifiers.

**Step 3: Revise the personal voice SKILL.md.**
Apply all fixes. Remove the `<!-- PASS2 -->` comments from Pass 2 (those changes are
now baseline). The SKILL.md should be clean and final.

Ensure the final SKILL.md:
- Has no inline comments or TODO markers
- Has corpus evidence citations for all patterns
- Has clean formatting (consistent headers, no broken markdown)
- Is fully self-contained (no references to analysis files or external context)
- Has the complete LLM-ism ban list
- If bilingual: has both primary and secondary language sections

**Step 4: Re-generate 2 calibration samples.**
Using the finalized SKILL.md, re-generate:
- The sample that was rated worst (most OFF) in the feedback
- One secondary language sample (if bilingual), or another primary language sample

Write these to `analysis/final-calibration-check.md` so you can do a quick sanity check.

**Step 5: Update plugin.json version.**
Change version to "1.0.0" in `output/.claude-plugin/plugin.json`.

**Step 6: Final file inventory.**
Write `output/MANIFEST.md` listing every file in output/ with a one-line description.
````

**After running Prompt 3B**: Read the re-generated calibration samples in
`analysis/final-calibration-check.md`. If they still sound off, you can iterate:
add more feedback to `notes.md` and re-run Prompt 3B. Typically one iteration is enough.

### Pass 3 Verification Criteria

| # | Criterion | How to check | Pass? |
|---|-----------|-------------|-------|
| 1 | All calibration feedback items are addressed | Cross-ref notes.md with SKILL.md | |
| 2 | No `<!-- PASS2 -->` comments remain | Search the file | |
| 3 | No TODO markers or placeholder text | Search for "TODO", "PLACEHOLDER", "TBD" | |
| 4 | Re-generated samples sound better than originals | Your judgment | |
| 5 | plugin.json version is "1.0.0" | Read the file | |
| 6 | MANIFEST.md lists all output files | Cross-ref with `find output/` | |
| 7 | SKILL.md has no dangling refs to analysis files | Read through | |

---

## Part 5: SKILL.md Skeleton

Use this skeleton when building the personal voice SKILL.md in Prompt 1B. Sections marked
`[DERIVED]` are populated from corpus analysis. Sections marked `[STANDARD]` use the
template text provided.

```markdown
---
name: your-voice
description: >
  Apply [Your Name]'s writing voice and style when drafting [list content types].
  Use this skill when asked to write, draft, review, or improve text in [your] voice.
  [If bilingual: Works in both [primary language] and [secondary language].]
---

# [Your Name] - Writing Voice Guide

This skill defines [Your Name]'s writing voice, extracted from [N] writing samples
across [M] content types. The voice represents patterns consistently found in your
writing, filtered for platform-specific conventions.

## CRITICAL: Zero tolerance for LLM-isms

[Your Name] DESPISES typical AI-generated language. Before producing ANY text using
this skill, mentally scan your output for every item in the ban lists below and replace
or remove them. This is the single most important rule in this skill. If the output
reads like it could have been generated by a chatbot, it has failed.

The ban lists below are not exhaustive. The general principle: if a word or phrase feels
like something a corporate chatbot would write, do not use it.

### Banned words ([primary language])

[STANDARD base list from Kobak/Liang research]
[DERIVED exceptions: words from the ban list that the author genuinely uses, with context]

Organize by part of speech: Adjectives, Nouns, Verbs, Adverbs.

### Banned phrases and patterns ([primary language])

[STANDARD base list of phrase patterns]
[DERIVED additions: any author-specific phrases to ban]

Cover these categories:
- Scene-setting openers
- Importance-flagging filler
- Grandiose action phrases
- Description templates
- Sycophantic openers
- Formulaic closings
- Narrator voice / story-announcing
- Overused transitions
- Structural gimmicks

### Banned words and phrases ([secondary language])

[DERIVED: secondary language LLM-isms]
[Only include this section if bilingual]

### Formatting rules (anti-LLM)

[DERIVED: formatting patterns that distinguish this author's writing from AI output]

Common patterns to check:
- Em dash usage (many humans don't use them; LLMs overuse them)
- Paragraph length uniformity (LLMs produce uniform lengths; humans vary)
- Bold/emphasis patterns
- Emoji usage rules
- Punctuation quirks

## CRITICAL: Inclusive and gender-neutral language

[STANDARD - include if found in corpus or as a default best practice]

### Terminology replacements

| Never use | Use instead |
|-----------|-------------|
| blacklist | ban list, block list, deny list |
| whitelist | allow list, permit list |
| master/slave | primary/replica, leader/follower |
| master branch | main branch |
| sanity check | confidence check, coherence check |
| dummy value | placeholder value, sample value |

### Pronouns and gender

- Use "they/them" as default singular pronoun when gender is unknown
- Never assume gender from names, roles, or context
- Use gender-neutral job titles

## CRITICAL: No performative writing

[STANDARD]

When you study someone's writing to imitate it, the temptation is to turn their natural
habits into theatrical "signature moves". Don't.

Rules:
- **Don't manufacture catchphrases.** If a phrase was used once in a specific context,
  that doesn't make it a trademark.
- **Don't announce the narrative.** Don't say "so here's the story" or "buckle up"
  or "let me tell you about". Just start talking about the thing.
- **Don't perform casualness.** If the text feels like it's trying hard to sound casual,
  it's failing.
- **Don't over-polish the arc.** Real writing is a bit messy. Not every paragraph needs
  to land perfectly.
- **When in doubt, be boring rather than performative.** A straightforward sentence that
  says what happened will always sound more like you than a cleverly constructed one.

## CRITICAL: Question-Led vs Affirmative Writing

[DERIVED: determine from corpus whether the author writes affirmatively or uses
rhetorical question setups. Most people should default to affirmative.]

Include concrete wrong/correct examples derived from the corpus:
- **WRONG (Question-led)**: "I asked myself: what am I actually getting?"
- **CORRECT (Affirmative)**: "I realized I was spending too much for too little."

## Core Voice DNA

[DERIVED: 3-5 sentence summary of the author's voice personality]

### Personality in writing

[DERIVED: bullet points describing personality traits visible in writing]

## Language-Specific Patterns

### [Primary Language] Voice

[DERIVED: extracted patterns organized by:]

**Sentence structure:**
[DERIVED]

**Opening patterns:**
[DERIVED: how the author starts each format type]

**Vocabulary and register:**
[DERIVED]

**Structural patterns:**
[DERIVED]

### [Secondary Language] Voice

[DERIVED or INFERRED]
[Only include this section if bilingual]

[DERIVED: secondary language patterns organized by:]
- Register (formal/informal conventions)
- Vocabulary and expressions
- Code-switching patterns
- Greeting/closing conventions

If INFERRED (no secondary language samples in corpus), mark:
<!-- INFERRED: These patterns are derived from the primary language voice
combined with standard conventions. Validate in Pass 3. -->

## Context-Specific Voice Modes

[DERIVED: determine the author's default mode from the corpus]

**CRITICAL**: Not everything is a story. Default to informational unless the format
specifically calls for narrative.

### Mode 1: Informational (DEFAULT)
**Use for**: Internal docs, reports, technical documentation, Slack messages, READMEs

Characteristics:
- Lead with the key information
- Explain what and why, skip the journey
- Direct, factual statements
- No narrative arc

### Mode 2: Narrative (ONLY when specifically writing blog posts/published content)
**Use for**: Blog posts, opinion pieces, LinkedIn posts

Characteristics:
- Can have Problem > Solution structure
- Personal angle and motivation
- Still no LLM-isms or performative writing

[DERIVED: determine which content types fall into which mode]

## Format-Specific Guidelines

[DERIVED: one subsection per content type found in the corpus]

### [Content Type 1, e.g., Blog Posts]
[DERIVED]

### [Content Type 2, e.g., Slack Messages]
[DERIVED]

### [Content Type N]
[DERIVED]

## What [Your Name] Does NOT Do

[DERIVED: explicit anti-patterns observed from the corpus]
[Things the author never does, stated as prohibitions]

## Adaptation Rules

[STANDARD with DERIVED elements]

When writing for [Your Name]:

1. **Ask about the audience** if not obvious - the register shifts between formats.
2. **Default to direct.** You can always soften later, starting too formal feels wrong.
3. **Same voice across languages.** If the primary language is punchy, the secondary
   language should be too.
4. **When translating between languages,** preserve the energy level and directness.
5. **Technical accuracy is non-negotiable.**
6. **After writing, re-read for LLM-isms AND performative writing.** Two separate passes.
   First kill the chatbot phrases. Then kill anything that feels like it's trying too hard
   to sound like a character rather than a person.
```

---

## Part 6: Edge Cases

### Monolingual authors

If you only write in one language, skip the secondary language sections throughout:
- No secondary language ban list needed
- No secondary language voice patterns section
- No bilingual calibration samples in Pass 3
- The SKILL.md will be shorter (and that's fine)

### Corpus too small

**Minimum viable**: 5 documents across 2 content types. The process will still run, but:
- Pattern confidence will be lower
- Platform filtering will be less reliable (fewer data points)
- Pass 2 (your feedback) becomes more important to compensate
- Consider adding more samples during Pass 2 or Pass 3

**Ideal**: 15+ documents across 3+ content types. This gives the extraction enough data
to find patterns you didn't know you had.

### Platform conventions vs voice patterns

Some patterns exist in a gray zone. A formal email greeting might be your choice or
the platform's convention. The process classifies these as BORDERLINE in Prompt 1A.

**Resolution**: In Pass 2, you decide. If you use formal greetings because you want to
(not because the platform expects it), promote it to VOICE. If you'd write differently
on another platform, it's PLATFORM.

### AI-generated content in corpus

If you used AI to help write some samples:
- **Heavily AI-edited** (you prompted, AI wrote, you published): exclude from corpus
- **Lightly AI-edited** (you wrote, AI fixed grammar/typos): probably fine to include
- **AI-assisted brainstorming** (AI helped outline, you wrote the text): fine to include
- **When in doubt**: exclude. Better to have fewer genuine samples than contaminated ones

Prompt 1A flags suspected AI content. If it flags something you genuinely wrote,
note it in Pass 2 feedback.

---

## Part 7: Deployment

### 7.1 File Inventory

After completing all 3 passes, `output/` should contain:

```
output/
├── .claude-plugin/
│   └── plugin.json                    # Plugin metadata, version 1.0.0
├── skills/
│   └── your-voice/
│       └── SKILL.md                   # Voice skill (final, no comments/TODOs)
├── README.md                          # Plugin documentation
└── MANIFEST.md                        # File inventory with descriptions
```

### 7.2 Installation

**Option A: Local plugin**

```bash
# Copy the output to your plugins directory
cp -r output/ ~/claude-plugins/your-voice/

# Register in Claude Code
claude plugin add ~/claude-plugins/your-voice/
```

**Option B: Git repository**

Push the output directory as a standalone repo or as part of a plugin collection:

```bash
# As a standalone plugin repo
cd output/
git init
git add .
git commit -m "Personal voice skill v1.0.0"
git remote add origin <your-repo-url>
git push -u origin main

# Then install via marketplace
/plugin marketplace add <your-username>/<repo-name>
/plugin install your-voice@<marketplace-name>
```

**Verification**: Start a new Claude Code session and ask it to write a short text
in your voice. Verify the output matches your style.

### 7.3 Maintenance

Your voice will drift over time. Re-run the process when:
- You notice Claude's output no longer sounds like you
- You've started writing in a new format (add samples, re-run)
- Your style has genuinely evolved (yearly re-extraction is reasonable)

To update, keep the `corpus/` and `analysis/` directories alongside the output.
Adding new samples and re-running from Pass 1 is the cleanest approach.

---

## Appendix: Reference

This process was derived from the 3-pass extraction methodology described in
[Building a custom voice skill for Claude Code](https://dropbars.be/blog/building-custom-voice-skill-claude-code/).
