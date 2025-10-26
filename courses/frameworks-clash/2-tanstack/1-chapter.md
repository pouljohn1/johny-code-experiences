---
title: Configuration
description: We will start with creating and configuring new Tanstack Start project
---

In this chapter, we'll create a fresh TanStack Start project and explore what the framework gives us out of the box. By the end, you'll have a running TanStack Start application ready for building our AI chat app.

> **Need Help?**
> 
> If you encounter any difficulties, you can always reference the complete code from this GitHub repository:
>
> [https://github.com/pouljohn1/tanstack-chat](https://github.com/pouljohn1/tanstack-chat)

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

Let's create our TanStack Start application. Open your terminal and run:

```bash
bun create @tanstack/start@latest
```

> **Prefered options**: After starting that command you will be asked on what features to use. We recommend following config
> - Name: tanstack-chat
> - Tailwind CSS: Yes
> - Toolchain: Biome
> - Add-ons: Shadcn
> - Example: No


Now do

```bash
cd tanstack-chat
bun --bun run dev
```
Let's break down what each command does:

> **About Bun**: In this course, we're using Bun as our package manager because it's fast and modern. However, you're not limited to Bun - feel free to use npm, yarn, or pnpm instead. Just replace `bun` with your preferred package manager in all commands throughout this course.

<Woz
title="Something is not working?"
description="Ask our almighty woz if you need any help."
context="User seems to have a problem with creation of TanStack Start app in Creating the Project section. Please help him."
prompt=""
placeholder="Write what's your problem here..."/>

## What Has Been Created?

After running the create command, you'll see a new `tanstack-chat` directory with the following structure:

```
nextjs-chat/
├── node_modules/
├── public/
│   ├── favicon.ico
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   ├── robots.txt
│   ├── tanstack-circle-logo.png
│   └── tanstack-word-logo-white.svg
├── src/
│   ├── components/
│   ├── data/
│   ├── lib/
│   ├── routes/
│   └── logo.svg
│   └── router.tsx
│   └── styles.css
├── .cta.json
├── .cursorrules
├── .gitignore
├── biome.json
├── components.json
├── package.json
├── README.md
└── tsconfig.json
└── vite.config.ts
```

### Understanding the File Structure

Let's explore the key files and directories:

#### `src/routes/` Directory
This is the heart of your TanStack Start application using **file-based routing**.

- **`__root.tsx`** - The root route component that wraps all pages. This is where you define shared UI elements like headers, footers, or providers that persist across navigation. It also includes the `<Outlet />` component where child routes are rendered.
  
- **`index.tsx`** - The home page of your application (maps to `/` route). Each `.tsx` file in the `routes` directory creates a new route based on the file name.

#### `src/` Directory Structure

- **`router.tsx`** - The router configuration file that defines your application's routing setup and connects all your routes together.

- **`styles.css`** - Global styles applied across your entire application. By default, it includes Tailwind CSS directives.

- **`components/`** - Directory for your reusable React components (including shadcn/ui components if you selected that option).

- **`lib/`** - Utility functions and helpers. Contains `utils.ts` for component utilities when using shadcn/ui.

- **`data/`** - Directory for data fetching logic and API utilities.

#### `public/` Directory
Static assets that are served directly. Files in this directory can be referenced from the root URL. For example, `public/logo.png` would be accessible at `/logo.png`.

#### Configuration Files

- **`vite.config.ts`** - Vite configuration file. TanStack Start is built on top of Vite, and this is where you can customize the build process, add plugins, configure environment variables, and more.

- **`package.json`** - Lists your project dependencies and scripts. You'll see:
  ```json
  {
    "scripts": {
      "dev": "vite dev --port 3000",
      "build": "vite build",
      "serve": "vite preview",
      "test": "vitest run",
      "format": "biome format",
      "lint": "biome lint",
      "check": "biome check"
    }
  }
  ```

- **`tsconfig.json`** - TypeScript configuration. TanStack Start comes with sensible defaults pre-configured.

- **`biome.json`** - Biome configuration for code formatting and linting (if you selected Biome as your toolchain).

- **`components.json`** - shadcn/ui configuration file (if you selected shadcn as an add-on during setup).

#### `node_modules/`
Contains all installed dependencies. This directory is generated automatically and should never be edited manually.

## Verifying the Installation

If the dev server started successfully, open your browser and navigate to:

```
http://localhost:3000
```

You should see the default TanStack Start welcome page with the TanStack logo and starter content.

## shadcn/ui Setup

If you selected **shadcn** as an add-on during project creation, it's already configured and ready to use! The `components.json` file contains the configuration, and `src/lib/utils.ts` has the utility functions for component styling.

If you didn't select shadcn during setup but want to add it now, run:

```bash
bunx --bun shadcn@latest init
```

The CLI will detect your TanStack Start setup and configure everything automatically.

## Understanding TanStack Start's Architecture

TanStack Start is built on top of **TanStack Router** and **Vinxi**, which gives you:

- **File-based routing** - Routes are automatically created based on your file structure in `src/routes/`
- **Type-safe routing** - Full TypeScript support with auto-generated route types
- **Server functions** - Write server-side code directly in your components using `createServerFn`
- **Streaming SSR** - Built-in support for React Server Components and streaming

The development server uses Vinxi (a Vite-based meta-framework) that handles both client and server bundles seamlessly.

## Next Steps

Now that you have a running TanStack Start application, you're ready to start building! In the next chapter, we'll:
- Clean up the default boilerplate
- Create a chat list component
- Learn how to use server functions with TanStack Start
- Build our database and action layer

Keep your dev server running (`bun --bun run dev`) as we'll be making changes and seeing them update in real-time.

---

**Troubleshooting:**
- If port 3000 is already in use, TanStack Start will automatically try port 3001, 3002, etc.
- If you encounter any errors, make sure you have Bun (or Node.js 18.17+) installed
- Check that you're in the correct directory before running `bun --bun run dev`
- Make sure to use `bun --bun run dev` (not just `bun dev`) to ensure Bun's runtime is used
