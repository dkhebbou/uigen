# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup        # First-time setup: install deps + prisma generate + migrate
npm run dev          # Start dev server with Turbopack on port 3000
npm run build        # Production build
npm run lint         # ESLint check
npm run test         # Run Vitest tests
npm run db:reset     # Reset database (destructive)
```

Requires a `.env` file with `ANTHROPIC_API_KEY` (falls back to mock responses without it) and `JWT_SECRET`.

## Communication

Communiquer avec l'utilisateur en français.

## Code Style

Use comments sparingly — only for genuinely complex logic.

## Architecture

UIGen is an AI-powered React component generator. Users chat with Claude, which generates/edits files in a **virtual in-memory file system** (no disk I/O). Components are rendered live in an iframe via client-side Babel transpilation.

### Request Flow

```
Chat message → /api/chat (POST) → Claude + tools → VirtualFileSystem → FileSystemContext → Preview/Editor
```

The AI chat endpoint (`src/app/api/chat/route.ts`) receives messages and the current file system state, streams back Claude's response with tool calls that modify virtual files. On completion, the project is saved to SQLite via Prisma.

### Key Abstractions

- **VirtualFileSystem** (`src/lib/file-system.ts`) — In-memory file store with create/update/delete/rename and serialize/deserialize. No real disk writes.
- **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`) — React context wrapping VirtualFileSystem; processes tool calls from Claude; exposes `useFileSystem()` hook.
- **ChatContext** (`src/lib/contexts/chat-context.tsx`) — Manages message history and streaming state; exposes `useChat()` hook.
- **AI Tools** (`src/lib/tools/`) — `str_replace.ts` handles view/create/str_replace/insert operations; `file_manager.ts` handles rename/delete.
- **JSX Transformer** (`src/lib/transform/jsx-transformer.ts`) — Converts virtual files to runnable iframe HTML using `@babel/standalone` and import maps. Entry point is always `/App.jsx`.
- **Provider** (`src/lib/provider.ts`) — Selects Claude Haiku 4.5; falls back to `MockLanguageModel` if no API key.

### UI Layout (`src/app/main-content.tsx`)

Three-panel resizable interface:
- **Left (35%):** Chat interface
- **Right (65%):** Toggle between:
  - **Preview:** iframe rendering the generated component
  - **Code:** FileTree (40%) + Monaco editor (60%)

### Pages

- `src/app/page.tsx` — Home; redirects authenticated users to their latest project, shows anonymous users the main UI.
- `src/app/[projectId]/page.tsx` — Protected project page; loads project from DB and passes serialized data to main content.

### Database (Prisma + SQLite)

Schema: `User` (id, email, bcrypt password) → `Project` (id, name, userId, messages JSON, data JSON). Projects store the full message history and serialized VirtualFileSystem as JSON blobs. Auth uses JWT sessions via `jose`.

### System Prompt

`src/lib/prompts/generation.tsx` instructs Claude to:
- Always create `/App.jsx` as the entry point
- Use Tailwind CSS (not inline styles)
- Use `@/` alias for all inter-file imports within the virtual FS
- Operate through `str_replace_editor` and `file_manager` tools
