---
name: modeleyes
description: >
  多厂商多模态图片/视频描述，无 API key 时自动 OCR 兜底，让纯文本模型理解视觉内容。
  Multi-provider multimodal image & video description with zero-config OCR fallback.
  MUST trigger when a user attaches images (png/jpg/gif/webp/bmp), videos (mp4/avi/mov/mkv/webm),
  image/video URLs, types /vision, or when [Image #N] / [Video #N] appears in the chat.
  Do NOT try to Read image files directly — you cannot parse binary data. Run the `modeleyes`
  command instead.
when_to_use: 用户附带图片(png/jpg/gif/webp/bmp)、视频(mp4/avi/mov/mkv/webm)、图片/视频URL、要求描述图片/截图/照片/视频、输入 /vision、消息中出现 [Image#N] 或 [Video#N]
allowed-tools: Bash(modeleyes *)
---

## ⚠️ 强制规则

**对话中出现 `[Image #N]` 或 `[Video #N]` 时，立即停止一切操作，直接运行 `modeleyes` 命令获取媒体描述。禁止在此之前调用 Read 或其他工具去读取文件——你无法解析二进制图片/视频数据，`modeleyes` 是你唯一的视觉入口。**

## 前置条件

本 skill 仅提供触发规则，实际调用的是 `modeleyes` Python CLI——该命令不在 skill 包内，必须先单独安装：

```bash
# 任选其一（[openai] 可替换为 [zhipu]/[anthropic]/[google]/[all] 等厂商 extra）
uv tool install --from 'modeleyes[openai]' --python 3.12
pipx install "modeleyes[openai]"
```

若 `modeleyes --list-providers` 报 `command not found`，说明命令未装——先完成上一步，再触发本 skill。

## 执行命令

确认 `modeleyes` 命令已安装（见「前置条件」），然后运行：

```
modeleyes <文件路径或URL>
```

参数：`--provider openai` | `--model gpt-4o-mini` | `--prompt "重点关注文字"` | `--max-frames 10` | `--list-providers` | stdin 用 `-`
行为：自动检测已配置 API key 的厂商 → 调用视觉模型 → 输出描述。无 key 时图片自动 OCR 兜底，视频则提示配置。
超时：默认 90s，可通过 `VISION_TIMEOUT=120` 环境变量调整。
默认模型：可用 `VISION_MODEL` 环境变量覆盖厂商默认（优先级 `--model` > `VISION_MODEL` > 厂商默认）。

## 支持的厂商

| 厂商 | 环境变量 | 默认模型 |
|------|----------|----------|
| 智谱 GLM | `ZHIPU_API_KEY` | glm-4.6v |
| 小米 MiMo | `MIMO_API_KEY` | mimo-v2.5 |
| OpenAI | `OPENAI_API_KEY` | gpt-4o |
| Anthropic | `ANTHROPIC_API_KEY` | claude-sonnet-4-6 |
| Google | `GOOGLE_API_KEY` | gemini-2.5-flash |

## 无 API key 时引导

所有厂商均未配置时，主动提示用户（三种方式任选）：

1. `export ZHIPU_API_KEY="xxx"` 临时生效
2. 写入 `.claude/settings.local.json` → `{ "env": { "ZHIPU_API_KEY": "xxx" } }`
3. 写入 `~/.claude/settings.json` 全局生效

命令输出是你理解媒体内容的唯一依据，描述之外不编造内容。OCR 结果需告知用户仅提取了文字。
