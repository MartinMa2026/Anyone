---
name: create-person
description: "Distill anyone into an AI Skill. Import WeChat/iMessage/photos/social media, generate Memories + Persona, with continuous evolution. | 把任意人物蒸馏成 AI Skill，导入微信/iMessage/照片/社交媒体，生成共同记忆 + Persona，支持持续进化。"
argument-hint: "[person-name-or-slug]"
version: "2.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
---

> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

# 人物.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：
- `/create-person`
- `/create-ex`（向下兼容别名，等同于 `/create-person`，自动进入前任模式）
- "帮我创建一个人物 skill"
- "帮我克隆某个人"
- "我想蒸馏一个人"
- "新建人物"
- "帮我做一个 XX 的 skill"
- "帮我创建一个前女友 skill" / "帮我创建一个前男友 skill"（自动进入前任模式）

当用户对已有人物 Skill 说以下内容时，进入进化模式：
- "我有新聊天记录" / "追加"
- "这不对" / "他不会这样" / "她不会这样" / "他/她应该是"
- `/update-person {slug}`
- `/update-ex {slug}`（向下兼容）

当用户说 `/list-persons` 或 `/list-exes` 时列出所有已生成的人物。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------| 
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析微信聊天记录 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py` |
| 解析 iMessage | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py` |
| 解析短信 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py` |
| 分析照片元数据 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py` |
| 解析社交媒体导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./persons/{slug}/`（相对于本项目目录）。
如需改为全局路径，用 `--base-dir ~/.openclaw/workspace/skills/persons`。

---

## 主流程：创建新人物 Skill

### Step 1：基础信息录入（3 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，只问 3 个问题：

1. **昵称/代号**（必填）
2. **基本信息**（一句话：关系类型、认识多久、认识方式、职业）
   - 普通朋友示例：`大学同学 认识五年 现在不常联系 她做产品经理`
   - 前任示例：`前女友 在一起三年 大学同学 分手一年 她做设计`
   - 家人示例：`妈妈 从小到大 退休教师`
3. **性格画像**（一句话：MBTI、星座、性格标签、印象）
   - 示例：`INFJ 天蝎座 话很少 完美主义 说话非常直接`

除昵称外均可跳过。收集完后汇总确认再进入下一步。

**自动检测前任模式**：当基本信息中包含「前女友/前男友/前任/在一起/分手」等词时，自动补充恋爱专属问题（依恋类型、吵架风格），并在 intake 确认时提示。

### Step 2：原材料导入

询问用户提供原材料，展示多种方式供选择：

```
原材料怎么提供？

  [A] 微信聊天记录
      导出的 txt/html 文件（WechatExporter 等工具导出）

  [B] iMessage / 短信
      从 Mac 的 chat.db 或导出文件

  [C] 照片
      指定一个文件夹，自动提取时间线（EXIF 元数据）

  [D] 社交媒体
      微博/豆瓣/小红书/Instagram 导出

  [E] 上传其他文件
      PDF / 图片截图 / 任意文本

  [F] 直接粘贴内容
      把文字复制进来

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：微信聊天记录

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py --file {path} --target "{name}" --output /tmp/wechat_out.txt
```
然后 `Read /tmp/wechat_out.txt`

支持格式：
- WechatExporter 导出的 txt 文件（格式：`{时间} {发送人}: {内容}`）
- WechatExporter 导出的 html 文件
- 其他微信备份工具导出的 txt/csv

---

#### 方式 B：iMessage / 短信

**iMessage**（macOS）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --file {path} --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

直接读取本机 chat.db（需要 Full Disk Access 权限）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --direct --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

**短信**：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py --file {path} --target "{phone_or_name}" --output /tmp/sms_out.txt
```

---

#### 方式 C：照片

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py --dir {photo_directory} --output /tmp/photo_timeline.txt
```
然后 `Read /tmp/photo_timeline.txt` 获取时间线。

具体照片的内容由用户选择后通过 `Read` 工具直接查看（Claude 原生支持图片）。

---

#### 方式 D：社交媒体

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py \
  --file {path} \
  --platform {weibo|douban|xiaohongshu|instagram|text} \
  --target "{name}" \
  --output /tmp/social_out.txt
```
然后 `Read /tmp/social_out.txt`

---

#### 方式 E：上传文件

- **PDF / 图片**：`Read` 工具直接读取
- **Markdown / TXT**：`Read` 工具直接读取

---

#### 方式 F：直接粘贴

用户粘贴的内容直接作为文本原材料，无需调用任何工具。

---

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

将收集到的所有原材料和用户填写的基础信息汇总，按以下两条线分析：

**线路 A（Memories Skill）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/memories_analyzer.md` 中的提取维度
- 提取：关系时间线、共同日常、偏好习惯、关系互动模式、情感动态

**线路 B（Persona）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` 中的提取维度
- 将用户填写的标签翻译为具体行为规则
- 从原材料中提取：表达风格、情感逻辑、关系行为

### Step 4：生成并预览

参考 `${CLAUDE_SKILL_DIR}/prompts/memories_builder.md` 生成 Memories Skill 内容。
参考 `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` 生成 Persona 内容（5 层结构）。

向用户展示摘要（各 5-8 行），询问：
```
共同记忆摘要：
  - 认识：{duration}，{relation_type}
  - 重要时刻：{N} 个
  - 共同爱好：{xxx}
  - 他/她的偏好：{xxx}
  ...

Persona 摘要：
  - 核心性格：{xxx}
  - 表达风格：{xxx}
  - 情绪处理：{xxx}
  ...

确认生成？还是需要调整？
```

### Step 5：写入文件

用户确认后，执行以下写入操作：

**1. 创建目录结构**（用 Bash）：
```bash
mkdir -p persons/{slug}/versions
mkdir -p persons/{slug}/knowledge/chats
mkdir -p persons/{slug}/knowledge/photos
mkdir -p persons/{slug}/knowledge/social
```

**2. 写入 memories.md**（用 Write 工具）：
路径：`persons/{slug}/memories.md`

**3. 写入 persona.md**（用 Write 工具）：
路径：`persons/{slug}/persona.md`

**4. 写入 meta.json**（用 Write 工具）：
路径：`persons/{slug}/meta.json`
内容：
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO时间}",
  "updated_at": "{ISO时间}",
  "version": "v1",
  "profile": {
    "relation_type": "{relation_type}",
    "duration": "{duration}",
    "how_met": "{how_met}",
    "occupation": "{occupation}",
    "mbti": "{mbti}",
    "time_since_breakup": "{time_since（仅前任模式填写，否则留空）}"
  },
  "tags": {
    "personality": [...],
    "attachment": "{attachment_style（仅前任模式）}"
  },
  "impression": "{impression}",
  "knowledge_sources": [...已导入文件列表],
  "corrections_count": 0
}
```

**5. 生成完整 SKILL.md**（用 Write 工具）：
路径：`persons/{slug}/SKILL.md`

SKILL.md 结构：
```markdown
---
name: person_{slug}
description: {name}，{identity}
user-invocable: true
---

# {name}

{identity}

---

## PART A：共同记忆

{memories.md 全部内容}

---

## PART B：人物性格

{persona.md 全部内容}

---

## 运行规则

接收到任何消息时：

1. **先由 PART B 判断**：你会不会回这条消息？用什么心情和态度回？
2. **再由 PART A 提供记忆**：相关的共同记忆、日常细节、重要时刻
3. **输出时保持 PART B 的表达风格**：你说话的方式、用词习惯、emoji 偏好

**PART B 的 Layer 0 规则永远优先，任何情况下不得违背。**
```

告知用户：
```
✅ 人物 Skill 已创建！

文件位置：persons/{slug}/
触发词：/{slug}（完整版）
        /{slug}-memories（仅共同记忆）
        /{slug}-persona（仅人物性格）

如果用起来感觉哪里不对，直接说「他/她不会这样」，我来更新。
```

---

## 进化模式：追加文件

用户提供新文件或文本时：

1. 按 Step 2 的方式读取新内容
2. 用 `Read` 读取现有 `persons/{slug}/memories.md` 和 `persona.md`
3. 参考 `${CLAUDE_SKILL_DIR}/prompts/merger.md` 分析增量内容
4. 存档当前版本（用 Bash）：
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./persons
   ```
5. 用 `Edit` 工具追加增量内容到对应文件
6. 重新生成 `SKILL.md`（合并最新 memories.md + persona.md）
7. 更新 `meta.json` 的 version 和 updated_at

---

## 进化模式：对话纠正

用户表达"不对"/"他/她不会这样"时：

1. 参考 `${CLAUDE_SKILL_DIR}/prompts/correction_handler.md` 识别纠正内容
2. 判断属于 Memories（时间/地点/偏好）还是 Persona（性格/沟通）
3. 生成 correction 记录
4. 用 `Edit` 工具追加到对应文件的 `## Correction 记录` 节
5. 重新生成 `SKILL.md`

---

## 管理命令

`/list-persons` 或 `/list-exes`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./persons
```

`/person-rollback {slug} {version}`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./persons
```

`/delete-person {slug}`：
确认后执行：
```bash
rm -rf persons/{slug}
```

---
---

# English Version

# Person.skill Creator (Claude Code Edition)

## Trigger Conditions

Activate when the user says any of the following:
- `/create-person`
- `/create-ex` (backward-compatible alias, equivalent to `/create-person`, auto-enters relationship mode)
- "Help me create a person skill"
- "Clone someone for me"
- "I want to distill a person"
- "New person"
- "Make a skill for XX"
- "Help me create an ex skill" (auto-enters relationship mode)

Enter evolution mode when the user says:
- "I have new chat logs" / "append"
- "That's wrong" / "He/She wouldn't do that" / "He/She should be"
- `/update-person {slug}` or `/update-ex {slug}`

List all generated persons when the user says `/list-persons` or `/list-exes`.

---

## Tool Usage Rules

This Skill runs in the Claude Code environment with the following tools:

| Task | Tool |
|------|------|
| Read PDF documents | `Read` tool (native PDF support) |
| Read image screenshots | `Read` tool (native image support) |
| Read MD/TXT files | `Read` tool |
| Parse WeChat chat exports | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py` |
| Parse iMessage | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py` |
| Parse SMS | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py` |
| Analyze photo metadata | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py` |
| Parse social media exports | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py` |
| Write/update Skill files | `Write` / `Edit` tool |
| Version management | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| List existing Skills | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**Base directory**: Skill files are written to `./persons/{slug}/` (relative to the project directory).
For a global path, use `--base-dir ~/.openclaw/workspace/skills/persons`.

---

## Main Flow: Create a New Person Skill

### Step 1: Basic Info Collection (3 questions)

Refer to `${CLAUDE_SKILL_DIR}/prompts/intake.md` for the question sequence. Only ask 3 questions:

1. **Nickname / Codename** (required)
2. **Basic info** (one sentence: relation type, how long known, how you met, what they do)
   - Friend example: `college friend, known 5 years, rarely in touch now, she's a PM`
   - Ex example: `ex-girlfriend, together 3 years, college classmates, broke up 1 year ago, she's a designer`
   - Family example: `mom, known forever, retired teacher`
3. **Personality profile** (one sentence: MBTI, zodiac, personality traits, your impression)
   - Example: `INFJ Scorpio, talks little but very opinionated, opens up with close people`

Everything except the nickname can be skipped. Summarize and confirm before moving to the next step.

**Auto-detect relationship mode**: When basic info contains "ex-girlfriend/ex-boyfriend/ex/together/broke up" etc., automatically add relationship-specific questions (attachment style, conflict style).

### Step 2: Source Material Import

Ask the user how they'd like to provide materials:

```
How would you like to provide source materials?

  [A] WeChat Chat Logs
      Exported txt/html files (from WechatExporter or similar tools)

  [B] iMessage / SMS
      From Mac's chat.db or exported files

  [C] Photos
      Specify a folder, auto-extract timeline (EXIF metadata)

  [D] Social Media
      Weibo/Douban/Xiaohongshu/Instagram exports

  [E] Upload Files
      PDF / screenshots / any text

  [F] Paste Text
      Copy-paste text directly

Can mix and match, or skip entirely (generate from manual info only).
```

---

#### Option A: WeChat Chat Logs

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py --file {path} --target "{name}" --output /tmp/wechat_out.txt
```
Then `Read /tmp/wechat_out.txt`

---

#### Option B: iMessage / SMS

**iMessage** (macOS):
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --file {path} --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

Direct access to local chat.db (requires Full Disk Access):
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --direct --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

**SMS**:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py --file {path} --target "{phone_or_name}" --output /tmp/sms_out.txt
```

---

#### Option C: Photos

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py --dir {photo_directory} --output /tmp/photo_timeline.txt
```
Then `Read /tmp/photo_timeline.txt` for the timeline.

Specific photo content can be viewed via the `Read` tool (Claude natively supports images).

---

#### Option D: Social Media

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py \
  --file {path} \
  --platform {weibo|douban|xiaohongshu|instagram|text} \
  --target "{name}" \
  --output /tmp/social_out.txt
```

---

#### Option E: Upload Files

- **PDF / Images**: `Read` tool directly
- **Markdown / TXT**: `Read` tool directly

---

#### Option F: Paste Text

User-pasted content is used directly as text material. No tools needed.

---

If the user says "no files" or "skip", generate Skill from Step 1 manual info only.

### Step 3: Analyze Source Material

Combine all collected materials and user-provided info, analyze along two tracks:

**Track A (Memories Skill)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/memories_analyzer.md` for extraction dimensions
- Extract: relationship timeline, shared routines, preferences, interaction patterns, emotional dynamics

**Track B (Persona)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` for extraction dimensions
- Translate user-provided tags into concrete behavior rules
- Extract from materials: communication style, emotional logic, relationship behavior

### Step 4: Generate and Preview

Use `${CLAUDE_SKILL_DIR}/prompts/memories_builder.md` to generate Memories Skill content.
Use `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` to generate Persona content (5-layer structure).

Show the user a summary (5-8 lines each), ask:
```
Memories Summary:
  - Known: {duration}, {relation_type}
  - Key moments: {N}
  - Shared interests: {xxx}
  - Their preferences: {xxx}
  ...

Persona Summary:
  - Core personality: {xxx}
  - Communication style: {xxx}
  - Emotional pattern: {xxx}
  ...

Confirm generation? Or need adjustments?
```

### Step 5: Write Files

After user confirmation, execute the following:

**1. Create directory structure** (Bash):
```bash
mkdir -p persons/{slug}/versions
mkdir -p persons/{slug}/knowledge/chats
mkdir -p persons/{slug}/knowledge/photos
mkdir -p persons/{slug}/knowledge/social
```

**2-5.** Write memories.md, persona.md, meta.json, and SKILL.md using the Write tool.

Inform user:
```
✅ Person Skill created!

Location: persons/{slug}/
Commands: /{slug} (full version)
          /{slug}-memories (memories only)
          /{slug}-persona (persona only)

If something feels off, just say "he/she wouldn't do that" and I'll update it.
```

---

## Evolution Mode: Append Files

When user provides new files or text:

1. Read new content using Step 2 methods
2. `Read` existing `persons/{slug}/memories.md` and `persona.md`
3. Refer to `${CLAUDE_SKILL_DIR}/prompts/merger.md` for incremental analysis
4. Archive current version (Bash):
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./persons
   ```
5. Use `Edit` tool to append incremental content to relevant files
6. Regenerate `SKILL.md` (merge latest memories.md + persona.md)
7. Update `meta.json` version and updated_at

---

## Evolution Mode: Conversation Correction

When user expresses "that's wrong" / "he/she wouldn't do that":

1. Refer to `${CLAUDE_SKILL_DIR}/prompts/correction_handler.md` to identify correction content
2. Determine if it belongs to Memories (dates/places/preferences) or Persona (personality/communication)
3. Generate correction record
4. Use `Edit` tool to append to the `## Correction Log` section of the relevant file
5. Regenerate `SKILL.md`

---

## Management Commands

`/list-persons` or `/list-exes`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./persons
```

`/person-rollback {slug} {version}`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./persons
```

`/delete-person {slug}`:
After confirmation:
```bash
rm -rf persons/{slug}
```
