## 🤖 Claude Code 配置 (Windows)

### 1. 安装 Node.js

从官网下载安装：https://nodejs.org/（选择 LTS 版本）

```
# 或使用 winget 安装
winget install OpenJS.NodeJS.LTS

# 验证安装 (PowerShell)
node -v; npm -v
```

### 2. 安装 Claude Code

```
npm install -g @anthropic-ai/claude-code
```

### 3. 更新 Claude Code

```
# 更新到最新版本
npm update -g @anthropic-ai/claude-code

# 或重新安装最新版
npm install -g @anthropic-ai/claude-code@latest


# 查看当前版本
claude --version
```

### 4. 配置 API

⚠️ 重要：先设置环境变量

新版 Claude Code 客户端会先检测环境变量，如果没有设置环境变量则不会读取配置文件。请先设置以下环境变量：

```
# PowerShell 临时设置（当前会话有效）
$env:ANTHROPIC_BASE_URL = "https://api.lclaitech.com"
$env:ANTHROPIC_AUTH_TOKEN = "sk-你的API密钥"
$env:API_TIMEOUT_MS = "3000000"
$env:CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC = "1"

# PowerShell 永久设置（用户级别）
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://api.lclaitech.com", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-你的API密钥", "User")
[Environment]::SetEnvironmentVariable("API_TIMEOUT_MS", "3000000", "User")
[Environment]::SetEnvironmentVariable("CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC", "1", "User")


# 或通过 CMD 设置
setx ANTHROPIC_BASE_URL "https://api.lclaitech.com"
setx ANTHROPIC_AUTH_TOKEN "sk-你的API密钥"
setx API_TIMEOUT_MS "3000000"
setx CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC "1"


# 设置后需要重启终端或 IDE 才能生效
```

**环境变量说明：**

- `ANTHROPIC_BASE_URL` - API 服务地址
- `ANTHROPIC_AUTH_TOKEN` - 您的 API 密钥
- `API_TIMEOUT_MS` - 请求超时时间（毫秒），建议设置较大值避免长任务超时
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` - 禁用非必要网络请求（遥测等）

**配置文件位置：**`%USERPROFILE%\.claude\settings.json`

📄 配置文件内容

```
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-你的API密钥",
    "ANTHROPIC_BASE_URL": "https://api.lclaitech.com",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },
  "model": "opus"
}
```

