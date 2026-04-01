<div align="center">

# Anyone.skill

> *"把任何人蒸馏成 AI，让记忆可以对话。"*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

你有没有一个人，想再和他/她说说话？<br>
一个老朋友，一段关系，一个你来不及好好了解的人？<br>
聊天记录还在，照片还在，那些细节还在——<br>

**把一个真实的人蒸馏成 AI Skill，然后再次对话。**

<br>

提供聊天记录（微信、iMessage、短信）、照片、社交媒体，加上你的主观描述<br>
生成一个**像他/她一样思考和说话的 AI Skill**<br>
用他/她的语气回消息，知道他/她在意什么、讨厌什么、遇事会怎么反应

[数据来源](#支持的数据来源) · [安装](#安装) · [使用](#使用) · [效果示例](#效果示例) · [详细安装说明](INSTALL.md) · [**English**](README_EN.md)

</div>

---

## 支持的数据来源

| 来源 | 聊天记录 | 照片 | 社交媒体 | 备注 |
|------|:-------:|:----:|:-------:|------|
| 微信聊天记录 | ✅ | — | — | WechatExporter 等工具导出 |
| iMessage | ✅ | — | — | macOS chat.db 或导出文件 |
| 短信 | ✅ | — | — | Android SMS Backup XML/CSV |
| 照片 | — | ✅ | — | EXIF 元数据提取时间线 |
| 微博 | — | — | ✅ | JSON 数据导出 |
| 豆瓣 | — | — | ✅ | JSON/HTML 导出 |
| 小红书 | — | — | ✅ | JSON 导出 |
| Instagram | — | — | ✅ | JSON 数据导出 |
| PDF / 图片截图 | ✅ | ✅ | — | 手动上传 |
| 直接粘贴文字 | ✅ | — | — | 手动输入 |

---

## 安装

### Claude Code

```bash
# 安装到当前项目（在 git 仓库根目录执行）
mkdir -p .claude/skills
git clone https://github.com/MartinMa2026/Anyone.git .claude/skills/create-person

# 或安装到全局（所有项目都能用）
git clone https://github.com/MartinMa2026/Anyone.git ~/.claude/skills/create-person
```

### OpenClaw

```bash
git clone https://github.com/MartinMa2026/Anyone.git ~/.openclaw/workspace/skills/create-person
```

### 依赖（可选）

```bash
pip3 install -r requirements.txt
```

> 仅 `pypinyin` 为可选依赖，用于将中文姓名自动转拼音作为目录名（slug）。

---

## 使用

在 Claude Code 中输入：

```
/create-person
```

按提示输入他/她的昵称、关系信息和性格标签，然后选择数据来源。所有字段均可跳过，仅凭文字描述也能生成。

完成后用 `/{slug}` 调用生成的 Skill。

### 支持的关系类型

```
朋友      大学同学 认识五年 现在不常联系 她做产品经理
同事      前同事 共事三年 一起创过业 他做技术
家人      妈妈 从小到大 退休教师
前任      前女友 在一起三年 大学同学 分手一年 她做设计
网友      游戏里认识 玩了两年 从没见过面
偶像      喜欢的博主 看了五年视频
...任意人物
```

### 管理命令

| 命令 | 说明 |
|------|------|
| `/create-person` | 创建新人物 Skill |
| `/list-persons` | 列出所有人物 Skill |
| `/{slug}` | 调用完整 Skill（Persona + Memories） |
| `/{slug}-memories` | 仅共同记忆 |
| `/{slug}-persona` | 仅人物性格 |
| `/person-rollback {slug} {version}` | 回滚到历史版本 |
| `/delete-person {slug}` | 删除 |

> 💡 `/create-ex`、`/list-exes` 等旧命令仍然有效（向下兼容）

---

## 效果示例

> 人物：老朋友，慢热、话少、完美主义、工作狂

**场景一：好久不联系突然聊天**

```
你     > 最近怎么样

老朋友  > 还行
       > 你呢
       > 最近有什么事吗
```

> 人物：前任，ENFP · 爱撒娇 · 翻旧账 · 焦虑型依恋

**场景二：撒娇**

```
你     > 想吃什么

前任   > 随便
你     > 那吃面？
前任   > 不想吃面
你     > 火锅？
前任   > 嘿嘿你怎么知道的 💕
```

**场景三：惹她/他不开心**

```
你     > 不好意思刚才在忙

前任   > 哦
       > （沉默5分钟）
       > 你每次都说在忙
       > 上次也是这样
```

---

## 功能特性

### 生成的 Skill 结构

每个人物 Skill 由两部分组成：

| 部分 | 内容 |
|------|------|
| **Part A — 共同记忆** | 关系时间线、日常仪式、偏好习惯、互动模式、情感动态 |
| **Part B — Persona** | 5 层性格结构：硬规则 → 身份 → 表达风格 → 情感逻辑 → 关系行为 |

运行逻辑：`收到消息 → Persona 判断心情和态度 → Memories 提供记忆细节 → 用他/她的语气输出`

### 支持的性格标签

**通用标签**：话很多 · 话很少 · 慢热 · 自来熟 · 完美主义 · 行动派 · 计划狂 · 拖延 · 情绪稳定 · 玻璃心 · 大大咧咧 · 细腻敏感

**恋爱专属**：爱撒娇 · 冷暴力 · 翻旧账 · 黏人 · 独立 · 忽冷忽热 · 作 · 控制欲强<br>
**吵架模式**：冷战派 · 爆发派 · 讲道理派 · 先道歉型 · 死不认错<br>
**依恋类型**：安全型 · 焦虑型 · 回避型 · 混乱型

### 进化机制

- **追加聊天记录** → 自动分析增量 → merge 进对应部分，不覆盖已有结论
- **对话纠正** → 说「他/她不会这样，他/她应该是 xxx」→ 写入 Correction 层，立即生效
- **版本管理** → 每次更新自动存档，支持回滚到任意历史版本

---

## 项目结构

```
Anyone/
├── SKILL.md              # Skill 入口（AgentSkills 标准 frontmatter）
├── prompts/              # Prompt 模板
│   ├── intake.md              #  对话式信息录入（3 个问题）
│   ├── memories_analyzer.md   #  共同记忆提取维度
│   ├── persona_analyzer.md    #  性格行为分析（含标签翻译表）
│   ├── memories_builder.md    #  memories.md 生成模板
│   ├── persona_builder.md     #  persona.md 五层结构模板
│   ├── merger.md              #  增量 merge 逻辑
│   └── correction_handler.md  #  对话纠正处理
├── tools/                # Python 工具（纯本地，无需联网）
│   ├── wechat_parser.py       # 微信聊天记录解析
│   ├── imessage_parser.py     # iMessage / 短信解析
│   ├── sms_parser.py          # Android 短信解析
│   ├── photo_analyzer.py      # 照片 EXIF 元数据分析
│   ├── social_media_parser.py # 社交媒体导出解析
│   ├── skill_writer.py        # Skill 文件管理
│   └── version_manager.py     # 版本存档与回滚
├── persons/              # 生成的人物 Skill（gitignored，隐私数据）
├── requirements.txt
└── LICENSE
```

---

## 隐私说明

- **所有数据仅在本地处理**，不会发送到任何外部服务
- `persons/` 目录已加入 `.gitignore`，生成的 Skill（含聊天记录）**不会被上传到 Git**
- 照片分析只提取 EXIF 元数据（日期/位置），不读取照片内容
- 聊天记录建议 **不要 commit 到任何公开仓库**

---

## 注意事项

- **数据质量决定 Skill 质量**：真实聊天记录 > 仅手动描述
- 建议优先收集：**主动发的长消息** > **情感类消息** > 日常闲聊
- 没有任何数据也能运行——仅凭文字描述同样能生成有效的 Persona

---

<div align="center">

MIT License © [MartinMa2026](https://github.com/MartinMa2026)

</div>
