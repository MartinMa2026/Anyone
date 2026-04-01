# Anyone.skill 安装说明

---

## 选择你的平台

### A. Claude Code（推荐）

本项目遵循 [AgentSkills](https://agentskills.io) 标准，整个 repo 就是 skill 目录。

```bash
# 方式 1：安装到当前项目（在 git 仓库根目录执行）
mkdir -p .claude/skills
git clone https://github.com/MartinMa2026/Anyone.git .claude/skills/create-person

# 方式 2：安装到全局（所有项目都能用）
git clone https://github.com/MartinMa2026/Anyone.git ~/.claude/skills/create-person
```

然后在 Claude Code 中说 `/create-person` 即可启动。

生成的人物 Skill 默认写入 `./persons/` 目录。

---

### B. OpenClaw

```bash
git clone https://github.com/MartinMa2026/Anyone.git ~/.openclaw/workspace/skills/create-person
```

重启 OpenClaw session，说 `/create-person` 启动。

---

## 依赖安装

```bash
# 基础（Python 3.9+）
pip3 install pypinyin        # 中文姓名转拼音 slug（可选但推荐）

# iMessage 直接读取（macOS）
# 不需要额外依赖，但需要在 系统设置 → 隐私与安全性 → Full Disk Access 中授权终端

# 照片 EXIF 分析
# 不需要额外依赖（使用内置 JPEG EXIF 解析）
# 如需支持更多格式（HEIC/TIFF 等）：
pip3 install Pillow
```

---

## 数据来源准备指南

### 微信聊天记录

推荐使用 [WechatExporter](https://github.com/BlueMatthew/WechatExporter) 导出：

1. iPhone 通过 iTunes/Finder 备份到 Mac（**不加密**）
2. 运行 WechatExporter，选择备份文件
3. 在联系人列表中勾选要导出的对象
4. 导出为 **txt 文本**格式
5. 在 `/create-person` 流程中选择 [A] 微信聊天记录，提供 txt 文件路径

> 💡 无需全量导出所有联系人——只勾选你需要的那个人即可

### iMessage

**方式 1：直接读取**（macOS）
- 需要在 系统设置 → 隐私与安全性 → Full Disk Access 中授权终端或 Claude Code
- 在 `/create-person` 流程中选择 [B]，使用 `--direct` 参数直接读取

**方式 2：导出文件**
- 使用 [iMazing](https://imazing.com/) 等工具导出
- 导出为 txt 或 csv 格式

### 照片

将相关照片放在一个文件夹中：
- 工具会自动提取 EXIF 元数据（拍摄日期、位置）
- 生成时间线摘要
- 具体照片内容由你选择后通过 Claude 的 Read 工具查看（原生支持图片）

### 社交媒体

**微博**：设置 → 安全中心 → 个人信息与资料下载

**Instagram**：设置 → 你的活动 → 下载你的信息

**小红书/豆瓣**：使用浏览器开发者工具或第三方导出工具

---

## 快速验证

```bash
cd ~/.claude/skills/create-person   # 或你的项目 .claude/skills/create-person

# 测试微信解析器
python3 tools/wechat_parser.py --help

# 测试 iMessage 解析器
python3 tools/imessage_parser.py --help

# 测试照片分析器
python3 tools/photo_analyzer.py --help

# 列出已有人物 Skill
python3 tools/skill_writer.py --action list --base-dir ./persons
```

---

## 目录结构说明

```
Anyone/                   ← clone 到 .claude/skills/create-person/
├── SKILL.md              # Skill 入口（AgentSkills frontmatter）
├── prompts/              # 分析和生成的 Prompt 模板
├── tools/                # Python 工具脚本
├── requirements.txt
│
└── persons/              # 生成的人物 Skill 存放处（.gitignore 排除）
    └── {slug}/
        ├── SKILL.md              # 完整 Skill（Persona + Memories）
        ├── memories.md           # 共同记忆
        ├── persona.md            # 人物性格
        ├── meta.json             # 元数据
        ├── versions/             # 历史版本
        └── knowledge/            # 原始材料归档
            ├── chats/
            ├── photos/
            └── social/
```

---

## 隐私说明

- `persons/` 目录已加入 `.gitignore`，生成的 Skill 文件**不会被 Git 追踪**
- 所有数据处理在本地完成，不联网，不上传
- 聊天记录等原始材料建议**不要 commit 到任何公开仓库**
