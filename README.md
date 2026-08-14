# modeleyes

> 多厂商多模态图片/视频描述 CLI + OCR 兜底——让纯文本大模型也能"看图"（model eyes）。

**简体中文** | [English](README_EN.md)

`modeleyes` 把图片/视频转成结构化文字描述，供 DeepSeek、GLM 等**纯文本大模型**理解视觉内容。多厂商视觉 API 自动检测、无 key 时 OCR 兜底，一行命令安装、三平台一致。

## 特性

- 🖼️ **图片描述**：结构化 7 段式（场景/背景/主体/文字/色彩/细节/印象），专为纯文本模型设计
- 🎬 **视频描述**：关键帧提取 → 逐帧描述 → 汇总总结
- 🔌 **多厂商自动切换**：智谱 / OpenAI / Anthropic / Google / 小米 MiMo，按注册顺序检测可用 API
- 📝 **OCR 兜底**：无 API key 时降级 easyocr → pytesseract → Pillow 基础信息
- 📋 **多输入**：本地文件、远程 URL、stdin 管道、多文件批处理
- 🔍 **魔数识别**：无扩展名的 TUI 临时文件也能识别
- 🌐 **跨平台**：Windows / macOS / Linux 安装与调用一致

## 快速开始

```bash
# 安装（任选其一）
uv tool install --from 'modeleyes[openai]' --python 3.12
pipx install "modeleyes[openai]"

# 配置 API key
export OPENAI_API_KEY="sk-..."

# 使用
modeleyes photo.jpg
```

## 安装

### 包安装（uv 首选 / pipx）

```bash
# 单厂商
uv tool install --from 'modeleyes[openai]' --python 3.12
# 全厂商 API（轻量，无 torch）
uv tool install --from 'modeleyes[all]' --python 3.12
# 全厂商 + OCR（含 torch，体积大）
uv tool install --from 'modeleyes[full]' --python 3.12

# 或用 pipx
pipx install "modeleyes[openai]"
```

> ⚠️ **OCR 用途需 pin Python 3.12**：`easyocr` 依赖 `torch`，而 torch 对 Python 3.14 的轮子尚不齐全，故 `[ocr]`/`[full]` 在 3.14 上会安装失败。纯 API 用途（`[openai]`/`[all]`）任意版本安全。uv 默认选最新 Python，故安装命令显式 `--python 3.12`。

### 可选 extras

| extra | 厂商/能力 | 依赖 |
|---|---|---|
| `[zhipu]` | 智谱 GLM | zai-sdk |
| `[openai]` | OpenAI + 小米 MiMo（复用 SDK） | openai |
| `[anthropic]` | Anthropic Claude | anthropic |
| `[google]` | Google Gemini | google-genai |
| `[ocr]` | easyocr 离线 OCR（含 torch） | easyocr |
| `[tesseract]` | tesseract OCR | pytesseract |
| `[all]` | 上述 4 个 API 厂商（轻量） | — |
| `[full]` | all + ocr + tesseract | — |

### 可选系统二进制

- **ffmpeg**（视频关键帧提取）：Windows `winget install ffmpeg` / macOS `brew install ffmpeg` / Linux `sudo apt install ffmpeg`
- **tesseract**（pytesseract OCR）：Windows 官方安装器或 `choco install tesseract` / macOS `brew install tesseract` / Linux `sudo apt install tesseract-ocr tesseract-ocr-chi-sim`

## 用法

```bash
modeleyes --list-providers                    # 列出厂商配置状态
modeleyes photo.jpg                           # 图片描述
cat screenshot.png | modeleyes -              # stdin 管道
modeleyes https://example.com/a.jpg           # 远程 URL
modeleyes video.mp4 --max-frames 8            # 视频（需 ffmpeg + key）
modeleyes a.jpg b.jpg                         # 多文件
modeleyes photo.jpg --provider zhipu --model glm-4.6v   # 指定厂商/模型
modeleyes photo.jpg --prompt "重点关注文字"    # 自定义提示词
python -m modeleyes photo.jpg                 # 模块调用（venv 内）
```

## 支持的厂商

| 厂商 | 环境变量 | 默认模型 |
|---|---|---|
| 智谱 GLM | `ZHIPU_API_KEY` | glm-4.6v |
| 小米 MiMo | `MIMO_API_KEY` | mimo-v2.5 |
| OpenAI | `OPENAI_API_KEY` | gpt-4o |
| Anthropic | `ANTHROPIC_API_KEY` | claude-sonnet-4-6 |
| Google | `GOOGLE_API_KEY` | gemini-2.5-flash |

## 配置 API key

三种方式任选：

```bash
# 1. 环境变量（临时）
export OPENAI_API_KEY="your-key"

# 2. 项目级（仅当前项目，推荐用于 Claude Code）
# 编辑 .claude/settings.local.json: { "env": { "OPENAI_API_KEY": "your-key" } }

# 3. 全局
# 编辑 ~/.claude/settings.json 的 env 字段
```

超时：默认 90s，可用 `VISION_TIMEOUT=120` 调整。

### 用环境变量覆盖默认模型

不必每次传 `--model`，用 `VISION_MODEL` 设置默认模型（优先级：`--model` 参数 > `VISION_MODEL` > 厂商默认值）：

```bash
export VISION_MODEL="gpt-4o-mini"   # OpenAI 用 gpt-4o-mini 而非默认 gpt-4o
modeleyes photo.jpg

export VISION_MODEL="glm-4.6v"      # 智谱用 glm-4.6v
modeleyes photo.jpg
```

临时覆盖用 `--model`（优先级最高）：

```bash
modeleyes photo.jpg --model gpt-4o   # 本次用 gpt-4o，忽略 VISION_MODEL
```

## 平台支持

| 平台 | 包安装 | 视频抽帧 | OCR |
|---|---|---|---|
| Windows | ✅ `winget` 装 ffmpeg/tesseract | ✅ | ✅ |
| macOS | ✅ `brew` 装 ffmpeg/tesseract | ✅ | ✅ |
| Linux | ✅ `apt` 装 ffmpeg/tesseract | ✅ | ✅ |

## 注意

仅在**纯文本主模型**下启用本工具。若主模型本身是多模态（如 GPT-4o、Claude Opus），它会冗余调用一次视觉 API，造成无谓开销与延迟。

## 致谢

基于 [LearningByDoingNow/vision-skill](https://github.com/LearningByDoingNow/vision-skill) 改造，感谢原作者。

## 许可证

[MIT](LICENSE)
