# Resemble Detect Skill

An agent skill for deepfake detection and media intelligence — powered by [Resemble AI](https://resemble.ai).

Give any AI agent the ability to detect AI-generated audio, images, and video, then analyze completed detection results using Resemble's Detect and Intelligence APIs.

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

This skill teaches your agent to use Resemble's Detect and Intelligence APIs directly:

| Capability | Description |
|---|---|
| **Deepfake Detection** | Analyze audio, image, and video for synthetic manipulation with labels, scores, status, and visualizations |
| **Direct Uploads** | Submit local/private files directly to `POST /detect` with multipart upload, or use secure upload tokens for larger/non-public media |
| **Audio Source Tracing** | Identify which AI platform, such as ElevenLabs or Resemble, may have synthesized detected fake audio |
| **Intelligence** | Extract speaker info, emotion, transcription, misinformation signals, abnormalities, and image/video context |
| **Detect Intelligence** | Ask natural-language follow-up questions about completed detection results |


## Requirements

- A [Resemble AI](https://app.resemble.ai) API key
- `curl` for direct API calls
- One of these media inputs for detection:
  - a public HTTPS URL,
  - a local file upload up to 150 MB, or
  - a secure upload token for larger/non-public media

## Direct API Usage

This skill is built around direct Resemble REST API calls. Agents do **not** need an MCP server to run detection workflows; they can call the API with `curl` using a Resemble API key.

```bash
export RESEMBLE_API_KEY="..."
BASE_URL="https://app.resemble.ai/api/v2"

curl --request POST "${BASE_URL}/detect" \
  -H "Authorization: Bearer ${RESEMBLE_API_KEY}" \
  -H "Prefer: wait" \
  -H "Content-Type: application/json" \
  --data '{
    "url": "https://example.com/media.mp4",
    "intelligence": true,
    "visualize": true,
    "audio_source_tracing": true,
    "zero_retention_mode": true
  }'
```

For private/local media, `POST /detect` also supports direct `multipart/form-data` file upload up to 150 MB. For larger or non-public media, use the Secure Upload flow and pass the returned `media_token` into `POST /detect`. See [`SKILL.md`](./SKILL.md) for copy-pasteable workflows.

## Optional: Pair With the Resemble MCP Server for Docs

If your agent supports MCP, you can still pair this skill with the **[Resemble MCP server](https://github.com/resemble-ai/resemble-mcp)** for live documentation and endpoint schema lookup. MCP is optional; it is not required for the skill's Detect or Intelligence workflows.

Hosted endpoint:

```text
https://mcp.resemble.ai/sse
```

See the [Resemble MCP README](https://github.com/resemble-ai/resemble-mcp) for per-agent config snippets.

## How It Works

The skill is a single markdown file (`SKILL.md`) that provides your AI agent with:

- **Decision tree** — maps user intent to Detect, Intelligence, or Detect Intelligence endpoints
- **Direct API examples** — curl-first workflows for URL, file upload, secure upload, and polling
- **Score interpretation** — how to read and present detection confidence scores
- **Workflow templates** — full media forensics and quick authenticity checks
- **Red flags** — anti-patterns the agent should catch in its own reasoning
- **Error handling** — common status codes with cause and resolution

Agents like [Hermes Agent](https://github.com/nousresearch/hermes-agent) with self-improving skill systems will automatically refine their use of this skill over time. [OpenClaw](https://github.com/openclaw/openclaw)'s 100+ prebuilt skills ecosystem makes it a natural fit — drop `detect.md` in and it works alongside existing skills immediately.

## Example Prompts

Once installed, try asking your agent:

- *"Is this audio file a deepfake?"*
- *"Analyze this video for AI manipulation and tell me what platform might have generated it."*
- *"Run detection with intelligence on this image URL."*
- *"Ask a follow-up question about this completed detection result."*
- *"What can you tell me about this audio — speaker, emotion, language, any abnormalities?"*

## Links

- [Resemble AI](https://resemble.ai) — Platform
- [Detect API Documentation](https://docs.resemble.ai/detect.md) — Deepfake detection docs
- [Submit Detection Job](https://docs.resemble.ai/detect/create.md) — `POST /detect`
- [Intelligence Documentation](https://docs.resemble.ai/detect/intelligence.md) — Media intelligence docs
- [`mcp.resemble.ai/mcp`](https://mcp.resemble.ai/mcp) — Optional hosted MCP endpoint for docs/schema lookup
- [resemble-ai/resemble-mcp](https://github.com/resemble-ai/resemble-mcp) — Optional MCP docs server
- [skills.sh](https://skills.sh) — The Open Agent Skills Ecosystem
- [OpenClaw](https://github.com/openclaw/openclaw) — Open-source AI agent (formerly Clawdbot)
- [Hermes Agent](https://github.com/nousresearch/hermes-agent) — Self-improving AI agent by Nous Research

## License

Apache-2.0
