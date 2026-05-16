---
name: bilibili-transcript
description: Transcribe Bilibili (B站) videos to text so agents can learn from video content. Use whenever the user provides a B站 video link (bilibili.com, b23.tv), asks to transcribe a B站 video, or wants to extract/summarize content from a 哔哩哔哩 video. Supports existing subtitles (fast) and AI-generated transcription via Whisper (slower but always available).
---

# Bilibili Video Transcription

Transcribes B站 videos to text using the `video-toolkit` MCP server.

## Workflow

### Step 1: Resolve short links

B23.tv short links (`https://b23.tv/XXXXX`) must be resolved first:

```bash
curl -sI "https://b23.tv/XXXXX" | grep -i location
```

This gives the full `bilibili.com/video/BVxxxxx` URL.

### Step 2: Try existing subtitles first

Call `get-transcript` from the video-toolkit MCP server:

```
MCP tool: video-toolkit → get-transcript
Args: { "url": "<bilibili video URL>", "lang": "zh" }
```

- If the video has CC subtitles and you're logged in to B站 → instant result
- If no subtitles or not logged in → will fail, proceed to Step 3

### Step 3: Generate subtitles with Whisper

Call `generate-subtitles` from the video-toolkit MCP server:

```
MCP tool: video-toolkit → generate-subtitles
Args: { "url": "<bilibili video URL>", "language": "zh" }
```

This downloads the audio, converts it, and transcribes with local Whisper (small model).

**Timing** (CPU, ~4 cores):
| Video length | Transcription time |
|---|---|
| 5 min | ~1 min |
| 15 min | ~3 min |
| 30 min | ~6.5 min |
| 1 hour | ~13 min |

### Step 4: Use the transcript

Once transcribed, read the returned text and answer the user's question. The transcript includes timestamps — use them to cite specific moments if needed.

## Configuration

The video-toolkit MCP server must be configured in `opencode.jsonc`:

```json
"video-toolkit": {
  "type": "local",
  "command": ["node", "/path/to/video-toolkit-mcp/dist/index.js"],
  "enabled": true,
  "timeout": 600000,
  "environment": {
    "YT_DLP_PATH": "yt-dlp",
    "FFMPEG_PATH": "ffmpeg",
    "TRANSCRIPT_MCP_WHISPER_ENGINE": "local",
    "WHISPER_BINARY_PATH": "/home/ubuntu/.local/bin/whisper",
    "WHISPER_MODEL_PATH": "small"
  }
}
```

The local whisper wrapper is bundled at `scripts/whisper`. Copy it to `~/.local/bin/whisper` and make it executable.

## Dependencies

- `yt-dlp` — downloads B站 video audio
- `ffmpeg` — audio format conversion
- `faster-whisper` (`pip install faster-whisper`) — local transcription engine
- `whisper` wrapper script at `~/.local/bin/whisper`

## Limitations

- **B站 subtitles require login**: `get-transcript` (fetching existing subtitles) only works when authenticated with B站 cookies. When not logged in, always fall through to `generate-subtitles`.
- **Transcription speed**: CPU-bound; roughly 4-5x realtime with the small model.
- **Accuracy**: Good for clear speech; background noise or heavy accents may reduce quality.
- **Long videos**: Videos over 1 hour may exceed context windows. Consider summarizing in chunks.
