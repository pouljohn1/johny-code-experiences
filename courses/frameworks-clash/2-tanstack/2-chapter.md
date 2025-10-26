---
title: Creating a chat list
description: We will create a UI for the chat list component
---

In this chapter, we'll redesign our layout to include a sidebar with a chat list on the left and chat content as the main area. For now, the main content will remain empty as we focus on building the chat list functionality.

> **Note** 
>
> Some parts of this chapter are similar to or the same as the previous Next.js module

## Prerequisites

First, we need to install some shadcn components. Run this command in your terminal:

```bash
bunx --bun shadcn@latest add sidebar button skeleton
```

If the CLI doesn't install icons automatically, run `bun add lucide-react`.

This creates new components in the `components/ui` folder. Feel free to explore them to see how they're built.

## Cleaning Up the Default Content

Now we need to remove Tanstack Start example web page content. First let's move to `src/routes/index.tsx` and remove everything and leave just simple div component:

```typescript
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/")({ component: App });

function App() {
	return <div className="bg-white dark:bg-black"></div>;
}
```

`index.tsx` is a main web page of the tanstack app.

## Configuring the Layout

Now let's configure the main layout. In Next.js there was a `layout.tsx` file that contained the layout of the whole app. 

In the TanStack Start app we have it in the `src/routes/__root.tsx` file. When you open it you will see that it exports one property called `Route` which is being created by calling `createRootRoute`. `createRootRoute` has a `shellComponent` property that is basically an equivalent to `layout.tsx` in Next.js. 

Let's change our RootDocument to match what we had in the Next.js app. Replace the whole content with:

```typescript fileName=src/routes/__root.tsx
import { TanStackDevtools } from "@tanstack/react-devtools";
import { createRootRoute, HeadContent, Scripts } from "@tanstack/react-router";
import { TanStackRouterDevtoolsPanel } from "@tanstack/react-router-devtools";
import ChatSidebar from "@/components/ChatSidebar";
import { SidebarProvider } from "@/components/ui/sidebar";
import appCss from "../styles.css?url";

export const Route = createRootRoute({
	head: () => ({
		meta: [
			{
				charSet: "utf-8",
			},
			{
				name: "viewport",
				content: "width=device-width, initial-scale=1",
			},
			{
				title: "TanStack Start Starter",
			},
		],
		links: [
			{
				rel: "stylesheet",
				href: appCss,
			},
		],
	}),

	shellComponent: RootDocument,
});

function RootDocument({ children }: { children: React.ReactNode }) {
	return (
		<html lang="en">
			<head>
				<HeadContent />
			</head>
			<body>
				<SidebarProvider>
					<ChatSidebar />
					<main className="flex-1 overflow-auto bg-white dark:bg-black">
						{children}
					</main>
				</SidebarProvider>
				<TanStackDevtools
					config={{
						position: "bottom-right",
					}}
					plugins={[
						{
							name: "Tanstack Router",
							render: <TanStackRouterDevtoolsPanel />,
						},
					]}
				/>
				<Scripts />
			</body>
		</html>
	);
}
```

You'll see an error about a missing `@/components/ChatSidebar` file. Let's create it!

Create a new file `ChatSidebar.tsx` in the `src/components/` folder with the following content:

```typescript fileName=src/components/ChatSidebar.tsx
import { Sidebar, SidebarContent } from "./ui/sidebar";

export default function ChatSidebar() {
	return (
		<Sidebar>
			<SidebarContent className="flex flex-col gap-4 p-4">
				Hello Sidebar
			</SidebarContent>
		</Sidebar>
	);
}
```

Great! Now we have the backbone of our application. In your browser, you should see something like this:

![Basic Layout](/assets/tanstack-2-layout.png)

## Creating the Chat Storage

Next, we'll create the code for storing chats. First, we need to define a `Chat` type. Create a new folder and file at `src/types/chat.ts` with this content:

```typescript fileName=src/types/chat.ts
export type Chat = {
  id: number;
  title: string;
  timestamp: string;
};
```

Next, we'll add a simple in-memory storage with database functions. Create `src/db/chat.ts` with the following content:

```typescript fileName=src/db/chat.ts
import type { Chat } from "@/types/chat";

const chats: Chat[] = [];

export async function getChats() {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return chats;
}

export async function createChat(title: string) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  const newChat: Chat = { id: chats.length + 1, title, timestamp: new Date().toISOString() }
  chats.push(newChat);
  return newChat;
}
```

Contents of the file seems to be obvious - we've created two functions here:

- **`getChats`** - Retrieves chats from the local `chats` array.
- **`createChat`** - Creates a new chat. 


<Woz
title="Any questions?"
description="Do you have any issues or questions so far? You can always ask Woz!"
placeholder="Why is it so complicated, Woz?"/>

## Creating Server Functions

Now we have database-related code, let's add some Server Functions (remember that Next.js had also server functions?). Differently from Next.js we will create a folder `/src/api` and will create `chat.ts` file in it with a content:

```typescript fileName=src/api/chat.ts
import { createServerFn } from "@tanstack/react-start";
import { createChat, getChats } from "@/db/chat";

export const addChatFn = createServerFn({ method: "POST" })
	.inputValidator((title: string) => title)
	.handler(async ({ data }) => {
		return await createChat(data);
	});

export const getChatsFn = createServerFn({ method: "GET" }).handler(
	async () => {
		return await getChats();
	},
);
```

The first thing you'll notice is the `createServerFn` method. This is similar to Next.js's `use server` directive but written in a more functional style with the ability to specify which HTTP method it should use.

Also, we've created two Server Functions instead of one as it was in Next.js. `getChatsFn` also needs to be created and we cannot use `getChats` directly like we could with Next.js Server Components.

## Fetching Data

Now we have all the infrastructure needed to connect it to our components. Go to `src/routes/__root.tsx` and add `getChatsFn` as follows:

```typescript add={4,32,36-37,48} remove={47} hide={29-40,45-51} fileName=src/routes/__root.tsx
import { TanStackDevtools } from "@tanstack/react-devtools";
import { createRootRoute, HeadContent, Scripts } from "@tanstack/react-router";
import { TanStackRouterDevtoolsPanel } from "@tanstack/react-router-devtools";
import { getChatsFn } from "@/api/chat";
import ChatSidebar from "@/components/ChatSidebar";
import { SidebarProvider } from "@/components/ui/sidebar";
import appCss from "../styles.css?url";

export const Route = createRootRoute({
	head: () => ({
		meta: [
			{
				charSet: "utf-8",
			},
			{
				name: "viewport",
				content: "width=device-width, initial-scale=1",
			},
			{
				title: "TanStack Start Starter",
			},
		],
		links: [
			{
				rel: "stylesheet",
				href: appCss,
			},
		],
	}),

	shellComponent: RootDocument,
	loader: async () => await getChatsFn(),
});

function RootDocument({ children }: { children: React.ReactNode }) {
	const chats = Route.useLoaderData();

	return (
		<html lang="en">
			<head>
				<HeadContent />
			</head>
			<body>
				<SidebarProvider>
        	<ChatSidebar />
					<ChatSidebar chats={chats} />
					<main className="flex-1 overflow-auto bg-white dark:bg-black">
						{children}
					</main>
				</SidebarProvider>
				<TanStackDevtools
					config={{
						position: "bottom-right",
					}}
					plugins={[
						{
							name: "Tanstack Router",
							render: <TanStackRouterDevtoolsPanel />,
						},
					]}
				/>
				<Scripts />
			</body>
		</html>
	);
}
```

As you can see, we made several changes:
- added `loader` prop to `createRootRoute` function. `loader` is an async function where we need to fetch all backend related data. That data is then passed to component
- Added `Route.useLoaderData()` located in react component which get's previously fetched data in `loader`
- Passed the `chats` data to `ChatSidebar`

## Displaying Chats

Now let's update `ChatSidebar.tsx` to display the chats:

```typescript add={1,3-5,8,13-45} remove={7,12} fileName=/src/components/ChatSidebar.tsx
import { Chat } from "@/types/chat";
import { Sidebar, SidebarContent } from "./ui/sidebar";
import { Link } from "@tanstack/react-router";
import { MessageSquare } from "lucide-react";
import { Button } from "./ui/button";

export default function ChatSidebar() {
export default function ChatSidebar({ chats }: { chats: Chat[] }) {
	return (
		<Sidebar>
			<SidebarContent className="flex flex-col gap-4 p-4">
        Hello Sidebar
				<div className="flex flex-col gap-2">
					<h2 className="text-sm font-semibold text-muted-foreground px-2">
						Recent Chats
					</h2>
					{chats.length > 0 ? (
						<div className="flex flex-col gap-1">
							{chats.map((chat) => (
								<Link
									key={chat.id}
									className="flex items-start gap-3 rounded-lg px-3 py-2.5 text-left hover:bg-accent transition-colors"
                  to={`/`}
								>
									<MessageSquare className="size-4 mt-0.5 shrink-0 text-muted-foreground" />
									<div className="flex flex-col gap-0.5 min-w-0 flex-1">
										<span className="text-sm font-medium truncate">
											{chat.title}
										</span>
										<span className="text-xs text-muted-foreground">
											{chat.timestamp}
										</span>
									</div>
								</Link>
							))}
						</div>
					) : (
						<div className="flex flex-col items-center justify-center py-8 px-4 text-center">
							<MessageSquare className="size-8 text-muted-foreground/50 mb-2" />
							<p className="text-sm text-muted-foreground">
								No chats yet. Start a new conversation!
							</p>
						</div>
					)}
				</div>
			</SidebarContent>
		</Sidebar>
	);
}
```

What we've done here is iterate over the received chats and create an item for each one. If there are no chats, we display a "no chats" message.

You should see something like this:

![No chats image](/assets/tanstack-2-empty.png)

## Creating a New Chat

Now let's add the ability to create new chats! We will create a section at the start of the `<SidebarContent>` component with an add button and will make use of server functions to add new chats: 

```typescript add={4-8,10,13-19,23-35} remove={1-3} wrap={37-100} fileName=/src/components/ChatSidebar.tsx
import { Chat } from "@/types/chat";
import { Sidebar, SidebarContent } from "./ui/sidebar";
import { MessageSquare } from "lucide-react";
import { Link, useRouter } from "@tanstack/react-router";
import { MessageSquare, Plus } from "lucide-react";
import { useCallback } from "react";
import { addChatFn } from "@/api/chat";
import type { Chat } from "@/types/chat";
import { Button } from "./ui/button";
import { Sidebar, SidebarContent } from "./ui/sidebar";

export default function ChatSidebar({ chats }: { chats: Chat[] }) {
	const router = useRouter();

	const submitChat = async () => {
		await addChatFn({ data: "New Chat" });
		router.invalidate();
	};

	return (
		<Sidebar>
			<SidebarContent className="flex flex-col gap-4 p-4">
				<div className="flex flex-col gap-3">
					<h1 className="text-xl font-bold">Chat App</h1>

					<Button
						className="w-full justify-start gap-2"
						variant="outline"
						type="submit"
						onClick={submitChat}
					>
						<Plus className="size-4" />
						New Chat
					</Button>
				</div>
				<div className="flex flex-col gap-2">
					<h2 className="text-sm font-semibold text-muted-foreground px-2">
						Recent Chats
					</h2>
					{chats.length > 0 ? (
						<div className="flex flex-col gap-1">
							{chats.map((chat) => (
								<Link
									key={chat.id}
									className="flex items-start gap-3 rounded-lg px-3 py-2.5 text-left hover:bg-accent transition-colors"
									to={`/`}
								>
									<MessageSquare className="size-4 mt-0.5 shrink-0 text-muted-foreground" />
									<div className="flex flex-col gap-0.5 min-w-0 flex-1">
										<span className="text-sm font-medium truncate">
											{chat.title}
										</span>
										<span className="text-xs text-muted-foreground">
											{chat.timestamp}
										</span>
									</div>
								</Link>
							))}
						</div>
					) : (
						<div className="flex flex-col items-center justify-center py-8 px-4 text-center">
							<MessageSquare className="size-8 text-muted-foreground/50 mb-2" />
							<p className="text-sm text-muted-foreground">
								No chats yet. Start a new conversation!
							</p>
						</div>
					)}
				</div>
			</SidebarContent>
		</Sidebar>
	);
}
```

Let's review what we did here: 
1. First, we imported all necessary components, icons, and function requirements 
2. We added an `Add Chat` section with a button that calls the `submitChat` function
3. The `submitChat` function calls the `addChatFn` Server Function and then invalidates the page, which causes the `loader` function to be called again and the data to be refreshed


Now you can test it! Click the **New Chat** button in your browser. After a few clicks, you should see a similar view:

![Chats image](/assets/tanstack-2-chats.png)

## Better Storage

Now we can create and view chats, but there are some problems with our current approach:

> Chats are not persistent after page refresh

Why does this happen? The issue is that we're storing chats in an **in-memory array** that lives only for the lifetime of the server process. In development, the server may restart, and in production deployments (especially serverless environments like Vercel, Cloudflare Workers, or Netlify), each function invocation may get a fresh instance with a new empty `chats` array. This means our data disappears on every refresh or deployment.

TanStack Start can run in both traditional Node.js servers and serverless environments, but in-memory storage is ephemeral in both cases. We need persistent storage.

We need a better approach for this. Normally you would use a database or cloud storage, but for this course we'll store data locally as a file. This will persist through refreshes and across server function calls.

Let's get started! First, we need to create utilities for file storage. Create `src/lib/fileUtils.ts` with the following content:

```typescript fileName=src/lib/fileUtils.ts
import fs from "node:fs/promises";
import path from "node:path";

async function ensureDataDir(fileName: string) {
	const dir = path.dirname(path.join(process.cwd(), "data", fileName));
	try {
		await fs.access(dir);
	} catch {
		await fs.mkdir(dir, { recursive: true });
	}
}

export async function readContent<T>(fileName: string): Promise<T | null> {
	try {
		await ensureDataDir(fileName);
		const data = await fs.readFile(
			path.join(process.cwd(), "data", fileName),
			"utf-8",
		);
		return JSON.parse(data);
	} catch (error) {
		// If file doesn't exist or is invalid, return empty array
		return null;
	}
}

export async function writeContent<T>(
	fileName: string,
	content: T,
): Promise<void> {
	await ensureDataDir(fileName);
	await fs.writeFile(
		path.join(process.cwd(), "data", fileName),
		JSON.stringify(content, null, 2),
		"utf-8",
	);
}
```

This utility provides two functions:
- **`readContent`** - Reads and parses JSON data from a file, creating the directory if it doesn't exist
- **`writeContent`** - Writes data to a file as formatted JSON

Next, we need to update our `src/db/chat.ts` functions to use these utilities:

```typescript remove={4-5,8,14-15} add={1,9,16-24} fileName=src/db/chat.ts
import { readContent, writeContent } from "@/lib/fileUtils";
import type { Chat } from "@/types/chat";

const chats: Chat[] = [];

export async function getChats() {
	await new Promise((resolve) => setTimeout(resolve, 1000));
  return chats;
	return (await readContent<Chat[]>("chats.json")) ?? [];
}

export async function createChat(title: string) {
	await new Promise((resolve) => setTimeout(resolve, 1000));
  const newChat: Chat = { id: chats.length + 1, title, timestamp: new Date().toISOString() }
  return newChat;
	const chats = await getChats();
	const newChat: Chat = {
		id: chats.length + 1,
		title,
		timestamp: new Date().toISOString(),
	};
	chats.push(newChat);
	await writeContent<Chat[]>("chats.json", chats);
	return newChat;
}

```

Key changes we made:
- Removed the in-memory `chats` array
- `getChats()` now reads from the `chats.json` file
- `createChat()` reads existing chats, adds the new chat, and writes back to the file

Now when you add chats and refresh the page, they should persist. Great work!

<Woz
title="Test Your Knowledge"
description="Let's check with Woz if you were paying attention!"
context={`Ask user the following questions, one by one:
1. What is the TanStack Start createServerFn function and how does it work?
2. How do you update web page data after you update data on the backend?
3. Why did our chats disappear after refresh?`}
prompt="Ask me! I know everything!"/>

## Next Steps

Great! We now have our first working feature. Users can create and view chat objects. Remember, this isn't just frontend code - we already have a backend with React components, server functions, and persistent storage!