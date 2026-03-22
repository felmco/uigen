# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components via chat, Claude generates code using tool calls (str_replace_editor, file_manager), and components render in a sandboxed iframe with Babel transpilation.

**Stack**: Next.js 15 (App Router), React 19, TypeScript 5, Tailwind CSS v4, Prisma (SQLite), Vercel AI SDK + Anthropic Claude, Vitest.

## Commands

```bash
npm run dev          # Dev server with Turbopack
npm run build        # Production build
npm run start        # Production server
npm run lint         # ESLint (Next.js config)
npm run test         # Vitest test suite
npm run setup        # Install deps + Prisma generate + migrations
npm run db:reset     # Reset database (destructive)
```

All commands require `NODE_OPTIONS='--require ./node-compat.cjs'` (set in package.json scripts).

## Architecture

### Core Flow
1. User sends message → `/api/chat` route streams Claude response
2. Claude calls tools (`str_replace_editor`, `file_manager`) to create/edit files
3. `FileSystemContext` applies tool calls to the in-memory `VirtualFileSystem`
4. Preview pipeline: Babel transpiles JSX → import map resolves deps via esm.sh → iframe renders

### Key Directories
- `src/app/` — Next.js pages and API routes. Main workspace at `/[projectId]`
- `src/components/chat/` — Chat UI (MessageList, MessageInput, ChatInterface)
- `src/components/editor/` — Code editor and file tree
- `src/components/preview/` — Iframe sandbox preview
- `src/components/ui/` — Shadcn/ui primitives (generated, New York style)
- `src/lib/contexts/` — ChatContext (AI conversation), FileSystemContext (virtual FS)
- `src/lib/tools/` — AI tool definitions (str-replace.ts, file-manager.ts)
- `src/lib/prompts/` — System prompts for Claude
- `src/lib/transform/` — Babel JSX transformation and import map generation
- `src/lib/file-system.ts` — VirtualFileSystem class (tree-based, in-memory)
- `src/actions/` — Server actions for auth and project persistence
- `prisma/schema.prisma` — User and Project models (SQLite)

### State Management
- **ChatContext**: Wraps Vercel AI SDK's `useChat`, manages messages and streaming status
- **FileSystemContext**: Manages VirtualFileSystem instance, processes AI tool calls, triggers preview refresh

### Authentication
JWT sessions (7-day expiry) stored in `auth-token` httpOnly cookie. Middleware verifies on protected routes. Passwords hashed with bcrypt.

### LLM Integration
- Provider in `src/lib/provider.ts` — uses Anthropic Claude or mock provider if no API key
- Two AI tools: `str_replace_editor` (create/view/replace/insert) and `file_manager` (rename/delete)
- Entry point file is always `/App.jsx`

## Conventions

- **Imports**: Always use `@/` alias (mapped to `src/`), never relative paths
- **Components**: PascalCase filenames matching export names
- **Server/client boundary**: `"use server"` in `/actions`, `"use client"` in interactive components
- **Styling**: Tailwind classes + `cn()` utility from `@/lib/utils` for conditional classes
- **Tests**: Colocated in `__tests__/` directories, use Testing Library + vi.mock for contexts
- **Prisma**: Singleton client in `src/lib/prisma.ts`, generated types in `src/generated/prisma` (don't edit). Reference `prisma/schema.prisma` to understand the database structure
- **Comments**: Use sparingly — only comment complex code
