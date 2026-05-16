# Bilibili Transcript Skill

OpenCode skill for transcribing Bilibili (B站) videos to text, enabling AI agents to learn from video content.

## What it does

When you send a B站 video link to your AI agent, this skill:

1. Fetches existing subtitles if available (instant)
2. Falls back to downloading audio and transcribing with local Whisper (1-2 min for typical videos)
3. Returns the full transcript your agent can read and summarize

## Setup

See [references/setup.md](references/setup.md) for full installation instructions.

Quick check:

```bash
yt-dlp --version && ffmpeg -version | head -1 && whisper --help > /dev/null && echo "All good"
```

## Performance

CPU transcription with `faster-whisper` small model (~4 cores):

| Video length | Time |
|---|---|
| 5 min | ~1 min |
| 15 min | ~3 min |
| 30 min | ~6.5 min |
| 1 hour | ~13 min |

## MCP Tools Used

- `video-toolkit` → `get-transcript` — existing subtitles (requires B站 login)
- `video-toolkit` → `generate-subtitles` — AI transcription via local Whisper

## License

MIT
