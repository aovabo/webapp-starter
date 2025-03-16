Using AI SDK with Mastra

One of the most popular TypeScript AI libraries is Vercel's AI SDK. AI SDK offers model routing (a unified interface on top of OpenAI, Anthropic, etc), structured output, and tool calling. It's a great foundation for building chatbots and prototyping proof of concepts.

We built Mastra as a layer on top of AI SDK to help teams productionize their proof-of-concepts quickly and easily.

AI SDK diagram
Mastra agents with the AI SDK
While AI SDK provides the basic infrastructure for tool calling, the basic Mastra agent is built on top of the AI SDK structured output, model routing, and tool calling.

import { openai } from "@ai-sdk/openai";
import { Agent } from "@mastra/core/agent";

const agent = new Agent({
  name: "WeatherAgent",
  instructions: "Instructions for the agent...",
  model: openai("gpt-4-turbo"), // Model comes directly from AI SDK
});

// Use the agent
const result = await agent.generate("What is the weather like?");
But there are lots more things you get with Mastra, too:

1. Agent Memory
To work effectively, AI agents need the right context from previous interactions. But the right context is often hard to define programmatically! This is where agent memory comes in.

Implementing a memory system for agents usually involves:

Persisting memory in backend storage
Storing and retrieving conversation history
Finding relevant context from past interactions using semantic search
So we built a Memory API for agents in Mastra. This includes:

Conversation context management
Semantic search over past interactions
Storage backend abstraction
Thread sharing between agents
Memory retrieval
Memory Configuration
Here's a reasonable set of parameters for message retrieval and search that you might give an agent.

It includes both a number of recent messages, as well as older messages that are semantically similar, as well as the context surrounding those messages.

await agent.stream("Message text", {
  memoryOptions: {
    lastMessages: 10,
    semanticRecall: {
      topK: 3,
      messageRange: 5,
    },
  },
});
Threads and Resources
When making requests, you can specify some global parameters for the agent. You can specify conversation threads and resources (usually scoped to users or projects), to group context.

resourceId: Identifier for the user/entity making the request
threadId: Identifier for the conversation thread
await agent.stream("Message text", {
  resourceId: "user_123",
  threadId: "thread_123",
});
2. Workflow Graphs
Diagram showing workflow with parallel steps
Mastra provides a workflow graph system for building more deterministic AI pipelines.

Graph-based workflow engine built on XState
Simple control flow syntax for branching (.step()), chaining (.then()), merging (.after()), conditions (when: { ...}), and interrupts (.suspend(), .resume())
Built-in Otel logging
More on this in our workflow blog post.

3. Agent Development Environment
Mastra Dev provides a powerful local development environment for building and testing AI applications.

Mastra Dev Interface
Some things this includes: you can chat with your agents, visualize their state and memory, debug tool calls and execution paths, and test different prompts and configurations.

You can curl agents/workflows, chat with agents, view evals and traces across runs, and iterate on prompts with an assistant. The playground uses a local storage layer powered by libsql (thanks Turso team!) and runs on localhost with npm run dev (no Docker).

Here's docs and a prompt demo

4. RAG Pipeline Primitives
We abstracted the main RAG verbs like .chunk(), embed(), .upsert(),’ .query(), and rerank()` across document types (text, HTML, Markdown, JSON) and vector DBs (Pincone, Pgvector, Qdrant).

We abstracted a metadata querying layer with a MongoDB/sift-like querying syntax on top. ($not, $and, $or, $in, etc)

Here's our RAG documentation.

5. Evals
We shipped an eval runner with 15 evals around accuracy and reliability, context, and output quality. We noticed a lot of teams seemed hesistant to even start with evals due to lack of knowledge, so we wanted to ship evals that we ready to use, or be modified, as well as the ability to write your own.

Here's our eval documentation.

A Personal Note
Technology choice is always a human decision, too. We spent most of the last decade building Gatsby.

Along the way, we gained a lot of respect for what Guillermo and the Vercel team have built. We've spent quite a bit of time with the AI SDK team as well. Their stuff is solid, and we're excited to build on top.