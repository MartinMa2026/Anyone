<div align="center">

# Anyone.skill

> *"Distill anyone into an AI — and talk to them again."*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)
[![Based on ex-skill](https://img.shields.io/badge/Based%20on-ex--skill-orange)](https://github.com/perkfly/ex-skill)

<br>

Is there someone you wish you could talk to again?<br>
An old friend, a lost relationship, someone you never got to know well enough?<br>
The chat logs are still there. The photos are still there. The details are still there.<br>

**Distill a real person into an AI Skill — and have that conversation again.**

<br>

Provide chat logs (WeChat, iMessage, SMS), photos, social media — plus your own description.<br>
Generate an **AI Skill that thinks and speaks like them**.<br>
It knows their tone, what they care about, how they react when upset, and what makes them light up.

[Data Sources](#supported-data-sources) · [Install](#installation) · [Usage](#usage) · [Examples](#examples) · [中文](README.md)

</div>

---

## Supported Data Sources

| Source | Chat Logs | Photos | Social Media | Notes |
|--------|:---------:|:------:|:------------:|-------|
| WeChat | ✅ | — | — | Export via WechatExporter or similar |
| iMessage | ✅ | — | — | macOS chat.db or exported file |
| SMS | ✅ | — | — | Android SMS Backup XML/CSV |
| Photos | — | ✅ | — | EXIF metadata → timeline |
| Weibo | — | — | ✅ | JSON export |
| Douban | — | — | ✅ | JSON/HTML export |
| Xiaohongshu | — | — | ✅ | JSON export |
| Instagram | — | — | ✅ | JSON data export |
| PDF / Screenshot | ✅ | ✅ | — | Manual upload |
| Paste Text | ✅ | — | — | Direct input |

---

## Installation

### Claude Code

```bash
# Install to current project (run from git repo root)
mkdir -p .claude/skills
git clone https://github.com/MartinMa2026/Anyone.git .claude/skills/create-person

# Or install globally (available in all projects)
git clone https://github.com/MartinMa2026/Anyone.git ~/.claude/skills/create-person
```

### OpenClaw

```bash
git clone https://github.com/MartinMa2026/Anyone.git ~/.openclaw/workspace/skills/create-person
```

### Dependencies (optional)

```bash
pip3 install -r requirements.txt
```

> Only `pypinyin` is optional — used to auto-convert Chinese names into slugs for directory naming.

---

## Usage

In Claude Code, type:

```
/create-person
```

You'll be guided through 3 short questions: nickname, relationship info, and personality traits. Everything except the name can be skipped — a description alone is enough to generate a working Skill.

Call the generated Skill with `/{slug}`.

### Supported Relationship Types

```
Friend       college friend, known 5 years, rarely in touch, she's a PM
Colleague    ex-coworker, worked together 3 years, he's in tech
Family       mom, known forever, retired teacher
Ex           ex-girlfriend, together 3 years, college, broke up 1 year ago
Online       met in a game, played 2 years, never met IRL
Idol         favorite creator, been watching for 5 years
...anyone
```

### Commands

| Command | Description |
|---------|-------------|
| `/create-person` | Create a new Person Skill |
| `/list-persons` | List all Person Skills |
| `/{slug}` | Invoke full Skill (Persona + Memories) |
| `/{slug}-memories` | Memories only |
| `/{slug}-persona` | Persona only |
| `/person-rollback {slug} {version}` | Roll back to a previous version |
| `/delete-person {slug}` | Delete a Skill |

> 💡 Legacy commands `/create-ex`, `/list-exes` still work (backward compatible)

---

## Examples

> Clone any kind of relationship — not just exes.

**Case 1: A late grandmother (family)**
> Input: scanned handwritten letters + family description: "quiet, blunt but warm-hearted, always nagged about eating"

```
You      > Grandma, work has been so stressful lately

Grandma  > Did you eat?
         > Stress means you need to eat more
         > You've always been like this — bottling everything up
```

**Case 2: A college best friend (lost touch)**
> Input: 3 years of WeChat logs, INFP · slow to warm up · quiet · quietly opinionated

```
You      > Miss you, how've you been?

Friend   > Good enough
         > You?
         > Something happen? You never just reach out
```

**Case 3: A former colleague / mentor (work relationship)**
> Input: team group chats + email threads, rational · high standards · economical with words

```
You      > What do you think of this proposal?

Mentor   > Where's the conclusion?
         > Cite your data sources
         > Logic holds overall, but there's a gap in point three
```

**Case 4: An ex (romantic relationship)**
> Input: 2 years of iMessage, ENFP · clingy · brings up old stuff · anxious attachment

```
You      > What do you want to eat?

Ex       > Anything is fine
You      > Noodles?
Ex       > Not noodles
You      > Hot pot?
Ex       > Hehe how did you know 💕
```

---

## Features

### Skill Structure

Each Person Skill has two parts:

| Part | Contents |
|------|----------|
| **Part A — Shared Memories** | Relationship timeline, daily rituals, preferences, interaction patterns, emotional dynamics |
| **Part B — Persona** | 5-layer personality: Hard rules → Identity → Communication style → Emotional logic → Relationship behavior |

Runtime logic: `Receive message → Persona decides mood and attitude → Memories supply specific details → Respond in their voice`

### Personality Tags

**Universal:** talks a lot · quiet · slow to warm up · naturally outgoing · perfectionist · action-oriented · planner · procrastinator · emotionally stable · sensitive · easygoing · detail-oriented

**Relationship-specific:** clingy · cold shoulder · brings up old arguments · independent · runs hot and cold · tests you · possessive<br>
**Conflict style:** silent treatment · explosive · logical · apologizes first · never admits fault<br>
**Attachment style:** secure · anxious · avoidant · disorganized

### Evolution

- **Append new chat logs** → incremental analysis → merged into existing Skill without overwriting
- **Conversation correction** → say "they wouldn't do that, they'd actually..." → written to Correction layer, takes effect immediately
- **Version management** → every update is auto-archived, roll back to any previous version

---

## Project Structure

```
Anyone/
├── SKILL.md              # Skill entry point (AgentSkills frontmatter)
├── prompts/              # Prompt templates
│   ├── intake.md              #  3-question onboarding flow
│   ├── memories_analyzer.md   #  Memory extraction dimensions
│   ├── persona_analyzer.md    #  Personality analysis (with tag translation table)
│   ├── memories_builder.md    #  memories.md generation template
│   ├── persona_builder.md     #  5-layer persona template
│   ├── merger.md              #  Incremental merge logic
│   └── correction_handler.md  #  Conversation correction handler
├── tools/                # Python tools (local only, no network)
│   ├── wechat_parser.py       # WeChat chat log parser
│   ├── imessage_parser.py     # iMessage / SMS parser
│   ├── sms_parser.py          # Android SMS parser
│   ├── photo_analyzer.py      # Photo EXIF metadata analyzer
│   ├── social_media_parser.py # Social media export parser
│   ├── skill_writer.py        # Skill file manager
│   └── version_manager.py     # Version archiving and rollback
├── persons/              # Generated Person Skills (gitignored)
├── requirements.txt
└── LICENSE
```

---

## Privacy

- **All data is processed locally** — nothing is sent to any external service
- The `persons/` directory is gitignored — generated Skills and chat logs **are never uploaded**
- Photo analysis only reads EXIF metadata (date/location), not image content
- Do **not** commit chat logs or generated Skills to any public repository

---

## Notes

- **Data quality determines Skill quality** — real chat logs beat manual descriptions
- Prioritize: **long unprompted messages** > **emotional messages** > casual chat
- Works with zero data — a text description alone generates a usable Persona

---

## Credits

Anyone.skill is built on top of [perkfly/ex-skill](https://github.com/perkfly/ex-skill).

ex-skill is a Skill focused on distilling romantic relationships (ex-girlfriends/ex-boyfriends) into AI. It established the complete chat log parsing pipeline, 5-layer Persona architecture, and memory evolution mechanism that Anyone.skill is built on.

Anyone.skill extends the scope from romantic relationships to **any person**, and adds a universal personality tag system, multi-relationship-type support, and auto relationship mode detection.

---

<div align="center">

MIT License © [MartinMa2026](https://github.com/MartinMa2026)

</div>
