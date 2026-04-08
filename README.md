# Claude Code Haha

A **locally runnable version** based on a fix of the leaked Claude Code source, supporting any Anthropic-compatible API (such as MiniMax, OpenRouter, etc.).

> The original leaked source cannot run directly. This repository fixes several blocking issues in the startup chain so that the full Ink TUI interface works locally.

<p align="center">
  <img src="docs/00runtime.png" alt="Runtime screenshot" width="800">
</p>

## Features

- Full Ink TUI interface (matching the official Claude Code)
- `--print` headless mode (for scripts / CI scenarios)
- Support for MCP servers, plugins, and Skills
- Support for custom API endpoints and models
- Fallback Recovery CLI mode

---

## Architecture Overview

<table>
  <tr>
    <td align="center" width="25%"><img src="docs/01-overall-architecture.png" alt="Overall Architecture"><br><b>Overall Architecture</b></td>
    <td align="center" width="25%"><img src="docs/02-request-lifecycle.png" alt="Request Lifecycle"><br><b>Request Lifecycle</b></td>
    <td align="center" width="25%"><img src="docs/03-tool-system.png" alt="Tool System"><br><b>Tool System</b></td>
    <td align="center" width="25%"><img src="docs/04-multi-agent.png" alt="Multi-Agent Architecture"><br><b>Multi-Agent Architecture</b></td>
  </tr>
  <tr>
    <td align="center" width="25%"><img src="docs/05-terminal-ui.png" alt="Terminal UI"><br><b>Terminal UI</b></td>
    <td align="center" width="25%"><img src="docs/06-permission-security.png" alt="Permissions & Security"><br><b>Permissions & Security</b></td>
    <td align="center" width="25%"><img src="docs/07-services-layer.png" alt="Services Layer"><br><b>Services Layer</b></td>
    <td align="center" width="25%"><img src="docs/08-state-data-flow.png" alt="State & Data Flow"><br><b>State & Data Flow</b></td>
  </tr>
</table>

---

## Quick Start

### 1. Install dependencies

Requires [Bun](https://bun.sh) >= 1.1 and Node.js >= 18.

```bash
npm install
```

### 2. Configure environment variables

Copy the example file and fill in your API key:

```bash
cp .env.example .env
```

Edit `.env`:

```env
# API authentication (choose one)
ANTHROPIC_API_KEY=sk-xxx          # Standard API key (x-api-key header)
ANTHROPIC_AUTH_TOKEN=sk-xxx       # Bearer token (Authorization header)

# API endpoint (optional, defaults to Anthropic official)
ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic

# Model configuration
ANTHROPIC_MODEL=MiniMax-M2.7-highspeed
ANTHROPIC_DEFAULT_SONNET_MODEL=MiniMax-M2.7-highspeed
ANTHROPIC_DEFAULT_HAIKU_MODEL=MiniMax-M2.7-highspeed
ANTHROPIC_DEFAULT_OPUS_MODEL=MiniMax-M2.7-highspeed

# Timeout (milliseconds)
API_TIMEOUT_MS=3000000

# Disable telemetry and non-essential network requests
DISABLE_TELEMETRY=1
CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1
```

### 3. Launch

```bash
# Interactive TUI mode (full interface)
./bin/claude-haha

# Headless mode (single-shot Q&A)
./bin/claude-haha -p "your prompt here"

# Pipe input
echo "explain this code" | ./bin/claude-haha -p

# View all options
./bin/claude-haha --help
```

---

## Environment Variables

| Variable | Required | Description |
|------|------|------|
| `ANTHROPIC_API_KEY` | One of two | API key, sent via the `x-api-key` header |
| `ANTHROPIC_AUTH_TOKEN` | One of two | Auth token, sent via the `Authorization: Bearer` header |
| `ANTHROPIC_BASE_URL` | No | Custom API endpoint, defaults to Anthropic official |
| `ANTHROPIC_MODEL` | No | Default model |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | No | Sonnet-tier model mapping |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | No | Haiku-tier model mapping |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | No | Opus-tier model mapping |
| `API_TIMEOUT_MS` | No | API request timeout, defaults to 600000 (10 min) |
| `DISABLE_TELEMETRY` | No | Set to `1` to disable telemetry |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | No | Set to `1` to disable non-essential network requests |

---

## Fallback Mode

If the full TUI has problems, you can use the simplified readline interactive mode:

```bash
CLAUDE_CODE_FORCE_RECOVERY_CLI=1 ./bin/claude-haha
```

---

## Fixes Relative to the Original Leaked Source

The leaked source cannot run directly. The following issues were fixed:

| Issue | Root Cause | Fix |
|------|------|------|
| TUI does not start | The entry script routed parameter-less startup to the recovery CLI | Restored routing through the full `cli.tsx` entry |
| Startup hangs | The `verify` skill imports a missing `.md` file, and Bun's text loader hangs indefinitely | Created stub `.md` files |
| `--print` hangs | `filePersistence/types.ts` is missing | Created a type stub file |
| `--print` hangs | `ultraplan/prompt.txt` is missing | Created a resource stub file |
| **Enter key unresponsive** | The `modifiers-napi` native package is missing; `isModifierPressed()` throws, which interrupts `handleEnter` so `onSubmit` never runs | Added a try-catch fallback |
| Setup is skipped | `preload.ts` automatically sets `LOCAL_RECOVERY=1`, skipping all initialization | Removed the default setting |

---

## Project Structure

```
bin/claude-haha          # Entry script
preload.ts               # Bun preload (sets MACRO global variables)
.env.example             # Environment variable template
src/
├── entrypoints/cli.tsx  # CLI main entry
├── main.tsx             # TUI main logic (Commander.js + React/Ink)
├── localRecoveryCli.ts  # Fallback Recovery CLI
├── setup.ts             # Startup initialization
├── screens/REPL.tsx     # Interactive REPL interface
├── ink/                 # Ink terminal rendering engine
├── components/          # UI components
├── tools/               # Agent tools (Bash, Edit, Grep, etc.)
├── commands/            # Slash commands (/commit, /review, etc.)
├── skills/              # Skill system
├── services/            # Services layer (API, MCP, OAuth, etc.)
├── hooks/               # React hooks
└── utils/               # Utility functions
```

---

## Tech Stack

| Category | Technology |
|------|------|
| Runtime | [Bun](https://bun.sh) |
| Language | TypeScript |
| Terminal UI | React + [Ink](https://github.com/vadimdemedes/ink) |
| CLI parsing | Commander.js |
| API | Anthropic SDK |
| Protocols | MCP, LSP |

---

## Disclaimer

This repository is based on the Claude Code source code leaked from the Anthropic npm registry on 2026-03-31. All original source code is copyright [Anthropic](https://www.anthropic.com). For learning and research purposes only.
