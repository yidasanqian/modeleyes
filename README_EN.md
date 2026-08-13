# llm-vision

> Multi-provider multimodal image/video description CLI with OCR fallback — lets text-only LLMs "see".

[简体中文](README.md) | **English**

`llm-vision` turns images/videos into structured text descriptions so that text-only LLMs (DeepSeek, GLM, ...) can understand visual content. It auto-detects among vision API providers and falls back to OCR when no API key is set. One-line install, consistent across platforms.

Refactored from [vision-skill](https://github.com/LearningByDoingNow/vision-skill) into a standalone general-purpose CLI.

## Features

- 🖼️ **Image description**: structured 7-section output (scene/background/subject/text/color/detail/impression), designed for text-only models
- 🎬 **Video description**: keyframe extraction → per-frame description → summary
- 🔌 **Multi-provider auto-switch**: Zhipu / OpenAI / Anthropic / Google / Xiaomi MiMo, detected in registry order
- 📝 **OCR fallback**: degrades to easyocr → pytesseract → Pillow basic info when no API key
- 📋 **Multiple inputs**: local files, remote URLs, stdin pipe, batch processing
- 🔍 **Magic-number detection**: identifies TUI temp files without extensions
- 🌐 **Cross-platform**: consistent install & usage on Windows / macOS / Linux

## Quick start

```bash
# Install (either)
uv tool install --from 'llm-vision[openai]' --python 3.12
pipx install "llm-vision[openai]"

# Set API key
export OPENAI_API_KEY="sk-..."

# Use
llm-vision photo.jpg
```

## Installation

### Package (uv recommended / pipx)

```bash
# Single provider
uv tool install --from 'llm-vision[openai]' --python 3.12
# All API providers (lightweight, no torch)
uv tool install --from 'llm-vision[all]' --python 3.12
# All providers + OCR (includes torch, large)
uv tool install --from 'llm-vision[full]' --python 3.12

# or pipx
pipx install "llm-vision[openai]"
```

> ⚠️ **Pin Python 3.12 for OCR**: `easyocr` depends on `torch`, whose wheels for Python 3.14 are not yet complete, so `[ocr]`/`[full]` fail to install on 3.14. Pure API usage (`[openai]`/`[all]`) is safe on any version. uv defaults to the latest Python, hence the explicit `--python 3.12`.

### Optional extras

| extra | provider / capability | dependency |
|---|---|---|
| `[zhipu]` | Zhipu GLM | zai-sdk |
| `[openai]` | OpenAI + Xiaomi MiMo (shared SDK) | openai |
| `[anthropic]` | Anthropic Claude | anthropic |
| `[google]` | Google Gemini | google-genai |
| `[ocr]` | easyocr offline OCR (includes torch) | easyocr |
| `[tesseract]` | tesseract OCR | pytesseract |
| `[all]` | the 4 API providers above (lightweight) | — |
| `[full]` | all + ocr + tesseract | — |

### Optional system binaries

- **ffmpeg** (video keyframe extraction): Windows `winget install ffmpeg` / macOS `brew install ffmpeg` / Linux `sudo apt install ffmpeg`
- **tesseract** (pytesseract OCR): Windows installer or `choco install tesseract` / macOS `brew install tesseract` / Linux `sudo apt install tesseract-ocr tesseract-ocr-chi-sim`

## Usage

```bash
llm-vision --list-providers                    # list provider status
llm-vision photo.jpg                           # describe image
cat screenshot.png | llm-vision -              # stdin pipe
llm-vision https://example.com/a.jpg           # remote URL
llm-vision video.mp4 --max-frames 8            # video (needs ffmpeg + key)
llm-vision a.jpg b.jpg                         # multiple files
llm-vision photo.jpg --provider zhipu --model glm-4.6v   # specify provider/model
llm-vision photo.jpg --prompt "focus on text"  # custom prompt
python -m llm_vision photo.jpg                 # module call (in venv)
```

## Supported providers

| Provider | env var | default model |
|---|---|---|
| Zhipu GLM | `ZHIPU_API_KEY` | glm-4.6v |
| Xiaomi MiMo | `MIMO_API_KEY` | mimo-v2.5 |
| OpenAI | `OPENAI_API_KEY` | gpt-4o |
| Anthropic | `ANTHROPIC_API_KEY` | claude-sonnet-4-6 |
| Google | `GOOGLE_API_KEY` | gemini-2.5-flash |

## Configure API keys

```bash
# 1. env var (temporary)
export OPENAI_API_KEY="your-key"

# 2. project-level (Claude Code): .claude/settings.local.json
# { "env": { "OPENAI_API_KEY": "your-key" } }

# 3. global: ~/.claude/settings.json env field
```

Model priority: `--model` arg > `VISION_MODEL` env > provider default.
Timeout: default 90s, adjustable via `VISION_TIMEOUT=120`.

## Platform support

| Platform | install | video | OCR |
|---|---|---|---|
| Windows | ✅ `winget` ffmpeg/tesseract | ✅ | ✅ |
| macOS | ✅ `brew` ffmpeg/tesseract | ✅ | ✅ |
| Linux | ✅ `apt` ffmpeg/tesseract | ✅ | ✅ |

## Note

Enable this tool only with a **text-only** main model. If the main model is already multimodal (e.g. GPT-4o, Claude Opus), this tool redundantly calls a vision API, adding needless cost and latency.

## Acknowledgements

Refactored from [LearningByDoingNow/vision-skill](https://github.com/LearningByDoingNow/vision-skill). Credits to the original author.

## License

[MIT](LICENSE)
