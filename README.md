# Resemble Detect Skill

An agent skill for deepfake detection and media safety — powered by [Resemble AI](https://resemble.ai).

Give any AI coding agent (Claude Code, Cursor, Copilot, Windsurf, Gemini CLI, and others) the ability to detect AI-generated audio, images, and video using Resemble's detection platform.

## Install

```bash
npx skills add resemble-ai/detect-skill
```

Or manually copy [`detect.md`](./detect.md) into your project's `.claude/skills/` directory (or equivalent for your agent).

## What It Does

This skill teaches your agent to use Resemble's full detection and media safety suite:

| Capability | Description |
|---|---|
| **Deepfake Detection** | Analyze audio, image, and video for synthetic manipulation with confidence scores and visualizations |
| **Intelligence** | Extract speaker info, emotion, transcription, misinformation signals, and abnormalities from any media |
| **Detect Intelligence** | Ask natural-language follow-up questions about completed detection results |
| **Audio Source Tracing** | Identify which AI platform (ElevenLabs, Resemble, etc.) synthesized detected fake audio |
| **Watermarking** | Apply invisible watermarks for provenance tracking, or detect existing watermarks in media |
| **Identity Verification** | Create voice profiles and match unknown speakers against them (Beta) |

## Requirements

- A [Resemble AI](https://app.resemble.ai) API key
- Media files accessible via public HTTPS URLs

For the richest experience, pair this skill with the [Resemble MCP server](https://github.com/resemble-ai/resemble-mcp) which gives your agent direct access to Resemble's documentation and API reference tools.

## How It Works

The skill is a single markdown file (`detect.md`) that provides your AI agent with:

- **Decision tree** — maps user intent to the correct API capability and endpoint
- **Complete API reference** — every endpoint, parameter, and response schema
- **Score interpretation** — how to read and present detection confidence scores
- **Workflow templates** — full forensics, quick checks, and provenance pipelines
- **Red flags** — anti-patterns the agent should catch in its own reasoning
- **Error handling** — every status code with cause and resolution

## Example Prompts

Once installed, try asking your agent:

- *"Is this audio file a deepfake?"*
- *"Analyze this video for AI manipulation and tell me what platform might have generated it"*
- *"Apply a watermark to this image for provenance tracking"*
- *"Check if this speaker matches the voice profile we have on file"*
- *"What can you tell me about this audio — speaker, emotion, language, any abnormalities?"*

## Links

- [Resemble AI](https://resemble.ai) — Platform
- [Resemble MCP Server](https://github.com/resemble-ai/resemble-mcp) — Full MCP integration
- [API Documentation](https://docs.resemble.ai) — Resemble API docs
- [skills.sh](https://skills.sh) — The Open Agent Skills Ecosystem

## License

Apache-2.0
