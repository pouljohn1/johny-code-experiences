---
title: Next steps and deploying your Next.js app
description: Wrap up, publish to Vercel, add a real database, and ideas for improvements
---

You now have a full‑stack Next.js app with:
- Server Components, Server Actions, and cached data (`use cache` + tags)
- A persistent chat list (file‑based storage)
- A working AI chat backed by the AI SDK and OpenAI

This chapter shows how to ship it and where to take it next.

## Deploying to Vercel

Vercel is the easiest way to host Next.js.

### 1) Prepare your repo
- Commit your code to a Git provider (GitHub, GitLab, Bitbucket).
- Ensure `.env.local` is in `.gitignore` (never commit secrets).

### 2) Add environment variables
In Vercel, go to Settings → Environment Variables and add:
- OPENAI_API_KEY: your OpenAI key

You can also add variables via the CLI:
```bash
vercel env add OPENAI_API_KEY
```

Locally, prefer `.env.local`:
```bash
OPENAI_API_KEY="sk-..."
```

### 3) Import and deploy
- Visit `https://vercel.com/new`
- Import your repository
- Framework preset: Next.js (auto)
- Click Deploy

Vercel will detect your app and build it. Subsequent pushes to your default branch auto‑deploy.

## Move from file storage to a database

File storage is great to start, but you’ll want a real database for reliability and scale. Popular options:
- Vercel Postgres (managed, serverless)
- Neon or Supabase (Postgres)
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

2) Set `DATABASE_URL` in `.env` (and in Vercel → Environment Variables)
```bash
DATABASE_URL="postgresql://user:password@host:5432/db?sslmode=require"
```

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

5) Update `db/chat.ts` to read/write via Prisma instead of file utils. Keep your cache tags (`cacheTag`/`updateTag`) so UI revalidations continue to work.

### Persisting messages
Add a `Message` model with `chatId`, `role`, `content`, and timestamps. Store/send the history in your API route so the AI has context after refreshes.

## Production hardening checklist

- Error handling: return helpful errors from API routes; guard empty inputs
- Rate limiting: protect your AI endpoint (e.g., Upstash Ratelimit)
- Secrets: never expose `OPENAI_API_KEY` to the client; use server routes/actions
- Logging/monitoring: Vercel Analytics, Sentry, or OpenTelemetry
- Performance: use cached components where possible; invalidate precisely with tags
- Accessibility: labels, focus management, keyboard navigation
- Testing: 
  - Unit: Vitest/Jest for utilities and DB logic
  - E2E: Playwright/Cypress for core chat flows

## Ideas to extend the app

- Multiple models and providers (OpenAI, Azure OpenAI, Anthropic)
- Chat title auto‑generation based on the first message
- Message editing / regeneration
- Delete chats and confirm dialogs
- Streaming UI polish (typing indicator, markdown rendering)
- Authentication (Auth.js, Clerk) to protect user data

<Woz
title="Where should I go next?"
description="Woz can suggest a tailored path based on your goals."
context={`User finished the core Next.js app and wants to advance. Offer a personalized plan:
1) Deploy to production (Vercel) and set OPENAI_API_KEY
2) Switch to a real database (Vercel Postgres or Neon) with Prisma
3) Persist messages and improve the chat UI (markdown, copy, code blocks)
4) Add authentication and per-user data isolation
5) Add rate limiting and observability
6) Optional: multi-model support and system prompts library`}
prompt="Suggest my next 3 steps based on my goals."/>

## Wrap up

You’ve built a real, deployable product. Ship it to Vercel, move storage to a database when you’re ready, and iterate on UX, reliability, and cost controls. From here, you can scale features confidently while keeping your stack simple and modern.


