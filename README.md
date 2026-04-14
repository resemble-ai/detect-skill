# Resemble Detect Skill

An agent skill for deepfake detection and media safety — powered by [Resemble AI](https://resemble.ai).

Give any AI agent the ability to detect AI-generated audio, images, and video using Resemble's detection platform.

## Compatible Agents

Works with any agent that supports markdown skills:

| Agent | Install Method |
|---|---|
| [Claude Code](https://claude.ai/code) | `npx skills add resemble-ai/detect-skill` or copy to `.claude/skills/` |
| [OpenClaw](https://github.com/openclaw/openclaw) (formerly Clawdbot) | Copy to skills directory or import via built-in skill loader |
| [Hermes Agent](https://github.com/nousresearch/hermes-agent) | Add to skills directory — Hermes will auto-discover and self-improve on it |
| [Cursor](https://cursor.com) | Add to `.cursor/skills/` or project rules |
| [GitHub Copilot](https://github.com/features/copilot) | Add to `.github/copilot-instructions.md` or reference in prompt |
| [Windsurf](https://codeium.com/windsurf) | Add to project rules |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Add to `.gemini/skills/` |

## Install

**Via skills.sh (recommended):**
```bash
npx skills add resemble-ai/detect-skill
```

**Manual:** Copy [`SKILL.md`](./SKILL.md) into your agent's skills directory.

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

## Pair With the Resemble MCP Server (Recommended)

For the richest experience, pair this skill with the **[Resemble MCP server](https://github.com/resemble-ai/resemble-mcp)** so your agent gets live access to Resemble's full documentation, API reference, and endpoint schemas — no guessing, no stale examples.

### Easiest setup — hosted SSE endpoint (zero install)

```
https://mcp.resemble.ai/sse
```

Point any MCP-compatible agent at this URL and you're done. No cloning, no installing, no server to run.

**Cursor** — add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "resemble": {
      "url": "https://mcp.resemble.ai/sse"
    }
  }
}
```

**Claude Desktop / Claude Code** — add to `claude_desktop_config.json` (or `.claude/mcp.json`):

```json
{
  "mcpServers": {
    "resemble": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.resemble.ai/sse"]
    }
  }
}
```

**Any other MCP-compatible agent** (OpenClaw, Hermes Agent, Windsurf, Cline, Continue, etc.) — point the MCP client at `https://mcp.resemble.ai/sse`. See the [Resemble MCP README](https://github.com/resemble-ai/resemble-mcp) for per-agent config snippets.

**Prefer self-hosting?** Clone [resemble-ai/resemble-mcp](https://github.com/resemble-ai/resemble-mcp) and run it locally.

Once connected, the agent gets tools like `resemble_docs_lookup`, `resemble_api_endpoint`, and `resemble_api_search` — this skill is built to use them when available.

## How It Works

The skill is a single markdown file (`SKILL.md`) that provides your AI agent with:

- **Decision tree** — maps user intent to the correct API capability and endpoint
- **Complete API reference** — every endpoint, parameter, and response schema
- **Score interpretation** — how to read and present detection confidence scores
- **Workflow templates** — full forensics, quick checks, and provenance pipelines
- **Red flags** — anti-patterns the agent should catch in its own reasoning
- **Error handling** — every status code with cause and resolution

Agents like [Hermes Agent](https://github.com/nousresearch/hermes-agent) with self-improving skill systems will automatically refine their use of this skill over time. [OpenClaw](https://github.com/openclaw/openclaw)'s 100+ prebuilt skills ecosystem makes it a natural fit — drop `detect.md` in and it works alongside existing skills immediately.

## Example Prompts

Once installed, try asking your agent:

- *"Is this audio file a deepfake?"*
- *"Analyze this video for AI manipulation and tell me what platform might have generated it"*
- *"Apply a watermark to this image for provenance tracking"*
- *"Check if this speaker matches the voice profile we have on file"*
- *"What can you tell me about this audio — speaker, emotion, language, any abnormalities?"*

## Links

- [Resemble AI](https://resemble.ai) — Platform
- [`mcp.resemble.ai/sse`](https://mcp.resemble.ai/sse) — Hosted MCP endpoint (zero install)
- [resemble-ai/resemble-mcp](https://github.com/resemble-ai/resemble-mcp) — Self-host the MCP server
- [API Documentation](https://docs.resemble.ai) — Resemble API docs
- [skills.sh](https://skills.sh) — The Open Agent Skills Ecosystem
- [OpenClaw](https://github.com/openclaw/openclaw) — Open-source AI agent (formerly Clawdbot)
- [Hermes Agent](https://github.com/nousresearch/hermes-agent) — Self-improving AI agent by Nous Research

## License

Apache-2.0
