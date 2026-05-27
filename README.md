# 9Router Starter Kit

> 5-minute setup for AI model routing across Claude Code, Cursor, Codex, Gemini CLI, and more

English | [中文](README.zh-CN.md)

## Why This Exists

AI coding tools each have different model providers. Switching between them manually is painful. 9Router gives you a local proxy that routes requests to the right model with fallback chains — configured in 5 minutes.

Use this starter kit when you want one **llm proxy** and **model router** for Claude Code, Cursor, Cline, Codex CLI, Gemini CLI, and OpenClaw. It helps with ai coding model switching, quota fallback, cost control, and 多模型路由 without editing every tool by hand.

Common problems it solves:

- Quota exhausted mid-session → automatically fallback to the next available model
- Premium models wasted on low-complexity tasks → route Haiku-style tasks to cheaper models such as GLM
- Manual model switching across tools → use aliases and combo presets instead
- Different endpoint formats → expose a local OpenAI-compatible proxy at `localhost:20128/v1`

## Quick Start

```bash
# Install and configure in one go
./install.sh && ./configure.sh

# Optional: verify the generated routing setup
./verify.sh
```

The interactive setup checks whether 9Router is running, asks which AI coding tools you use, collects provider credentials, writes 9Router providers/aliases/combos through the REST API, generates tool-specific config, and verifies that the local model router works.

## Requirements

- Node.js >= 18
- `curl` and `jq` (`brew install jq` on macOS if needed)
- At least one model provider API key or Claude OAuth subscription connection

## Supported Tools

| Tool | Config Generated |
|------|-----------------|
| Claude Code | `.claude.json` with model routing |
| Cursor | Guide for settings.json |
| Codex CLI | Setup shell script |
| Gemini CLI | `.gemini/settings.json` |
| Cline | Guide for VS Code settings |
| OpenClaw | Configuration file |

## Combo Presets

| Combo | Models | Use Case |
|-------|--------|----------|
| full-subscription | All premium models | Power users with API keys |
| hybrid | Mix of free and paid | Cost-conscious developers |
| domestic-only | China-based models (GLM, DeepSeek, Qwen, etc.) | No overseas API needed |
| free-tier | Free models only | Zero-cost setup |

`configure.sh` matches your selected providers to the closest preset, then writes a fallback chain into 9Router so requests can move from the preferred model to backup providers when quota, network, or provider errors occur.

## Architecture

AI coding tools send OpenAI-compatible requests to `localhost:20128/v1`. 9Router receives each request, resolves the `model` field through model aliases, applies combo fallback rules when needed, and forwards the request to the first available provider.

Read [docs/architecture.md](docs/architecture.md) for data flow and storage schema.

## Generated Files

```text
├── install.sh          # One-command install for 9Router and pm2 process management
├── configure.sh        # Interactive setup for tools, providers, API keys, aliases, and combos
├── verify.sh           # Five-stage verification script
├── uninstall.sh        # Rollback script
├── config/             # Preset templates
│   ├── combos/         # Four combo presets
│   ├── aliases.json    # Default model aliases
│   └── tool-configs/   # Templates for supported tools
├── claude-md/          # CLAUDE.md routing rule snippets
├── output/             # Generated configs, ignored by git
└── docs/
    ├── architecture.md
    └── faq.md
```

## Rollback

```bash
./uninstall.sh
```

This restores previous local tool configuration and stops 9Router-managed processes where applicable.

## FAQ

### What if `install.sh` says Node.js is not installed?

Install Node.js 18 or newer first. We recommend nvm:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
nvm install 22
```

### What if port `20128` is already in use?

Run `lsof -i :20128`. If the process is 9Router, it usually means 9Router is already installed. If another process owns the port, stop that process before installing.

### Why does `configure.sh` require `jq`?

The setup script uses `jq` to parse API responses and generated JSON. On macOS, install it with `brew install jq`; on Linux, use your package manager such as `apt install jq` or `yum install jq`.

### I have both Claude Max and a Claude API key. Which should I choose?

Choose Claude Max subscription / OAuth if available. OAuth can refresh tokens automatically and avoids pay-as-you-go API key billing for supported workflows. Use an API key when you specifically want metered API usage.

### Can I configure only one model?

Yes, but the main value of 9Router is multi-provider fallback. For reliable ai model routing, we recommend configuring at least two providers.

### Is routing Haiku-style tasks to GLM good enough?

For low-complexity sub-tasks such as review, testing, and summarization, GLM-5.1 is usually sufficient. Strong reasoning tasks can still route to Opus or Sonnet through aliases and combo rules.

### How do I configure Cursor?

Cursor settings are GUI-based, so they cannot be fully scripted. `configure.sh` generates `output/cursor/guide.md` with step-by-step instructions for Cursor Settings → Models.

### Where are 9Router logs?

Use `pm2 logs 9router` or inspect `~/.9router/logs/`.

## More from AIFlowLearn

- [9router-starter-kit](https://github.com/aiflowlearn/9router-starter-kit) — AI model routing in 5 minutes
- [awesome-claude-code](https://github.com/aiflowlearn/awesome-claude-code) — Claude Code learning toolkit
- [zero2claude](https://github.com/aiflowlearn/zero2claude) — Learn Claude Code from zero
- [zero2codex](https://github.com/aiflowlearn/zero2codex) — Learn Codex CLI from zero
- [zero2cursor](https://github.com/aiflowlearn/zero2cursor) — Learn Cursor IDE from zero
- [zero2codewhale](https://github.com/aiflowlearn/zero2codewhale) — Learn CodeWhale from zero
- [ai-coding-skillpacks](https://github.com/aiflowlearn/ai-coding-skillpacks) — 21 AI coding learning paths

## Online Practice

Practice AI coding workflows and model routing patterns on [AIFlowLearn](https://aiflowlearn.net), a browser-based platform for interactive AI coding courses, labs, and skill packs.

## Contributing

Contributions welcome! Please see the issues tab or submit a PR.

## License

MIT License — Copyright (c) 2026 AIFlowLearn

---

*Sponsored by [AIFlowLearn](https://aiflowlearn.net) — AI-native learning platform*

**[AIFlowLearn / AI智流学社](https://aiflowlearn.net)** — Browser-based AI coding practice environments, interactive courses, and skill packs.

[Zero2Claude](https://zero2claude.aiflowlearn.net/) | [Zero2Codex](https://zero2codex.aiflowlearn.net/) | [All Skill Packs](https://aiflowlearn.net/skillpacks)
