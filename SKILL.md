---
name: video-transcript
description: Transcribe online videos (Bilibili/B站, YouTube, Vimeo, Twitch, and any yt-dlp supported platform) to text so agents can learn from video content. Use whenever the user provides a video link (bilibili.com, b23.tv, youtube.com, youtu.be, vimeo.com, twitch.tv, etc.), asks to transcribe a video, or wants to extract/summarize content from a video. Supports existing subtitles (fast) and AI-generated transcription via local Whisper (slower but always available).
---

# Video Transcription

Transcribes online videos to text using the `video-toolkit` MCP server. Works with **any platform supported by yt-dlp** — Bilibili (B站), YouTube, Vimeo, Twitch, and hundreds more.

## Workflow

### Step 1: Resolve short links

Short links like `b23.tv/XXXXX` or `youtu.be/XXXXX` must be resolved first:

```bash
curl -sI "https://b23.tv/XXXXX" | grep -i location
```

### Step 2: Try existing subtitles first

Call `get-transcript` from the video-toolkit MCP server:

```
MCP tool: video-toolkit → get-transcript
Args: { "url": "<video URL>", "lang": "zh" }
```

- If the video has subtitles → instant result
- If no subtitles → fails, proceed to Step 3

### Step 3: Generate subtitles with Whisper

Call `generate-subtitles` from the video-toolkit MCP server:

```
MCP tool: video-toolkit → generate-subtitles
Args: { "url": "<video URL>", "language": "zh" }
```

Downloads audio, converts to 16kHz mono, transcribes with local Whisper (small model).

**Timing** (CPU, ~4 cores):
| Video length | Transcription time |
|---|---|
| 5 min | ~1 min |
| 15 min | ~3 min |
| 30 min | ~6.5 min |
| 1 hour | ~13 min |

### Step 4: Use the transcript

Read the returned text and answer the user's question. Timestamps included — cite specific moments if needed.

### Step 5: Check available languages

```
MCP tool: video-toolkit → list-transcript-languages
Args: { "url": "<video URL>" }
```

## Supported Platforms

Anything [yt-dlp supports](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md):

**Bilibili** (bilibili.com, b23.tv) · **YouTube** (youtube.com, youtu.be) · **Vimeo** · **Twitch** · Podcast feeds · Direct media URLs · And hundreds more

## Configuration

The `video-toolkit` MCP server must be in `opencode.jsonc`:

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

The local whisper wrapper is bundled at `scripts/whisper`. Copy to `~/.local/bin/whisper` and `chmod +x`.

## Dependencies

- `yt-dlp` — downloads video/audio
- `ffmpeg` — audio format conversion
- `faster-whisper` (`pip install faster-whisper`) — local transcription
- `whisper` wrapper script at `~/.local/bin/whisper`

## Limitations

- **Some platforms require authentication** for subtitles (e.g. B站). `generate-subtitles` with local Whisper always works as fallback.
- **Transcription speed**: CPU-bound, ~4-5x realtime with small model.
- **Accuracy**: Good for clear speech; noise or heavy accents reduce quality.
- **Long videos** (>1h) may exceed context windows. Process in chunks.
