# OpenClaw Configuration Audit

Complete reference of every configuration surface, what's on/off by default, and where each setting lives.

**Config file location:** `~/.openclaw/openclaw.json` (JSON5 format)

**Environment variable precedence** (highest wins):
1. Process environment (`export VAR=...`)
2. `./.env` (repo root)
3. `~/.openclaw/.env` (user home)
4. `openclaw.json` `env` block
5. Direct config keys override env fallbacks

---

## Table of Contents

1. [Quick Reference: Enabled by Default](#enabled-by-default)
2. [Quick Reference: Disabled by Default](#disabled-by-default)
3. [Security-Critical Settings](#security-critical-settings)
4. [Environment Variables](#environment-variables)
5. [Gateway Config](#gateway-config-gateway)
6. [Agent Config](#agent-config-agents)
7. [Channel Config](#channel-config-channels)
8. [Model Config](#model-config-models)
9. [Skills Config](#skills-config-skills)
10. [Hooks Config](#hooks-config-hooks)
11. [Session Config](#session-config-session)
12. [Security Config](#security-config)
13. [Build & Dev Config](#build--dev-config)
14. [Deployment Config](#deployment-config)

---

## Enabled by Default

These features are **on** unless you explicitly disable them:

| Feature | Setting | Default |
|---------|---------|---------|
| Control UI (web dashboard) | `gateway.controlUi.enabled` | `true` |
| Canvas host | `gateway.canvasHost.enabled` | `true` (skip via `OPENCLAW_SKIP_CANVAS_HOST=1`) |
| mDNS discovery | `gateway.discovery.mdns` | `"minimal"` |
| TLS auto-generate | `gateway.tls.autoGenerate` | `true` |
| Auth rate limiting | `gateway.auth.rateLimit.*` | 10 attempts / 60s window / 5min lockout |
| Loopback exemption from rate limit | `gateway.auth.rateLimit.exemptLoopback` | `true` |
| Agent max concurrent (main) | `agents.defaults.maxConcurrent` | `4` |
| Sub-agent max concurrent | Sub-agent limit | `8` |
| Session scope | `session.scope` | `"per-sender"` |
| Typing mode | `session.typingMode` | `"message"` |
| Reply mode | `session.replyMode` | `"text"` |
| File/image URL input | `gateway.fileInput.allowUrl` | `true` |
| Talk interrupt on speech | `gateway.talk.interruptOnSpeech` | `true` |
| Discord/Telegram native commands | `commands.native` | Auto-enabled on Discord, Telegram |
| Bundled skills | `skills.allowBundled` | Enabled (all bundled skills) |
| Config hot reload | `gateway.reload.mode` | `"hybrid"` |

## Disabled by Default

These features are **off** unless you explicitly enable them:

| Feature | Setting | Default |
|---------|---------|---------|
| OpenAI Chat Completions endpoint | `gateway.chatCompletions.enabled` | `false` |
| OpenAI Responses endpoint | `gateway.openResponses.enabled` | `false` |
| Remote gateway | `gateway.remote.*` | Not configured |
| Tailscale integration | `gateway.tailscale.mode` | `"off"` |
| TLS (manual) | `gateway.tls.enabled` | `false` (unless configured) |
| Sandbox mode | `agents.defaults.sandbox.mode` | `"off"` |
| Human delay simulation | `agents.defaults.humanDelay.mode` | `"off"` |
| Hooks / triggers | `hooks.*` | Not configured |
| Slack native commands | `commands.native` (Slack) | Auto-disabled |
| Bedrock model discovery | `models.bedrock.discovery` | Not configured |

---

## Security-Critical Settings

> **Read before exposing anything.** These settings directly affect who can access your system.

| Setting | Risk | Recommendation |
|---------|------|----------------|
| `gateway.bind` | Controls network exposure. Default `"loopback"` only listens on 127.0.0.1. Setting to `"lan"` exposes to all interfaces. | Keep `"loopback"` unless you need LAN access. Requires auth when set to `"lan"`. |
| `gateway.auth.mode` | `"token"` (shared secret), `"password"`, or `"trusted-proxy"`. | Always set a token or password for non-loopback binds. |
| `OPENCLAW_GATEWAY_TOKEN` | Shared secret for HMAC auth. | Generate with `openssl rand -hex 32`. Never commit to git. |
| `channels.*.dmPolicy` | `"pairing"` (default) requires approval code. `"open"` allows anyone. | Keep `"pairing"`. Only use `"open"` with `allowFrom` allowlists. |
| `channels.*.allowFrom` | Allowlist of phone numbers / user IDs. `"*"` allows everyone. | Never set `"*"` on public-facing channels without `dmPolicy="pairing"`. |
| `agents.defaults.sandbox.mode` | `"off"` runs tools on host. `"non-main"` sandboxes group sessions. `"all"` sandboxes everything. | Use `"non-main"` for multi-user or group scenarios. |
| `gateway.controlUi.allowInsecureAuth` | Allows unencrypted auth over HTTP. | Leave `false`. Only for local dev. |
| `gateway.controlUi.dangerouslyDisableDeviceAuth` | Disables device authentication. | Never enable in production. |

---

## Environment Variables

Defined in `.env.example`. Set in process env, `.env` file, or `openclaw.json` `env` block.

### Core

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENCLAW_GATEWAY_TOKEN` | *(none)* | Shared secret for gateway HMAC authentication. Generate with `openssl rand -hex 32`. |
| `OPENCLAW_GATEWAY_PASSWORD` | *(none)* | Alternative: password-based auth instead of token. |
| `OPENCLAW_STATE_DIR` | `~/.openclaw` | Config directory (openclaw.json, credentials, sessions). |
| `OPENCLAW_CONFIG_PATH` | *(auto)* | Explicit path to openclaw.json. |
| `OPENCLAW_HOME` | *(auto)* | Home directory override. |
| `OPENCLAW_LOAD_SHELL_ENV` | `0` | Set to `1` to import missing env keys from login shell profile. |
| `OPENCLAW_SHELL_ENV_TIMEOUT_MS` | `15000` | Timeout for shell env loading. |
| `OPENCLAW_SKIP_CANVAS_HOST` | `0` | Set to `1` to skip starting the canvas host server. |
| `OPENCLAW_PREFER_PNPM` | `0` | Set to `1` to prefer pnpm over bun for scripts. |
| `OPENCLAW_E2E_WORKERS` | *(auto)* | Override e2e test worker count. |
| `OPENCLAW_E2E_VERBOSE` | `0` | Set to `1` for verbose e2e test output. |

### Model Provider API Keys

Set at least one provider key to enable the agent:

| Variable | Provider |
|----------|----------|
| `ANTHROPIC_API_KEY` | Anthropic (Claude) |
| `OPENAI_API_KEY` | OpenAI (GPT, Codex) |
| `GEMINI_API_KEY` | Google Gemini |
| `OPENROUTER_API_KEY` | OpenRouter (multi-provider) |
| `GROQ_API_KEY` | Groq |
| `ZAI_API_KEY` | ZAI |
| `AI_GATEWAY_API_KEY` | AI Gateway |
| `MINIMAX_API_KEY` | MiniMax |
| `SYNTHETIC_API_KEY` | Synthetic |

### Channel Tokens

| Variable | Channel |
|----------|---------|
| `TELEGRAM_BOT_TOKEN` | Telegram (or `channels.telegram.botToken`) |
| `DISCORD_BOT_TOKEN` | Discord (or `channels.discord.token`) |
| `SLACK_BOT_TOKEN` | Slack (or `channels.slack.botToken`) |
| `SLACK_APP_TOKEN` | Slack app-level token (or `channels.slack.appToken`) |
| `MATTERMOST_BOT_TOKEN` | Mattermost |
| `MATTERMOST_URL` | Mattermost server URL |
| `ZALO_BOT_TOKEN` | Zalo |
| `OPENCLAW_TWITCH_ACCESS_TOKEN` | Twitch |

### Tools & Media

| Variable | Purpose |
|----------|---------|
| `BRAVE_API_KEY` | Brave Search API |
| `PERPLEXITY_API_KEY` | Perplexity research API |
| `FIRECRAWL_API_KEY` | Web scraping |
| `ELEVENLABS_API_KEY` or `XI_API_KEY` | ElevenLabs TTS |
| `DEEPGRAM_API_KEY` | Deepgram speech recognition |

---

## Gateway Config (`gateway.*`)

All settings under the `gateway` key in `openclaw.json`.

### Network & Binding

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.port` | `number` | `18789` | WebSocket server port. |
| `gateway.bind` | `"auto" \| "lan" \| "loopback" \| "custom" \| "tailnet"` | `"loopback"` | Network bind mode. `"loopback"` = 127.0.0.1 only. `"lan"` = all interfaces (requires auth). `"custom"` = use `gateway.customBindHost`. |
| `gateway.customBindHost` | `string` | *(none)* | Custom host when `bind="custom"`. |

### Authentication

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.auth.mode` | `"token" \| "password" \| "trusted-proxy"` | `"token"` | Authentication method. |
| `gateway.auth.token` | `string` | *(env: `OPENCLAW_GATEWAY_TOKEN`)* | Shared secret for HMAC token auth. |
| `gateway.auth.password` | `string` | *(env: `OPENCLAW_GATEWAY_PASSWORD`)* | Password for password auth. |
| `gateway.auth.trustedProxy.userHeader` | `string` | *(none)* | Header containing username (e.g., Pomerium). |
| `gateway.auth.rateLimit.maxAttempts` | `number` | `10` | Max failed auth attempts before lockout. |
| `gateway.auth.rateLimit.windowMs` | `number` | `60000` | Rate limit window in ms (60 seconds). |
| `gateway.auth.rateLimit.lockoutMs` | `number` | `300000` | Lockout duration in ms (5 minutes). |
| `gateway.auth.rateLimit.exemptLoopback` | `boolean` | `true` | Skip rate limiting for localhost connections. |

### Control UI

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.controlUi.enabled` | `boolean` | `true` | Serve the web dashboard and Control UI. |
| `gateway.controlUi.basePath` | `string` | `"/"` | URL base path for the UI. |
| `gateway.controlUi.root` | `string` | *(built-in)* | Custom UI root directory. |
| `gateway.controlUi.allowedOrigins` | `string[]` | *(auto)* | CORS allowed origins. |
| `gateway.controlUi.allowInsecureAuth` | `boolean` | `false` | Allow unencrypted HTTP auth (dev only). |
| `gateway.controlUi.dangerouslyDisableDeviceAuth` | `boolean` | `false` | Disable device authentication (never in prod). |

### TLS

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.tls.enabled` | `boolean` | `false` | Enable TLS encryption. |
| `gateway.tls.autoGenerate` | `boolean` | `true` | Auto-generate self-signed certs if enabled. |
| `gateway.tls.certPath` | `string` | *(auto)* | Path to TLS certificate. |
| `gateway.tls.keyPath` | `string` | *(auto)* | Path to TLS private key. |
| `gateway.tls.caPath` | `string` | *(none)* | Path to CA certificate. |

### Tailscale

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.tailscale.mode` | `"off" \| "serve" \| "funnel"` | `"off"` | Tailscale integration mode. `"serve"` = tailnet-only HTTPS. `"funnel"` = public HTTPS (requires password auth). |
| `gateway.tailscale.resetOnExit` | `boolean` | `false` | Undo serve/funnel on gateway shutdown. |

### Config Reload

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.reload.mode` | `"off" \| "restart" \| "hot" \| "hybrid"` | `"hybrid"` | How config changes are applied. `"hot"` = apply without restart. `"hybrid"` = hot when possible, restart otherwise. |
| `gateway.reload.debounceMs` | `number` | `300` | Debounce delay for file change detection. |

### Discovery

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.discovery.mdns` | `"off" \| "minimal" \| "full"` | `"minimal"` | mDNS/Bonjour discovery mode. `"minimal"` advertises basic presence. `"full"` includes metadata. |

### Canvas Host

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.canvasHost.enabled` | `boolean` | `true` | Start the Canvas HTTP/WS server for agent-driven UI. |
| `gateway.canvasHost.root` | `string` | *(auto)* | Custom canvas root directory. |
| `gateway.canvasHost.port` | `number` | `18793` | Canvas host port. |
| `gateway.canvasHost.liveReload` | `boolean` | `false` | Enable live reload for canvas development. |

### Talk (Voice)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.talk.voiceId` | `string` | *(provider default)* | Default voice ID for TTS. |
| `gateway.talk.voiceAliases` | `Record<string, string>` | *(none)* | Friendly aliases for voice IDs. |
| `gateway.talk.modelId` | `string` | *(provider default)* | TTS model ID. |
| `gateway.talk.outputFormat` | `string` | *(provider default)* | Audio output format. |
| `gateway.talk.interruptOnSpeech` | `boolean` | `true` | Stop speaking when user starts talking. |

### HTTP Endpoints

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.chatCompletions.enabled` | `boolean` | `false` | Expose OpenAI-compatible `/v1/chat/completions` endpoint. |
| `gateway.openResponses.enabled` | `boolean` | `false` | Expose `/v1/responses` endpoint. |
| `gateway.openResponses.maxBodyBytes` | `number` | `20971520` | Max request body size (20 MB). |
| `gateway.openResponses.maxUrlParts` | `number` | `8` | Max URL segments. |

### File/Image Input

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.fileInput.allowUrl` | `boolean` | `true` | Allow URL-based file inputs. |
| `gateway.fileInput.urlAllowlist` | `string[]` | *(none)* | Restrict allowed URL domains. |
| `gateway.fileInput.allowedMimes` | `string[]` | *(all)* | Restrict allowed MIME types. |
| `gateway.fileInput.maxBytes` | `number` | *(unlimited)* | Max file size. |
| `gateway.fileInput.maxRedirects` | `number` | `3` | Max URL redirects to follow. |
| `gateway.fileInput.timeout` | `number` | `10000` | URL fetch timeout in ms. |

### PDF Processing

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.pdf.maxPages` | `number` | `4` | Max PDF pages to process. |
| `gateway.pdf.maxPixels` | `number` | `4000000` | Max pixels per page (4M). |
| `gateway.pdf.minTextChars` | `number` | `200` | Min text chars before falling back to image extraction. |

### Remote Gateway

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.remote.url` | `string` | *(none)* | Remote gateway WebSocket URL. |
| `gateway.remote.transport` | `"ssh" \| "direct"` | *(none)* | Connection transport. |
| `gateway.remote.token` | `string` | *(none)* | Auth token for remote gateway. |
| `gateway.remote.password` | `string` | *(none)* | Auth password for remote gateway. |
| `gateway.remote.tlsFingerprint` | `string` | *(none)* | Expected TLS certificate fingerprint. |

### Node Host

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.nodeHost.allowCommands` | `string[]` | *(all)* | Allowlisted node commands. |
| `gateway.nodeHost.denyCommands` | `string[]` | *(none)* | Denylisted node commands. |

### Tools

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `gateway.tools.allow` | `string[]` | *(all)* | Tool allowlist (restricts available tools). |
| `gateway.tools.deny` | `string[]` | *(none)* | Tool denylist (blocks specific tools). |

---

## Agent Config (`agents.*`)

### Defaults (applied to all agents unless overridden per-agent)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `agents.defaults.maxConcurrent` | `number` | `4` | Max concurrent agent sessions (main). |
| `agents.defaults.workspace` | `string` | `~/.openclaw/workspace` | Agent workspace root directory. |
| `agents.defaults.model` | `string` | *(from config)* | Default model ID (e.g., `"anthropic/claude-opus-4-6"`). |
| `agents.defaults.humanDelay.mode` | `"off" \| "natural" \| "custom"` | `"off"` | Simulate natural typing delay before responses. |
| `agents.defaults.humanDelay.minMs` | `number` | *(varies)* | Min delay in ms (for `"custom"` mode). |
| `agents.defaults.humanDelay.maxMs` | `number` | *(varies)* | Max delay in ms (for `"custom"` mode). |
| `agents.defaults.compaction.reserveTokensFloor` | `number` | `20000` | Min tokens reserved before compaction triggers. |

### Sandbox

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `agents.defaults.sandbox.mode` | `"off" \| "non-main" \| "all"` | `"off"` | Sandbox mode. `"non-main"` sandboxes group/channel sessions. `"all"` sandboxes everything including main. |
| `agents.defaults.sandbox.scope` | `"session"` | `"session"` | Sandbox scope (per-session containers). |
| `agents.defaults.sandbox.workspace` | `"none" \| "ro" \| "rw"` | *(varies)* | Workspace access from sandbox. |

### Docker Sandbox Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `agents.defaults.sandbox.docker.image` | `string` | *(default sandbox image)* | Docker image for sandbox containers. |
| `agents.defaults.sandbox.docker.containerPrefix` | `string` | *(auto)* | Container name prefix. |
| `agents.defaults.sandbox.docker.workdir` | `string` | *(auto)* | Working directory inside container. |
| `agents.defaults.sandbox.docker.readOnlyRoot` | `boolean` | `false` | Mount root filesystem read-only. |
| `agents.defaults.sandbox.docker.network` | `string` | *(default)* | Docker network mode. |
| `agents.defaults.sandbox.docker.pidsLimit` | `number` | *(unlimited)* | Max PIDs in container. |
| `agents.defaults.sandbox.docker.memory` | `string` | *(unlimited)* | Memory limit (e.g., `"512m"`). |
| `agents.defaults.sandbox.docker.cpus` | `number` | *(unlimited)* | CPU limit. |

### Browser Sandbox

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `agents.defaults.sandbox.browser.enabled` | `boolean` | `false` | Enable sandboxed browser (Chromium in Docker). |
| `agents.defaults.sandbox.browser.image` | `string` | *(default browser image)* | Docker image for browser sandbox. |
| `agents.defaults.sandbox.browser.headless` | `boolean` | `true` | Run browser headlessly. |
| `agents.defaults.sandbox.browser.enableNoVnc` | `boolean` | `false` | Enable noVNC web viewer for the sandboxed browser. |
| `agents.defaults.sandbox.browser.autoStart` | `boolean` | `false` | Auto-start browser sandbox with gateway. |

### Per-Agent Config

Each agent in the `agents.list` array can override any default:

```json5
{
  agents: {
    defaults: { /* ... */ },
    list: [
      {
        id: "research",
        name: "Research Agent",
        workspace: "~/research-workspace",
        model: "anthropic/claude-sonnet-4-5-20250929",
        skills: ["github", "obsidian"],
        sandbox: { mode: "all" },
      },
    ],
  },
}
```

---

## Channel Config (`channels.*`)

Each channel has its own config block. Common patterns across channels:

### Shared Channel Settings

| Setting pattern | Type | Default | Description |
|-----------------|------|---------|-------------|
| `channels.<ch>.dmPolicy` | `"pairing" \| "open"` | `"pairing"` | DM access policy. `"pairing"` requires approval code. |
| `channels.<ch>.allowFrom` | `string[]` | `[]` | Allowlist of user IDs / phone numbers. `"*"` = allow all. |
| `channels.<ch>.groups` | `object \| string[]` | *(none)* | Group allowlist. Include `"*"` to allow all groups. |

### WhatsApp (`channels.whatsapp.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.whatsapp.allowFrom` | `string[]` | `[]` | Allowed phone numbers (E.164 format). |
| `channels.whatsapp.groups` | `string[]` | *(none)* | Group allowlist (JIDs or `"*"`). |

### Telegram (`channels.telegram.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.telegram.botToken` | `string` | *(env: `TELEGRAM_BOT_TOKEN`)* | Bot token from @BotFather. |
| `channels.telegram.allowFrom` | `string[]` | `[]` | Allowed user IDs or usernames. |
| `channels.telegram.groups` | `object` | *(none)* | Group config with `requireMention` per group. |
| `channels.telegram.webhookUrl` | `string` | *(none)* | Webhook URL (alternative to polling). |
| `channels.telegram.webhookSecret` | `string` | *(none)* | Webhook verification secret. |

### Discord (`channels.discord.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.discord.token` | `string` | *(env: `DISCORD_BOT_TOKEN`)* | Bot token. |
| `channels.discord.allowFrom` | `string[]` | `[]` | Allowed user IDs. |
| `channels.discord.guilds` | `string[]` | *(none)* | Allowed guild (server) IDs. |
| `channels.discord.mediaMaxMb` | `number` | *(default)* | Max media upload size in MB. |

### Slack (`channels.slack.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.slack.botToken` | `string` | *(env: `SLACK_BOT_TOKEN`)* | Bot OAuth token. |
| `channels.slack.appToken` | `string` | *(env: `SLACK_APP_TOKEN`)* | App-level token (for Socket Mode). |
| `channels.slack.allowFrom` | `string[]` | `[]` | Allowed Slack user IDs. |

### Signal (`channels.signal.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.signal.*` | `object` | *(none)* | Requires `signal-cli` installed. See [Signal docs](https://docs.openclaw.ai/channels/signal). |

### BlueBubbles / iMessage (`channels.bluebubbles.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.bluebubbles.serverUrl` | `string` | *(required)* | BlueBubbles server URL. |
| `channels.bluebubbles.password` | `string` | *(required)* | BlueBubbles server password. |
| `channels.bluebubbles.webhookPath` | `string` | *(auto)* | Webhook path for incoming messages. |

### Microsoft Teams (`channels.msteams.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `channels.msteams.allowFrom` | `string[]` | `[]` | Allowed user IDs. |
| `channels.msteams.groupAllowFrom` | `string[]` | `[]` | Allowed group/team IDs. |
| `channels.msteams.groupPolicy` | `"pairing" \| "open"` | `"pairing"` | Group access policy. |

---

## Model Config (`models.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `models.mode` | `"merge" \| "replace"` | `"merge"` | `"merge"` = implicit providers (from env) + explicit overrides. `"replace"` = explicit config only. |

### Providers

Each provider in the `models.providers` object:

```json5
{
  models: {
    mode: "merge",
    providers: {
      anthropic: {
        // baseUrl, apiKey auto-detected from env
        models: [
          { id: "claude-opus-4-6", name: "Claude Opus 4.6" },
        ],
      },
      custom: {
        baseUrl: "http://localhost:11434/v1",
        apiKey: "ollama",
        models: [
          { id: "llama3.2", name: "Llama 3.2" },
        ],
      },
    },
  },
}
```

| Provider Setting | Type | Description |
|------------------|------|-------------|
| `models.providers.<name>.baseUrl` | `string` | API base URL. |
| `models.providers.<name>.apiKey` | `string` | API key (or use env var). |
| `models.providers.<name>.models` | `ModelDefinitionConfig[]` | Available models. |
| `models.providers.<name>.models[].id` | `string` | Model identifier. |
| `models.providers.<name>.models[].contextWindow` | `number` | Context window size. |
| `models.providers.<name>.models[].maxTokens` | `number` | Max output tokens. |

### Bedrock Discovery

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `models.bedrock.discovery.regions` | `string[]` | *(none)* | AWS regions to scan for available Bedrock models. |

### Implicit Providers (auto-detected from env)

These providers are automatically available when their API key is set:

| Provider | Env Variable | Notes |
|----------|-------------|-------|
| Anthropic | `ANTHROPIC_API_KEY` | Claude models |
| OpenAI | `OPENAI_API_KEY` | GPT, Codex models |
| Gemini | `GEMINI_API_KEY` | Google Gemini |
| OpenRouter | `OPENROUTER_API_KEY` | Multi-provider gateway |
| Groq | `GROQ_API_KEY` | Fast inference |
| Ollama | *(auto at localhost:11434)* | Local models |
| DeepSeek | `DEEPSEEK_API_KEY` | DeepSeek models |
| GitHub Copilot | OAuth token | Token exchange |
| AWS Bedrock | AWS SDK credentials | Regional auto-discovery |

---

## Skills Config (`skills.*`)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `skills.allowBundled` | `string[]` | *(all bundled)* | Allowlist of bundled skill names. |
| `skills.load.extraDirs` | `string[]` | `[]` | Additional directories to scan for skills. |
| `skills.load.watch` | `boolean` | `false` | Watch skill directories for changes. |
| `skills.load.watchDebounceMs` | `number` | *(default)* | Debounce for file change detection. |
| `skills.install.preferBrew` | `boolean` | `false` | Prefer Homebrew for skill dependencies. |
| `skills.install.nodeManager` | `"npm" \| "pnpm" \| "yarn" \| "bun"` | *(auto)* | Package manager for skill installs. |

### Per-Skill Config

```json5
{
  skills: {
    weather: {
      enabled: true,
      apiKey: "...",
      config: { units: "metric" },
    },
    github: {
      enabled: true,
      env: { GITHUB_TOKEN: "..." },
    },
  },
}
```

---

## Hooks Config (`hooks.*`)

### Hook Mappings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `hooks.mappings[].match.path` | `string` | *(required)* | URL path to match (webhook endpoint). |
| `hooks.mappings[].match.source` | `string` | *(none)* | Source identifier filter. |
| `hooks.mappings[].action` | `"wake" \| "agent"` | *(required)* | Action on match: wake an agent or run inline. |
| `hooks.mappings[].wakeMode` | `string` | *(default)* | Wake mode for the triggered agent. |
| `hooks.mappings[].agentId` | `string` | *(default agent)* | Target agent ID. |
| `hooks.mappings[].model` | `string` | *(default)* | Model override for this hook. |
| `hooks.mappings[].thinking` | `string` | *(default)* | Thinking level override. |
| `hooks.mappings[].timeoutSeconds` | `number` | *(default)* | Hook execution timeout. |

### Gmail Hooks

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `hooks.gmail.account` | `string` | *(required)* | Gmail account to watch. |
| `hooks.gmail.label` | `string` | `"INBOX"` | Gmail label to monitor. |
| `hooks.gmail.topic` | `string` | *(required)* | Google Pub/Sub topic. |
| `hooks.gmail.subscription` | `string` | *(required)* | Google Pub/Sub subscription. |
| `hooks.gmail.includeBody` | `boolean` | `false` | Include email body in hook payload. |
| `hooks.gmail.maxBytes` | `number` | *(default)* | Max email body size. |
| `hooks.gmail.renewEveryMinutes` | `number` | *(default)* | Pub/Sub watch renewal interval. |

### Webhook Server

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `hooks.serve.bind` | `string` | *(gateway bind)* | Webhook server bind address. |
| `hooks.serve.port` | `number` | *(gateway port)* | Webhook server port. |
| `hooks.serve.path` | `string` | `"/hooks"` | Base path for webhook endpoints. |

---

## Session Config

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `session.scope` | `"per-sender" \| "global"` | `"per-sender"` | Session isolation. `"per-sender"` = each user gets their own session. `"global"` = shared session. |
| `session.dmScope` | `"main" \| "per-peer" \| "per-channel-peer" \| "per-account-channel-peer"` | `"main"` | DM session granularity. |
| `session.resetMode` | `"daily" \| "idle"` | *(none)* | Auto-reset sessions daily or after idle timeout. |
| `session.typingMode` | `"never" \| "instant" \| "thinking" \| "message"` | `"message"` | When to show typing indicators. |
| `session.replyMode` | `"text" \| "command"` | `"text"` | Reply format. |
| `session.maintenance` | `string` | *(none)* | Maintenance mode message. |

### Logging

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `logging.level` | `"silent" \| "error" \| "warn" \| "info" \| "debug" \| "trace"` | `"info"` | Console log level. |
| `logging.file` | `boolean \| string` | `false` | Log to file (true = default path, string = custom path). |
| `logging.ansiStyle` | `"pretty" \| "compact" \| "json"` | `"pretty"` | Console output format. |
| `logging.redaction` | `boolean` | `false` | Redact sensitive values in logs. |

### Markdown

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `markdown.tables` | `"off" \| "bullets" \| "code"` | `"off"` | How to render markdown tables in channel output. |

### Block Streaming

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `blockStreaming.coalesce` | `object` | *(auto)* | Coalesce settings for streaming blocks. |
| `blockStreaming.chunk` | `object` | *(auto)* | Chunk settings for block replies. |

### Diagnostics (OpenTelemetry)

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `diagnostics.endpoint` | `string` | *(none)* | OTLP endpoint URL. |
| `diagnostics.cacheTraces` | `boolean` | `false` | Cache traces locally. |
| `diagnostics.protocol` | `"http/protobuf" \| "grpc"` | `"http/protobuf"` | OTLP transport protocol. |

---

## Security Config

### Commands

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `commands.native` | `boolean \| object` | Auto (on: Discord/Telegram, off: Slack) | Enable native slash commands on channels. |
| `commands.text` | `boolean` | *(varies)* | Enable text-based commands. |
| `commands.useAccessGroups` | `boolean` | `false` | Use access group permissions for commands. |

### Approvals

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `approvals.*` | `object` | *(none)* | Tool approval workflows (require human approval before executing certain tools). |

### Browser

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `browser.enabled` | `boolean` | *(auto)* | Enable browser tool. |
| `browser.color` | `string` | *(none)* | Browser theme color. |

---

## Build & Dev Config

### TypeScript (`tsconfig.json`)

| Setting | Value | Notes |
|---------|-------|-------|
| Target | `ES2023` | Modern JavaScript output. |
| Module | `NodeNext` | ESM with Node.js resolution. |
| Strict | `true` | Full strict type checking. |
| Decorators | `experimentalDecorators: true` | Legacy decorators (Lit components). |
| Path aliases | `openclaw/plugin-sdk → src/plugin-sdk/` | Plugin SDK import mapping. |

### Linting (`.oxlintrc.json`)

| Rule | Severity | Notes |
|------|----------|-------|
| `no-explicit-any` | `error` | No `any` types allowed. |
| `curly` | `error` | Braces required on all blocks. |
| `no-await-in-loop` | `off` | Await in loops is allowed. |
| Correctness rules | `error` | All correctness rules enforced. |
| Performance rules | `error` | All perf rules enforced. |
| Suspicious rules | `error` | All suspicious pattern rules enforced. |

### Testing (`vitest.config.ts`)

| Setting | Value | Notes |
|---------|-------|-------|
| Pool | `forks` | Process isolation between tests. |
| Test timeout | `120,000 ms` | 2 minutes per test. |
| Hook timeout | `180,000 ms` (Windows) / `120,000 ms` (Unix) | Setup/teardown timeout. |
| Workers | `4-16` (local), `2-3` (CI) | Auto-scaled. |
| Coverage threshold | `70%` lines/functions/statements, `55%` branches | V8 coverage provider. |

### Pre-commit Hooks (`.pre-commit-config.yaml`)

| Hook | Tool | What it checks |
|------|------|----------------|
| Trailing whitespace | built-in | Removes trailing spaces. |
| End of file | built-in | Ensures newline at EOF. |
| Large files | built-in | Blocks files > 500 KB. |
| Secret detection | detect-secrets | Scans for leaked secrets/keys. |
| Shell linting | shellcheck | Shell script errors. |
| Actions linting | actionlint | GitHub Actions workflow errors. |
| Actions security | zizmor | GitHub Actions security issues. |
| TypeScript lint | oxlint | `oxlint --type-aware src test`. |
| TypeScript format | oxfmt | `oxfmt --check src test`. |
| Swift lint | swiftlint | macOS app Swift code. |
| Swift format | swiftformat | macOS app Swift formatting. |

---

## Deployment Config

### Docker (`docker-compose.yml`)

| Setting | Default | Description |
|---------|---------|-------------|
| Gateway bind | `${OPENCLAW_GATEWAY_BIND:-lan}` | Network bind mode in container. |
| Gateway port | `18789` | WebSocket port (mapped to host). |
| Bridge port | `18790` | Bridge port (mapped to host). |
| Restart policy | `unless-stopped` | Auto-restart on crash. |
| Init | `true` | Proper signal handling with tini. |
| User | `node` (uid 1000) | Non-root user in container. |

### Fly.io (`fly.toml`)

| Setting | Value | Description |
|---------|-------|-------------|
| Region | `iad` | US East (Virginia). |
| VM | `shared-cpu-2x` | 2 shared CPU cores. |
| Memory | `2048 MB` | 2 GB RAM. |
| Internal port | `3000` | App listens on port 3000. |
| Force HTTPS | `true` | Redirect HTTP to HTTPS. |
| Auto-stop | `false` | Keep running (persistent WS connections). |
| Auto-start | `true` | Start on first request. |
| Min machines | `1` | Always-on. |
| Volume | `openclaw_data` at `/data` | Persistent storage. |
| `NODE_OPTIONS` | `--max-old-space-size=1536` | 1.5 GB heap limit. |

### Render.com (`render.yaml`)

| Setting | Value | Description |
|---------|-------|-------------|
| Plan | `starter` | Entry-level plan. |
| Health check | `/health` | HTTP health endpoint. |
| Port | `8080` | Internal app port. |
| Disk | `1 GB` at `/data` | Persistent storage. |
| `OPENCLAW_GATEWAY_TOKEN` | Auto-generated | Secure token created at deploy. |
