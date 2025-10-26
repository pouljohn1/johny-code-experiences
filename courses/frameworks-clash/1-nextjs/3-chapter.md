---
title: Creating an AI chat
description: We will create an AI chat using ai-sdk
---

In this chapter, we'll create an AI chat where you can have real conversations with an AI assistant!

> **Need Help?**
> 
> If you encounter any difficulties, you can always reference the complete code from this GitHub repository:
>
> [https://github.com/pouljohn1/nextjs-chat](https://github.com/pouljohn1/nextjs-chat)

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

Let's create a new page to display individual chat conversations. In your `app` folder, create the path `app/chat/[chatId]/page.tsx`:

```typescript filename=app/chat/[chatId]/page.tsx
import ChatView from "@/components/ChatView";
import { Skeleton } from "@/components/ui/skeleton";
import { getChat } from "@/db/chat";
import { redirect } from "next/navigation";
import { Suspense } from "react";

export default async function ChatPage({ params }: PageProps<"/chat/[chatId]">) {
  const chatPromise = getChat(Number.parseInt(params.chatId, 10))
    .then(chat => chat ?? redirect('/404'));

  return (
    <Suspense fallback={
      <div className="p-4">
        <Skeleton className="w-full h-[40px] mb-4" />
      </div>
    }>
      <ChatView chatPromise={chatPromise} />
    </Suspense>
  )
}
```

> **What is Suspense?**
>
> Suspense is a React component that displays a fallback UI while its children are loading. This allows us to show an instant loading state (like a skeleton) and then seamlessly switch to the actual chat view once the data is ready.

Now we have the Next.js page, but we need to implement the `ChatView` component. Create `components/ChatView.tsx`:

```typescript fileName=components/ChatView.tsx
'use client';

import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { use, useState } from 'react';
import { Input } from './ui/input';
import { Button } from './ui/button';
import { Chat } from '@/types/chat';

export default function ChatView({ chatPromise }: { chatPromise: Promise<Chat> }) {
  const chat = use(chatPromise);

  const { messages, sendMessage, status } = useChat({
    id: chat.id.toString(),
    transport: new DefaultChatTransport({
      api: `/api/chat/${chat.id}`,
    }),
  });
  const [input, setInput] = useState('');

  return (
    <div className="p-4">
      {messages.map(message => (
        <div key={message.id}>
          {message.role === 'user' ? 'User: ' : 'AI: '}
          {message.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null,
          )}
        </div>
      ))}

      <form
        className="flex gap-2"
        onSubmit={e => {
          e.preventDefault();
          if (input.trim()) {
            sendMessage({ text: input });
            setInput('');
          }
        }}
      >
        <Input
          value={input}
          onChange={e => setInput(e.target.value)}
          disabled={status !== 'ready'}
          placeholder="Say something..."
        />
        <Button type="submit" disabled={status !== 'ready'}>
          Submit
        </Button>
      </form>
    </div>
  );
}
```

Let's break down what's happening in this component:

**`'use client'` directive**: This tells Next.js that this component should run on the client side. It's required because we're using React hooks (`useState`, `useChat`) and handling user interactions. Server components can't use hooks or browser APIs, so we need to mark this as a client component.

**`use()` hook**: This is a new React hook that unwraps a Promise. It allows us to use async data in a synchronous way within our component. When the promise resolves, the component re-renders with the data.

**`useChat()` hook**: This is from the AI SDK and provides everything we need for chat functionality:
- `messages`: Array of all messages in the conversation
- `sendMessage`: Function to send a new message to the AI
- `status`: Current state of the chat ('ready', 'loading', etc.)
- `id`: Chat identifier for managing multiple conversations
- `transport`: Configures how messages are sent to your API endpoint

The hook handles all the complexity of managing message state, streaming responses, and API communication for us.

## Linking to Chats

Now we need a way to open individual chats. Update `components/ChatSidebar.tsx` to make chat items clickable:

```typescript add={4,10,13,18} remove={9,17} fileName=components/ChatSidebar.tsx
import { Plus, MessageSquare } from "lucide-react";
import { Chat } from "@/types/chat";
import { createChatAction } from "@/actions/chat";
import Link from "next/link";

...

              {chats.map((chat) => (
                <button
                <Link
                  key={chat.id}
                  className="flex items-start gap-3 rounded-lg px-3 py-2.5 text-left hover:bg-accent transition-colors"
                  href={`/chat/${chat.id}`}

...

                </button>
                </Link>

...
```

We've replaced the `<button>` with Next.js's `<Link>` component to enable navigation to individual chat pages. The `Link` component provides client-side navigation, making the app feel faster and more responsive.

Now we have all the UI components we need. Let's dive into the backend and API!

## Adding the Get Chat Function

First, we need a function in `db/chat.ts` to fetch a chat by ID. This will be used in our API to verify the chat exists.

Add this code at the end of `db/chat.ts`:

```typescript fileName=db/chat.ts
export async function getChat(id: number) {
  const chats = await readContent<Chat[]>('chats.json') ?? [];
  return chats.find(chat => chat.id === id);
}
```

## Creating the Chat API

Our AI SDK will communicate with `/api/chat/[chatId]`, so let's create that endpoint. Create the file `app/api/chat/[chatId]/route.ts`:

```typescript fileName=app/api/chat/[chatId]/route.ts
import { getChat } from '@/db/chat';
import { openai } from '@ai-sdk/openai';
import { convertToModelMessages, streamText, UIMessage } from 'ai';
import { NextRequest } from 'next/server';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: NextRequest, { params }: { params: Promise<{ chatId: string }> }
) {
  const { chatId } = await params;
  const chat = await getChat(Number.parseInt(chatId, 10));
  if (!chat) {
    return new Response('Chat not found', { status: 404 });
  }
  
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4.1'),
    system: 'You are a helpful assistant.',
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

Let's break down what's happening in this API route:

### Understanding the Code

1. **`maxDuration = 30`**: This allows streaming responses for up to 30 seconds. AI responses can take time, especially for longer answers, so we need to extend the default timeout.

2. **POST handler**: This function handles incoming chat messages:
   - Extracts the `chatId` from the URL
   - Verifies the chat exists using `getChat()`
   - Parses the incoming messages from the request body
   - Returns a 404 if the chat doesn't exist

3. **`streamText()`**: This is the core AI SDK function that:
   - Takes a model (we're using GPT-4.1)
   - Takes a system prompt (instructions for the AI's behavior)
   - Takes the conversation messages
   - Returns a streaming response

4. **`toUIMessageStreamResponse()`**: Converts the AI stream into a format that the `useChat` hook can understand and display in real-time.

### How AI Chat Works

When you send a message to an AI like ChatGPT, several things happen:

1. **System Message**: This sets the AI's behavior and personality (e.g., "You are a helpful assistant"). It's like giving instructions to the AI about how it should act.

2. **Message History**: The AI receives the entire conversation history, so it has context for your questions.

3. **Prompt**: Your actual message or question to the AI.

4. **Streaming Response**: Instead of waiting for the complete response, the AI streams text back token by token (word by word), making the experience feel more natural and responsive.

5. **Conversation Context**: Each message builds on the previous ones, allowing for coherent multi-turn conversations.

## Testing Your Chat

You're ready to test! Create a new chat, open it, and start chatting with the AI. You should see something like this:

![Chat image](/assets/chapter-3-chat.png)

<Woz
title="Test Your Knowledge"
description="Let's check with Woz if you were paying attention!"
context={`Ask user the following questions, one by one:
1. What is Suspense and why are we using it?
2. What is the 'use client' directive and how does it work?
3. How does AI chat, like ChatGPT, work?
`}
prompt="Ask me! I know everything!"/>

## What's Next

Congratulations! You now have a fully functional AI chat application built with Next.js. You're using cached components, static views where appropriate, persistent data storage, and real AI conversations. This is a complete full-stack application!

### Want to Challenge Yourself?

<Woz
title="Ready for More?"
description="Woz can suggest enhancements to improve your app"
context={`User finished the demo app but would like to do more and improve the code. Possible enhancements:
1. Update chat titles automatically based on the first message (requires modifying route.ts and adding an update function to db/chat.ts)
2. Add a welcome placeholder to the home page so it's not empty
3. Persist chat messages so conversations are saved between sessions
4. Add a delete chat button
5. Implement message editing or regeneration
6. Add different AI models to choose from
7. Use AI Elements (https://ai-sdk.dev/elements/overview) to enhance the UI`}
prompt="I'd like to do more, give me ideas!"/>