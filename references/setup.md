# Video Transcript Skill Setup

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

## Configure video-toolkit MCP (exposes native tools)

The MCP server is configured in `opencode.jsonc` and its tools are exposed as native `video-toolkit_*` tools.

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
    "YT_DLP_PATH": "/home/ubuntu/.local/bin/yt-dlp",
    "FFMPEG_PATH": "/home/ubuntu/.local/bin/ffmpeg",
    "FFPROBE_PATH": "/home/ubuntu/.local/bin/ffprobe",
    "TRANSCRIPT_MCP_WHISPER_ENGINE": "local",
    "WHISPER_BINARY_PATH": "/home/ubuntu/.local/bin/whisper",
    "WHISPER_MODEL_PATH": "small"
  }
}
```

Restart OpenCode to load the tools. Once loaded, tools are called natively (e.g., `video-toolkit_get-transcript`) — no `skill_mcp` wrapper needed.
