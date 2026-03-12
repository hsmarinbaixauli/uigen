# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup      # First-time setup: install deps, generate Prisma client, run migrations
npm run dev        # Dev server with Turbopack at http://localhost:3000
npm run build      # Production build
npm run lint       # ESLint check
npm run test       # Run all Vitest tests
npm run db:reset   # Reset the SQLite database
```

To run a single test file:
```bash
npx vitest run src/lib/__tests__/file-system.test.ts
```

## Architecture

This is an AI-powered React component generator. Users describe a component in chat and the AI writes/edits code in a **virtual file system** (in-memory, no disk writes) with live preview.

### Request flow

1. User submits a message → `POST /api/chat` (`src/app/api/chat/route.ts`)
2. The route calls the language model (Anthropic Claude or `MockLanguageModel` fallback) with a system prompt from `src/lib/prompts/`
3. The model calls tools: `str_replace_editor` (file edits) and `file_manager` (create/delete/list files) defined in `src/lib/tools/`
4. Tool results stream back to the client via Vercel AI SDK
5. The frontend `ChatProvider` (`src/lib/contexts/chat-context.tsx`) processes streamed tool calls and updates the `FileSystemProvider` (`src/lib/contexts/file-system-context.tsx`)
6. `PreviewFrame` renders the virtual files live using a JSX transformer (`src/lib/transform/`)

### Key abstractions

- **`VirtualFileSystem`** (`src/lib/file-system.ts`): In-memory file store. The entire component workspace lives here.
- **`FileSystemProvider`** / **`ChatProvider`**: React contexts that are the source of truth for file state and chat history. `MainContent` (`src/app/main-content.tsx`) wires them together with resizable panels.
- **LLM provider** (`src/lib/provider.ts`): Returns Anthropic model if `ANTHROPIC_API_KEY` is set in `.env`, otherwise returns `MockLanguageModel` for demo/testing.
- **Auth** (`src/lib/auth.ts`): JWT-based sessions stored in HTTP-only cookies. Prisma/SQLite stores users and projects.

### Database

Prisma with SQLite (`prisma/dev.db`). Two models: `User` and `Project`. Projects store chat messages and file data as JSON blobs. Generated client is output to `src/generated/prisma`.

### Path alias

`@/*` maps to `src/*` throughout the project.
