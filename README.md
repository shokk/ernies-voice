# ernies-voice

A Claude Code plugin that makes Claude write like me instead of like a corporate chatbot.

Most AI-generated text has a tell. It's the words: "actionable," "holistic," "transformative."
It's the em dashes. It's the balanced-on-one-hand-on-the-other structure when one side is
clearly right. It's the sycophantic opener and the formulaic close. This plugin teaches Claude
to avoid all of that and write in my actual voice instead.

## What this is

A [Claude Code](https://claude.ai/code) skill plugin. Install it once, and whenever you ask
Claude to write something in your voice, it pulls the skill automatically. No prompting
required beyond "write this in my voice" or "draft a blog post."

The skill is built from real writing: blog posts at [shokk.com](https://shokk.com), Reddit
comments at [/u/shokk](https://reddit.com/user/shokk), and Threads posts at
[@shokk](https://threads.com/@shokk). IT and security professional. NJ-based. Opinionated
about AI, infosec, and tech policy, with a recurring moral lens on all three.

## Installation

```bash
/plugin marketplace add shokk/ernies-voice
/plugin install ernies-voice@shokk
```

Or locally, if you've cloned the repo:

```bash
claude plugin add /path/to/ernies-voice/
```

## Usage

Once installed, the skill triggers automatically when you ask Claude to write in your voice.
Phrases that activate it: "write this for me," "draft a post," "in my style," "my voice,"
"write a blog post," "Threads post."

Examples:

```
Write a blog post in my voice about the EU AI Act's enforcement gap.

Draft a Threads post about the SolarWinds ruling.

Rewrite this paragraph in my voice.
```

## What's in this repo

```
.claude-plugin/
  plugin.json              Plugin metadata
skills/
  ernies-voice/
    SKILL.md               The voice definition: banned words, opening patterns,
                           format rules, and everything else Claude needs
templates/
  personal-voice-extraction-process.md
                           The 3-pass methodology used to build this skill from
                           a writing corpus — useful if you want to build your own
```

## Building your own

The `templates/` directory contains the full extraction process: how to collect a writing
corpus, run it through three passes of analysis and calibration, and produce a SKILL.md
that actually sounds like you. It was originally developed by
[Sam Dumont](https://github.com/sam-dumont/claude-skills) and adapted here.

The short version: gather 10+ samples of your own writing across at least two formats,
follow the three-pass prompting process in the template, and iterate until the generated
samples sound like you wrote them on a normal day, not like you were trying to impress someone.

## License

MIT
