---
title: Creating an AI chat
description: We will create an AI chat using ai-sdk
---

In this chapter, we'll create an AI chat where you can have real conversations with an AI assistant!

> **Need Help?**
> 
> If you encounter any difficulties, you can always reference the complete code from this GitHub repository:
>
> [https://github.com/pouljohn1/tanstack-chat](https://github.com/pouljohn1/tanstack-chat)

## Prerequisites

### Dependencies

First, we need to install Vercel's AI SDK for connecting with AI models:

```bash
bun add ai @ai-sdk/react @ai-sdk/openai zod
```

We've installed:
- **`ai`** - Core AI SDK JavaScript library
- **`@ai-sdk/react`** - React hooks and utilities for chat functionality
- **`@ai-sdk/openai`** - OpenAI provider for the AI SDK
- **`zod`** - TypeScript schema validation library

### OpenAI API Key

We need an OpenAI API key for our chat to work. Here's how to get one:

1. Go to https://platform.openai.com/api-keys
2. Click **Create new secret key**
3. Optionally enter a name for your key
4. Click **Create secret key**
5. Copy your new API key
6. Create a file named `.env` in your project root and add:
   ```
   OPENAI_API_KEY="your_api_key_here"
   ```

Voilà, everything is set!

## Chat Details Page

Let's create a new page to display individual chat conversations. In your `src/routes` folder, create the path `src/routes/chat/$chatId.tsx`:

> **Have you noticed**
>
> Did you notice that Tanstack has a different way of writing path params? Here we are using `$chatId` instead of `[chatId]` in Next.js. In addition we are allowed to create file route `$chatId.tsx` without need of creating new folder for it - in Next.js we needed to do `[chatId]/page.tsx`.

```typescript filename=src/routes/chat/$chatId.tsx
import { useChat } from "@ai-sdk/react";
import { createFileRoute, notFound } from "@tanstack/react-router";
import { DefaultChatTransport } from "ai";
import { useState } from "react";
import { getChatByIdFn } from "@/api/chat";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Skeleton } from "@/components/ui/skeleton";

export const Route = createFileRoute("/chat/$chatId")({
	component: ChatDetails,
	pendingComponent: Loader,
	loader: async ({ params }) =>
		await getChatByIdFn({ data: Number.parseInt(params.chatId, 10) }),
});

function Loader() {
	return (
		<div className="p-4">
			<Skeleton className="w-full h-[40px] mb-4" />
		</div>
	);
}

function ChatDetails() {
	const chat = Route.useLoaderData();
	if (!chat) {
		throw notFound();
	}

	const { messages, sendMessage, status } = useChat({
		id: chat.id.toString(),
		transport: new DefaultChatTransport({
			api: `/api/chat/${chat.id}`,
		}),
	});
	const [input, setInput] = useState("");

	return (
		<div className="p-4">
			{messages.map((message) => (
				<div key={message.id}>
					{message.role === "user" ? "User: " : "AI: "}
					{message.parts.map((part, index) =>
						// biome-ignore lint/suspicious/noArrayIndexKey: not ideal but for now ok
						part.type === "text" ? <span key={index}>{part.text}</span> : null,
					)}
				</div>
			))}

			<form
				className="flex gap-2"
				onSubmit={(e) => {
					e.preventDefault();
					if (input.trim()) {
						sendMessage({ text: input });
						setInput("");
					}
				}}
			>
				<Input
					value={input}
					onChange={(e) => setInput(e.target.value)}
					disabled={status !== "ready"}
					placeholder="Say something..."
				/>
				<Button type="submit" disabled={status !== "ready"}>
					Submit
				</Button>
			</form>
		</div>
	);
}
```

> **Issues with TypeScript?**
>
> Remember that all routes in Tanstack Start are typed. To do this Tanstack generates `routeTree.gen.ts`, which is being generated when starting an app. So if you have any issues about not `createFileRoute` path then try to restart app with `bun dev`

Different than in Next.js we have created whole page in one component. The most important thing is what we are export, meaning `const Route` which value is result of calling function `createFileRoute`. In that function, similarly to what we had with `createRootRoute` we are specifying all configuration for that route. In our example we have:
- `component` - a react component that will be used for showing a content for that page
- `pendingComponent` - a loader component that will be shown while waiting for main component
- `loader` - we already know that - it is for fetching data from a backend

Let's break down what's happening inside component itself:

**`useChat()` hook**: This is from the AI SDK and provides everything we need for chat functionality:
- `messages`: Array of all messages in the conversation
- `sendMessage`: Function to send a new message to the AI
- `status`: Current state of the chat ('ready', 'loading', etc.)
- `id`: Chat identifier for managing multiple conversations
- `transport`: Configures how messages are sent to your API endpoint

The hook handles all the complexity of managing message state, streaming responses, and API communication for us.

## Linking to Chats

Now we need a way to open individual chats. Update `src/components/ChatSidebar.tsx` to make chat items clickable:

```typescript add={43-44} remove={42} hide={1-39,47-} fileName=src/components/ChatSidebar.tsx
import { Link, useRouter } from "@tanstack/react-router";
import { MessageSquare, Plus } from "lucide-react";
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
									to="/chat/$chatId"
									params={{ chatId: chat.id.toString() }}
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

We've replaced a path of `<Link>` component from `/` to `/chat/$chatId` and added `params` to that component. As you see all params, routes are strictly typed in Tanstack Start. No possible to make mistake here :)

Now we have all the UI components we need. Let's dive into the backend and API!

## Adding the Get Chat Function

First, we need a function in `src/db/chat.ts` to fetch a chat by ID. This will be used in our API to verify the chat exists.

Add this code at the end of `src/db/chat.ts`:

```typescript hide={1-18} add={21-25} fileName=src/db/chat.ts
import { readContent, writeContent } from "@/lib/fileUtils";
import type { Chat } from "@/types/chat";

export async function getChats() {
	await new Promise((resolve) => setTimeout(resolve, 1000));
	return (await readContent<Chat[]>("chats.json")) ?? [];
}

export async function createChat(title: string) {
	await new Promise((resolve) => setTimeout(resolve, 1000));
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

export async function getChatById(chatId: number) {
	const chats = await getChats();
	return chats.find((chat) => chat.id === chatId);
}

```

## Creating the Chat API

Our AI SDK will communicate with `/api/chat/$chatId`, so let's create that endpoint. Create the file `src/routes/api/chat.[chatId].ts`:

```typescript fileName=src/routes/api/chat.[chatId].ts
import { openai } from "@ai-sdk/openai";
import { createFileRoute } from "@tanstack/react-router";
import { convertToModelMessages, streamText, type UIMessage } from "ai";
import { getChatById } from "@/db/chat";

export const Route = createFileRoute("/api/chat/$chatId")({
	server: {
		handlers: {
			POST: async ({ request, params }) => {
				const { chatId } = params;
				const chat = await getChatById(Number.parseInt(chatId, 10));
				if (!chat) {
					return new Response("Chat not found", { status: 404 });
				}

				const { messages }: { messages: UIMessage[] } = await request.json();

				const result = streamText({
					model: openai("gpt-4.1"),
					system: "You are a helpful assistant.",
					messages: convertToModelMessages(messages),
				});

				return result.toUIMessageStreamResponse();
			},
		},
	},
});
```

> **Did you notice?**
>
> Look how we named a route file. Instead of creating folders we created a file with dots. Tanstack Start allows much more flexibility when creating route paths.

Let's break down what's happening in this API route:

### Understanding the Code

1. **createFileRoute**: We are still using the same function as we did while creating normal page, you know this already! Such route is name **Server Route**. 

2. **server.handlers.POST**: This function handles incoming chat messages:
   - Extracts the `chatId` from the URL by `params` field
   - Verifies the chat exists using `getChatById()`
   - Parses the incoming messages from the request body
   - Returns a 404 if the chat doesn't exist

3. **`streamText()`**: This is the core AI SDK function that:
   - Takes a model (we're using GPT-4.1)
   - Takes a system prompt (instructions for the AI's behavior)
   - Takes the conversation messages
   - Returns a streaming response

4. **`toUIMessageStreamResponse()`**: Converts the AI stream into a format that the `useChat` hook can understand and display in real-time.

## Testing Your Chat

You're ready to test! Create a new chat, open it, and start chatting with the AI. You should see something like this:

![Chat image](/assets/tanstack-3-chat.png)

<Woz
title="Test Your Knowledge"
description="Let's check with Woz if you were paying attention!"
context={`Ask user the following questions, one by one:
1. What is \`createFileRoute\` function and what is it used for?
2. Is path \`routes/profile.$profileId.hobby.tsx\` can be used in Tanstack? What data may be on such page?
3. What makes a route a "Server Route" in Tanstack Start, and what can you do with it that you can't do with a regular route component?`}
prompt="Ask me! I know everything!"/>

## What's Next

Congratulations! You now have a fully functional AI chat application built with Tanstack Start. You're using server functions, server routes, static views, persistent data storage, and real AI conversations. This is a complete full-stack application!

### Want to Challenge Yourself?

<Woz
title="Ready for More?"
description="Woz can suggest enhancements to improve your app"
context={`User finished the demo app but would like to do more and improve the code. Possible enhancements:
1. Update chat titles automatically based on the first message (requires modifying Server Route and adding an update function to src/db/chat.ts)
2. Add a welcome placeholder to the home page so it's not empty
3. Persist chat messages so conversations are saved between sessions
4. Add a delete chat button
5. Implement message editing or regeneration
6. Add different AI models to choose from
7. Use AI Elements (https://ai-sdk.dev/elements/overview) to enhance the UI

Don't list every possibility at once. Just randomly pick and tell user that you can show different possibilities if they want.`}
prompt="I'd like to do more, give me ideas!"/>