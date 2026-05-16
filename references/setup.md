# Bilibili Transcript Skill Setup

## Install Dependencies

```bash
# yt-dlp for video/audio downloading
pip install yt-dlp

# ffmpeg for audio processing (static build if no package manager)
# Download from: https://johnvansickle.com/ffmpeg/

# faster-whisper for local transcription
pip install faster-whisper
```

## Install Whisper Wrapper

Copy the bundled `scripts/whisper` to a PATH location:

```bash
cp scripts/whisper ~/.local/bin/whisper
chmod +x ~/.local/bin/whisper
```

## Configure video-toolkit MCP

Clone and build:

```bash
git clone https://github.com/JamesANZ/video-transcript-mcp.git
cd video-transcript-mcp
npm install && npm run build
```

Add to `opencode.jsonc` under `mcp`:

```json
"video-toolkit": {
  "type": "local",
  "command": ["node", "/absolute/path/to/video-toolkit-mcp/dist/index.js"],
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

Restart OpenCode to load the new MCP server.
