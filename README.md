# install gemini-cli with vox-box, kokoro-fastapi-zh, voicemode-zh(diyism/gemini-cli-voice):
    #1. install whisper.cpp for voicemode STT/ASR:
    #whisper-server of whisper.cpp is not an openai compatible api:
    #git clone https://github.com/ggml-org/whisper.cpp.git
    #cd whisper.cpp
    #sh ./models/download-ggml-model.sh base
    #sudo mkdir -p /usr/local/share/whisper.cpp/models
    #sudo cp ./models/ggml-base.bin /usr/local/share/whisper.cpp/models/
    #cmake -B build
    #cmake --build build --config Release
    #sudo install ./build/bin/whisper-server /usr/bin/
    #whisper-server -l zh -m /usr/local/share/whisper.cpp/models/ggml-base.bin --port 2022

    #whisper-cpp-server(https://github.com/litongjava/whisper-cpp-server) is also not an openai compatible api:
    #docker run -dit --name=whisper-server -p 2022:8080 -v /usr/local/share/whisper.cpp/models/ggml-base.bin:/models/ggml-base.bin litongjava/whisper-cpp-server:1.0.0 /app/whisper_http_server_base_httplib -m /models/ggml-base.bin

    #vox-box is an openai compatible api server:
    pip install vox-box==0.0.12
    mkdir -p ~/.cache/huggingface/hub/cache/huggingface/
    ln -s ~/.cache/huggingface/hub/models--Systran--faster-whisper-small ~/.cache/huggingface/hub/cache/huggingface/
    vox-box start --huggingface-repo-id Systran/faster-whisper-small --data-dir ~/.cache/huggingface/hub --host 127.0.0.1 --port 2022
    export VOICEMODE_STT_BASE_URLS="http://127.0.0.1:2022/v1"
    #test port:
    curl http://127.0.0.1:2022/v1/audio/transcriptions -F "a=b"
    #{"detail":[{"type":"missing","loc":["body","file"],"msg":"Field required","input":null},{"type":"missing","loc":["body","model"],"msg":"Field required","input":null}]}

    #2. install kokoro-fastapi-zh for voicemode TTS:
    git clone --depth 1 https://github.com/diyism/Kokoro-FastAPI-zh
    cd Kokoro-FastAPI-zh

    git lfs install
    cd api/src/models
    git clone https://huggingface.co/hexgrad/Kokoro-82M-v1.1-zh
    #if failed, "cd Kokoro-82M-v1.1-zh", "git lfs pull", "cd .."
    mv Kokoro-82M-v1.1-zh v1_1-zh
    cp -r v1_1-zh/voices ../voices/v1_1-zh
    cd ../../../
    
    ./start-cpu.sh       #if in conda, need first "conda deactivate", because this script will create .venv
    #maybe need to do: source .venv/bin/actiate; uv pip install "numpy==1.26.4"; deactivate; then ./start-cpu.sh again
    export VOICEMODE_TTS_BASE_URLS="http://127.0.0.1:8880/v1"
    export VOICEMODE_TTS_VOICES="zf_xiaobei"      #available: "zf_xiaobei", "zf_xiaoni", "zf_xiaoxiao", "zf_xiaoyi", "zm_yunjian", "zm_yunxi", "zm_yunxia", "zm_yunyang"
    #test port:
    curl http://127.0.0.1:8880/v1/audio/speech -F "a=b"
    #{"detail":[{"type":"model_attributes_type","loc":["body"],"msg":"Input should be a valid dictionary or object to extract fields from"...

    #3. install gemini-cli:
    #first install npm, ref: https://nodejs.org/en/download
    npm install -g @google/gemini-cli
    #get first project id from: https://console.cloud.google.com/
    #if you're not a google cloud user, skip this step
    #enable gemini api in: https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com?project=<my project id>
    export GOOGLE_CLOUD_PROJECT=<my project id>

    #4. install voice-mode mcp into gemini-cli:
    #if you installed the official version(mbailey/voicemode), first uninstall:
    pip uninstall voice-mode -y
    git clone https://github.com/diyism/gemini-cli-voice
    cd gemini-cli-voice
    pip install -e .
    voice-mode
    #if started successfully, ctrl+c to stop it
    nano ~/.gemini/settings.json
    #insert and save, now it likes:
    {
      "theme": "GitHub Light",
      "selectedAuthType": "oauth-enterprise",
      "mcpServers": {
        "voice-mode": {
          "command": "voice-mode"
        }
      }
    }

    #5. start gemini-cli with voicemode mcp:
    export VOICEMODE_STT_BASE_URLS="http://127.0.0.1:2022/v1"
    export VOICEMODE_TTS_BASE_URLS="http://127.0.0.1:8880/v1"
    export VOICEMODE_TTS_VOICES="zf_xiaobei"
    #export GOOGLE_CLOUD_PROJECT=<my project id>
    gemini
    #first time, click login google in the popped browser tab
    #type 'voice_mode_info' the check if all green
    #type 'converse "开始语音对话, 始终用汉语回答"' to start voice mode
    #select 'Yes, always allow all tools from server "voice-mode"'

![](./screenshot.png)

# Voice Mode

> **Install via:** `uvx voice-mode` | `pip install voice-mode` | [getvoicemode.com](https://getvoicemode.com)

[![PyPI Downloads](https://static.pepy.tech/badge/voice-mode)](https://pepy.tech/project/voice-mode)
[![PyPI Downloads](https://static.pepy.tech/badge/voice-mode/month)](https://pepy.tech/project/voice-mode)
[![PyPI Downloads](https://static.pepy.tech/badge/voice-mode/week)](https://pepy.tech/project/voice-mode)
[![Documentation](https://readthedocs.org/projects/voice-mode/badge/?version=latest)](https://voice-mode.readthedocs.io/en/latest/?badge=latest)

Natural voice conversations for AI assistants. Voice Mode brings human-like voice interactions to Claude, ChatGPT, and other LLMs through the Model Context Protocol (MCP).

## 🖥️ Compatibility

**Runs on:** Linux • macOS • Windows (WSL) | **Python:** 3.10+

## ✨ Features

- **🎙️ Voice conversations** with Claude - ask questions and hear responses
- **🔄 Multiple transports** - local microphone or LiveKit room-based communication  
- **🗣️ OpenAI-compatible** - works with any STT/TTS service (local or cloud)
- **⚡ Real-time** - low-latency voice interactions with automatic transport selection
- **🔧 MCP Integration** - seamless with Claude Desktop and other MCP clients
- **🎯 Silence detection** - automatically stops recording when you stop speaking (no more waiting!)

## 🎯 Simple Requirements

**All you need to get started:**

1. **🔑 OpenAI API Key** (or compatible service) - for speech-to-text and text-to-speech
2. **🎤 Computer with microphone and speakers** OR **☁️ LiveKit server** ([LiveKit Cloud](https://docs.livekit.io/home/cloud/) or [self-hosted](https://github.com/livekit/livekit))

## Quick Start

> 📖 **Using a different tool?** See our [Integration Guides](docs/integrations/README.md) for Cursor, VS Code, Gemini CLI, and more!

```bash
npm install -g @anthropic-ai/claude-code
curl -LsSf https://astral.sh/uv/install.sh | sh
claude mcp add --scope user voice-mode uvx voice-mode
export OPENAI_API_KEY=your-openai-key
claude converse
```

## 🎬 Demo

Watch Voice Mode in action with Claude Code:

[![Voice Mode Demo](https://img.youtube.com/vi/cYdwOD_-dQc/maxresdefault.jpg)](https://www.youtube.com/watch?v=cYdwOD_-dQc)

### Voice Mode with Gemini CLI

See Voice Mode working with Google's Gemini CLI (their implementation of Claude Code):

[![Voice Mode with Gemini CLI](https://img.youtube.com/vi/HC6BGxjCVnM/maxresdefault.jpg)](https://www.youtube.com/watch?v=HC6BGxjCVnM)

## Example Usage

Once configured, try these prompts with Claude:

### 👨‍💻 Programming & Development
- `"Let's debug this error together"` - Explain the issue verbally, paste code, and discuss solutions
- `"Walk me through this code"` - Have Claude explain complex code while you ask questions
- `"Let's brainstorm the architecture"` - Design systems through natural conversation
- `"Help me write tests for this function"` - Describe requirements and iterate verbally

### 💡 General Productivity  
- `"Let's do a daily standup"` - Practice presentations or organize your thoughts
- `"Interview me about [topic]"` - Prepare for interviews with back-and-forth Q&A
- `"Be my rubber duck"` - Explain problems out loud to find solutions

### 🎯 Voice Control Features
- `"Read this error message"` (Claude speaks, then waits for your response)
- `"Just give me a quick summary"` (Claude speaks without waiting)
- Use `converse("message", wait_for_response=False)` for one-way announcements

The `converse` function makes voice interactions natural - it automatically waits for your response by default, creating a real conversation flow.

## Supported Tools

Voice Mode works with your favorite AI coding assistants:

- 🤖 **[Claude Code](docs/integrations/claude-code/README.md)** - Anthropic's official CLI
- 🖥️ **[Claude Desktop](docs/integrations/claude-desktop/README.md)** - Desktop application
- 🌟 **[Gemini CLI](docs/integrations/gemini-cli/README.md)** - Google's CLI tool
- ⚡ **[Cursor](docs/integrations/cursor/README.md)** - AI-first code editor
- 💻 **[VS Code](docs/integrations/vscode/README.md)** - With MCP preview support
- 🦘 **[Roo Code](docs/integrations/roo-code/README.md)** - AI dev team in VS Code
- 🔧 **[Cline](docs/integrations/cline/README.md)** - Autonomous coding agent
- ⚡ **[Zed](docs/integrations/zed/README.md)** - High-performance editor
- 🏄 **[Windsurf](docs/integrations/windsurf/README.md)** - Agentic IDE by Codeium
- 🔄 **[Continue](docs/integrations/continue/README.md)** - Open-source AI assistant

## Installation

### Prerequisites
- Python >= 3.10
- [Astral UV](https://github.com/astral-sh/uv) - Package manager (install with `curl -LsSf https://astral.sh/uv/install.sh | sh`)
- OpenAI API Key (or compatible service)

#### System Dependencies

<details>
<summary><strong>Ubuntu/Debian</strong></summary>

```bash
sudo apt install python3-dev libasound2-dev libportaudio2 portaudio19-dev ffmpeg
```
</details>

<details>
<summary><strong>Fedora/RHEL</strong></summary>

```bash
sudo dnf install python3-devel alsa-lib-devel portaudio-devel ffmpeg
```
</details>

<details>
<summary><strong>macOS</strong></summary>

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install portaudio ffmpeg
```
</details>

<details>
<summary><strong>Windows (WSL)</strong></summary>

Follow the Ubuntu/Debian instructions above within WSL.
</details>

### Quick Install

```bash
# Using Claude Code (recommended)
claude mcp add --scope user voice-mode uvx voice-mode

# Using UV
uvx voice-mode

# Using pip
pip install voice-mode
```

### Configuration for AI Coding Assistants

> 📖 **Looking for detailed setup instructions?** Check our comprehensive [Integration Guides](docs/integrations/README.md) for step-by-step instructions for each tool!

Below are quick configuration snippets. For full installation and setup instructions, see the integration guides above.

<details>
<summary><strong>Claude Code (CLI)</strong></summary>

```bash
claude mcp add voice-mode -- uvx voice-mode
```

Or with environment variables:
```bash
claude mcp add voice-mode --env OPENAI_API_KEY=your-openai-key -- uvx voice-mode
```
</details>

<details>
<summary><strong>Claude Desktop</strong></summary>

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

<details>
<summary><strong>Cline</strong></summary>

Add to your Cline MCP settings:

**Windows**:
```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "cmd",
      "args": ["/c", "uvx", "voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```

**macOS/Linux**:
```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

<details>
<summary><strong>Continue</strong></summary>

Add to your `.continue/config.json`:
```json
{
  "experimental": {
    "modelContextProtocolServers": [
      {
        "transport": {
          "type": "stdio",
          "command": "uvx",
          "args": ["voice-mode"],
          "env": {
            "OPENAI_API_KEY": "your-openai-key"
          }
        }
      }
    ]
  }
}
```
</details>

<details>
<summary><strong>Cursor</strong></summary>

Add to `~/.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

<details>
<summary><strong>VS Code</strong></summary>

Add to your VS Code MCP config:
```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

<details>
<summary><strong>Windsurf</strong></summary>

```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

<details>
<summary><strong>Zed</strong></summary>

Add to your Zed settings.json:
```json
{
  "context_servers": {
    "voice-mode": {
      "command": {
        "path": "uvx",
        "args": ["voice-mode"],
        "env": {
          "OPENAI_API_KEY": "your-openai-key"
        }
      }
    }
  }
}
```
</details>

<details>
<summary><strong>Roo Code</strong></summary>

Add to your Roo Code MCP configuration:
```json
{
  "mcpServers": {
    "voice-mode": {
      "command": "uvx",
      "args": ["voice-mode"],
      "env": {
        "OPENAI_API_KEY": "your-openai-key"
      }
    }
  }
}
```
</details>

### Alternative Installation Options

<details>
<summary><strong>Using Docker</strong></summary>

```bash
docker run -it --rm \
  -e OPENAI_API_KEY=your-openai-key \
  --device /dev/snd \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  ghcr.io/mbailey/voicemode:latest
```
</details>

<details>
<summary><strong>Using pipx</strong></summary>

```bash
pipx install voice-mode
```
</details>

<details>
<summary><strong>From source</strong></summary>

```bash
git clone https://github.com/mbailey/voicemode.git
cd voicemode
pip install -e .
```
</details>

## Tools

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `converse` | Have a voice conversation - speak and optionally listen | `message`, `wait_for_response` (default: true), `listen_duration` (default: 30s), `transport` (auto/local/livekit) |
| `listen_for_speech` | Listen for speech and convert to text | `duration` (default: 5s) |
| `check_room_status` | Check LiveKit room status and participants | None |
| `check_audio_devices` | List available audio input/output devices | None |
| `start_kokoro` | Start the Kokoro TTS service | `models_dir` (optional, defaults to ~/Models/kokoro) |
| `stop_kokoro` | Stop the Kokoro TTS service | None |
| `kokoro_status` | Check the status of Kokoro TTS service | None |

**Note:** The `converse` tool is the primary interface for voice interactions, combining speaking and listening in a natural flow.

## Configuration

- 📖 **[Integration Guides](docs/integrations/README.md)** - Step-by-step setup for each tool
- 🔧 **[Configuration Reference](docs/configuration.md)** - All environment variables
- 📁 **[Config Examples](config-examples/)** - Ready-to-use configuration files

### Quick Setup

The only required configuration is your OpenAI API key:

```bash
export OPENAI_API_KEY="your-key"
```

### Optional Settings

```bash
# Custom STT/TTS services (OpenAI-compatible)
export STT_BASE_URL="http://127.0.0.1:2022/v1"  # Local Whisper
export TTS_BASE_URL="http://127.0.0.1:8880/v1"  # Local TTS
export TTS_VOICE="alloy"                        # Voice selection

# LiveKit (for room-based communication)
# See docs/livekit/ for setup guide
export LIVEKIT_URL="wss://your-app.livekit.cloud"
export LIVEKIT_API_KEY="your-api-key"
export LIVEKIT_API_SECRET="your-api-secret"

# Debug mode
export VOICEMODE_DEBUG="true"

# Save all audio (TTS output and STT input)
export VOICEMODE_SAVE_AUDIO="true"

# Audio format configuration (default: pcm)
export VOICEMODE_AUDIO_FORMAT="pcm"         # Options: pcm, mp3, wav, flac, aac, opus
export VOICEMODE_TTS_AUDIO_FORMAT="pcm"     # Override for TTS only (default: pcm)
export VOICEMODE_STT_AUDIO_FORMAT="mp3"     # Override for STT upload

# Format-specific quality settings
export VOICEMODE_OPUS_BITRATE="32000"       # Opus bitrate (default: 32kbps)
export VOICEMODE_MP3_BITRATE="64k"          # MP3 bitrate (default: 64k)
```

### Audio Format Configuration

Voice Mode uses **PCM** audio format by default for TTS streaming for optimal real-time performance:

- **PCM** (default for TTS): Zero latency, best streaming performance, uncompressed
- **MP3**: Wide compatibility, good compression for uploads
- **WAV**: Uncompressed, good for local processing
- **FLAC**: Lossless compression, good for archival
- **AAC**: Good compression, Apple ecosystem
- **Opus**: Small files but NOT recommended for streaming (quality issues)

The audio format is automatically validated against provider capabilities and will fallback to a supported format if needed.

## Local STT/TTS Services

For privacy-focused or offline usage, Voice Mode supports local speech services:

- **[Whisper.cpp](docs/whisper.cpp.md)** - Local speech-to-text with OpenAI-compatible API
- **[Kokoro](docs/kokoro.md)** - Local text-to-speech with multiple voice options

These services provide the same API interface as OpenAI, allowing seamless switching between cloud and local processing.

### OpenAI API Compatibility Benefits

By strictly adhering to OpenAI's API standard, Voice Mode enables powerful deployment flexibility:

- **🔀 Transparent Routing**: Users can implement their own API proxies or gateways outside of Voice Mode to route requests to different providers based on custom logic (cost, latency, availability, etc.)
- **🎯 Model Selection**: Deploy routing layers that select optimal models per request without modifying Voice Mode configuration
- **💰 Cost Optimization**: Build intelligent routers that balance between expensive cloud APIs and free local models
- **🔧 No Lock-in**: Switch providers by simply changing the `BASE_URL` - no code changes required

Example: Simply set `OPENAI_BASE_URL` to point to your custom router:
```bash
export OPENAI_BASE_URL="https://router.example.com/v1"
export OPENAI_API_KEY="your-key"
# Voice Mode now uses your router for all OpenAI API calls
```

The OpenAI SDK handles this automatically - no Voice Mode configuration needed!

## Architecture

```
┌─────────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│   Claude/LLM        │     │  LiveKit Server  │     │  Voice Frontend     │
│   (MCP Client)      │◄────►│  (Optional)     │◄───►│  (Optional)         │
└─────────────────────┘     └──────────────────┘     └─────────────────────┘
         │                            │
         │                            │
         ▼                            ▼
┌─────────────────────┐     ┌──────────────────┐
│  Voice MCP Server   │     │   Audio Services │
│  • converse         │     │  • OpenAI APIs   │
│  • listen_for_speech│◄───►│  • Local Whisper │
│  • check_room_status│     │  • Local TTS     │
│  • check_audio_devices    └──────────────────┘
└─────────────────────┘
```

## Troubleshooting

### Common Issues

- **No microphone access**: Check system permissions for terminal/application
  - **WSL2 Users**: See [WSL2 Microphone Access Guide](docs/troubleshooting/wsl2-microphone-access.md)
- **UV not found**: Install with `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **OpenAI API error**: Verify your `OPENAI_API_KEY` is set correctly
- **No audio output**: Check system audio settings and available devices

### Debug Mode

Enable detailed logging and audio file saving:

```bash
export VOICEMODE_DEBUG=true
```

Debug audio files are saved to: `~/voicemode_recordings/`

### Audio Diagnostics

Run the diagnostic script to check your audio setup:

```bash
python scripts/diagnose-wsl-audio.py
```

This will check for required packages, audio services, and provide specific recommendations.

### Audio Saving

To save all audio files (both TTS output and STT input):

```bash
export VOICEMODE_SAVE_AUDIO=true
```

Audio files are saved to: `~/voicemode_audio/` with timestamps in the filename.

## Documentation

📚 **[Read the full documentation at voice-mode.readthedocs.io](https://voice-mode.readthedocs.io)**

### Getting Started
- **[Integration Guides](docs/integrations/README.md)** - Step-by-step setup for all supported tools
- **[Configuration Guide](docs/configuration.md)** - Complete environment variable reference

### Development
- **[Using uv/uvx](docs/uv.md)** - Package management with uv and uvx
- **[Local Development](docs/local-development-uvx.md)** - Development setup guide
- **[Audio Formats](docs/audio-format-migration.md)** - Audio format configuration and migration
- **[Statistics Dashboard](docs/statistics-dashboard.md)** - Performance monitoring and metrics

### Service Guides
- **[Whisper.cpp Setup](docs/whisper.cpp.md)** - Local speech-to-text configuration
- **[Kokoro Setup](docs/kokoro.md)** - Local text-to-speech configuration
- **[LiveKit Integration](docs/livekit/README.md)** - Real-time voice communication

### Troubleshooting
- **[WSL2 Microphone Access](docs/troubleshooting/wsl2-microphone-access.md)** - WSL2 audio setup
- **[Migration Guide](docs/migration-guide.md)** - Upgrading from older versions

## Links

- **Website**: [getvoicemode.com](https://getvoicemode.com)
- **Documentation**: [voice-mode.readthedocs.io](https://voice-mode.readthedocs.io)
- **GitHub**: [github.com/mbailey/voicemode](https://github.com/mbailey/voicemode)
- **PyPI**: [pypi.org/project/voice-mode](https://pypi.org/project/voice-mode/)
- **npm**: [npmjs.com/package/voicemode](https://www.npmjs.com/package/voicemode)

### Community

- **Discord**: [Join our community](https://discord.gg/Hm7dF3uCfG)
- **Twitter/X**: [@getvoicemode](https://twitter.com/getvoicemode)
- **YouTube**: [@getvoicemode](https://youtube.com/@getvoicemode)

## See Also

- 🚀 [Integration Guides](docs/integrations/README.md) - Setup instructions for all supported tools
- 🔧 [Configuration Reference](docs/configuration.md) - Environment variables and options
- 🎤 [Local Services Setup](docs/kokoro.md) - Run TTS/STT locally for privacy
- 🐛 [Troubleshooting](docs/troubleshooting/README.md) - Common issues and solutions

## License

MIT - A [Failmode](https://failmode.com) Project

---

<sub>[Project Statistics](docs/project-stats/README.md)</sub>
