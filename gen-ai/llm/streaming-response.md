---
aliases:
  - Streaming Response
Source 1: https://dev.to/pockit_tools/the-complete-guide-to-streaming-llm-responses-in-web-applications-from-sse-to-real-time-ui-3534
Source 2: https://websocket.org/guides/use-cases/ai-streaming/
---

# AI Token Streaming: From SSE to Durable Sessions

* **Key Points:**
  - "Every major AI provider — OpenAI, Anthropic, Google — streams tokens via Server-Sent Events. The pattern is the same across all of them: client sends a prompt over HTTP POST, server responds with a stream of token events."
* **Technical Entities (Classes/Functions/APIs):** `OpenAI`, `Anthropic`, `Google`, `SSE` (Server-Sent Events)

## Where SSE works fine
* **Key Points:**
  - "For single-turn chat — user sends prompt, model streams back — SSE is the right choice. It is simple, built into browsers, works through every CDN and proxy, has automatic reconnection, and needs no library."
  - "If you are building a basic chatbot, a search summarizer, or a single-turn Q&A interface, SSE handles it well. Do not add complexity you do not need. The threshold for switching away from SSE is when your AI interaction becomes stateful, bidirectional, or long-running."
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `EventSource`

## Where SSE breaks: the Gen 2 AI problem
* **Key Points:**
  - "As AI moves from single-turn chat to agentic workflows, SSE hits fundamental limits. These are not edge cases — they are the core interaction patterns of production AI systems."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Connection drops lose generation state
* **Key Points:**
  - "SSE reconnects automatically via EventSource, but the generation state is gone. The model has no concept of 'resume from token 847.' A 5-minute agent task that drops at minute 4 means restarting from scratch — wasted compute, wasted money, frustrated user. At current API pricing, a dropped 10,000-token generation costs $0.03-0.30 in wasted compute depending on the model. Multiply by thousands of daily users on mobile networks, and the cost of not having resumability becomes measurable."
  - "On mobile networks, connections drop constantly. Wi-Fi to cellular handoffs, elevator rides, subway tunnels. Every drop restarts the entire generation."
* **Technical Entities (Classes/Functions/APIs):** `EventSource`

### Tool call approval needs bidirectionality
* **Key Points:**
  - "Modern AI agents propose tool calls: 'I want to run this database query' or 'I need to call this API.' The user needs to approve. SSE is server-to-client only. The approval requires a separate HTTP request back to the server, with all the coordination complexity that implies — correlating the approval to the right tool call, handling race conditions if the stream continues, and managing timeouts."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Live steering has no path
* **Key Points:**
  - "The user wants to interrupt mid-generation: 'stop, go back, try a different approach.' This is barge-in, and it requires client-to-server messaging on the same session. With SSE, you close the connection (losing state) and start a new request. There is no way to send a 'change direction' signal on an SSE stream."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Multi-device continuity does not exist
* **Key Points:**
  - "Start a conversation with an AI agent on your laptop, walk to a meeting, pull out your phone. With SSE, each connection is independent. There is no session identity, no way to subscribe a second device to the same generation stream."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Multi-agent coordination lacks ordering guarantees
* **Key Points:**
  - "Multiple agents working on a shared task — a coding agent, a testing agent, a review agent — need to read each other's output in order. SSE provides no message ordering guarantees across multiple streams and no shared state primitive."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Background completion has nowhere to go
* **Key Points:**
  - "An agent finishes a long research task after the user closes the tab. With SSE, the stream had one consumer and it is gone. The results vanish. The user comes back and the work is lost."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

## The framework ecosystem is signaling the shift
* **Key Points:**
  - "The AI framework ecosystem has started building abstractions specifically to replace SSE as the default transport: Vercel AI SDK deprecated its SSE-based StreamingTextResponse in favor of a pluggable ChatTransport interface that accepts any bidirectional transport; TanStack AI introduced a ConnectionAdapter abstraction, decoupling streaming from any specific transport; AG-UI (Agent-User Interaction Protocol) designed transport agnosticity from day one, with SSE as just one option; MCP (Model Context Protocol) deprecated its HTTP+SSE transport entirely, replacing it with Streamable HTTP"
  - "These are not fringe projects. They are the dominant AI application frameworks, and they are all creating extension points because developers need alternatives to SSE."
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `Vercel AI SDK`, `TanStack AI`, `AG-UI`, `MCP` (Model Context Protocol)

## Durable sessions: the emerging pattern
* **Key Points:**
  - "A durable session is a persistent, addressable interaction layer between agents and users that outlives any single connection."
  - "It is not a connection (which breaks). It is not a channel (which is a transport primitive). It is the stateful layer that persists across disconnects, device switches, and agent handoffs."
  - "The analogy is durable execution. Just as Temporal and Restate make backend workflows crash-proof by persisting state across failures, durable sessions make user-facing AI experiences crash-proof. The session is the unit of persistence, not the connection."
  - "The term 'durable sessions' is emerging, not yet standardized. ElectricSQL coined it in late 2025; other vendors use different names for similar patterns. The concept matters more than the label."
* **Technical Entities (Classes/Functions/APIs):** `Temporal`, `Restate`, `ElectricSQL`

### What durable sessions provide
* **Key Points:**
  - "Resumable streaming. The client reconnects at the last-acknowledged offset. No duplicate tokens, no restart. The 5-minute agent task that drops at minute 4 resumes at minute 4. The session tracks what was delivered, not what was sent."
  - "Multi-device fan-out. All tabs and devices subscribe to the same session. Start on laptop, continue on phone. Both see the same token stream, in order, without gaps."
  - "Bidirectional interaction. Tool call proposals flow server-to-client. User approvals flow client-to-server. Steering signals, cancellation, preference updates — all through the same session, with ordering guarantees."
  - "Presence awareness. The session knows if a user is actively consuming tokens. If nobody is watching, the system can defer expensive generation, batch results, or reduce streaming priority to save compute."
  - "Asynchronous participation. Join a session after the fact and get the full history. An agent finishes while you are away — the results are waiting when you return, with the complete interaction log intact."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```javascript
// Conceptual: what a durable session client looks like
const session = durableSession.connect("session_abc123", {
  lastOffset: localStorage.getItem("lastOffset") || 0,
});

session.on("token", (token, offset) => {
  renderToken(token);
  localStorage.setItem("lastOffset", offset);
});

session.on("toolCall", (call) => {
  // Agent proposes a tool call — user approves or rejects
  showApprovalDialog(call, (approved) => {
    session.send({ type: "toolResponse", callId: call.id, approved });
  });
});

// Works across disconnects, device switches, tab reloads
// The session layer handles resumption automatically
```

### Who is building this layer
* **Key Points:**
  - "Multiple companies have converged on this pattern independently"
  - Table: ElectricSQL (Durable Streams), Ably (AI Transport), Upstash (Resumable AI SDK Streams), Convex (Agent Component) across: Approach, Transport, Persistence, Open source, Best for
  - "ElectricSQL (Durable Streams). An open protocol for persistent, addressable, real-time streams. Built on HTTP with offset-based resumability and CDN compatibility. Open source under Apache 2.0. ElectricSQL is building structured Durable Sessions on top, targeting collaborative AI with TanStack DB integration."
  - "Ably (AI Transport). Durable sessions built on Ably's global pub/sub infrastructure. Provides resumable token streaming, live steering and barge-in, tool call coordination, and presence-aware cost controls."
  - "Upstash (Resumable AI SDK Streams). Uses Redis Streams as the persistence layer, giving AI SDK streams durability through an existing infrastructure primitive. Tokens persist in Redis, so clients reconnect and resume from the last received offset."
  - "Convex (Agent Component). Persistent threads and real-time sync backed by their reactive database. The agent state lives in the database, so conversations survive disconnects and support real-time subscriptions from multiple clients."
  - "The convergence is the signal — this is not one vendor's feature, it is an emerging architectural layer."
* **Technical Entities (Classes/Functions/APIs):** `ElectricSQL`, `Ably`, `Upstash`, `Convex`, `Redis Streams`, `TanStack DB`

## Why WebSocket matters underneath
* **Key Points:**
  - "Durable sessions need a transport, and WebSocket is the strongest option for token delivery."
  - "Frame overhead. WebSocket uses 2-6 bytes of framing per message. SSE sends full HTTP headers with every event. At hundreds of tokens per second, this adds up — especially on metered mobile connections."
  - "Native bidirectionality. Tool call approvals, steering signals, and presence updates need to flow client-to-server. WebSocket carries both directions on a single TCP connection. SSE requires a separate HTTP request for every client-to-server message."
  - "Lower latency. No HTTP overhead per message means faster token delivery. For real-time AI interactions, the difference between 1ms and 50ms framing overhead is perceptible when multiplied across thousands of tokens."
  - "The durable session layer sits on top of WebSocket (or HTTP as a fallback) and adds persistence, resumability, and state management. WebSocket is the transport; the durable session is the abstraction."
* **Technical Entities (Classes/Functions/APIs):** `WebSocket`, `SSE`, `HTTP`

## Backpressure: the problem nobody talks about
* **Key Points:**
  - "What happens when the server streams tokens faster than the client can render? On a fast model generating hundreds of tokens per second to a slow mobile device, the client buffers tokens in memory. Without flow control, this buffer grows until the browser tab crashes."
  - "SSE has no backpressure mechanism. The server sends, the client receives. There is no way for the client to signal 'slow down' over an SSE stream."
  - "WebSocket enables backpressure through TCP flow control. When the client stops reading from the socket, TCP's receive window fills up, and the server naturally slows down. Some implementations add explicit pause/resume signals at the application layer"
  - "In practice, most implementations rely on TCP flow control rather than application-level signals. When the client's receive buffer fills, TCP back-pressure naturally slows the sender. The application-level approach above is useful when you need finer-grained control — for example, pausing generation entirely rather than just slowing the stream."
  - "For long-running agent tasks that produce large outputs, consider token budgets — a per-session limit on buffered-but-unrendered tokens. When the budget is exceeded, the server pauses generation until the client catches up. This prevents both memory exhaustion and wasted compute."
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `WebSocket`, `TCP`

## FAQ
### Why do most AI APIs use SSE instead of WebSocket?
* **Key Points:**
  - "SSE works over plain HTTP, needs no library beyond the browser's built-in EventSource, and passes through every CDN and proxy without special configuration. It fits the request-response pattern of chat completions naturally: POST the prompt, stream back tokens. For single-turn interactions, SSE has a lower integration cost than WebSocket and fewer infrastructure surprises. The trade-off only becomes visible when you need bidirectionality or session persistence."
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `EventSource`, `WebSocket`

### What is a durable session in AI streaming?
* **Key Points:**
  - "A durable session is a persistent, addressable interaction layer between an agent and a user that outlives any single connection. It survives disconnects, device switches, and agent handoffs by persisting state and enabling offset-based resumption. Think of it as the AI equivalent of durable execution (Temporal, Restate) - instead of making backend workflows crash-proof, durable sessions make user-facing AI experiences crash-proof."
* **Technical Entities (Classes/Functions/APIs):** `Temporal`, `Restate`

### When should I use WebSocket instead of SSE for AI?
* **Key Points:**
  - "When you need bidirectional communication on the same connection: tool call approvals where the user confirms before the agent acts, live steering to redirect mid-generation, or presence awareness to know if anyone is consuming tokens. SSE is server-to-client only, so any client-to-server interaction requires a separate HTTP request with its own latency and coordination logic. If your AI integration is simple prompt-in, tokens-out with no interruption, SSE is the better choice."
* **Technical Entities (Classes/Functions/APIs):** `WebSocket`, `SSE`

### What happens when an SSE connection drops during generation?
* **Key Points:**
  - "The EventSource API reconnects automatically using the Last-Event-ID header, but the generation state on the server is typically gone. The model has no concept of where it left off - it was streaming into a response that no longer has a consumer. For a 5-minute agent task that drops at minute 4, that means restarting from scratch and paying for the compute again. Durable sessions solve this by tracking delivery offsets server-side, so the client resumes at exactly the token where it disconnected."
* **Technical Entities (Classes/Functions/APIs):** `EventSource`, `Last-Event-ID`

### How does backpressure work in AI token streaming?
* **Key Points:**
  - "Without flow control, a fast-streaming server fills the client's memory buffer until the browser tab crashes. WebSocket supports application-level pause/resume signals and benefits from TCP flow control - when the client stops reading, the TCP receive window fills and the server naturally slows. SSE has no backpressure mechanism at all. For long-running agent tasks or slow mobile clients, consider implementing token budgets: a per-session cap on buffered-but-unrendered tokens that pauses generation until the client catches up."

---

* **Technical Entities (Classes/Functions/APIs):** `WebSocket`, `TCP`, `SSE`

# The Complete Guide to Streaming LLM Responses in Web Applications: From SSE to Real-Time UI


* **Key Points:**
  - "Streaming LLM responses is one of the most common challenges developers face when building AI-powered applications in 2024-2025. The difference between a sluggish app that makes users wait 10+ seconds for a complete response and a responsive interface that starts showing content immediately can mean the difference between user adoption and abandonment."
  - "In this comprehensive guide, we'll dive deep into the mechanics of streaming LLM responses, from the underlying protocols to production-ready implementations."
* **Technical Entities (Classes/Functions/APIs):** `OpenAI`, `Anthropic`, `Ollama`

## Why Streaming Matters: The Psychology of Perceived Performance
* **Key Points:**
  - "Without streaming, your users stare at a loading spinner for the entire response time. With streaming, they start seeing content within the TTFT window—a 10-20x improvement in perceived responsiveness."
  - "Users perceive streaming interfaces as 40% faster than buffered responses, even when total time is identical; Progressive content display reduces perceived wait time and user anxiety; The 'typewriter effect' creates a sense of the AI 'thinking,' which paradoxically increases trust"
* **Technical Entities (Classes/Functions/APIs):** `TTFT` (Time to First Token)

## Understanding the Streaming Pipeline
* **Key Points:**
  - Table: LLM API Layer → Backend Server → Transport Protocol → Frontend
  - "LLM API Layer: The provider sends tokens as they're generated"
  - "Backend Server: Transforms API responses into a format suitable for your transport"
  - "Transport Protocol: How you send data to the browser (SSE, WebSockets, or HTTP streaming)"
  - "Frontend Rendering: Efficiently updating the DOM without causing jank"
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `WebSockets`, `HTTP streaming`

## Part 1: Server-Sent Events (SSE) — The Industry Standard
* **Key Points:**
  - "Server-Sent Events (SSE) is the de facto standard for LLM streaming. It's what OpenAI, Anthropic, and most LLM APIs use natively."
* **Technical Entities (Classes/Functions/APIs):** `SSE` (Server-Sent Events)

### What is SSE?
* **Key Points:**
  - "SSE is a web standard that allows servers to push data to web clients over HTTP. Unlike WebSockets, it's: Unidirectional: Server → Client only; HTTP-based: Works through proxies, CDNs, and load balancers; Auto-reconnecting: Built-in reconnection with Last-Event-ID; Text-based: Each message is a text event"
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `WebSockets`

### The SSE Protocol Format
* **Key Points:**
  - "Key rules: Each field is on its own line: field: value; Messages are separated by double newlines (\n\n); The data: field contains your payload (usually JSON); Optional event:, id:, and retry: fields"
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### Node.js Backend Implementation
* **Key Points:**
  - "Let's build a production-ready SSE endpoint that proxies OpenAI's streaming API"
* **Technical Entities (Classes/Functions/APIs):** `Express`, `OpenAI`, `cors`
* **Code Snippet:**
```javascript
// server.js - Express + OpenAI Streaming
import express from 'express';
import OpenAI from 'openai';
import cors from 'cors';

const app = express();
app.use(cors());
app.use(express.json());

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

app.post('/api/chat/stream', async (req, res) => {
  const { messages, model = 'gpt-4-turbo-preview' } = req.body;

  // Set SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('X-Accel-Buffering', 'no'); // Disable nginx buffering

  // Flush headers immediately
  res.flushHeaders();

  try {
    const stream = await openai.chat.completions.create({
      model,
      messages,
      stream: true,
      stream_options: { include_usage: true }, // Get token usage in stream
    });

    let totalTokens = 0;

    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content;
      const finishReason = chunk.choices[0]?.finish_reason;

      // Track usage if available (last chunk contains usage)
      if (chunk.usage) {
        totalTokens = chunk.usage.total_tokens;
      }

      if (content) {
        // Send content chunk
        res.write(`data: ${JSON.stringify({ 
          type: 'content', 
          content 
        })}\n\n`);
      }

      if (finishReason) {
        // Send completion signal with metadata
        res.write(`data: ${JSON.stringify({ 
          type: 'done', 
          finishReason,
          usage: { totalTokens }
        })}\n\n`);
      }
    }
  } catch (error) {
    console.error('Stream error:', error);

    // Send error event
    res.write(`data: ${JSON.stringify({ 
      type: 'error', 
      message: error.message 
    })}\n\n`);
  } finally {
    res.end();
  }
});

app.listen(3001, () => {
  console.log('Server running on http://localhost:3001');
});
```

### Critical Implementation Details
* **Key Points:**
  - "1. Proxy and Load Balancer Buffering: Nginx, Cloudflare, and many reverse proxies buffer responses by default. This destroys streaming."
  - "2. Connection Timeouts: Long-running streams can hit timeout limits. Implement heartbeats"
  - "3. Backpressure Handling: If the client can't consume data fast enough, you need to handle backpressure"
* **Technical Entities (Classes/Functions/APIs):** `Nginx`, `Cloudflare`, `X-Accel-Buffering`

## Part 2: Frontend Implementation with React
* **Key Points:**
  - "Now let's build a React frontend that consumes our SSE stream with proper error handling, cancellation, and a smooth UI."
* **Technical Entities (Classes/Functions/APIs):** `React`

### The Core Streaming Hook
* **Technical Entities (Classes/Functions/APIs):** `useStreamingChat`, `AbortController`, `fetch`, `TextDecoder`
* **Code Snippet:**
```typescript
// hooks/useStreamingChat.ts
import { useState, useCallback, useRef } from 'react';

interface Message {
  role: 'user' | 'assistant';
  content: string;
}

interface StreamState {
  isStreaming: boolean;
  error: string | null;
  usage: { totalTokens: number } | null;
}

export function useStreamingChat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [streamState, setStreamState] = useState<StreamState>({
    isStreaming: false,
    error: null,
    usage: null,
  });

  const abortControllerRef = useRef<AbortController | null>(null);

  const sendMessage = useCallback(async (userMessage: string) => {
    // Cancel any existing stream
    abortControllerRef.current?.abort();
    abortControllerRef.current = new AbortController();

    const newMessages: Message[] = [
      ...messages,
      { role: 'user', content: userMessage },
    ];

    setMessages([...newMessages, { role: 'assistant', content: '' }]);
    setStreamState({ isStreaming: true, error: null, usage: null });

    try {
      const response = await fetch('/api/chat/stream', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: newMessages }),
        signal: abortControllerRef.current.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const reader = response.body?.getReader();
      if (!reader) throw new Error('No response body');

      const decoder = new TextDecoder();
      let buffer = '';
      let assistantContent = '';

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        buffer += decoder.decode(value, { stream: true });

        // Process complete SSE messages
        const lines = buffer.split('\n\n');
        buffer = lines.pop() || ''; // Keep incomplete message in buffer

        for (const line of lines) {
          if (!line.startsWith('data: ')) continue;

          const jsonStr = line.slice(6); // Remove 'data: ' prefix
          if (jsonStr === '[DONE]') continue;

          try {
            const data = JSON.parse(jsonStr);

            if (data.type === 'content') {
              assistantContent += data.content;
              setMessages(prev => {
                const updated = [...prev];
                updated[updated.length - 1] = {
                  role: 'assistant',
                  content: assistantContent,
                };
                return updated;
              });
            } else if (data.type === 'done') {
              setStreamState(prev => ({
                ...prev,
                usage: data.usage,
              }));
            } else if (data.type === 'error') {
              throw new Error(data.message);
            }
          } catch (parseError) {
            console.warn('Failed to parse SSE message:', line);
          }
        }
      }
    } catch (error) {
      if ((error as Error).name === 'AbortError') {
        // User cancelled, not an error
        return;
      }

      setStreamState(prev => ({
        ...prev,
        error: (error as Error).message,
      }));

      // Remove empty assistant message on error
      setMessages(prev => prev.slice(0, -1));
    } finally {
      setStreamState(prev => ({ ...prev, isStreaming: false }));
    }
  }, [messages]);

  const cancelStream = useCallback(() => {
    abortControllerRef.current?.abort();
    setStreamState(prev => ({ ...prev, isStreaming: false }));
  }, []);

  const clearMessages = useCallback(() => {
    setMessages([]);
    setStreamState({ isStreaming: false, error: null, usage: null });
  }, []);

  return {
    messages,
    streamState,
    sendMessage,
    cancelStream,
    clearMessages,
  };
}
```

### Rendering Streaming Content Efficiently
* **Key Points:**
  - "When content updates 10-50 times per second during streaming, naive React rendering can cause performance issues. Here's how to optimize"
* **Technical Entities (Classes/Functions/APIs):** `memo`, `useMemo`, `marked`, `DOMPurify`
* **Code Snippet:**
```typescript
// components/MessageContent.tsx
import { memo, useMemo } from 'react';
import { marked } from 'marked';
import DOMPurify from 'dompurify';

interface MessageContentProps {
  content: string;
  isStreaming: boolean;
}

// Memoize to prevent re-renders from parent updates
export const MessageContent = memo(function MessageContent({
  content,
  isStreaming,
}: MessageContentProps) {
  // Only parse markdown when not streaming (expensive operation)
  const renderedContent = useMemo(() => {
    if (isStreaming) {
      // During streaming, just show raw text with basic formatting
      return content.split('\n').map((line, i) => (
        <span key={i}>
          {line}
          {i < content.split('\n').length - 1 && <br />}
        </span>
      ));
    }

    // After streaming complete, render full markdown
    const html = marked(content, { breaks: true, gfm: true });
    const sanitized = DOMPurify.sanitize(html);
    return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
  }, [content, isStreaming]);

  return (
    <div className="message-content">
      {renderedContent}
      {isStreaming && <span className="cursor-blink">▊</span>}
    </div>
  );
});
```

### The Complete Chat Component
* **Technical Entities (Classes/Functions/APIs):** `useStreamingChat`, `MessageContent`
* **Code Snippet:**
```typescript
// components/StreamingChat.tsx
import { useState, useRef, useEffect, FormEvent } from 'react';
import { useStreamingChat } from '../hooks/useStreamingChat';
import { MessageContent } from './MessageContent';

export function StreamingChat() {
  const [input, setInput] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const inputRef = useRef<HTMLTextAreaElement>(null);

  const {
    messages,
    streamState,
    sendMessage,
    cancelStream,
    clearMessages,
  } = useStreamingChat();

  // Auto-scroll to bottom on new messages
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  // Auto-resize textarea
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.style.height = 'auto';
      inputRef.current.style.height = `${inputRef.current.scrollHeight}px`;
    }
  }, [input]);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    if (!input.trim() || streamState.isStreaming) return;

    const message = input.trim();
    setInput('');
    await sendMessage(message);
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSubmit(e);
    }
  };

  return (
    <div className="chat-container">
      {/* Messages Area */}
      <div className="messages-area">
        {messages.length === 0 ? (
          <div className="empty-state">
            <h2>Start a conversation</h2>
            <p>Send a message to begin chatting with AI</p>
          </div>
        ) : (
          messages.map((message, index) => (
            <div
              key={index}
              className={`message message-${message.role}`}
            >
              <div className="message-avatar">
                {message.role === 'user' ? '👤' : '🤖'}
              </div>
              <MessageContent
                content={message.content}
                isStreaming={
                  streamState.isStreaming &&
                  index === messages.length - 1 &&
                  message.role === 'assistant'
                }
              />
            </div>
          ))
        )}
        <div ref={messagesEndRef} />
      </div>

      {/* Error Display */}
      {streamState.error && (
        <div className="error-banner">
          <span>⚠️ {streamState.error}</span>
          <button onClick={clearMessages}>Dismiss</button>
        </div>
      )}

      {/* Token Usage */}
      {streamState.usage && (
        <div className="usage-info">
          Tokens used: {streamState.usage.totalTokens}
        </div>
      )}

      {/* Input Area */}
      <form onSubmit={handleSubmit} className="input-area">
        <textarea
          ref={inputRef}
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="Type a message... (Shift+Enter for new line)"
          disabled={streamState.isStreaming}
          rows={1}
        />

        {streamState.isStreaming ? (
          <button type="button" onClick={cancelStream} className="cancel-btn">
            ⏹ Stop
          </button>
        ) : (
          <button type="submit" disabled={!input.trim()}>
            Send →
          </button>
        )}
      </form>
    </div>
  );
}
```

## Part 3: Alternative Approaches — When SSE Isn't Enough
* **Key Points:**
  - "While SSE covers 90% of use cases, some scenarios require different approaches."
* **Technical Entities (Classes/Functions/APIs):** `SSE`

### WebSockets for Bidirectional Communication
* **Key Points:**
  - "Use WebSockets when you need: Real-time interrupts: Allow users to stop generation mid-stream; Multiplexing: Multiple concurrent streams over one connection; Bidirectional control: Server-initiated status updates"
* **Technical Entities (Classes/Functions/APIs):** `WebSockets`
* **Code Snippet:**
```javascript
// WebSocket streaming implementation
class StreamingWebSocket {
  private ws: WebSocket;
  private messageHandlers = new Map<string, (data: any) => void>();

  constructor(url: string) {
    this.ws = new WebSocket(url);

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      const handler = this.messageHandlers.get(data.streamId);
      if (handler) handler(data);
    };
  }

  async streamChat(
    messages: Message[],
    onChunk: (content: string) => void
  ): Promise<{ streamId: string; cancel: () => void }> {
    const streamId = crypto.randomUUID();

    this.messageHandlers.set(streamId, (data) => {
      if (data.type === 'content') {
        onChunk(data.content);
      }
    });

    this.ws.send(JSON.stringify({
      type: 'start_stream',
      streamId,
      messages,
    }));

    return {
      streamId,
      cancel: () => {
        this.ws.send(JSON.stringify({ type: 'cancel', streamId }));
        this.messageHandlers.delete(streamId);
      },
    };
  }
}
```

### HTTP/2 Server Push
* **Key Points:**
  - "If your infrastructure supports HTTP/2, you can leverage server push for even lower latency. The key advantage is multiplexing multiple streams over a single TCP connection without head-of-line blocking."
* **Technical Entities (Classes/Functions/APIs):** `HTTP/2`

### Using the Vercel AI SDK
* **Key Points:**
  - "For production applications, consider the Vercel AI SDK, which abstracts away much of this complexity"
* **Technical Entities (Classes/Functions/APIs):** `Vercel AI SDK`, `useChat`
* **Code Snippet:**
```javascript
// Using Vercel AI SDK
import { useChat } from 'ai/react';

export function Chat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
  });

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>{m.role}: {m.content}</div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
        <button type="submit" disabled={isLoading}>Send</button>
      </form>
    </div>
  );
}
```

## Part 4: Production Considerations
### Error Recovery and Retry Logic
* **Key Points:**
  - "Implement exponential backoff for transient failures"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```javascript
async function streamWithRetry(
  messages: Message[],
  maxRetries = 3
): AsyncGenerator<string> {
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      yield* streamFromAPI(messages);
      return; // Success, exit
    } catch (error) {
      attempt++;

      if (attempt >= maxRetries) throw error;

      // Exponential backoff: 1s, 2s, 4s
      const delay = Math.pow(2, attempt - 1) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Rate Limiting and Queue Management
* **Key Points:**
  - "When multiple users are streaming simultaneously"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```javascript
// Simple token bucket rate limiter
class RateLimiter {
  private tokens: number;
  private lastRefill: number;

  constructor(
    private maxTokens: number,
    private refillRate: number // tokens per second
  ) {
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
  }

  async acquire(): Promise<boolean> {
    this.refill();

    if (this.tokens >= 1) {
      this.tokens -= 1;
      return true;
    }

    return false;
  }

  private refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(
      this.maxTokens,
      this.tokens + elapsed * this.refillRate
    );
    this.lastRefill = now;
  }
}
```

### Monitoring and Observability
* **Key Points:**
  - "Track these key metrics: Time to First Token (TTFT): When users first see content; Tokens per Second (TPS): Generation speed; Stream Completion Rate: Percentage of streams that complete without error; Connection Duration: How long streams stay open"
* **Technical Entities (Classes/Functions/APIs):** `Prometheus`, `Counter`, `Histogram`
* **Code Snippet:**
```javascript
// Prometheus-style metrics
import { Counter, Histogram } from 'prom-client';

const streamDuration = new Histogram({
  name: 'llm_stream_duration_seconds',
  help: 'Duration of LLM streaming requests',
  buckets: [0.5, 1, 2, 5, 10, 30, 60],
});

const tokensGenerated = new Counter({
  name: 'llm_tokens_generated_total',
  help: 'Total tokens generated',
});

const streamErrors = new Counter({
  name: 'llm_stream_errors_total',
  help: 'Total streaming errors',
  labelNames: ['error_type'],
});
```

### Cost Optimization
* **Key Points:**
  - "Streaming doesn't reduce API costs, but you can optimize: Token estimation: Predict output length and warn users before expensive operations; Caching: Cache common responses (with appropriate invalidation); Model selection: Use cheaper models for simple tasks, expensive models for complex ones"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```javascript
// Intelligent model selection
function selectModel(prompt: string): string {
  const estimatedComplexity = analyzePromptComplexity(prompt);

  if (estimatedComplexity < 0.3) {
    return 'gpt-3.5-turbo'; // $0.0005/1K tokens
  } else if (estimatedComplexity < 0.7) {
    return 'gpt-4-turbo-preview'; // $0.01/1K tokens  
  } else {
    return 'gpt-4'; // $0.03/1K tokens
  }
}
```

## Part 5: Handling Edge Cases
### Very Long Responses
* **Key Points:**
  - "For responses that might exceed browser memory"
* **Technical Entities (Classes/Functions/APIs):** `react-window`, `FixedSizeList`
* **Code Snippet:**
```javascript
// Virtual scrolling for very long responses
import { FixedSizeList as List } from 'react-window';

function VirtualizedMessage({ content }: { content: string }) {
  const lines = content.split('\n');

  return (
    <List
      height={400}
      itemCount={lines.length}
      itemSize={24}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{lines[index]}</div>
      )}
    </List>
  );
}
```

### Code Blocks in Streaming Content
* **Key Points:**
  - "Detect incomplete code blocks to avoid syntax highlighting flicker"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```javascript
function isCompleteCodeBlock(content: string): boolean {
  const openBlocks = (content.match(/```/g) || []).length;
  return openBlocks % 2 === 0; // Even number means all blocks are closed
}

function MessageContent({ content, isStreaming }: Props) {
  const shouldHighlight = !isStreaming || isCompleteCodeBlock(content);

  // Only apply syntax highlighting when appropriate
  const processed = shouldHighlight 
    ? highlightCode(content) 
    : escapeHtml(content);

  return <div dangerouslySetInnerHTML={{ __html: processed }} />;
}
```

### Mobile and Low-Bandwidth Considerations
* **Key Points:**
  - "Adaptive chunking based on connection speed"
* **Technical Entities (Classes/Functions/APIs):** `navigator.connection`
* **Code Snippet:**
```javascript
// Adaptive chunking based on connection speed
function getChunkingStrategy(): 'immediate' | 'batched' {
  if ('connection' in navigator) {
    const connection = (navigator as any).connection;

    if (connection.effectiveType === '2g' || 
        connection.effectiveType === 'slow-2g') {
      return 'batched'; // Reduce render frequency
    }
  }

  return 'immediate';
}
```

## Conclusion
* **Key Points:**
  - "Streaming LLM responses is no longer optional—it's expected by users who've been trained by ChatGPT and Claude. The good news is that with the patterns in this guide, you can build streaming experiences that match or exceed what the major providers offer."
  - "Key takeaways: SSE is your default choice: Simple, HTTP-based, works through proxies; Handle backpressure and timeouts: Production environments are harsh; Optimize rendering: Markdown parsing and DOM updates are expensive during streaming; Monitor everything: TTFT, TPS, and error rates are your key metrics; Plan for edge cases: Long responses, code blocks, and mobile users need special handling"
  - "The code examples in this guide are production-tested patterns. Take them, adapt them to your stack, and build AI experiences that feel magical."
* **Technical Entities (Classes/Functions/APIs):** `SSE`, `TTFT`, `TPS`