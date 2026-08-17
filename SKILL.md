---
name: resemble-detect
description: Deepfake detection and media safety — detect AI-generated audio, images, and video, trace synthesis sources, and analyze media intelligence using direct Resemble AI API calls
---

# Resemble Detect — Deepfake Detection & Media Safety

Analyze audio, image, and video for synthetic manipulation, AI-generated content, and media intelligence using **direct Resemble AI API calls**.

## Core Principle — THE IRON LAW

**"NEVER DECLARE MEDIA AS REAL OR FAKE WITHOUT A COMPLETED DETECTION RESULT."**

Do not guess, infer, or speculate about media authenticity. Every authenticity claim must be backed by a completed Resemble Detect job with a returned `label`, `score`, and `status: "completed"`. If the detection is still `processing`, wait. If it `failed`, say so — do not substitute your own judgment.

## When to Use

Use this skill whenever the user's request involves any of these:

- Checking if audio, video, or image is AI-generated or manipulated
- Detecting deepfakes in any media format
- Verifying media authenticity or provenance
- Identifying which AI platform synthesized audio (source tracing)
- Analyzing media for speaker info, emotion, transcription, or misinformation
- Asking natural-language questions about detection results
- Running a full investigation workflow — insurance claim, breaking news, ID check, submitted evidence — where detection is one input among several
- Any mention of: "deepfake", "fake detection", "synthetic media", "media forensics", "authenticity check", "source tracing", "is this real"

**Do NOT use** for text-to-speech generation, voice cloning, or speech-to-text transcription — those are separate Resemble capabilities.

## Required Setup

- **API key:** Bearer token from the Resemble dashboard: <https://app.resemble.ai/account/api>
- **Environment variable:** prefer `RESEMBLE_API_KEY`
- **Base URL:** `https://app.resemble.ai/api/v2`
- **Auth header:** `Authorization: Bearer $RESEMBLE_API_KEY`
- **Media inputs:** `POST /detect` accepts exactly one of:
  - direct `multipart/form-data` file upload as `file` (up to 150 MB),
  - public HTTPS `url`, or
  - `media_token` from `POST /secure_uploads`.

Never print API keys or paste bearer tokens into chat. Use environment variables in examples and commands.

## Capability Decision Tree

| User wants to...                                      | Use this                  | API endpoint               |
|-------------------------------------------------------|---------------------------|----------------------------|
| Check if media is AI-generated / deepfake             | **Deepfake Detection**    | `POST /detect`, then `GET /detect/{uuid}` |
| Upload a private/local file without public hosting    | **Direct Upload**         | `POST /detect` multipart `file=@...` |
| Analyze a file larger than 150 MB without public URL  | **Secure Upload**         | `POST /secure_uploads`, then `POST /detect` with `media_token` |
| Know *which AI platform* made fake audio              | **Audio Source Tracing**  | `POST /detect` with `audio_source_tracing: true` |
| Get speaker info, emotion, transcription from media   | **Intelligence**          | `POST /intelligence`       |
| Ask questions about a completed detection             | **Detect Intelligence**   | `POST /detects/{uuid}/intelligence`, then poll answer |
| Run a managed multi-step investigation with a verdict | **Detect Agents**         | `GET /agents`, then `POST /agents/{preset_id}/run` (SSE) |

When multiple media capabilities apply, combine them in a single `POST /detect` call using flags such as `intelligence: true`, `audio_source_tracing: true`, `visualize: true`, `use_reverse_search: true`, and `zero_retention_mode: true` instead of making separate jobs.

## Direct API Call Rules

1. **Use direct HTTP requests first.** This skill is intentionally written around `curl` and the Resemble REST API, not MCP tool calls.
2. **Use `Prefer: wait` when a synchronous result is acceptable.** Without it, submit the job, capture the returned UUID, and poll.
3. **Poll async jobs until terminal status.** Terminal statuses are `completed` and `failed`.
4. **Use zero retention for sensitive media.** Set `zero_retention_mode: true` for media detection when privacy matters.
5. **Only report completed results.** Pending/processing jobs are not verdicts.

## Reusable Shell Setup

Use this at the start of any command sequence:

```bash
: "${RESEMBLE_API_KEY:?Set RESEMBLE_API_KEY first}"
BASE_URL="https://app.resemble.ai/api/v2"
AUTH_HEADER="Authorization: Bearer ${RESEMBLE_API_KEY}"
```

If you need JSON extraction and `jq` is available, use it. If not, use `python3 -c 'import json,sys; ...'`.

---

## Phase 1: Deepfake Detection

Submit any audio, image, or video for AI-generated content analysis.

### Submit a Detection from a Public URL

Use this when the media is already reachable via HTTPS:

```bash
curl --request POST "${BASE_URL}/detect" \
  -H "$AUTH_HEADER" \
  -H "Prefer: wait" \
  -H "Content-Type: application/json" \
  --data '{
    "url": "https://example.com/media.mp4",
    "visualize": true,
    "intelligence": true,
    "audio_source_tracing": true,
    "use_reverse_search": true,
    "zero_retention_mode": true
  }'
```

For asynchronous mode, omit `Prefer: wait`, capture `.item.uuid`, then poll `GET /detect/{uuid}`.

### Submit a Detection from a Local File

Direct file uploads are supported for files up to 150 MB:

```bash
curl --request POST "${BASE_URL}/detect" \
  -H "$AUTH_HEADER" \
  -H "Prefer: wait" \
  -F "file=@/path/to/media.mp4" \
  -F "intelligence=true" \
  -F "visualize=true" \
  -F "audio_source_tracing=true" \
  -F "frame_length=2"
```

Allowed direct-upload extensions include `.wav`, `.mp3`, `.m4a`, `.ogg`, `.aac`, `.flac`, `.amr`, `.3gp`, `.3gpp`, `.mp4`, `.mov`, `.avi`, `.mkv`, `.webm`, `.jpg`, `.jpeg`, `.png`, `.gif`, and `.webp`.

### Submit a Detection with a Secure Upload Token

Use secure uploads when the file is larger than 150 MB or should not be hosted publicly. First upload the file:

```bash
curl --request POST "${BASE_URL}/secure_uploads" \
  -H "$AUTH_HEADER" \
  -F "file=@/path/to/media.mp4"
```

Then submit the returned token as `media_token`:

```bash
curl --request POST "${BASE_URL}/detect" \
  -H "$AUTH_HEADER" \
  -H "Content-Type: application/json" \
  --data '{
    "media_token": "SECURE_UPLOAD_TOKEN",
    "intelligence": true,
    "visualize": true,
    "zero_retention_mode": true
  }'
```

Secure upload tokens are short-lived. Use them promptly.

### Detection Parameters

| Parameter              | Type    | Required | Description                                              |
|------------------------|---------|----------|----------------------------------------------------------|
| `file`                 | file    | One of   | Multipart file upload, max 150 MB                        |
| `url`                  | string  | One of   | Public HTTPS URL to audio, image, or video file          |
| `media_token`          | string  | One of   | Token from `POST /secure_uploads`                        |
| `callback_url`         | string  | No       | Webhook URL for async completion notification             |
| `visualize`            | boolean | No       | Generate heatmap/treeview visualization artifacts         |
| `intelligence`         | boolean | No       | Run multimodal intelligence analysis alongside detection  |
| `audio_source_tracing` | boolean | No       | Identify which AI platform synthesized fake audio         |
| `frame_length`         | integer | No       | Audio/video analysis window size in seconds (1–4, default 2) |
| `start_region`         | number  | No       | Start of segment to analyze (seconds)                    |
| `end_region`           | number  | No       | End of segment to analyze (seconds)                      |
| `max_video_secs`       | number  | No       | Cap processed video duration                             |
| `model_types`          | string  | No       | `"image"` or `"talking_head"` for video face-swap detection |
| `use_reverse_search`   | boolean | No       | Enable reverse image search (image only)                 |
| `use_ood_detector`     | boolean | No       | Enable out-of-distribution detection                     |
| `zero_retention_mode`  | boolean | No       | Auto-delete submitted media after detection completes    |

Exactly one of `file`, `url`, or `media_token` must be supplied.

### Poll for Detection Results

```bash
DETECT_UUID="..."
curl --request GET "${BASE_URL}/detect/${DETECT_UUID}" \
  -H "$AUTH_HEADER"
```

Polling best practice: start at 2-second intervals, back off to 5 seconds, then 10 seconds. Stop when `item.status` is `completed` or `failed`.

If you need a small polling helper:

```bash
DETECT_UUID="..."
for delay in 2 2 5 5 10 10 10 10 10 10; do
  response=$(curl -sS "${BASE_URL}/detect/${DETECT_UUID}" -H "$AUTH_HEADER")
  printf '%s\n' "$response"
  status=$(printf '%s' "$response" | python3 -c 'import json,sys; print(json.load(sys.stdin).get("item",{}).get("status",""))')
  [ "$status" = "completed" ] && break
  [ "$status" = "failed" ] && break
  sleep "$delay"
done
```

### Reading Results by Media Type

**Audio results** — in `item.metrics`:

```json
{
  "label": "fake",
  "score": ["0.92", "0.88", "0.95"],
  "consistency": "0.91",
  "aggregated_score": "0.92",
  "image": "https://..."
}
```

- `label`: `"fake"` or `"real"` — the verdict
- `score`: per-chunk prediction scores
- `aggregated_score`: overall confidence (0.0–1.0, higher = more likely synthetic)
- `consistency`: how consistent the prediction is across chunks
- `image`: visualization heatmap URL if `visualize: true`

**Image results** — in `item.image_metrics`:

```json
{
  "type": "FinalResult",
  "label": "Fake",
  "score": 0.87,
  "image": "https://...",
  "ifl": { "score": 0.82, "heatmap": "https://..." },
  "reverse_image_search_sources": [
    { "url": "...", "title": "...", "verdict": "known_fake", "similarity": 0.95 }
  ]
}
```

**Video results** — in `item.video_metrics`, with audio metrics in `item.metrics` when the video has audio:

```json
{
  "label": "Fake",
  "score": 0.89,
  "certainty": 0.91,
  "treeview": "https://...",
  "children": [
    {
      "type": "VideoResult",
      "conclusion": "Fake",
      "score": 0.89,
      "timestamp": 2.5,
      "children": []
    }
  ]
}
```

### Interpreting Scores

| Score Range | Interpretation                                      |
|-------------|-----------------------------------------------------|
| 0.0 – 0.3   | Strong indication of authentic/real media           |
| 0.3 – 0.5   | Inconclusive — recommend additional analysis        |
| 0.5 – 0.7   | Likely synthetic — flag for review                  |
| 0.7 – 1.0   | High confidence synthetic/AI-generated              |

Always present scores with context. Say "The detection returned a score of 0.87, indicating high confidence that this media is AI-generated" — never just "it's fake."

---

## Phase 2: Intelligence — Media Analysis

Analyze media for rich structured insights independently or alongside detection.

### Standalone Intelligence

```bash
curl --request POST "${BASE_URL}/intelligence" \
  -H "$AUTH_HEADER" \
  -H "Content-Type: application/json" \
  --data '{
    "url": "https://example.com/audio.mp3",
    "media_type": "audio"
  }'
```

By default, `POST /intelligence` is synchronous. If you provide `callback_url`, it becomes asynchronous and returns an intelligence record that you can poll with `GET /intelligences/{uuid}`.

**Parameters:**

| Parameter      | Type   | Required | Description                                              |
|----------------|--------|----------|----------------------------------------------------------|
| `url`          | string | One of   | HTTPS URL to media file                                  |
| `media_token`  | string | One of   | Token from secure upload                                 |
| `detect_id`    | string | No       | UUID of existing detect to associate                     |
| `media_type`   | string | No       | `"audio"`, `"video"`, or `"image"` (auto-detected if omitted) |
| `callback_url` | string | No       | Webhook for async completion                             |

**Audio/video intelligence may include:** speaker info, language/dialect, emotion, speaking style, context, message summary, abnormalities, transcription, translation, and misinformation analysis.

**Image intelligence may include:** scene description, subjects, authenticity analysis, context/setting, abnormalities, and misinformation analysis.

### Get Intelligence

```bash
INTELLIGENCE_UUID="..."
curl --request GET "${BASE_URL}/intelligences/${INTELLIGENCE_UUID}" \
  -H "$AUTH_HEADER"
```

### Detect Intelligence — Ask Questions About Completed Detections

After a detection completes, submit natural-language questions about it:

```bash
DETECT_UUID="..."
curl --request POST "${BASE_URL}/detects/${DETECT_UUID}/intelligence" \
  -H "$AUTH_HEADER" \
  -H "Content-Type: application/json" \
  --data '{"query": "Summarize the detection results in plain language."}'
```

This returns a question UUID. Poll until the question status is `completed` or `failed`:

```bash
QUESTION_UUID="..."
curl --request GET "${BASE_URL}/detects/${DETECT_UUID}/intelligence/${QUESTION_UUID}" \
  -H "$AUTH_HEADER"
```

Good questions to suggest:

- "Summarize the detection results in plain language."
- "What specific indicators suggest this is AI-generated?"
- "How do the audio and video detection results differ?"
- "What is the confidence level and what does it mean?"
- "Are there any inconsistencies in the analysis?"

Prerequisite: the detection must have `status: "completed"`. Asking about a processing or failed detection can return `422`.

---

## Phase 3: Audio Source Tracing

When audio is detected as synthetic, identify which AI platform generated it.

Enable it in the `POST /detect` request:

```json
{
  "url": "https://example.com/audio.wav",
  "audio_source_tracing": true
}
```

Result appears in the detection response under `item.audio_source_tracing`:

```json
{
  "label": "elevenlabs",
  "error_message": null
}
```

Known source labels include `resemble_ai`, `elevenlabs`, `real`, and others as the model expands.

Standalone lookup endpoints:

```bash
curl --request GET "${BASE_URL}/audio_source_tracings" -H "$AUTH_HEADER"
curl --request GET "${BASE_URL}/audio_source_tracings/${TRACE_UUID}" -H "$AUTH_HEADER"
```

Important: source tracing is most useful when audio is labeled `fake`. If the audio is `real`, a source tracing result may be absent or identify the media as real.

---

## Phase 4: Detect Agents

Detect Agents are six Resemble-managed investigators that wrap a detection in a multi-step workflow: they run Detect, pull in supporting evidence and web research, and end on a written assessment. Use one when the question is "should this claim/post/document be trusted?" rather than "is this file synthetic?".

| Agent | `preset_id` |
|-------|-------------|
| Investigate Social Media Content | `investigate_social_content` |
| Review an Insurance Claim        | `review_insurance_claim`     |
| Verify Breaking News Media       | `verify_breaking_news`       |
| Verify a Document or Receipt     | `verify_document`            |
| Verify Submitted Evidence        | `verify_evidence`            |
| Verify an ID                     | `verify_id`                  |

List them (the `uuid` is the same stable identifier as `preset_id`):

```bash
curl --request GET "${BASE_URL}/agents" -H "$AUTH_HEADER"
```

### Run an Investigation

`multipart/form-data`, with exactly one primary media source (`file` or `url`). The response is a Server-Sent Events stream, so pass `--no-buffer`:

```bash
curl --no-buffer --request POST "${BASE_URL}/agents/verify_document/run" \
  -H "$AUTH_HEADER" \
  -H "Accept: text/event-stream" \
  -F "file=@/path/to/receipt.pdf" \
  -F "query=Is this receipt genuine?" \
  -F "evidence[]=@/path/to/order-confirmation.png" \
  -F "check_urls=https://example.com/original-listing"
```

| Field        | Required    | Description                                          |
|--------------|-------------|------------------------------------------------------|
| `file`       | One of      | Media to analyze                                     |
| `url`        | One of      | Public HTTPS media URL                               |
| `query`      | No          | The investigation question or objective              |
| `evidence[]` | No          | Supporting files; repeat the field for multiple      |
| `check_urls` | No          | Additional URLs for the agent to check               |

### Reading the Stream

Each frame is `data: {json}`. The frames that matter:

- `run_started` — carries `run_id`. Save it; the run is persisted server-side even if you disconnect.
- `detect` — the Resemble Detect evidence, with `label` and `score`. **This is the authenticity verdict under the Iron Law.**
- `gate` — whether the investigative agent proceeded past detection (`agent_ran`).
- `tool_call` / `tool_result` — research and detection steps as they happen.
- `token` / `agent_message` / `message_end` — incremental agent narration.
- `final_verdict` — the written assessment. `intelligence` is a string, and may contain serialized JSON when the agent uses a schema.
- `done` — completed. `error` — failed after the stream opened (HTTP stays `200`).

**Iron Law applies unchanged.** `final_verdict` is the agent's reasoning, not a detection result. Never report media as real or fake on the strength of the narration alone — cite the `label` and `score` from the `detect` frame. If no `detect` frame arrived, say the detection did not complete.

### Retrieve Past Runs

```bash
curl --request GET "${BASE_URL}/agents/verify_document/runs" -H "$AUTH_HEADER"
curl --request GET "${BASE_URL}/agents/verify_document/runs/${RUN_ID}" -H "$AUTH_HEADER"
```

Run detail adds the full event transcript, the config snapshot, and the agent's memory before/after — enough to replay the investigation.

### Access

Listing agents and reading run history need nothing beyond a valid API key. **Starting a run** requires entitlement: the D-Agent tier bundle on the team, or one of the team's **5 lifetime free runs**. When neither applies the run endpoint returns `402 Payment Required` as plain JSON *before* the stream opens — check for that before assuming a stream. `GET /agents` also returns `free_runs_remaining`, `free_runs_limit`, and `entitled`, so prefer checking those over triggering a 402.

Creating or editing agents is not available over the API; the six presets are managed by Resemble and activate on first run.

---

## Recommended Workflows

### Full Media Forensics (Most Thorough)

1. Submit one `POST /detect` job with all useful flags enabled:
   ```json
   {
     "url": "https://example.com/suspect.mp4",
     "visualize": true,
     "intelligence": true,
     "audio_source_tracing": true,
     "use_reverse_search": true,
     "zero_retention_mode": true
   }
   ```
2. Poll `GET /detect/{uuid}` until `status: "completed"`.
3. Read `metrics`, `image_metrics`, or `video_metrics` for the verdict.
4. Read `intelligence.description` if intelligence was requested.
5. If audio is synthetic, check `audio_source_tracing.label` for the likely source platform.
6. Ask a follow-up via `POST /detects/{uuid}/intelligence` only after the detection is complete.

### Quick Authenticity Check (Fastest)

1. Submit minimal detection using `Prefer: wait`:
   ```bash
   curl --request POST "${BASE_URL}/detect" \
     -H "$AUTH_HEADER" \
     -H "Prefer: wait" \
     -H "Content-Type: application/json" \
     --data '{"url": "https://example.com/media.wav"}'
   ```
2. Confirm `item.status` is `completed`.
3. Check `item.metrics.label` and `item.metrics.aggregated_score` for audio, or `item.image_metrics.label` / `item.video_metrics.label` and `score` for image/video.
4. Report the result with score context and detector caveats.

---

## Red Flags — Stop and Reassess

- **Declaring authenticity without a completed detection result** — never say media is real or fake based on visual/auditory inspection alone.
- **Ignoring status** — `processing` is not a verdict; `failed` requires reporting the failure.
- **Ignoring score and reporting only label** — a `fake` label with score 0.51 is very different from score 0.95.
- **Submitting multiple media sources** — `file`, `url`, and `media_token` are mutually exclusive for detection.
- **Uploading files larger than 150 MB directly** — use secure upload or public URL.
- **Polling too aggressively** — start at 2 seconds and back off; do not loop at sub-second intervals.
- **Asking Detect Intelligence questions before detection completes** — this can return `422`.
- **Expecting source tracing on authentic audio** — source tracing is most useful for synthetic audio.
- **Leaking credentials** — never print bearer tokens, `.env` files, or authorization headers with real secrets.

## Response Presentation Guidelines

When presenting results to users:

1. **Lead with the detector verdict** — "Resemble Detect classified this audio as likely AI-generated."
2. **Include status and score** — only report authenticity when status is `completed`.
3. **Name the fields used** — e.g. `item.metrics.aggregated_score`, `item.image_metrics.score`, or `item.video_metrics.score`.
4. **Mention limitations** — detection is probabilistic, not absolute proof or legal evidence.
5. **Include operational details** — whether intelligence, reverse search, OOD, source tracing, or zero retention was used.
6. **For inconclusive scores (0.3–0.5)** — explicitly state the result is inconclusive and recommend additional analysis or manual review.

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| 400 | Invalid request body, missing media source, unsupported file, or multiple media sources | Check payload and supply exactly one `file`, `url`, or `media_token` |
| 401 | Invalid or missing API key | Verify `RESEMBLE_API_KEY` and auth header |
| 404 | Detection/question/intelligence UUID not found | Verify the UUID and endpoint path |
| 422 | Detection not completed for Detect Intelligence or request validation failed | Wait for completion or fix the request |
| 429 | Rate limited | Back off and retry with exponential delay |
| 500 | Server error | Retry once, then report failure |

## Privacy & Compliance Notes

- **Zero retention mode:** set `zero_retention_mode: true` on media detection to auto-delete submitted media after analysis. Responses redact media URLs when enabled.
- **Callbacks:** if using `callback_url`, ensure it is HTTPS and authenticated on your side.
- **Secrets:** keep API keys in environment variables or secret managers, never in skill files or prompts.
