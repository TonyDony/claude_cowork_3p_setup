# Claude Desktop 第三方模型部署指南

> 在 macOS 上配置 Claude Desktop (3p 模式) 通过 claude-remap 代理调用第三方大模型（mimo / GLM / DeepSeek 等）

**在线预览**: https://tonydony.github.io/claude_cowork_3p_setup/

---

## 工作原理

```
Claude Desktop → claude-remap (本地代理 :9960) → 第三方模型 API
```

Claude Desktop 发送请求时使用 Anthropic 模型名（如 `claude-sonnet-4-6`），本地代理在转发前将模型名替换为目标模型名（如 `mimo-v2.5-pro`、`glm-4-plus`），API 响应再原路返回。

---

## 前置条件

- macOS (Intel 或 Apple Silicon)
- Node.js 18+ 已安装（`node -v` 验证）
- Claude Desktop 已安装（3p 版本，即第三方部署模式）
- 已获取第三方模型的 API Key

---

## 启用开发者工具（可选）

方便通过 `Cmd+Option+I` 调试网络请求。

文件路径：`~/Library/Application Support/Claude-3p/developer_settings.json`

```json
{
  "allowDevTools": true
}
```

---

## 安装 claude-remap 代理

### 3.1 确认 Node.js 环境

```bash
# 检查 Node.js 版本（需要 18+）
node -v

# 检查 npm 是否可用
npm -v
```

> **注意**: 如果未安装 Node.js，前往 https://nodejs.org 下载 LTS 版本，或使用 Homebrew：`brew install node`

### 3.2 安装 claude-remap

```bash
# 方式一：全局安装（推荐，后续可直接运行）
npm install -g claude-remap@1.12.0

# 验证安装
claude-remap --version

# 方式二：免安装，通过 npx 直接运行（每次会检查更新）
npx claude-remap@1.12.0 start
```

| 方式 | 优点 | 缺点 |
|------|------|------|
| 全局安装 | 启动快，LaunchAgent 配置简单 | 需手动更新版本 |
| npx 运行 | 无需预装，自动使用最新版 | 首次启动稍慢 |

---

## 配置 claude-remap 代理

### 4.1 创建配置文件

文件路径：`~/.claude-remap.json`

切换供应商时只需修改此文件中的 `target`、`key`、`models` 三个字段，然后重启代理即可。

| 字段 | 说明 |
|------|------|
| `target` | 供应商的 Anthropic 兼容 API 端点 |
| `key` | 该供应商的 API Key |
| `port` | 本地代理监听端口（固定 9960） |
| `models` | Claude 模型名 → 供应商模型名 的映射表 |

> **警告**: `models` 字段名不能写错为 `maps`，否则代理启动时会报 `TypeError: Cannot convert undefined or null to object`。

### 4.2 供应商配置示例

#### 小米 mimo

- **供应商**: 小米 mimo
- **API 格式**: Anthropic 兼容
- **获取 Key**: https://xiaomimimo.com

```json
{
  "target": "https://token-plan-cn.xiaomimimo.com/anthropic",
  "key": "tp-xxxxxxxx",
  "port": 9960,
  "models": {
    "claude-sonnet-4-6-20250514": "mimo-v2.5-pro",
    "claude-sonnet-4-6": "mimo-v2.5-pro",
    "claude-opus-4-6-20250514": "mimo-v2.5-pro",
    "claude-opus-4-6": "mimo-v2.5-pro",
    "claude-haiku-4-5-20251001": "mimo-v2.5",
    "claude-haiku-4-5": "mimo-v2.5"
  }
}
```

| Claude 模型 (显示名) | mimo 模型 (实际调用) |
|----------------------|---------------------|
| claude-opus-4-6 | mimo-v2.5-pro |
| claude-sonnet-4-6 | mimo-v2.5-pro |
| claude-haiku-4-5 | mimo-v2.5 |

#### 智谱 GLM

- **供应商**: 智谱 AI (GLM)
- **API 格式**: 需确认是否提供 Anthropic 兼容端点
- **获取 Key**: https://open.bigmodel.cn

```json
{
  "target": "GLM 的 Anthropic 兼容端点地址",
  "key": "你的GLM密钥",
  "port": 9960,
  "models": {
    "claude-sonnet-4-6-20250514": "glm-4-plus",
    "claude-sonnet-4-6": "glm-4-plus",
    "claude-opus-4-6-20250514": "glm-4-plus",
    "claude-opus-4-6": "glm-4-plus",
    "claude-haiku-4-5-20251001": "glm-4-flash",
    "claude-haiku-4-5": "glm-4-flash"
  }
}
```

| Claude 模型 (显示名) | GLM 模型 (实际调用) | 说明 |
|----------------------|---------------------|------|
| claude-opus-4-6 | glm-4-plus | 高性能模型 |
| claude-sonnet-4-6 | glm-4-plus | 高性能模型 |
| claude-haiku-4-5 | glm-4-flash | 快速模型 |

> **注意**: 智谱 GLM 原生使用 OpenAI 格式（`/v1/chat/completions`），需确认是否提供 Anthropic 兼容端点。如果仅有 OpenAI 格式，需要在 claude-remap 和 GLM 之间增加一层格式转换（如使用 `openai-to-anthropic` 适配器），或使用支持双格式转换的代理工具。

#### DeepSeek

- **供应商**: DeepSeek
- **API 格式**: 需确认是否提供 Anthropic 兼容端点
- **获取 Key**: https://platform.deepseek.com

```json
{
  "target": "DeepSeek 的 Anthropic 兼容端点地址",
  "key": "你的DeepSeek密钥",
  "port": 9960,
  "models": {
    "claude-sonnet-4-6-20250514": "deepseek-chat",
    "claude-sonnet-4-6": "deepseek-chat",
    "claude-opus-4-6-20250514": "deepseek-chat",
    "claude-opus-4-6": "deepseek-chat",
    "claude-haiku-4-5-20251001": "deepseek-chat",
    "claude-haiku-4-5": "deepseek-chat"
  }
}
```

| Claude 模型 (显示名) | DeepSeek 模型 (实际调用) | 说明 |
|----------------------|-------------------------|------|
| claude-opus-4-6 | deepseek-chat | DeepSeek-V3 |
| claude-sonnet-4-6 | deepseek-chat | DeepSeek-V3 |
| claude-haiku-4-5 | deepseek-chat | DeepSeek-V3 |

> **注意**: DeepSeek 原生使用 OpenAI 格式，与 GLM 相同，需确认是否提供 Anthropic 兼容端点或增加格式转换层。

### 4.3 切换供应商的操作步骤

```bash
# 1. 编辑代理配置，修改 target / key / models
nano ~/.claude-remap.json

# 2. 重启 claude-remap 代理（以使新配置生效）
kill $(lsof -t -i:9960) 2>/dev/null
npx claude-remap@1.12.0 start &

# 3. 无需修改 Claude Desktop 的网关配置，因为代理地址不变
```

> **提示**: 切换供应商只需改 `~/.claude-remap.json` 并重启代理，Claude Desktop 的网关配置（Step 6）不需要任何改动。

### 4.4 启动代理

```bash
# 前台启动（调试用，可看到请求日志）
npx claude-remap@1.12.0 start

# 后台启动
nohup npx claude-remap@1.12.0 start > ~/.claude-remap.log 2>&1 &
```

> **提示**: 首次运行 `npx` 会自动下载 claude-remap，后续直接使用缓存。验证代理是否运行：`curl http://localhost:9960`

---

## 配置 Claude Desktop 网关

在 Claude Desktop 的设置界面中选择 **"自定义 API 端点"**（Gateway 模式），填入本地代理地址和 API Key。配置会自动保存到以下文件。

文件路径：`~/Library/Application Support/Claude-3p/configLibrary/<你的用户ID>.json`

> **注意**: 文件名中的 `<你的用户ID>` 是一串 UUID，在 `configLibrary/` 目录下查看：`ls ~/Library/Application\ Support/Claude-3p/configLibrary/`

```json
{
  "disableDeploymentModeChooser": true,
  "inferenceProvider": "gateway",
  "inferenceGatewayBaseUrl": "http://localhost:9960",
  "inferenceGatewayApiKey": "你的API密钥",
  "inferenceGatewayAuthScheme": "x-api-key",
  "inferenceModels": [
    { "name": "claude-opus-4-6", "supports1m": true },
    { "name": "claude-sonnet-4-6", "supports1m": true },
    { "name": "claude-haiku-4-5" }
  ]
}
```

| 字段 | 说明 |
|------|------|
| `inferenceProvider` | 固定为 `"gateway"`，启用自定义 API 模式 |
| `inferenceGatewayBaseUrl` | 本地代理地址 |
| `inferenceGatewayApiKey` | API Key（转发时由代理替换） |
| `inferenceGatewayAuthScheme` | 认证方式，固定 `"x-api-key"` |
| `inferenceModels` | 下拉框显示的模型列表。对象格式，`supports1m: true` 启用 100 万 token 上下文窗口 |
| `disableDeploymentModeChooser` | 禁用部署模式切换，防止误操作 |

> **提示**: `supports1m: true` 为模型启用扩展上下文窗口（100 万 token）。不加此字段则使用默认上下文长度。部分版本也支持字符串简写格式 `"claude-opus-4-6[1m]"`。

---

## 重启并验证

```bash
# 1. 确保 claude-remap 代理正在运行
curl http://localhost:9960

# 2. 关闭 Claude Desktop
killall Claude

# 3. 重新打开 Claude Desktop
open -a "Claude"
```

### 验证清单

- [ ] Claude Desktop 正常启动，无 CPU 卡死
- [ ] 左下角模型下拉框显示 3 个模型选项
- [ ] 选择模型后发送消息，能正常收到回复
- [ ] 开发者工具 Network 面板中，请求发往 `localhost:9960`

---

## 设置开机自启（推荐）

使用 macOS 的 LaunchAgent 实现 claude-remap 开机自动启动。

文件路径：`~/Library/LaunchAgents/com.claude-remap.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude-remap</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/npx</string>
        <string>claude-remap@1.12.0</string>
        <string>start</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/{computer_name}/.claude-remap.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/{computer_name}/.claude-remap-error.log</string>
</dict>
</plist>
```

```bash
# 加载 LaunchAgent（立即生效 + 开机自启）
launchctl load ~/Library/LaunchAgents/com.claude-remap.plist

# 卸载（取消自启）
launchctl unload ~/Library/LaunchAgents/com.claude-remap.plist
```

> **注意**: 请将 plist 中的路径替换为你系统上的实际路径：
> - npx 路径：`which npx` 查看
> - 用户目录：将 `/Users/{computer_name}` 替换为你的用户目录
> - 如果全局安装了 claude-remap，可直接使用 `claude-remap` 命令路径（`which claude-remap` 查看）

---

## 常见问题

### Q: 模型下拉框只显示 Claude 模型名，能显示 mimo 模型名吗？

Claude Desktop v1.6259.1 及以上版本强制验证模型名必须为 Anthropic 官方模型名。因此下拉框显示的是 Claude 模型名（如 "Claude Sonnet 4.6"），但实际请求会通过代理自动转换为 mimo 模型。

如果需要下拉框直接显示自定义模型名，需要降级到 v1.5354.0 以下版本（但该版本功能较少且官方已停止分发）。

### Q: 发送消息后报 "Not supported model"

说明请求未经过 claude-remap 代理。检查：
1. 代理是否运行：`curl http://localhost:9960`
2. 网关配置中 `inferenceGatewayBaseUrl` 是否为 `http://localhost:9960`
3. `~/.claude-remap.json` 中模型映射是否正确

### Q: claude-remap 启动报 TypeError

确认 `~/.claude-remap.json` 中模型映射字段名为 `"models"`（不是 `"maps"`），且 JSON 格式合法。

### Q: 重启电脑后 Claude Desktop 无法使用

claude-remap 代理进程不会随系统重启自动启动。按 **Step 6** 配置 LaunchAgent 实现开机自启，或手动运行 `npx claude-remap@1.12.0 start`。

---

## 文件路径汇总

| 文件 | 路径 |
|------|------|
| 应用主配置 | `~/Library/Application Support/Claude-3p/claude_desktop_config.json` |
| 开发者设置 | `~/Library/Application Support/Claude-3p/developer_settings.json` |
| 网关配置 | `~/Library/Application Support/Claude-3p/configLibrary/<用户ID>.json` |
| 代理配置 | `~/.claude-remap.json` |
| 自启服务 | `~/Library/LaunchAgents/com.claude-remap.plist` |

---

## License

MIT
