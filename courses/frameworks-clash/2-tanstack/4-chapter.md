---
title: Next steps
description: Wrap up, publish to Railway, add a real database, and ideas for improvements
---

You now have a full‑stack Tanstack Start app with:
- Server functions, server routes, and loaders for data fetching
- A persistent chat list (file‑based storage)
- A working AI chat backed by the AI SDK and OpenAI

This chapter shows how to ship it and where to take it next.

## Deploying to Railway

Railway is an excellent platform for hosting Tanstack Start applications. It provides simple deployment with automatic builds and easy environment variable management.

### 1) Prepare your repo
- Commit your code to a Git provider (GitHub, GitLab, Bitbucket).
- Ensure `.env` is in `.gitignore` (never commit secrets).

### 2) Sign up and create a project
- Go to https://railway.app and sign up with GitHub
- Click **New Project** → **Deploy from GitHub repo**
- Select your repository
- Railway will auto-detect your project settings

### 3) Add environment variables
In Railway dashboard:
- Click on your deployment
- Go to the **Variables** tab
- Add your environment variable:
  - `OPENAI_API_KEY`: your OpenAI key

Locally, use `.env`:
```bash
OPENAI_API_KEY="sk-..."
```

### 4) Configure build settings (if needed)
Railway should auto-detect Bun. If not, set these in **Settings**:
- Build Command: `bun install && bun run build`
- Start Command: `bun run start`

Railway will automatically build and deploy your app. Subsequent pushes to your default branch will trigger automatic deployments.

## Move from file storage to a database

File storage is great to start, but you'll want a real database for reliability and scale. Popular options:
- Railway Postgres (managed, included with your deployment)
- Neon or Supabase (Postgres)
- Turso (SQLite, edge-ready)
- SQLite for simple/self‑hosted flows

Two common ORMs:
- Prisma (schema‑driven, batteries included)
- Drizzle (lightweight, SQL‑first)

### Example (Prisma + Postgres)
1) Install and init
```bash
bun add -d prisma
bunx prisma init
```

2) Set `DATABASE_URL` in `.env` (and in Railway → Variables)
```bash
DATABASE_URL="postgresql://user:password@host:5432/db?sslmode=require"
```

> **Railway tip**: You can add a PostgreSQL database directly in Railway by clicking **New** → **Database** → **Add PostgreSQL**. Railway will automatically set the `DATABASE_URL` environment variable for you.

3) Define a basic schema in `prisma/schema.prisma`
```prisma
model Chat {
  id        Int      @id @default(autoincrement())
  title     String
  timestamp DateTime @default(now())
}
```

4) Create the DB and types
```bash
bunx prisma migrate dev --name init
bunx prisma generate
```

5) Update `src/db/chat.ts` to read/write via Prisma instead of file utils. You may need to invalidate the router cache using `router.invalidate()` to refresh data after mutations.

### Persisting messages
Add a `Message` model with `chatId`, `role`, `content`, and timestamps. Store/send the history in your server route so the AI has context after refreshes.

## Production hardening checklist

- **Error handling**: Return helpful errors from server routes; guard empty inputs with proper validation
- **Rate limiting**: Protect your AI endpoint (e.g., Upstash Ratelimit or Railway's rate limiting)
- **Secrets**: Never expose `OPENAI_API_KEY` to the client; keep it in server routes only
- **Logging/monitoring**: Railway provides built-in logs; integrate Sentry or OpenTelemetry for deeper insights
- **Performance**: Use loaders effectively; leverage Tanstack Router's built-in caching and preloading
- **Accessibility**: Labels, focus management, keyboard navigation
- **Testing**: 
  - Unit: Vitest for utilities and DB logic (Tanstack Start supports Vitest out of the box)
  - E2E: Playwright or Cypress for core chat flows

## Ideas to extend the app

- **Multiple models and providers**: OpenAI, Azure OpenAI, Anthropic, or local models
- **Chat title auto‑generation**: Update titles based on the first message using AI
- **Message editing / regeneration**: Allow users to edit their messages or regenerate AI responses
- **Delete chats**: Add delete functionality with confirmation dialogs
- **Streaming UI polish**: Add typing indicators, markdown rendering with syntax highlighting
- **Authentication**: Use Clerk or Auth.js to protect user data and enable multi-user support
- **File uploads**: Add support for image or document uploads in chat
- **Export conversations**: Allow users to download chat history as markdown or PDF

<Woz
title="Where should I go next?"
description="Woz can suggest a tailored path based on your goals."
context={`User finished the core Tanstack Start app and wants to advance. Offer a personalized plan:
1) Deploy to production (Railway) and set OPENAI_API_KEY in environment variables
2) Switch to a real database (Railway Postgres, Neon, or Turso) with Prisma or Drizzle
3) Persist messages in the database and improve the chat UI (markdown rendering, copy button, code block syntax highlighting)
4) Add authentication (Clerk or Auth.js) and implement per-user data isolation
5) Add rate limiting using Upstash or Railway's features and add observability/logging
6) Optional: Add multi-model support, system prompts library, and file upload capabilities`}
prompt="Suggest my next 3 steps based on my goals."/>

## Wrap up

You've built a real, deployable Tanstack Start application! Ship it to Railway, move storage to a database when you're ready, and iterate on UX, reliability, and performance. From here, you can scale features confidently while enjoying Tanstack Start's excellent developer experience and type-safe routing.


