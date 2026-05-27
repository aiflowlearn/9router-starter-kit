# 9Router Starter Kit

> 5 分钟完成 Claude Code、Cursor、Codex、Gemini CLI 等 AI 编码工具的多模型路由配置

[English](README.md) | 中文

## 为什么需要它

AI 编码工具各自支持不同的模型服务商，手动切换模型、复制 API Key、维护 fallback 配置都很麻烦。9Router Starter Kit 提供一个本地 **llm proxy** 和 **model router**，把 Claude Code、Cursor、Cline、Codex CLI、Gemini CLI、OpenClaw 等工具统一接入本地代理，在 5 分钟内完成 ai model routing 和 多模型路由。

它主要解决这些问题：

- 额度耗尽就停工 → 自动 fallback 到下一个可用模型
- Opus/Sonnet 额度浪费在低复杂度任务 → haiku 子任务自动走 GLM 等更省成本的模型
- 多个工具手动切模型太麻烦 → 用 Alias 和 Combo 统一管理 ai coding model switching
- 各工具端点格式不同 → 统一走 `localhost:20128/v1` 的 OpenAI 兼容代理

## 快速开始

```bash
# 一次完成安装和交互式配置
./install.sh && ./configure.sh

# 可选：验证路由配置是否生效
./verify.sh
```

`configure.sh` 会检测 9Router 是否运行，询问你使用的 AI 编码工具和模型服务，收集 API Key，通过 9Router REST API 自动写入 Provider、Alias、Combo，并生成各工具配置文件。

## 前置要求

- Node.js >= 18
- `curl` 和 `jq`（macOS 可用 `brew install jq`）
- 至少一个 AI 模型服务 API Key，或 Claude OAuth / Claude Max 订阅连接

## 支持的工具

| 工具 | 生成的配置 |
|------|-----------|
| Claude Code | 带模型路由的 `.claude.json` |
| Cursor | settings.json 配置指南 |
| Codex CLI | 初始化 Shell 脚本 |
| Gemini CLI | `.gemini/settings.json` |
| Cline | VS Code 设置指南 |
| OpenClaw | 配置文件 |

## Combo 预设

| Combo | 模型组合 | 适合场景 |
|-------|----------|----------|
| full-subscription | 全部高级模型 | 拥有多个 API Key / 订阅的重度用户 |
| hybrid | 免费 + 付费模型混合 | 希望控制成本的开发者 |
| domestic-only | GLM、DeepSeek、Qwen 等国产模型 | 不依赖海外 API 的团队 |
| free-tier | 免费模型 | 零成本体验 |

`configure.sh` 会根据你选择的模型自动匹配最接近的套餐，并写入 fallback 链路。当某个 Provider 额度不足、网络失败或返回错误时，9Router 会自动尝试下一个模型。

## 架构

AI 编码工具把 OpenAI 兼容请求发送到 `localhost:20128/v1`。9Router 接收请求后解析 `model` 字段，先查模型别名 Alias；如果命中 Combo 名称，则按 fallback 策略选择第一个可用 Provider；Provider 成功响应后再返回给工具。

详细数据流和存储结构请阅读 [docs/architecture.md](docs/architecture.md)。

## 目录结构

```text
├── install.sh          # 一键安装 9Router + pm2 进程管理
├── configure.sh        # 交互式配置工具、模型、API Key、Alias、Combo
├── verify.sh           # 5 阶段自动验证
├── uninstall.sh        # 一键回滚
├── config/             # 预设模板
│   ├── combos/         # 4 套 combo 预设
│   ├── aliases.json    # 默认模型别名
│   └── tool-configs/   # 6 种工具配置模板
├── claude-md/          # CLAUDE.md 路由规则片段
├── output/             # configure.sh 生成的配置（gitignored）
└── docs/
    ├── architecture.md
    └── faq.md
```

## 回滚

```bash
./uninstall.sh
```

该脚本会恢复原有本地工具配置，并在需要时停止 9Router 相关进程。

## 常见问题

### install.sh 报 “Node.js 未安装” 怎么办？

需要先安装 Node.js 18 或更高版本。推荐使用 nvm：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
nvm install 22
```

### install.sh 报 “端口 20128 被占用” 怎么办？

运行 `lsof -i :20128` 查看占用进程。如果是 9Router，通常说明已经安装；如果是其他进程，需要先停止它。

### configure.sh 报 “jq 未安装” 怎么办？

macOS 执行 `brew install jq`；Linux 可执行 `apt install jq` 或 `yum install jq`。

### Claude OAuth 登录后 configure.sh 还是检测不到怎么办？

OAuth 连接存储在 9Router 的 `claude` provider 中。请先确认 9Router Dashboard 中 Claude provider 已连接，再重新运行 `./configure.sh`。

### 我有 Claude Max 订阅也有 API Key，选哪个？

优先选择 “Claude Max 订阅 / OAuth”。OAuth 模式可自动刷新 token，使用体验更省心；API Key 模式适合需要按量计费或独立账单的场景。

### 可以只配置一个模型吗？

可以。但 9Router 的核心价值是多模型冗余和 fallback。为了更稳定的 ai model routing，建议至少配置 2 个模型服务。

### haiku 路由到 GLM 后质量够用吗？

haiku 子任务通常是 review、测试、总结等低复杂度任务，GLM-5.1 一般可以胜任。强推理任务仍可通过 Alias 和 Combo 路由到 Opus 或 Sonnet。

### Cursor 怎么配置？

Cursor 是 GUI 设置，无法完全脚本化。`configure.sh` 会生成 `output/cursor/guide.md`，按指南在 Settings → Models 中填写即可。

### 9Router 日志在哪里？

可以查看 `pm2 logs 9router` 或 `~/.9router/logs/`。

## 更多 AIFlowLearn 项目

- [9router-starter-kit](https://github.com/aiflowlearn/9router-starter-kit) — 5 分钟完成 AI 模型路由配置
- [awesome-claude-code](https://github.com/aiflowlearn/awesome-claude-code) — Claude Code 学习工具包
- [zero2claude](https://github.com/aiflowlearn/zero2claude) — 从零学习 Claude Code
- [zero2codex](https://github.com/aiflowlearn/zero2codex) — 从零学习 Codex CLI
- [zero2cursor](https://github.com/aiflowlearn/zero2cursor) — 从零学习 Cursor IDE
- [zero2codewhale](https://github.com/aiflowlearn/zero2codewhale) — 从零学习 CodeWhale
- [ai-coding-skillpacks](https://github.com/aiflowlearn/ai-coding-skillpacks) — 21 条 AI 编程学习路径

## 在线练习

你可以在 [AIFlowLearn](https://aiflowlearn.net) 上练习 AI 编码工作流和模型路由模式。AIFlowLearn 提供浏览器里的互动课程、实战练习环境和技能包。

## 贡献

欢迎贡献！请查看 issues tab，或直接提交 PR。

## License

MIT License — Copyright (c) 2026 AIFlowLearn

---

*Sponsored by [AIFlowLearn](https://aiflowlearn.net) — AI-native learning platform*

**[AIFlowLearn / AI智流学社](https://aiflowlearn.net)** — 浏览器里的 AI 编码练习环境、互动课程与技能包。

[Zero2Claude](https://zero2claude.aiflowlearn.net/) | [Zero2Codex](https://zero2codex.aiflowlearn.net/) | [All Skill Packs](https://aiflowlearn.net/skillpacks)
