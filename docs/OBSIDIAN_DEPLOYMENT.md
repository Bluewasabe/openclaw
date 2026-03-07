# Obsidian Deployment Guide — SilverBullet + OpenClaw Agents

## Overview

This guide covers how to deploy a shared knowledge vault that both OpenClaw agents and humans can access. The approach uses **SilverBullet** — a lightweight, self-hosted, web-based markdown platform — pointed at the same vault directory that OpenClaw agents already mount.

**Architecture:**

```
┌──────────────────────┐     ┌──────────────────────┐
│   OpenClaw Agents    │     │     SilverBullet      │
│  (read/write .md)    │     │   (web UI + REST API) │
│                      │     │   http://host:3000    │
└──────────┬───────────┘     └──────────┬───────────┘
           │                            │
           ▼                            ▼
    ┌──────────────────────────────────────────┐
    │         Shared Vault Directory           │
    │         ${OBSIDIAN_VAULT_DIR}            │
    │                                          │
    │  vault/                                  │
    │  ├── agent-memory/   ← agents write here │
    │  │   ├── task-logs/                      │
    │  │   ├── research/                       │
    │  │   └── decisions/                      │
    │  ├── personal/       ← your notes        │
    │  └── projects/       ← shared            │
    └──────────────────────────────────────────┘
```

## Why SilverBullet?

- **Same files on disk** — no sync layer, no database, no conflicts
- **Browser access** — browse and edit notes from any device on your LAN
- **Built-in REST API** — agents can use HTTP instead of raw file writes
- **Lightweight** — ~100MB RAM (vs 500MB+ for full Obsidian-in-Docker)
- **PWA support** — installable as an app with offline capability
- **Wikilinks, backlinks, frontmatter** — core Obsidian-like features work

## Docker Compose Setup

Add this service to your existing `docker-compose.yml` or create a new compose file alongside your OpenClaw stack:

```yaml
services:
  silverbullet:
    image: silverbulletmd/silverbullet:latest
    container_name: silverbullet
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ${OBSIDIAN_VAULT_DIR}:/space
    environment:
      - SB_USER=admin:${SB_PASSWORD:-changeme}
    networks:
      - jarvis-net

networks:
  jarvis-net:
    external: true
```

### Environment Variables

Add to your `.env` file (same one used by OpenClaw):

```env
# Obsidian / SilverBullet
OBSIDIAN_VAULT_DIR=C:\Users\Bluew\.openclaw\workspace\vault
SB_PASSWORD=your-secure-password-here
```

### Start the Service

```bash
docker compose up -d silverbullet
```

Access at: `http://192.168.1.217:3000`

## Agent Access

### Option A: Direct Filesystem (Already Working)

OpenClaw agents mount the vault at `/home/node/.openclaw/workspace/vault` via the existing docker-compose volume:

```yaml
- ${OBSIDIAN_VAULT_DIR}:/home/node/.openclaw/workspace/vault
```

Agents can read/write `.md` files directly. SilverBullet detects changes immediately.

### Option B: SilverBullet REST API

Agents on the `jarvis-net` network can use SilverBullet's HTTP API:

```bash
# Read a note
curl http://silverbullet:3000/.fs/agent-memory/task-log.md

# Write/update a note
curl -X PUT \
  -H "Content-Type: text/markdown" \
  --data-binary @note.md \
  http://silverbullet:3000/.fs/agent-memory/task-log.md

# Delete a note
curl -X DELETE http://silverbullet:3000/.fs/agent-memory/task-log.md

# List all files
curl http://silverbullet:3000/.fs
```

The API is available at `http://silverbullet:3000` from any container on `jarvis-net`.

## Folder Structure Best Practices

Separate agent-written and human-written content to avoid conflicts:

```
vault/
├── agent-memory/           ← agents write here (do not manually edit)
│   ├── task-logs/          ← execution logs, summaries
│   ├── research/           ← web research, findings
│   ├── decisions/          ← architectural decisions, rationale
│   └── context/            ← session context, state
├── personal/               ← your notes (agents read-only)
├── projects/               ← shared project docs
│   ├── jarvis/
│   ├── openclaw/
│   └── ...
└── templates/              ← note templates
```

### Frontmatter Convention

Agents should tag their notes with frontmatter for filtering:

```yaml
---
created_by: jarvis
agent_id: main
created: 2026-03-03T12:00:00Z
tags: [agent-memory, task-log]
---
```

## Alternative: Desktop Obsidian (No Extra Containers)

If you prefer using the native Obsidian desktop app instead of a web UI:

1. Install Obsidian on Windows
2. Open `${OBSIDIAN_VAULT_DIR}` as a vault
3. Agents write files; Obsidian auto-detects filesystem changes
4. Optionally install the **Local REST API plugin** for agent HTTP access via `host.docker.internal:27124`

This uses zero extra resources but only works from your Windows desktop (no browser/mobile access).

## Alternative: Full Obsidian in Browser

If you need the real Obsidian experience (all plugins, graph view, Dataview) in a browser:

```yaml
services:
  obsidian:
    image: lscr.io/linuxserver/obsidian:latest
    container_name: obsidian-browser
    restart: unless-stopped
    ports:
      - "3100:3000"
      - "3101:3001"
    volumes:
      - ./obsidian-config:/config
      - ${OBSIDIAN_VAULT_DIR}:/config/obsidian-vault
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    shm_size: "1gb"
    networks:
      - jarvis-net
```

Access at `https://192.168.1.217:3101`. Requires ~500MB+ RAM and 1GB shared memory.

## Comparison

| Feature | SilverBullet | Desktop Obsidian | Obsidian in Docker |
|---------|-------------|-----------------|-------------------|
| Browser access | Yes | No | Yes (VNC) |
| Agent REST API | Built-in | Plugin needed | No |
| Obsidian plugins | No | Yes | Yes |
| Resource usage | ~100MB | 0 (no container) | ~500MB+ |
| Setup complexity | Minimal | None | Medium |
| Multi-device | Yes (LAN) | No | Yes (LAN) |
