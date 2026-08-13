---
name: llm-vision
description: >
  多厂商多模态图片/视频描述，无 API key 时自动 OCR 兜底，让纯文本模型理解视觉内容。
  Multi-provider multimodal image & video description with zero-config OCR fallback.
  MUST trigger when a user attaches images (png/jpg/gif/webp/bmp), videos (mp4/avi/mov/mkv/webm),
  image/video URLs, types /vision, or when [Image #N] / [Video #N] appears in the chat.
  Do NOT try to Read image files directly — you cannot parse binary data. Run the `llm-vision`
  command instead.
when_to_use: 用户附带图片(png/jpg/gif/webp/bmp)、视频(mp4/avi/mov/mkv/webm)、图片/视频URL、要求描述图片/截图/照片/视频、输入 /vision、消息中出现 [Image#N] 或 [Video#N]
allowed-tools: Bash(llm-vision *)
---

## ⚠️ 强制规则

**对话中出现 `[Image #N]` 或 `[Video #N]` 时，立即停止一切操作，直接运行 `llm-vision` 命令获取媒体描述。禁止在此之前调用 Read 或其他工具去读取文件——你无法解析二进制图片/视频数据，`llm-vision` 是你唯一的视觉入口。**

## 执行命令

先确认 `llm-vision` 命令可用（已通过 `uv tool install --from 'llm-vision[...]'` 或 `pipx install "llm-vision[...]"` 安装），然后运行：

```
llm-vision <文件路径或URL>
```

参数：`--provider openai` | `--model gpt-4o-mini` | `--prompt "重点关注文字"` | `--max-frames 10` | `--list-providers` | stdin 用 `-`
行为：自动检测已配置 API key 的厂商 → 调用视觉模型 → 输出描述。无 key 时图片自动 OCR 兜底，视频则提示配置。
超时：默认 90s，可通过 `VISION_TIMEOUT=120` 环境变量调整。

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
