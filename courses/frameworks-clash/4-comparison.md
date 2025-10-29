---
title: Comparison
---

### What we're comparing

- **Next.js** (App Router with Server Components, Server Actions, caching directives)
- **TanStack Start** (TanStack Router + isomorphic-by-default model + server functions)
- **React SPA** with **Vite + Hono** (separate frontend/backend; traditional SPA without SSR)

This page summarizes trade-offs, pulls context from the previous Next.js and TanStack modules, and adds guidance for a classic SPA (you’ll write that module later).

### TL;DR — when to choose what

| Choose this | When it fits best |
|---|---|
| **Next.js** | You want a widely adopted standard, great docs, first-class SSR/SSG/ISR/streaming, and don’t mind some magic and Vercel-oriented defaults. |
| **TanStack Start** | You want explicit, strongly typed, super-composable React with minimal magic, great DX, and easy multi-provider deployment. |
| **React SPA (Vite + Hono)** | You don’t need SEO/SSR, want strict frontend-backend separation, maximum freedom in libraries and infra, or plan multiple clients (web, mobile). |

### Side-by-side comparison

| Dimension | Next.js | TanStack Start | React SPA (Vite + Hono) |
|---|---|---|---|
| **Maturity & adoption** | High; industry standard in many companies | Newer; stable but less adopted | High as an approach; parts mature but you assemble them |
| **Community & docs** | Big community; great docs | Smaller community; weaker docs | Large but fragmented (depends on chosen libs) |
| **Learning curve** | Moderate; concepts evolve over time | Gentle; explicit primitives are easy to reason about | Varies; you own more setup and decisions |
| **“Magic” level** | Some magic (directives, conventions) | No magic; explicit and typed | No magic; everything you wire yourself |
| **Type-safety** | Strong TS support | Strong and pervasive; excellent types | Strong if you choose typed libs end-to-end |
| **Routing** | File-based App Router | File-based TanStack Router | Your choice (TanStack Router, React Router, etc.) |
| **Data fetching model** | Server Components, Server Actions, caching directives | Isomorphic by default with explicit server functions and loaders | Your choice (client-side fetch, BFF via Hono, tRPC, etc.) |
| **SSR/SSG/ISR/Streaming** | First-class and battle-tested | Supported; more explicit primitives | Typically SPA only (no SSR); can add SSR but non-trivial |
| **SEO** | Excellent (SSR/SSG) | Good (SSR available) | Weak by default in SPA; acceptable if SEO is not critical |
| **DX & composability** | Productive but some framework constraints | Very composable; easy to test | Highly flexible; your DX depends on your stack |
| **Testing** | Good; some framework specifics | Easy; fewer directives, plain React | Straightforward; you design the testing story |
| **Ecosystem integration** | Huge; many examples and libs target Next | Growing; great primitives, smaller ecosystem | Unlimited but DIY integration |
| **Deployment** | Best on Vercel; portability via adapters is improving | Easy across providers | Easiest anywhere; split FE/BE deploys |
| **Vendor lock-in risk** | Moderate (historically Vercel-oriented) | Low | Very low |
| **Operating cost control** | Good; best with Vercel features | Good; portable | Excellent; total control |
| **Multi-frontend readiness** | Fine, but full-stack coupling exists | Fine; explicit boundaries | Excellent; clean FE/BE separation |
| **Setup/config effort** | Low to medium | Medium | High (you configure more) |
| **Time-to-first-feature** | Fast | Fast | Slower initially |
| **Edge/Workers** | Great with the Next runtime (excellent on Vercel) | Good and portable | Great; Hono excels on edge |

### Pros and cons (grounded in your notes)

#### Next.js
- **Pros**
  - Standard used by many big companies
  - Big community and great documentation
  - Many built-ins that make static and dynamic apps easier
  - First-class SSR/SSG/ISR/streaming and caching primitives
- **Cons**
  - "Magic" via directives (`"use server"`, `"use client"`, `"use cache"`) and evolving patterns over time can be confusing
  - Historically hard to use different infra than Vercel; the Build Adapters story is improving but not universal

#### TanStack Start
- **Pros**
  - Newer but stable; explicit and strongly typed
  - No magic; very easy to understand
  - Very composable; easy to test; pleasant DX
  - Easy to deploy on many providers
- **Cons**
  - Still maturing; smaller community and weaker docs
  - Fewer built-in features than Next.js

#### React SPA (Vite + Hono)
- **Pros**
  - Clear frontend-backend split; scales well across teams and future clients (e.g., mobile)
  - You pick the exact libraries you want; full freedom
  - Very easy to deploy anywhere (including your own servers via e.g., Coolify)
  - Pleasant to work with if you know what you’re doing
- **Cons**
  - Poor SEO by default (no SSR); fine if SEO isn’t required
  - More time spent on configuration and owning the platform pieces
  - You must handle a lot that full-stack frameworks give you out of the box

### Practical recommendations

- **Pick Next.js if:**
  - You want the safest, most common choice with strong community support
  - SEO, SSR/SSG/ISR, or streaming responses matter
  - You’re fine with some framework conventions/magic and possibly deploying on Vercel

- **Pick TanStack Start if:**
  - You value explicitness, strong typing, and a very composable mental model
  - You want easy testing and clear control over when code runs (server vs client)
  - You want portability across infra providers without vendor lock-in
  - You're comfortable adopting newer technologies

- **Pick React SPA (Vite + Hono) if:**
  - You don’t need SEO/SSR (internal tools, authenticated apps, dashboards)
  - You want a clean separation of concerns and plan multiple clients (web + native)
  - You prefer maximum infra flexibility and cost control and don’t mind doing more setup

### Scenario-based guidance

- **Public marketing site + app shell**: Next.js for SSG/ISR and SEO, with app routes for the product
- **Internal dashboard or authenticated-only app**: SPA or TanStack Start (if you prefer stronger typing and explicit server functions)
- **Product with a mobile app and multiple clients**: SPA (Vite FE + Hono BFF) for the cleanest separation
- **Complex data fetching with edge streaming**: Next.js (best-in-class streaming) or TanStack for explicit control
- **On-prem or multi-cloud requirements**: TanStack Start or SPA for the easiest portability
- **Content-heavy site (docs/blog) + occasional interactive routes**: Next.js SSG/ISR

### Final verdict

- **There’s no single “best” — your constraints decide.**
  - Choose **Next.js** when you want the industry-standard experience, top-tier SEO/SSR, and rapid delivery with strong community support.
  - Choose **TanStack Start** when you want explicit, typed, composable building blocks and easy portability without directives or hidden magic.
  - Choose a **React SPA (Vite + Hono)** when SEO is not needed and you want maximum control, separation, and infra flexibility.

> You’ll add a dedicated SPA module later. The guidance above assumes the classic SPA model without SSR; if you decide to add SSR or pre-rendering to a Vite-based stack later, revisit the SEO and complexity rows accordingly.