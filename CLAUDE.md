# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zap-GPT 是一个 WhatsApp 聊天机器人，使用 wppconnect 连接 WhatsApp，并集成 OpenAI GPT (Assistants API) 或 Google Gemini 作为 AI 后端。

## Commands

```bash
# 开发模式（热重载）
bun run dev

# 构建项目
bun run build

# 生产环境运行（使用 pm2）
bun run start

# 停止生产服务
bun run stop

# 配置环境变量（交互式）
bun run config

# 代码检查
bun run lint

# 代码检查并自动修复
bun run lint:fix
```

## Environment Variables

复制 `.env.example` 到 `.env` 并配置:

- `AI_SELECTED`: 选择 AI 后端 (`GPT` 或 `GEMINI`)
- GPT 模式需要: `OPENAI_KEY`, `OPENAI_ASSISTANT`
- Gemini 模式需要: `GEMINI_KEY`, `GEMINI_PROMPT` (可选)

## Architecture

```
src/
├── index.ts           # 入口点，wppconnect 初始化，消息路由
├── service/
│   ├── openai.ts      # OpenAI Assistants API 集成（线程管理）
│   └── google.ts      # Google Gemini API 集成（会话历史管理）
└── util/index.ts      # 消息分割和延迟发送工具
```

**核心流程**:
1. `index.ts` 通过 wppconnect 监听 WhatsApp 消息
2. 消息缓冲 15 秒后合并发送给 AI（避免频繁请求）
3. 根据 `AI_SELECTED` 调用相应 service
4. AI 响应通过 `splitMessages` 拆分为多条消息
5. 使用 `sendMessagesWithDelay` 模拟打字延迟发送

**AI 服务特点**:
- OpenAI: 使用 Assistants API 的 Thread 机制维持会话
- Gemini: 使用内存 Map 存储会话历史

## Tech Stack

- Bun runtime
- TypeScript (strict mode)
- wppconnect: WhatsApp 连接
- Biome: 代码规范（linting + formatting）
- tsup: 构建工具
- pm2: 生产环境进程管理
