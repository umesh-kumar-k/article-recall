---
aliases:
  - Structured Ouput / JSON Mode
Source 1: https://docs.langchain.com/oss/javascript/langchain/structured-output
---
# Structured output

* **Key Points:**
  - "Structured output allows agents to return data in a specific, predictable format. Instead of parsing natural language responses, you get typed structured data."
  - "LangChain’s prebuilt ReAct agent createAgent handles structured output automatically. The user sets their desired structured output schema, and when the model generates the structured data, it’s captured, validated, and returned in the structuredResponse key of the agent’s state."
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `structuredResponse`, `ResponseFormat`, `ZodSchema`, `StandardSchema`, `JSON Schema`

## Response format
* **Key Points:**
  - "Controls how the agent returns structured data. You can provide a Zod schema, any Standard Schema-compatible schema, or a JSON Schema object."
  - "By default, the agent uses a tool calling strategy, in which the output is created by an additional tool call. Certain models support native structured output, in which case the agent will use that strategy instead."
  - "You can control the behavior by wrapping ResponseFormat in a toolStrategy or providerStrategy function call."
  - "The structured response is returned in the structuredResponse key of the agent’s final state."
  - "Support for native structured output features is read dynamically from the model’s profile data if using langchain>=1.1. If data are not available, use another condition or specify manually."
  - "If tools are specified, the model must support simultaneous use of tools and structured output."
* **Technical Entities (Classes/Functions/APIs):** `toolStrategy`, `providerStrategy`, `initChatModel`, `ModelProfile`, `structuredOutput`

## Provider strategy
* **Key Points:**
  - "Some model providers support structured output natively through their APIs (e.g. OpenAI, xAI (Grok), Gemini, Anthropic (Claude)). This is the most reliable method when available."
  - "Provider-native structured output provides high reliability and strict validation because the model provider enforces the schema. Use it when available."
  - "If the provider natively supports structured output for your model choice, it is functionally equivalent to write responseFormat: contactInfoSchema instead of responseFormat: providerStrategy(contactInfoSchema)."
  - "In either case, if structured output is not supported, the agent will fall back to a tool calling strategy."
* **Technical Entities (Classes/Functions/APIs):** `providerStrategy()`, `ProviderStrategy`, `ZodSchema`, `SerializableSchema`, `JsonSchemaFormat`
* **Code Snippet:**
```typescript
import * as z from "zod";
import { createAgent, providerStrategy } from "langchain";

const ContactInfo = z.object({
    name: z.string().describe("The name of the person"),
    email: z.string().describe("The email address of the person"),
    phone: z.string().describe("The phone number of the person"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: providerStrategy(ContactInfo)
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Extract contact info from: John Doe, john@example.com, (555) 123-4567"}]
});

console.log(result.structuredResponse);
// { name: "John Doe", email: "john@example.com", phone: "(555) 123-4567" }
```

## Tool calling strategy
* **Key Points:**
  - "For models that don't support native structured output, LangChain uses tool calling to achieve the same result. This works with all models that support tool calling (most modern models)."
* **Technical Entities (Classes/Functions/APIs):** `toolStrategy()`, `ToolStrategy`, `ToolStrategyOptions`, `JsonSchemaFormat`, `ZodSchema`, `SerializableSchema`
* **Code Snippet:**
```typescript
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ProductReview = z.object({
    rating: z.number().min(1).max(5).optional(),
    sentiment: z.enum(["positive", "negative"]),
    keyPoints: z.array(z.string()).describe("The key points of the review. Lowercase, 1-3 words each."),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(ProductReview)
})

const result = await agent.invoke({
    "messages": [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})

console.log(result.structuredResponse);
// { "rating": 5, "sentiment": "positive", "keyPoints": ["fast shipping", "expensive"] }
```

### Custom tool message content
* **Key Points:**
  - "The toolMessageContent parameter allows you to customize the message that appears in the conversation history when structured output is generated."
* **Technical Entities (Classes/Functions/APIs):** `toolStrategy()`, `ToolStrategyOptions`, `toolMessageContent`
* **Code Snippet:**
```typescript
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const MeetingAction = z.object({
    task: z.string().describe("The specific task to be completed"),
    assignee: z.string().describe("Person responsible for the task"),
    priority: z.enum(["low", "medium", "high"]).describe("Priority level"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(MeetingAction, {
        toolMessageContent: "Action item captured and added to meeting notes!"
    })
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "From our meeting: Sarah needs to update the project timeline as soon as possible"}]
});

console.log(result);
/**
 * {
 *   messages: [
 *     { role: "user", content: "From our meeting: Sarah needs to update the project timeline as soon as possible" },
 *     { role: "assistant", content: "Action item captured and added to meeting notes!", tool_calls: [ { name: "MeetingAction", args: { task: "update the project timeline", assignee: "Sarah", priority: "high" }, id: "call_456" } ] },
 *     { role: "tool", content: "Action item captured and added to meeting notes!", tool_call_id: "call_456", name: "MeetingAction" }
 *   ],
 *   structuredResponse: { task: "update the project timeline", assignee: "Sarah", priority: "high" }
 * }
 */
```

### Error handling
* **Key Points:**
  - "Models can make mistakes when generating structured output via tool calling. LangChain provides intelligent retry mechanisms to handle these errors automatically."
* **Technical Entities (Classes/Functions/APIs):** `ToolMessage`, `ToolStrategyError`, `handleError`

#### Multiple structured outputs error
* **Key Points:**
  - "When a model incorrectly calls multiple structured output tools, the agent provides error feedback in a ToolMessage and prompts the model to retry."
* **Technical Entities (Classes/Functions/APIs):** `ToolMessage`
* **Code Snippet:**
```typescript
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ContactInfo = z.object({
    name: z.string().describe("Person's name"),
    email: z.string().describe("Email address"),
});

const EventDetails = z.object({
    event_name: z.string().describe("Name of the event"),
    date: z.string().describe("Event date"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy([ContactInfo, EventDetails]),
});

const result = await agent.invoke({
    messages: [
        {
        role: "user",
        content:
            "Extract info: John Doe (john@email.com) is organizing Tech Conference on March 15th",
        },
    ],
});

console.log(result);

/**
 * {
 *   messages: [
 *     { role: "user", content: "Extract info: John Doe (john@email.com) is organizing Tech Conference on March 15th" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ContactInfo", args: { name: "John Doe", email: "john@email.com" }, id: "call_1" }, { name: "EventDetails", args: { event_name: "Tech Conference", date: "March 15th" }, id: "call_2" } ] },
 *     { role: "tool", content: "Error: Model incorrectly returned multiple structured responses (ContactInfo, EventDetails) when only one is expected.\n Please fix your mistakes.", tool_call_id: "call_1", name: "ContactInfo" },
 *     { role: "tool", content: "Error: Model incorrectly returned multiple structured responses (ContactInfo, EventDetails) when only one is expected.\n Please fix your mistakes.", tool_call_id: "call_2", name: "EventDetails" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ContactInfo", args: { name: "John Doe", email: "john@email.com" }, id: "call_3" } ] },
 *     { role: "tool", content: "Returning structured response: {'name': 'John Doe', 'email': 'john@email.com'}", tool_call_id: "call_3", name: "ContactInfo" }
 *   ],
 *   structuredResponse: { name: "John Doe", email: "john@email.com" }
 * }
 */
```

#### Schema validation error
* **Key Points:**
  - "When structured output doesn't match the expected schema, the agent provides specific error feedback."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```typescript
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ProductRating = z.object({
    rating: z.number().min(1).max(5).describe("Rating from 1-5"),
    comment: z.string().describe("Review comment"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(ProductRating),
});

const result = await agent.invoke({
    messages: [
        {
        role: "user",
        content: "Parse this: Amazing product, 10/10!",
        },
    ],
});

console.log(result);

/**
 * {
 *   messages: [
 *     { role: "user", content: "Parse this: Amazing product, 10/10!" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ProductRating", args: { rating: 10, comment: "Amazing product" }, id: "call_1" } ] },
 *     { role: "tool", content: "Error: Failed to parse structured output for tool 'ProductRating': 1 validation error for ProductRating\nrating\n  Input should be less than or equal to 5 [type=less_than_equal, input_value=10, input_type=int].\n Please fix your mistakes.", tool_call_id: "call_1", name: "ProductRating" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ProductRating", args: { rating: 5, comment: "Amazing product" }, id: "call_2" } ] },
 *     { role: "tool", content: "Returning structured response: {'rating': 5, 'comment': 'Amazing product'}", tool_call_id: "call_2", name: "ProductRating" }
 *   ],
 *   structuredResponse: { rating: 5, comment: "Amazing product" }
 * }
 */
```

#### Error handling strategies
* **Key Points:**
  - "You can customize how errors are handled using the handleErrors parameter."
* **Technical Entities (Classes/Functions/APIs):** `handleErrors`, `ToolInputParsingException`, `ToolStrategyError`, `CustomUserError`
* **Code Snippet:**
```typescript
// Custom error message:
const responseFormat = toolStrategy(ProductRating, {
    handleError: "Please provide a valid rating between 1-5 and include a comment."
)

// Handle specific exceptions only:
import { ToolInputParsingException } from "@langchain/core/tools";

const responseFormat = toolStrategy(ProductRating, {
    handleError: (error: ToolStrategyError) => {
        if (error instanceof ToolInputParsingException) {
        return "Please provide a valid rating between 1-5 and include a comment.";
        }
        return error.message;
    }
)

// Handle multiple exception types:
const responseFormat = toolStrategy(ProductRating, {
    handleError: (error: ToolStrategyError) => {
        if (error instanceof ToolInputParsingException) {
        return "Please provide a valid rating between 1-5 and include a comment.";
        }
        if (error instanceof CustomUserError) {
        return "This is a custom user error.";
        }
        return error.message;
    }
)

// No error handling:
const responseFormat = toolStrategy(ProductRating, {
    handleError: false  // All errors raised
)
```