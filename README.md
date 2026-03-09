# dictator-tts

> Global voice input (STT) and text-to-speech (TTS) for macOS, powered by Hammerspoon, Groq Whisper, and edge-tts.

Press a hotkey → speak → text appears in any app. Press another hotkey → selected text is read aloud. Completely free APIs, no subscription needed.

---

## Features

- **Speech-to-Text** — Press `Cmd+/` to start recording, press again to stop. Transcribed text is automatically pasted into the active application.
- **Text-to-Speech** — Press `Cmd+.` to read the selected text (or clipboard) aloud. Press again to stop.
- **Mixed language** — Handles Chinese and English in the same sentence.
- **AI correction** (optional) — Passes the transcript through an LLM to fix punctuation and dictation errors before pasting.
- **Multiple API providers** — Works with Groq (recommended, free), OpenAI, DeepInfra, Together AI, Fireworks AI, and Cloudflare Workers AI.
- **Rate limiting & watchdog** — Built-in per-minute rate limiter and a 90-second timeout to prevent stuck states.

---

## How It Works

```
Press hotkey
     │
     ▼
SoX records microphone → temp .wav file
     │
     ▼
Groq Whisper API (cloud, free)
     │
     ▼
(optional) LLM correction pass
     │
     ▼
Paste into active app via Cmd+V
```

All processing happens in Hammerspoon (Lua). Only the Whisper API call goes to the network — everything else is local.

---

## Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| macOS | 12+ | Hammerspoon uses macOS accessibility APIs |
| [Hammerspoon](https://www.hammerspoon.org/) | latest | macOS automation framework |
| [SoX](https://sox.sourceforge.net/) | any | command-line audio recorder |
| [edge-tts](https://github.com/rany2/edge-tts) | any | Microsoft Neural TTS (Python) |
| Python 3 | 3.8+ | required by edge-tts |
| [Groq API key](https://console.groq.com) | — | free account, no credit card needed |

---

## Installation

### Step 1 — Install Hammerspoon

```bash
brew install --cask hammerspoon
```

Open Hammerspoon from Applications. Grant **Accessibility** permission when prompted (System Settings → Privacy & Security → Accessibility).

### Step 2 — Install SoX

```bash
brew install sox
```

Verify it works:

```bash
sox --version
```

### Step 3 — Install edge-tts

```bash
pip3 install edge-tts
```

Verify it works:

```bash
edge-tts --text "hello" --write-media /tmp/test.mp3
```

### Step 4 — Install dictator-tts

Copy the Lua files into Hammerspoon's config directory:

```bash
git clone https://github.com/harvenstar/dictator-tts.git
cp dictator-tts/*.lua ~/.hammerspoon/
```

Or, if you manage your Hammerspoon config as a module, create a subdirectory:

```bash
mkdir -p ~/.hammerspoon/dictator-tts
cp dictator-tts/*.lua ~/.hammerspoon/dictator-tts/
```

Then add to your `~/.hammerspoon/init.lua`:

```lua
require("dictator-tts.init")
```

### Step 5 — Configure your Groq API key

1. Sign up at [console.groq.com](https://console.groq.com) (free, no credit card).
2. Go to [console.groq.com/keys](https://console.groq.com/keys) and create an API key.
3. Run the following in Terminal (replace `YOUR_KEY_HERE`):

```bash
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.apiKey" "YOUR_KEY_HERE"
```

### Step 6 — Reload Hammerspoon

Click the Hammerspoon menubar icon → **Reload Config**, or press `Cmd+Alt+R` inside Hammerspoon's console.

---

## Configuration

All settings are stored in macOS user defaults under the `org.hammerspoon.Hammerspoon` domain. Use `defaults write` to set them and `defaults read` to verify.

### Transcription (required)

```bash
# API key (Groq or any OpenAI-compatible provider)
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.apiKey" "YOUR_KEY"

# API base URL (default: Groq)
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.transcriptionApiBaseUrl" "https://api.groq.com/openai/v1"

# Whisper model
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.transcriptionModel" "whisper-large-v3-turbo"

# Language hint — "auto" detects Chinese/English automatically
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.language" "auto"
```

### AI correction (optional)

When enabled, the raw transcript is sent to an LLM to fix punctuation, common dictation errors, and filler words.

```bash
# Base URL for the correction model
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.correctionApiBaseUrl" "https://api.groq.com/openai/v1"

# Correction model (LLaMA 3.3 70B works well on Groq free tier)
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.correctionModel" "llama-3.3-70b-versatile"
```

Leave both fields empty (or unset) to skip AI correction entirely.

### Verify your config

```bash
defaults read org.hammerspoon.Hammerspoon | grep "com.simon.dictator"
```

---

## Hotkeys

| Hotkey | Action |
|--------|--------|
| `Cmd+/` | Start recording (press once). Press again to stop and transcribe. |
| `Cmd+.` | Read selected text aloud (or clipboard if nothing selected). Press again to stop. |

---

## Free API Quotas (Groq)

| Model | Daily limit | Used for |
|-------|-------------|----------|
| `whisper-large-v3-turbo` | 2,000 requests/day | Speech-to-text |
| `llama-3.3-70b-versatile` | 1,000 requests/day | Optional AI correction |

Groq is free with no credit card required. Limits reset daily at midnight UTC.

---

## Alternative API Providers

dictator-tts uses the OpenAI-compatible API format. You can point it at any provider:

| Provider | Base URL | Whisper model |
|----------|----------|---------------|
| Groq (recommended) | `https://api.groq.com/openai/v1` | `whisper-large-v3-turbo` |
| OpenAI | `https://api.openai.com/v1` | `whisper-1` |
| DeepInfra | `https://api.deepinfra.com/v1/openai` | `openai/whisper-large-v3` |
| Together AI | `https://api.together.xyz/v1` | check their model list |
| Fireworks AI | `https://api.fireworks.ai/inference/v1` | check their model list |

---

## Project Structure

```
dictator-tts/
├── init.lua           # Entry point: hotkey bindings, recording lifecycle, TTS
├── api.lua            # Whisper transcription + AI correction requests
├── api_client.lua     # HTTP client wrapper
├── api_filters.lua    # API response post-processing
├── audio.lua          # SoX recording management (start, stop, temp files)
├── config.lua         # Config read/write via hs.settings
├── config_defaults.lua # Default values and the AI correction system prompt
├── ui.lua             # Menubar icon and status display
├── utils.lua          # Helpers: context capture, text validation
└── rate_limiter.lua   # Per-minute API rate limiter
```

---

## Resource Usage

| Resource | Usage |
|----------|-------|
| Memory | ~64 MB (Hammerspoon process, always-on) |
| Disk | ~41 MB (Hammerspoon + Lua scripts) |
| Network | Only on transcription: one API call per recording |
| CPU | Idle between keypresses |

---

## Known Issues

- **TTS doesn't capture selection in some apps** — `Cmd+.` tries to read the selected text via the macOS Accessibility API (`AXSelectedText`). Some apps (e.g. Electron apps, games) don't expose this. Workaround: manually press `Cmd+C` to copy first, then press `Cmd+.` to read from clipboard.
- **edge-tts voice sounds synthetic** — Microsoft Neural TTS is free but not the most natural. Alternatives being explored: [Qwen3-TTS](https://github.com/QwenLM/Qwen-Audio) (local, voice cloning) or [Fish Audio](https://fish.audio/) API.

---

## Credits

- [Dictator](https://github.com/Glossardi/Dictator-Speech-to-Text) — the original Hammerspoon speech-to-text plugin this project is based on
- [Hammerspoon](https://www.hammerspoon.org/) — macOS automation framework
- [Groq](https://groq.com/) — free cloud Whisper API
- [edge-tts](https://github.com/rany2/edge-tts) — Microsoft Neural TTS Python client

---

## License

[MIT](./LICENSE)
