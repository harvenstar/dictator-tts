# Dictator TTS

macOS 全局语音输入 + TTS 朗读系统，基于 Hammerspoon。

## 功能

### 语音输入（Speech-to-Text）
- **快捷键**: `Cmd+/`（按一次开始录音，再按一次停止并转写）
- 支持中英文混合语音输入
- 转写结果自动复制到剪贴板并粘贴到当前应用
- 支持 AI 文本校正（可选）

### TTS 朗读（Text-to-Speech）
- **快捷键**: `Cmd+.`（朗读选中文字/剪贴板内容，再按一次停止）
- 使用 edge-tts（微软 Neural TTS），声音：zh-CN-XiaoxiaoNeural
- 支持中英文混合朗读

## 架构

```
用户按键 → Hammerspoon（热键监听 + 自动粘贴）
              ↓
         SoX（本地录音）
              ↓
         Groq Whisper API（语音转文字，云端）
              ↓
         剪贴板 + Cmd+V 自动粘贴到当前应用
```

- **Hammerspoon**: macOS 自动化框架（Lua），负责热键监听、菜单栏UI、自动粘贴
- **SoX**: 命令行录音工具，负责麦克风采集
- **Groq API**: 免费云端 API，运行 OpenAI Whisper Large V3 Turbo 模型
- **edge-tts**: 微软 Edge 浏览器的神经网络 TTS 引擎，免费

## 安装依赖

```bash
# Hammerspoon（macOS 自动化框架）
brew install --cask hammerspoon

# SoX（录音工具）
brew install sox

# edge-tts（TTS 引擎）
pip3 install edge-tts
```

## 配置 Groq API（免费）

1. 注册 Groq 账号: https://console.groq.com
2. 创建 API Key: https://console.groq.com/keys
3. 通过命令行写入配置:

```bash
# API Key
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.apiKey" "你的Groq_API_Key"

# 转写设置（Whisper）
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.transcriptionApiBaseUrl" "https://api.groq.com/openai/v1"
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.transcriptionModel" "whisper-large-v3-turbo"

# AI校正设置（可选）
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.correctionApiBaseUrl" "https://api.groq.com/openai/v1"
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.correctionModel" "llama-3.3-70b-versatile"

# 语言（auto = 自动检测中英文）
defaults write org.hammerspoon.Hammerspoon "com.simon.dictator.language" "auto"
```

4. 重新加载 Hammerspoon 配置

### Groq 免费额度

| 模型 | 每日限制 | 用途 |
|------|---------|------|
| Whisper Large V3 Turbo | 2,000 次/天 | 语音转文字 |
| LLaMA 3.3 70B | 1,000 次/天 | AI 文本校正 |

## 文件说明

| 文件 | 说明 |
|------|------|
| `init.lua` | 主入口：热键绑定、录音控制、TTS |
| `api.lua` | Whisper 转写 + AI 校正 API 调用 |
| `api_client.lua` | HTTP 客户端封装 |
| `api_filters.lua` | API 响应过滤 |
| `audio.lua` | SoX 录音管理 |
| `config.lua` | 配置管理（hs.settings） |
| `config_defaults.lua` | 默认配置值 |
| `ui.lua` | 菜单栏 UI |
| `utils.lua` | 工具函数（上下文捕获、验证等） |
| `rate_limiter.lua` | API 调用速率限制 |

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd+/` | 开始/停止录音（toggle 模式） |
| `Cmd+.` | 朗读选中文字（再按停止） |

## 已知问题

- [ ] **TTS 自动复制不生效**: `Cmd+.` 尝试通过无障碍 API（AXSelectedText）直接读取选中文字，但部分应用不支持。当前 workaround：先手动 `Cmd+C` 复制，再 `Cmd+.` 朗读剪贴板内容。
- [ ] **edge-tts 声音偏 AI 感**: 微软 Neural TTS 声音自然度有限，后续可考虑 Qwen3-TTS（本地部署，支持声音克隆）或 Fish Audio API。

## 其他 AI 提供商

除了 Groq，也支持:
- OpenAI（`https://api.openai.com/v1`，模型 `whisper-1`）
- DeepInfra（`https://api.deepinfra.com/v1/openai`）
- Together AI（`https://api.together.xyz/v1`）
- Fireworks AI（`https://api.fireworks.ai/inference/v1`）
- Cloudflare Workers AI

## 资源占用

- 内存: ~64MB（Hammerspoon 进程）
- 磁盘: ~41MB（Lua 脚本 + Hammerspoon）
- 网络: 仅录音结束时调用 Groq API

## 基于

- [Dictator](https://github.com/Glossardi/Dictator-Speech-to-Text) - 原始 Hammerspoon 语音输入插件
- [Hammerspoon](https://www.hammerspoon.org/) - macOS 自动化框架
- [Groq](https://groq.com/) - 免费 Whisper API
- [edge-tts](https://github.com/rany2/edge-tts) - 微软 Neural TTS
