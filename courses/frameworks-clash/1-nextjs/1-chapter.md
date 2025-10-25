---
title: Configuration
description: We will start with creating and configuring new Next.js project
---

# Setting Up Your Next.js Project

In this chapter, we'll create a fresh Next.js project and explore what the framework gives us out of the box. By the end, you'll have a running Next.js application ready for building our AI chat app.

## Prerequisites

Before we begin, make sure you have a JavaScript package manager installed on your system. You can choose any of the following:

- **[Bun](https://bun.sh/docs/installation)** - Fast all-in-one JavaScript runtime (what we'll use in this course)
- **[Node.js](https://nodejs.org/)** - Comes with npm pre-installed
- **[pnpm](https://pnpm.io/installation)** - Fast, disk space efficient package manager
- **[Yarn](https://yarnpkg.com/getting-started/install)** - Alternative package manager

To verify your installation, run one of these commands in your terminal:

```bash
# For Bun
bun --version

# For npm (comes with Node.js)
npm --version

# For pnpm
pnpm --version

# For Yarn
yarn --version
```

If the command returns a version number, you're good to go! If not, click on the links above to install your preferred package manager.

## Creating the Project

Let's create our Next.js application. Open your terminal and run:

```bash
bun create next-app@latest nextjs-chat --yes
cd nextjs-chat
bun dev
```

Let's break down what each command does:

1. **`bun create next-app@latest nextjs-chat --yes`**
   - `create next-app@latest` - Uses the latest Next.js template
   - `nextjs-chat` - Names our project directory
   - `--yes` - Automatically accepts all default options

2. **`cd nextjs-chat`**
   - Changes into our newly created project directory

3. **`bun dev`**
   - Starts the development server
   - Your app will be available at `http://localhost:3000`

> **About Bun**: In this course, we're using Bun as our package manager because it's fast and modern. However, you're not limited to Bun - feel free to use npm, yarn, or pnpm instead. Just replace `bun` with your preferred package manager in all commands throughout this course.

> **Note**: The `--yes` flag accepts the following defaults:
> - TypeScript: Yes
> - ESLint: Yes
> - Tailwind CSS: Yes
> - `src/` directory: No
> - App Router: Yes
> - Import alias: `@/*`

<Woz
title="Something is not working?"
description="Ask our almighty woz if you need any help."
context="User seems to have a problem with creation of Next.js app in Creating the Project section. Please help him."
prompt=""
placeholder="Write what's your problem here..."/>

## What Has Been Created?

After running the create command, you'll see a new `nextjs-chat` directory with the following structure:

```
nextjs-chat/
├── app/
│   ├── favicon.ico
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── node_modules/
├── public/
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── .gitignore
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── README.md
└── tsconfig.json
```

### Understanding the File Structure

Let's explore the key files and directories:

#### `app/` Directory
This is the heart of your Next.js application using the **App Router** (introduced in Next.js 13+).

- **`layout.tsx`** - The root layout that wraps all pages. This is where you define shared UI elements like headers, footers, or providers that persist across navigation.
  
- **`page.tsx`** - The home page of your application (maps to `/` route). Each `page.tsx` file in the `app` directory creates a new route.

- **`globals.css`** - Global styles applied across your entire application. By default, it includes Tailwind CSS directives.

#### `public/` Directory
Static assets that are served directly. Files in this directory can be referenced from the root URL. For example, `public/logo.png` would be accessible at `/logo.png`.

#### Configuration Files

- **`next.config.ts`** - Next.js configuration file. This is where you customize webpack, add environment variables, configure domains for images, and more.

- **`package.json`** - Lists your project dependencies and scripts. You'll see:
  ```json
  {
    "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "eslint"
    }
  }
  ```

- **`tsconfig.json`** - TypeScript configuration. Next.js comes with sensible defaults pre-configured.

- **`eslint.config.mjs`** - ESLint configuration for code quality and consistency.

#### `node_modules/`
Contains all installed dependencies. This directory is generated automatically and should never be edited manually.

## Verifying the Installation

If the dev server started successfully, open your browser and navigate to:

```
http://localhost:3000
```

You should see the default Next.js welcome page.

## Adding a Component Library

To make development faster and create a better user experience, we'll use **shadcn/ui** - a collection of accessible and customizable components. Unlike traditional libraries, shadcn/ui copies component code directly into your project, giving you full control to modify them as needed.

In your terminal, run:

```bash
bunx --bun shadcn@latest init
```

You'll be prompted with configuration questions. Accept the defaults or choose your preferences for style and colors. The CLI will automatically detect your project setup and configure everything for you.

After initialization, you'll have two new files: `components.json` (shadcn configuration) and `lib/utils.ts` (UI utilities we'll use later).

## Enabling Cached Components

Next.js 16+ introduced an `cacheComponents` feature that improves performance by caching component rendering results.

Open `next.config.ts` and update it to enable this feature:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  cacheComponents: true
};

export default nextConfig;
```

This configuration will allow us to use cached components throughout our application.

## Next Steps

Now that you have a running Next.js application, you're ready to start building! In the next chapter, we'll:
- Clean up the default boilerplate
- Set up our project structure for the AI chat application
- Create our first custom components

Keep your dev server running (`bun dev`) as we'll be making changes and seeing them update in real-time.

---

**Troubleshooting:**
- If port 3000 is already in use, Next.js will automatically try port 3001, 3002, etc.
- If you encounter any errors, make sure you have Bun (or Node.js 18.17+) installed
- Check that you're in the correct directory before running `bun dev`
