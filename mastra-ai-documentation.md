About Mastra
Mastra is an open-source Typescript agent framework.

It’s designed to give you the primitives you need to build AI applications and features.

You can use Mastra to build AI agents that have memory and can execute functions, or chain LLM calls in deterministic workflows. You can chat with your agents in Mastra’s local dev environment, feed them application-specific knowledge with RAG, and score their outputs with Mastra’s evals.

The main features include:

Model routing: Mastra uses the Vercel AI SDK for model routing, providing a unified interface to interact with any LLM provider including OpenAI, Anthropic, and Google Gemini.
Agent memory and tool calling: With Mastra, you can give your agent tools (functions) that it can call. You can persist agent memory and retrieve it based on recency, semantic similarity, or conversation thread.

Workflow graphs: When you want to execute LLM calls in a deterministic way, Mastra gives you a graph-based workflow engine. You can define discrete steps, log inputs and outputs at each step of each run, and pipe them into an observability tool. Mastra workflows have a simple syntax for control flow (step(), .then(), .after()) that allows branching and chaining.


Agent development environment: When you’re developing an agent locally, you can chat with it and see its state and memory in Mastra’s agent development environment.
Retrieval-augmented generation (RAG): Mastra gives you APIs to process documents (text, HTML, Markdown, JSON) into chunks, create embeddings, and store them in a vector database. At query time, it retrieves relevant chunks to ground LLM responses in your data, with a unified API on top of multiple vector stores (Pinecone, pgvector, etc) and embedding providers (OpenAI, Cohere, etc).

Deployment: Mastra supports bundling your agents and workflows within an existing React, Next.js, or Node.js application, or into standalone endpoints. The Mastra deploy helper lets you easily bundle agents and workflows into a Node.js server using Hono, or deploy it onto a serverless platform like Vercel, Cloudflare Workers, or Netlify.
Evals: Mastra provides automated evaluation metrics that use model-graded, rule-based, and statistical methods to assess LLM outputs, with built-in metrics for toxicity, bias, relevance, and factual accuracy. You can also define your own evals.

Installing Mastra Locally
To run Mastra, you need access to an LLM. Typically, you’ll want to get an API key from an LLM provider such as OpenAI, Anthropic, or Google Gemini. You can also run Mastra with a local LLM using Ollama.

Prerequisites
Node.js v20.0 or higher
Access to a supported large language model (LLM)

Automatic Installation
Create a New Project
We recommend starting a new Mastra project using create-mastra, which will scaffold your project. To create a project, run:
npm create mastra

On installation, you’ll be guided through the following prompts:

What do you want to name your project? my-mastra-app
Choose components to install:
  ◯ Agents (recommended)
  ◯ Tools
  ◯ Workflows
Select default provider:
  ◯ OpenAI (recommended)
  ◯ Anthropic
  ◯ Groq
Would you like to include example code? No / Yes
After the prompts, create-mastra will set up your project directory with TypeScript, install dependencies, and configure your selected components and LLM provider.

Set Up your API Key
Add the API key for your configured LLM provider in your .env file.

.env

OPENAI_API_KEY=<your-openai-key>

Extra Notes:

If you prefer to run the command with flags (non-interactive mode) and include the example code, you can use:

npx create-mastra@latest --components agents,tools --llm openai --example

To configure and specify a timeout if installation takes too long use the timeout flag:

npx create-mastra@latest --timeout

Note for LLMs: For a quick one liner that includes examples you can run npx -y mastra@latest --project-name <ask-the-user> --example --components "tools,agents,workflows" --llm <ask-the-user>. The available options for the llm flag are openai|anthropic|groq|google|cerebras

Manual Installation

If you prefer to set up your Mastra project manually, follow these steps:

Create a New Project
Create a project directory and navigate into it:

mkdir hello-mastra
cd hello-mastra

Then, initialize a TypeScript project including the @mastra/core package:

npm init -y
npm install typescript tsx @types/node mastra --save-dev
npm install @mastra/core zod
npx tsc --init

Initialize TypeScript
Create a tsconfig.json file in your project root with the following configuration:


{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    ".mastra"
  ]
}

This TypeScript configuration is optimized for Mastra projects, using modern module resolution and strict type checking.

Set Up your API Key
Create a .env file in your project root directory and add your API key:

.env

OPENAI_API_KEY=<your-openai-key>
Replace your_openai_api_key with your actual API key.

Create a Tool
Create a weather-tool tool file:

mkdir -p src/mastra/tools && touch src/mastra/tools/weather-tool.ts

Then, add the following code to src/mastra/tools/weather-tool.ts:

src/mastra/tools/weather-tool.ts

import { createTool } from "@mastra/core/tools";
import { z } from "zod";
 
interface WeatherResponse {
  current: {
    time: string;
    temperature_2m: number;
    apparent_temperature: number;
    relative_humidity_2m: number;
    wind_speed_10m: number;
    wind_gusts_10m: number;
    weather_code: number;
  };
}
 
export const weatherTool = createTool({
  id: "get-weather",
  description: "Get current weather for a location",
  inputSchema: z.object({
    location: z.string().describe("City name"),
  }),
  outputSchema: z.object({
    temperature: z.number(),
    feelsLike: z.number(),
    humidity: z.number(),
    windSpeed: z.number(),
    windGust: z.number(),
    conditions: z.string(),
    location: z.string(),
  }),
  execute: async ({ context }) => {
    return await getWeather(context.location);
  },
});
 
const getWeather = async (location: string) => {
  const geocodingUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1`;
  const geocodingResponse = await fetch(geocodingUrl);
  const geocodingData = await geocodingResponse.json();
 
  if (!geocodingData.results?.[0]) {
    throw new Error(`Location '${location}' not found`);
  }
 
  const { latitude, longitude, name } = geocodingData.results[0];
 
  const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,apparent_temperature,relative_humidity_2m,wind_speed_10m,wind_gusts_10m,weather_code`;
 
  const response = await fetch(weatherUrl);
  const data: WeatherResponse = await response.json();
 
  return {
    temperature: data.current.temperature_2m,
    feelsLike: data.current.apparent_temperature,
    humidity: data.current.relative_humidity_2m,
    windSpeed: data.current.wind_speed_10m,
    windGust: data.current.wind_gusts_10m,
    conditions: getWeatherCondition(data.current.weather_code),
    location: name,
  };
};
 
function getWeatherCondition(code: number): string {
  const conditions: Record<number, string> = {
    0: "Clear sky",
    1: "Mainly clear",
    2: "Partly cloudy",
    3: "Overcast",
    45: "Foggy",
    48: "Depositing rime fog",
    51: "Light drizzle",
    53: "Moderate drizzle",
    55: "Dense drizzle",
    56: "Light freezing drizzle",
    57: "Dense freezing drizzle",
    61: "Slight rain",
    63: "Moderate rain",
    65: "Heavy rain",
    66: "Light freezing rain",
    67: "Heavy freezing rain",
    71: "Slight snow fall",
    73: "Moderate snow fall",
    75: "Heavy snow fall",
    77: "Snow grains",
    80: "Slight rain showers",
    81: "Moderate rain showers",
    82: "Violent rain showers",
    85: "Slight snow showers",
    86: "Heavy snow showers",
    95: "Thunderstorm",
    96: "Thunderstorm with slight hail",
    99: "Thunderstorm with heavy hail",
  };
  return conditions[code] || "Unknown";
}
Create an Agent
Create a weather agent file:

mkdir -p src/mastra/agents && touch src/mastra/agents/weather.ts

Then, add the following code to src/mastra/agents/weather.ts:

src/mastra/agents/weather.ts

import { openai } from "@ai-sdk/openai";
import { Agent } from "@mastra/core/agent";
import { weatherTool } from "../tools/weather-tool";
 
export const weatherAgent = new Agent({
  name: "Weather Agent",
  instructions: `You are a helpful weather assistant that provides accurate weather information.
 
Your primary function is to help users get weather details for specific locations. When responding:
- Always ask for a location if none is provided
- Include relevant details like humidity, wind conditions, and precipitation
- Keep responses concise but informative
 
Use the weatherTool to fetch current weather data.`,
  model: openai("gpt-4o-mini"),
  tools: { weatherTool },
});
Register Agent
Finally, create the Mastra entry point in src/mastra/index.ts and register agent:

src/mastra/index.ts

import { Mastra } from "@mastra/core";
 
import { weatherAgent } from "./agents/weather";
 
export const mastra = new Mastra({
  agents: { weatherAgent },
});
This registers your agent with Mastra so that mastra dev can discover and serve it.

Existing Project Installation
To add Mastra to an existing project, see our Local dev docs on mastra init.

You can also checkout our framework specific docs e.g Next.js

Start the Mastra Server
Mastra provides commands to serve your agents via REST endpoints

Development Server
Run the following command to start the Mastra server:

npm run dev

If you have the mastra CLI installed, run:

mastra dev

This command creates REST API endpoints for your agents.

Test the Endpoint
You can test the agent’s endpoint using curl or fetch:

curl -X POST http://localhost:4111/api/agents/weatherAgent/generate \
-H "Content-Type: application/json" \
-d '{"messages": ["What is the weather in London?"]}'

Use Mastra on the Client
To use Mastra in your frontend applications, you can use our type-safe client SDK to interact with your Mastra REST APIs.

See our Client SDK documentation for detailed usage instructions.

Run from the command line
If you’d like to directly call agents from the command line, you can create a script to get an agent and call it:

src/index.ts

import { mastra } from "./mastra";
 
async function main() {
  const agent = await mastra.getAgent("weatherAgent");
 
  const result = await agent.generate("What is the weather in London?");
 
  console.log("Agent response:", result.text);
}
 
main();
Then, run the script to test that everything is set up correctly:

npx tsx src/index.ts

This should output the agent’s response to your console.

Project Structure
This page provides a guide for organizing folders and files in Mastra. Mastra is a modular framework, and you can use any of the modules separately or together.

You could write everything in a single file (as we showed in the quick start), or separate each agent, tool, and workflow into their own files.

We don’t enforce a specific folder structure, but we do recommend some best practices, and the CLI will scaffold a project with a sensible structure.

Using the CLI
mastra init is an interactive CLI that allows you to:

Choose a directory for Mastra files: Specify where you want the Mastra files to be placed (default is src/mastra).
Select components to install: Choose which components you want to include in your project:
Agents
Tools
Workflows
Select a default LLM provider: Choose from supported providers like OpenAI, Anthropic, or Groq.
Include example code: Decide whether to include example code to help you get started.


Example Project Structure
Assuming you select all components and include example code, your project structure will look like this:

Top-level Folders
Folder	Description
src/mastra	Core application folder
src/mastra/agents	Agent configurations and definitions
src/mastra/tools	Custom tool definitions
src/mastra/workflows	Workflow definitions
Top-level Files
File	Description
src/mastra/index.ts	Main configuration file for Mastra
.env	Environment variables

Agents Guide: Building a Chef Assistant
In this guide, we’ll walk through creating a “Chef Assistant” agent that helps users cook meals with available ingredients.

Prerequisites
Node.js installed
Mastra installed: npm install @mastra/core
Create the Agent
Define the Agent
Create a new file src/mastra/agents/chefAgent.ts and define your agent:

src/mastra/agents/chefAgent.ts

import { openai } from "@ai-sdk/openai";
import { Agent } from "@mastra/core/agent";
 
export const chefAgent = new Agent({
  name: "chef-agent",
  instructions:
    "You are Michel, a practical and experienced home chef" +
    "You help people cook with whatever ingredients they have available.",
  model: openai("gpt-4o-mini"),
});
Set Up Environment Variables
Create a .env file in your project root and add your OpenAI API key:

.env

OPENAI_API_KEY=your_openai_api_key
Register the Agent with Mastra
In your main file, register the agent:

src/mastra/index.ts

import { Mastra } from "@mastra/core";
 
import { chefAgent } from "./agents/chefAgent";
 
export const mastra = new Mastra({
  agents: { chefAgent },
});
Interacting with the Agent
Generating Text Responses
src/index.ts

async function main() {
  const query =
    "In my kitchen I have: pasta, canned tomatoes, garlic, olive oil, and some dried herbs (basil and oregano). What can I make?";
  console.log(`Query: ${query}`);
 
  const response = await chefAgent.generate([{ role: "user", content: query }]);
  console.log("\n👨‍🍳 Chef Michel:", response.text);
}
 
main();
Run the script:

npx bun src/index.ts

Output:

Query: In my kitchen I have: pasta, canned tomatoes, garlic, olive oil, and some dried herbs (basil and oregano). What can I make?
👨‍🍳 Chef Michel: You can make a delicious pasta al pomodoro! Here's how...
Streaming Responses
src/index.ts

async function main() {
  const query =
    "Now I'm over at my friend's house, and they have: chicken thighs, coconut milk, sweet potatoes, and some curry powder.";
  console.log(`Query: ${query}`);
 
  const stream = await chefAgent.stream([{ role: "user", content: query }]);
 
  console.log("\n Chef Michel: ");
 
  for await (const chunk of stream.textStream) {
    process.stdout.write(chunk);
  }
 
  console.log("\n\n✅ Recipe complete!");
}
 
main();
Output:

Query: Now I'm over at my friend's house, and they have: chicken thighs, coconut milk, sweet potatoes, and some curry powder.
👨‍🍳 Chef Michel:
Great! You can make a comforting chicken curry...
✅ Recipe complete!
Generating a Recipe with Structured Data
src/index.ts

import { z } from "zod";
 
async function main() {
  const query =
    "I want to make lasagna, can you generate a lasagna recipe for me?";
  console.log(`Query: ${query}`);
 
  // Define the Zod schema
  const schema = z.object({
    ingredients: z.array(
      z.object({
        name: z.string(),
        amount: z.string(),
      }),
    ),
    steps: z.array(z.string()),
  });
 
  const response = await chefAgent.generate(
    [{ role: "user", content: query }],
    { output: schema },
  );
  console.log("\n👨‍🍳 Chef Michel:", response.object);
}
 
main();
Output:

Query: I want to make lasagna, can you generate a lasagna recipe for me?
👨‍🍳 Chef Michel: {
  ingredients: [
    { name: "Lasagna noodles", amount: "12 sheets" },
    { name: "Ground beef", amount: "1 pound" },
    // ...
  ],
  steps: [
    "Preheat oven to 375°F (190°C).",
    "Cook the lasagna noodles according to package instructions.",
    // ...
  ]
}
Running the Agent Server
Using mastra dev
You can run your agent as a service using the mastra dev command:

mastra dev

This will start a server exposing endpoints to interact with your registered agents.

Accessing the Chef Assistant API
By default, mastra dev runs on http://localhost:4111. Your Chef Assistant agent will be available at:

POST http://localhost:4111/api/agents/chefAgent/generate
Interacting with the Agent via curl
You can interact with the agent using curl from the command line:

curl -X POST http://localhost:4111/api/agents/chefAgent/generate \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "I have eggs, flour, and milk. What can I make?"
      }
    ]
  }'

Sample Response:

{
  "text": "You can make delicious pancakes! Here's a simple recipe..."

  Stock Agent
We’re going to create a simple agent that fetches the last day’s closing stock price for a given symbol. This example will show you how to create a tool, add it to an agent, and use the agent to fetch stock prices.

Project Structure
stock-price-agent/
├── src/
│   ├── agents/
│   │   └── stockAgent.ts
│   ├── tools/
│   │   └── stockPrices.ts
│   └── index.ts
├── package.json
└── .env
Initialize the Project and Install Dependencies
First, create a new directory for your project and navigate into it:

mkdir stock-price-agent
cd stock-price-agent
Initialize a new Node.js project and install the required dependencies:

npm init -y
npm install @mastra/core zod
Set Up Environment Variables

Create a .env file at the root of your project to store your OpenAI API key.

.env

OPENAI_API_KEY=your_openai_api_key
Create the necessary directories and files:

mkdir -p src/agents src/tools
touch src/agents/stockAgent.ts src/tools/stockPrices.ts src/index.ts
Create the Stock Price Tool
Next, we’ll create a tool that fetches the last day’s closing stock price for a given symbol.

src/tools/stockPrices.ts
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
 
const getStockPrice = async (symbol: string) => {
  const data = await fetch(
    `https://mastra-stock-data.vercel.app/api/stock-data?symbol=${symbol}`,
  ).then((r) => r.json());
  return data.prices["4. close"];
};
 
export const stockPrices = createTool({
  id: "Get Stock Price",
  inputSchema: z.object({
    symbol: z.string(),
  }),
  description: `Fetches the last day's closing stock price for a given symbol`,
  execute: async ({ context: { symbol } }) => {
    console.log("Using tool to fetch stock price for", symbol);
    return {
      symbol,
      currentPrice: await getStockPrice(symbol),
    };
  },
});
Add the Tool to an Agent
We’ll create an agent and add the stockPrices tool to it.

src/agents/stockAgent.ts
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
 
import * as tools from "../tools/stockPrices";
 
export const stockAgent = new Agent<typeof tools>({
  name: "Stock Agent",
  instructions:
    "You are a helpful assistant that provides current stock prices. When asked about a stock, use the stock price tool to fetch the stock price.",
  model: openai("gpt-4o-mini"),
  tools: {
    stockPrices: tools.stockPrices,
  },
});
Set Up the Mastra Instance
We need to initialize the Mastra instance with our agent and tool.

src/index.ts
import { Mastra } from "@mastra/core";
 
import { stockAgent } from "./agents/stockAgent";
 
export const mastra = new Mastra({
  agents: { stockAgent },
});
Serve the Application
Instead of running the application directly, we’ll use the mastra dev command to start the server. This will expose your agent via REST API endpoints, allowing you to interact with it over HTTP.

In your terminal, start the Mastra server by running:

mastra dev --dir src
This command will allow you to test your stockPrices tool and your stockAgent within the playground.

This will also start the server and make your agent available at:

http://localhost:4111/api/agents/stockAgent/generate
Test the Agent with cURL
Now that your server is running, you can test your agent’s endpoint using curl:

curl -X POST http://localhost:4111/api/agents/stockAgent/generate \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "What is the current stock price of Apple (AAPL)?" }
    ]
  }'
Expected Response:

You should receive a JSON response similar to:

{
  "text": "The current price of Apple (AAPL) is $174.55.",
  "agent": "Stock Agent"
}
This indicates that your agent successfully processed the request, used the stockPrices tool to fetch the stock price, and returned the result.

Introduction
In this guide, you’ll learn how Mastra helps you build workflows with LLMs.

We’ll walk through creating a workflow that gathers information from a candidate’s resume, then branches to either a technical or behavioral question based on the candidate’s profile. Along the way, you’ll see how to structure workflow steps, handle branching, and integrate LLM calls.

Below is a concise version of the workflow. It starts by importing the necessary modules, sets up Mastra, defines steps to extract and classify candidate data, and then asks suitable follow-up questions. Each code block is followed by a short explanation of what it does and why it’s useful.

1. Imports and Setup
You need to import Mastra tools and Zod to handle workflow definitions and data validation.

src/mastra/index.ts

 
import { Mastra } from "@mastra/core";
import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
Add your OPENAI_API_KEY to the .env file.

.env

OPENAI_API_KEY=<your-openai-key>
2. Step One: Gather Candidate Info
You want to extract candidate details from the resume text and classify them as technical or non-technical. This step calls an LLM to parse the resume and return structured JSON, including the name, technical status, specialty, and the original resume text. The code reads resumeText from trigger data, prompts the LLM, and returns organized fields for use in subsequent steps.

src/mastra/index.ts

import { Agent } from '@mastra/core/agent';
import { openai } from "@ai-sdk/openai";
 
const recruiter = new Agent({
  instructions: `You are a recruiter.`,
  model: openai("gpt-4o-mini"),
})
 
const gatherCandidateInfo = new Step({
  id: "gatherCandidateInfo",
  inputSchema: z.object({
    resumeText: z.string(),
  }),
  outputSchema: z.object({
    candidateName: z.string(),
    isTechnical: z.boolean(),
    specialty: z.string(),
    resumeText: z.string(),
  }),
  execute: async ({ context }) => {
    const resumeText = context?.getStepResult<{
      resumeText: string;
    }>("trigger")?.resumeText;
 
    const prompt = `
          Extract details from the resume text:
          "${resumeText}"
        `;
 
    const res = await recruiter.generate(prompt, {
      output: z.object({
        candidateName: z.string(),
        isTechnical: z.boolean(),
        specialty: z.string(),
        resumeText: z.string(),
      }),
    });
 
    return res.object;
  },
});
3. Technical Question Step
This step prompts a candidate who is identified as technical for more information about how they got into their specialty. It uses the entire resume text so the LLM can craft a relevant follow-up question. The code generates a question about the candidate’s specialty.

src/mastra/index.ts

interface CandidateInfo {
  candidateName: string;
  isTechnical: boolean;
  specialty: string;
  resumeText: string;
}
 
const askAboutSpecialty = new Step({
  id: "askAboutSpecialty",
  outputSchema: z.object({
    question: z.string(),
  }),
  execute: async ({ context }) => {
    const candidateInfo = context?.getStepResult<CandidateInfo>(
      "gatherCandidateInfo",
    );
 
    const prompt = `
          You are a recruiter. Given the resume below, craft a short question
          for ${candidateInfo?.candidateName} about how they got into "${candidateInfo?.specialty}".
          Resume: ${candidateInfo?.resumeText}
        `;
    const res = await recruiter.generate(prompt);
 
    return { question: res?.text?.trim() || "" };
  },
});
4. Behavioral Question Step
If the candidate is non-technical, you want a different follow-up question. This step asks what interests them most about the role, again referencing their complete resume text. The code solicits a role-focused query from the LLM.

src/mastra/index.ts

const askAboutRole = new Step({
  id: "askAboutRole",
  outputSchema: z.object({
    question: z.string(),
  }),
  execute: async ({ context }) => {
    const candidateInfo = context?.getStepResult<CandidateInfo>(
      "gatherCandidateInfo",
    );
 
    const prompt = `
          You are a recruiter. Given the resume below, craft a short question
          for ${candidateInfo?.candidateName} asking what interests them most about this role.
          Resume: ${candidateInfo?.resumeText}
        `;
    const res = await recruiter.generate(prompt);
    return { question: res?.text?.trim() || "" };
  },
});
5. Define the Workflow
You now combine the steps to implement branching logic based on the candidate’s technical status. The workflow first gathers candidate data, then either asks about their specialty or about their role, depending on isTechnical. The code chains gatherCandidateInfo with askAboutSpecialty and askAboutRole, and commits the workflow.

src/mastra/index.ts

const candidateWorkflow = new Workflow({
  name: "candidate-workflow",
  triggerSchema: z.object({
    resumeText: z.string(),
  }),
});
 
candidateWorkflow
  .step(gatherCandidateInfo)
  .then(askAboutSpecialty, {
    when: { "gatherCandidateInfo.isTechnical": true },
  })
  .after(gatherCandidateInfo)
  .step(askAboutRole, {
    when: { "gatherCandidateInfo.isTechnical": false },
  });
 
candidateWorkflow.commit();
6. Execute the Workflow
src/mastra/index.ts

const mastra = new Mastra({
  workflows: {
    candidateWorkflow,
  },
});
 
(async () => {
  const { runId, start } = mastra.getWorkflow("candidateWorkflow").createRun();
 
  console.log("Run", runId);
 
  const runResult = await start({
    triggerData: { resumeText: "Simulated resume content..." },
  });
 
  console.log("Final output:", runResult.results);
})();
You’ve just built a workflow to parse a resume and decide which question to ask based on the candidate’s technical abilities. Congrats and happy hacking!

AI SDK
Mastra Leverages AI SDK’s model routing (a unified interface on top of OpenAI, Anthropic, etc), structured output, and tool calling. We explain this in greater detail in this blog post

Mastra + AI SDK
Mastra acts as a layer on top of AI SDK to help teams productionize their proof-of-concepts quickly and easily.

Agent interaction trace showing spans, LLM calls, and tool executions
Model routing
When creating agents in Mastra, you can specify any AI SDK-supported model:

import { openai } from "@ai-sdk/openai";
import { Agent } from "@mastra/core/agent";
 
const agent = new Agent({
  name: "WeatherAgent",
  instructions: "Instructions for the agent...",
  model: openai("gpt-4-turbo"), // Model comes directly from AI SDK
});
 
const result = await agent.generate("What is the weather like?");
AI SDK Hooks
Mastra is compatible with AI SDK’s hooks for seamless frontend integration:

useChat
The useChat hook enables real-time chat interactions in your frontend application

Works with agent data streams i.e. .toDataStreamResponse()
The useChat api defaults to /api/chat
Works with the Mastra REST API agent stream endpoint {MASTRA_BASE_URL}/agents/:agentId/stream for data streams, i.e. no structured output is defined.
app/api/chat/route.ts

 
import { mastra } from '@/src/mastra';
 
export async function POST(req: Request) {
  const { messages } = await req.json();
  const myAgent = mastra.getAgent('weatherAgent');
  const stream = await myAgent.stream(messages);
 
  return stream.toDataStreamResponse();
}
 
import { useChat } from '@ai-sdk/react';
 
export function ChatComponent() {
  const { messages, input, handleInputChange, handleSubmit } = useChat({
    api: '/path-to-your-agent-stream-api-endpoint'
  });
 
  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role}: {m.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Say something..."
        />
      </form>
    </div>
  );
}

Gotcha: When using useChat with agent memory functionality, make sure to check out the Agent Memory section for important implementation details.

useCompletion
For single-turn completions, use the useCompletion hook:

Works with agent data streams i.e. .toDataStreamResponse()
The useCompletion api defaults to /api/completion
Works with the Mastra REST API agent stream endpoint {MASTRA_BASE_URL}/agents/:agentId/stream for data streams, i.e. no structured output is defined.
app/api/completion/route.ts

 
import { mastra } from '@/src/mastra';
 
export async function POST(req: Request) {
  const { messages } = await req.json();
  const myAgent = mastra.getAgent('weatherAgent');
  const stream = await myAgent.stream(messages);
 
  return stream.toDataStreamResponse();
}
 
import { useCompletion } from "@ai-sdk/react";
 
export function CompletionComponent() {
  const {
    completion,
    input,
    handleInputChange,
    handleSubmit,
  } = useCompletion({
  api: '/path-to-your-agent-stream-api-endpoint'
  });
 
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Enter a prompt..."
        />
      </form>
      <p>Completion result: {completion}</p>
    </div>
  );
}
useObject
For consuming text streams that represent JSON objects and parsing them into a complete object based on a schema.

Works with agent text streams i.e. .toTextStreamResponse()
The useObject api defaults to /api/completion
Works with the Mastra REST API agent stream endpoint {MASTRA_BASE_URL}/agents/:agentId/stream for text streams, i.e. structured output is defined.
app/api/use-object/route.ts

 
import { mastra } from '@/src/mastra';
 
export async function POST(req: Request) {
  const { messages } = await req.json();
  const myAgent = mastra.getAgent('weatherAgent');
  const stream = await myAgent.stream(messages, {
    output: z.object({
      weather: z.string(),
    }),
  });
 
  return stream.toTextStreamResponse();
}
import { experimental_useObject as useObject } from '@ai-sdk/react';
 
export default function Page() {
  const { object, submit } = useObject({
    api: '/api/use-object',
    schema: z.object({
      weather: z.string(),
    }),
  });
 
  return (
    <div>
      <button onClick={() => submit('example input')}>Generate</button>
      {object?.content && <p>{object.content}</p>}
    </div>
  );
}
Tool Calling
AI SDK Tool Format
Mastra supports tools created using the AI SDK format, so you can use them directly with Mastra agents. See our tools doc on Vercel AI SDK Tool Format for more details.

Client-side tool calling
Mastra leverages AI SDK’s tool calling, so what applies in AI SDK applies here still. Agent Tools in Mastra are 100% percent compatible with AI SDK tools.

Mastra tools also expose an optional execute async function. It is optional because you might want to forward tool calls to the client or to a queue instead of executing them in the same process.

One way to then leverage client-side tool calling is to use the @ai-sdk/react useChat hook’s onToolCall property for client-side tool execution

Integrate Mastra in your Next.js project
There are two main ways to integrate Mastra with your Next.js application: as a separate backend service or directly integrated into your Next.js app.

1. Separate Backend Integration
Best for larger projects where you want to:

Scale your AI backend independently
Keep clear separation of concerns
Have more deployment flexibility
Create Mastra Backend
Create a new Mastra project using our CLI:

npm create mastra

nstall MastraClient
npm install @mastra/client-js

Use MastraClient
Create a client instance and use it in your Next.js application:

lib/mastra.ts

import { MastraClient } from '@mastra/client-js';
 
// Initialize the client
export const mastraClient = new MastraClient({
  baseUrl: process.env.NEXT_PUBLIC_MASTRA_API_URL || 'http://localhost:4111',
});
Example usage in your React component:

app/components/SimpleWeather.tsx

'use client'
 
import { mastraClient } from '@/lib/mastra'
 
export function SimpleWeather() {
  async function handleSubmit(formData: FormData) {
    const city = formData.get('city')
    const agent = mastraClient.getAgent('weatherAgent')
    
    try {
      const response = await agent.generate({
        messages: [{ role: 'user', content: `What's the weather like in ${city}?` }],
      })
      // Handle the response
      console.log(response.text)
    } catch (error) {
      console.error('Error:', error)
    }
  }
 
  return (
    <form action={handleSubmit}>
      <input name="city" placeholder="Enter city name" />
      <button type="submit">Get Weather</button>
    </form>
  )
}
Deployment
When you’re ready to deploy, you can use any of our platform-specific deployers (Vercel, Netlify, Cloudflare) or deploy to any Node.js hosting platform. Check our deployment guide for detailed instructions.

2. Direct Integration
Better for smaller projects or prototypes. This approach bundles Mastra directly with your Next.js application.

Initialize Mastra in your Next.js Root
First, navigate to your Next.js project root and initialize Mastra:

cd your-nextjs-app

Then run the initialization command:

npx mastra@latest init

This will set up Mastra in your Next.js project. For more details about init and other configuration options, see our mastra init reference.

Configure Next.js
Add to your next.config.js:

next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  serverExternalPackages: ["@mastra/*"],
  // ... your other Next.js config
}
 
module.exports = nextConfig
Server Actions Example
app/actions.ts

'use server'
 
import { mastra } from '@/mastra'
 
export async function getWeatherInfo(city: string) {
  const agent = mastra.getAgent('weatherAgent')
  
  const result = await agent.generate(`What's the weather like in ${city}?`)
 
  return result
}
Use it in your component:

app/components/Weather.tsx

'use client'
 
import { getWeatherInfo } from '../actions'
 
export function Weather() {
  async function handleSubmit(formData: FormData) {
    const city = formData.get('city') as string
    const result = await getWeatherInfo(city)
    // Handle the result
    console.log(result)
  }
 
  return (
    <form action={handleSubmit}>
      <input name="city" placeholder="Enter city name" />
      <button type="submit">Get Weather</button>
    </form>
  )
}
API Routes Example
app/api/chat/route.ts

import { mastra } from '@/mastra'
import { NextResponse } from 'next/server'
 
export async function POST(req: Request) {
  const { city } = await req.json()
  const agent = mastra.getAgent('weatherAgent')
 
  const result = await agent.stream(`What's the weather like in ${city}?`)
 
  return result.toDataStreamResponse()
}
Deployment
When using direct integration, your Mastra instance will be deployed alongside your Next.js application. Ensure you:

Set up environment variables for your LLM API keys in your deployment platform
Implement proper error handling for production use
Monitor your AI agent’s performance and costs
Observability
Mastra provides built-in observability features to help you monitor, debug, and optimize your AI operations. This includes:

Tracing of AI operations and their performance
Logging of prompts, completions, and errors
Integration with observability platforms like Langfuse and LangSmith
For detailed setup instructions and configuration options specific to Next.js local development, see our Next.js Observability Configuration Guide.

Creating and Calling Agents
Agents in Mastra are systems where the language model can autonomously decide on a sequence of actions to perform tasks. They have access to tools, workflows, and synced data, enabling them to perform complex tasks and interact with external systems. Agents can invoke your custom functions, utilize third-party APIs through integrations, and access knowledge bases you have built.

Agents are like employees who can be used for ongoing projects. They have names, persistent memory, consistent model configurations, and instructions across calls, as well as a set of enabled tools.

1. Creating an Agent
To create an agent in Mastra, you use the Agent class and define its properties:

src/mastra/agents/index.ts

import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
 
export const myAgent = new Agent({
  name: "My Agent",
  instructions: "You are a helpful assistant.",
  model: openai("gpt-4o-mini"),
});
Note: Ensure that you have set the necessary environment variables, such as your OpenAI API key, in your .env file:

.env

OPENAI_API_KEY=your_openai_api_key
Also, make sure you have the @mastra/core package installed:

npm install @mastra/core

Registering the Agent
Register your agent with Mastra to enable logging and access to configured tools and integrations:

src/mastra/index.ts

import { Mastra } from "@mastra/core";
import { myAgent } from "./agents";
 
export const mastra = new Mastra({
  agents: { myAgent },
});
2. Generating and streaming text
Generating text
Use the .generate() method to have your agent produce text responses:

src/mastra/index.ts

const response = await myAgent.generate([
  { role: "user", content: "Hello, how can you assist me today?" },
]);
 
console.log("Agent:", response.text);
Streaming responses
For more real-time responses, you can stream the agent’s response:

src/mastra/index.ts

const stream = await myAgent.stream([
  { role: "user", content: "Tell me a story." },
]);
 
console.log("Agent:");
 
for await (const chunk of stream.textStream) {
  process.stdout.write(chunk);
}
3. Structured Output
Agents can return structured data by providing a JSON Schema or using a Zod schema.

Using JSON Schema
const schema = {
  type: "object",
  properties: {
    summary: { type: "string" },
    keywords: { type: "array", items: { type: "string" } },
  },
  additionalProperties: false,
  required: ["summary", "keywords"],
};
 
const response = await myAgent.generate(
  [
    {
      role: "user",
      content:
        "Please provide a summary and keywords for the following text: ...",
    },
  ],
  {
    output: schema,
  },
);
 
console.log("Structured Output:", response.object);
Using Zod
You can also use Zod schemas for type-safe structured outputs.

First, install Zod:

npm install zod

Then, define a Zod schema and use it with the agent:

src/mastra/index.ts

import { z } from "zod";
 
// Define the Zod schema
const schema = z.object({
  summary: z.string(),
  keywords: z.array(z.string()),
});
 
// Use the schema with the agent
const response = await myAgent.generate(
  [
    {
      role: "user",
      content:
        "Please provide a summary and keywords for the following text: ...",
    },
  ],
  {
    output: schema,
  },
);
 
console.log("Structured Output:", response.object);
This allows you to have strong typing and validation for the structured data returned by the agent.

4. Running Agents
Mastra provides a CLI command mastra dev to run your agents behind an API. By default, this looks for exported agents in files in the src/mastra/agents directory.

Starting the Server
mastra dev
This will start the server and make your agent available at http://localhost:4111/api/agents/myAgent/generate.

Interacting with the Agent
You can interact with the agent using curl from the command line:

curl -X POST http://localhost:4111/api/agents/myAgent/generate \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "Hello, how can you assist me today?" }
    ]
  }'
Next Steps
Learn about Agent Memory in the Agent Memory guide.
Learn about Agent Tools in the Agent Tools guide.
See an example agent in the Chef Michel example.

Agent Memory
Agents in Mastra have a sophisticated memory system that stores conversation history and contextual information. This memory system supports both traditional message storage and vector-based semantic search, enabling agents to maintain state across interactions and retrieve relevant historical context.

Threads and Resources
In Mastra, you can organize conversations by a thread_id. This allows the system to maintain context and retrieve historical messages that belong to the same discussion.

Mastra also supports the concept of a resource_id, which typically represents the user involved in the conversation, ensuring that the agent’s memory and context are correctly associated with the right entity.

This separation allows you to manage multiple conversations (threads) for a single user or even share conversation context across users if needed.

import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
 
const agent = new Agent({
  name: "Project Manager",
  instructions:
    "You are a project manager. You are responsible for managing the project and the team.",
  model: openai("gpt-4o-mini"),
});
 
await agent.stream("When will the project be completed?", {
  threadId: "project_123",
  resourceId: "user_123",
});

Managing Conversation Context
The key to getting good responses from LLMs is feeding them the right context.

Mastra has a Memory API that stores and manages conversation history and contextual information. The Memory API uses a storage backend to persist conversation history and contextual information (more on this later).

The Memory API uses two main mechanisms to maintain context in conversations, recent message history and semantic search.

Recent Message History
By default, Memory keeps track of the 40 most recent messages in a conversation. You can customize this with the lastMessages setting:

const memory = new Memory({
  options: {
    lastMessages: 5, // Keep 5 most recent messages
  },
});
 
// When user asks this question, the agent will see the last 10 messages,
await agent.stream("Can you summarize the search feature requirements?", {
  memoryOptions: {
    lastMessages: 10,
  },
});

Semantic Search
Semantic search is enabled by default in Mastra. While FastEmbed (bge-small-en-v1.5) and LibSQL are included by default, you can use any embedder (like OpenAI or Cohere) and vector database (like PostgreSQL, Pinecone, or Chroma) that fits your needs.

This allows your agent to find and recall relevant information from earlier in the conversation:

const memory = new Memory({
  options: {
    semanticRecall: {
      topK: 10, // Include 10 most relevant past messages
      messageRange: 2, // Messages before and after each result
    },
  },
});
 
// Example: User asks about a past feature discussion
await agent.stream("What did we decide about the search feature last week?", {
  memoryOptions: {
    lastMessages: 10,
    semanticRecall: {
      topK: 3,
      messageRange: 2,
    },
  },
});

When semantic search is used:

The message is converted to a vector embedding
Similar messages are found using vector similarity search
Surrounding context is included based on messageRange
All relevant context is provided to the agent
You can also customize the vector database and embedder:

import { openai } from "@ai-sdk/openai";
import { PgVector } from "@mastra/pg";
 
const memory = new Memory({
  // Use a different vector database (libsql is default)
  vector: new PgVector("postgresql://user:pass@localhost:5432/db"),
  // Or a different embedder (fastembed is default)
  embedder: openai.embedding("text-embedding-3-small"),
});

Memory Configuration
The Mastra memory system is highly configurable and supports multiple storage backends. By default, it uses LibSQL for storage and vector search, and FastEmbed for embeddings.

Basic Configuration
For most use cases, you can use the default configuration:

import { Memory } from "@mastra/memory";
 
const memory = new Memory();

Custom Configuration
For more control, you can customize the storage backend, vector database, and memory options:

import { Memory } from "@mastra/memory";
import { PostgresStore, PgVector } from "@mastra/pg";
 
const memory = new Memory({
  storage: new PostgresStore({
    host: "localhost",
    port: 5432,
    user: "postgres",
    database: "postgres",
    password: "postgres",
  }),
  vector: new PgVector("postgresql://user:pass@localhost:5432/db"),
  options: {
    // Number of recent messages to include (false to disable)
    lastMessages: 10,
    // Configure vector-based semantic search (false to disable)
    semanticRecall: {
      topK: 3, // Number of semantic search results
      messageRange: 2, // Messages before and after each result
    },
  },
});

Overriding Memory Settings
When you initialize a Mastra instance with memory configuration, all agents will automatically use these memory settings when you call their stream() or generate() methods. You can override these default settings for individual calls:

// Use default memory settings from Memory configuration
const response1 = await agent.generate("What were we discussing earlier?", {
  resourceId: "user_123",
  threadId: "thread_456",
});
 
// Override memory settings for this specific call
const response2 = await agent.generate("What were we discussing earlier?", {
  resourceId: "user_123",
  threadId: "thread_456",
  memoryOptions: {
    lastMessages: 5, // Only inject 5 recent messages
    semanticRecall: {
      topK: 2, // Only get 2 semantic search results
      messageRange: 1, // Context around each result
    },
  },
});

Configuring Memory for Different Use Cases
You can adjust memory settings based on your agent’s needs:

// Customer support agent with minimal context
await agent.stream("What are your store hours?", {
  threadId,
  resourceId,
  memoryOptions: {
    lastMessages: 5, // Quick responses need minimal conversation history
    semanticRecall: false, // no need to search through earlier messages
  },
});
 
// Project management agent with extensive context
await agent.stream("Update me on the project status", {
  threadId,
  resourceId,
  memoryOptions: {
    lastMessages: 50, // Maintain longer conversation history across project discussions
    semanticRecall: {
      topK: 5, // Find more relevant project details
      messageRange: 3, // Number of messages before and after each result
    },
  },
});

Storage Options
Mastra currently supports several storage backends:

LibSQL Storage
import { LibSQLStore } from "@mastra/core/storage/libsql";
 
const storage = new LibSQLStore({
  config: {
    url: "file:example.db",
  },
});

PostgreSQL Storage
import { PostgresStore } from "@mastra/pg";
 
const storage = new PostgresStore({
  host: "localhost",
  port: 5432,
  user: "postgres",
  database: "postgres",
  password: "postgres",
});

Upstash KV Storage
import { UpstashStore } from "@mastra/upstash";
 
const storage = new UpstashStore({
  url: "http://localhost:8089",
  token: "your_token",
});

Vector Search
Mastra supports semantic search through vector embeddings. When configured with a vector store, agents can find relevant historical messages based on semantic similarity. To enable vector search:

Configure a vector store (currently supports PostgreSQL):
import { PgVector } from "@mastra/pg";
 
const vector = new PgVector(connectionString);
 
const memory = new Memory({ vector });

Configure embedding options:
const memory = new Memory({
  vector,
  embedder: openai.embedding("text-embedding-3-small"),
});

Enable vector search in memory configuration options:
const memory = new Memory({
  vector,
  embedder,
 
  options: {
    semanticRecall: {
      topK: 3, // Number of similar messages to find
      messageRange: 2, // Context around each result
    },
  },
});

Using Memory in Agents
Once configured, the memory system is automatically used by agents. Here’s how to use it:

// Initialize Agent with memory
const myAgent = new Agent({
  memory,
  // other agent options
});
// Add agent to mastra
const mastra = new Mastra({
  agents: { myAgent },
});
 
// Memory is automatically used in agent interactions when resourceId and threadId are added
const response = await myAgent.generate(
  "What were we discussing earlier about performance?",
  {
    resourceId: "user_123",
    threadId: "thread_456",
  },
);

The memory system will automatically:

Store all messages in the configured storage backend
Create vector embeddings for semantic search (if configured)
Inject relevant historical context into new conversations
Maintain conversation threads and context
useChat()
When using useChat from the AI SDK, you must send only the latest message or you will encounter message ordering bugs.

If the useChat() implementation for your framework supports experimental_prepareRequestBody, you can do the following:

const { messages } = useChat({
  api: "api/chat",
  experimental_prepareRequestBody({ messages, id }) {
    return { message: messages.at(-1), id };
  },
});
This will only ever send the latest message to the server. In your chat server endpoint you can then pass a threadId and resourceId when calling stream or generate and the agent will have access to the memory thread messages:

const { messages } = await request.json();
 
const stream = await myAgent.stream(messages, {
  threadId,
  resourceId,
});
 
return stream.toDataStreamResponse();
If the useChat() for your framework (svelte for example) doesn’t support experimental_prepareRequestBody, you can pick and use the last message before calling stream or generate:

const { messages } = await request.json();
 
const stream = await myAgent.stream([messages.at(-1)], {
  threadId,
  resourceId,
});
 
return stream.toDataStreamResponse();
See the AI SDK documentation on message persistence for more information.

Manually Managing Threads
While threads are automatically managed when using agent methods, you can also manually manage threads using the memory API directly. This is useful for advanced use cases like:

Creating threads before starting conversations
Managing thread metadata
Explicitly saving or retrieving messages
Cleaning up old threads
Here’s how to manually work with threads:

import { Memory } from "@mastra/memory";
import { PostgresStore } from "@mastra/pg";
 
// Initialize memory
const memory = new Memory({
  storage: new PostgresStore({
    host: "localhost",
    port: 5432,
    user: "postgres",
    database: "postgres",
    password: "postgres",
  }),
});
 
// Create a new thread
const thread = await memory.createThread({
  resourceId: "user_123",
  title: "Project Discussion",
  metadata: {
    project: "mastra",
    topic: "architecture",
  },
});
 
// Manually save messages to a thread
await memory.saveMessages({
  messages: [
    {
      id: "msg_1",
      threadId: thread.id,
      role: "user",
      content: "What's the project status?",
      createdAt: new Date(),
      type: "text",
    },
  ],
});
 
// Get messages from a thread with various filters
const messages = await memory.query({
  threadId: thread.id,
  selectBy: {
    last: 10, // Get last 10 messages
    vectorSearchString: "performance", // Find messages about performance
  },
});
 
// Get thread by ID
const existingThread = await memory.getThreadById({
  threadId: "thread_123",
});
 
// Get all threads for a resource
const threads = await memory.getThreadsByResourceId({
  resourceId: "user_123",
});
 
// Update thread metadata
await memory.updateThread({
  id: thread.id,
  title: "Updated Project Discussion",
  metadata: {
    status: "completed",
  },
});
 
// Delete a thread and all its messages
await memory.deleteThread(thread.id);

Note that in most cases, you won’t need to manage threads manually since the agent’s generate() and stream() methods handle thread management automatically. Manual thread management is primarily useful for advanced use cases or when you need more fine-grained control over the conversation history.

Working Memory
Working memory is a powerful feature that allows agents to maintain persistent information across conversations, even with minimal context. This is particularly useful for remembering user preferences, personal details, or any other contextual information that should persist throughout interactions.

Inspired by the working memory concept from the MemGPT whitepaper, our implementation improves upon it in several key ways:

No extra roundtrips or tool calls required
Full support for streaming messages
Seamless integration with the agent’s natural response flow
How It Works
Working memory operates through a system of XML tags and automatic updates:

Template Structure: Define what information should be remembered using XML tags. The Memory class comes with a comprehensive default template for user information, or you can create your own template to match your specific needs.

Automatic Updates: The Memory class injects special instructions into the agent’s system prompt that tell it to:

Store relevant information by including <working_memory>...</working_memory> tags in its responses
Update information proactively when anything changes
Maintain the XML structure while updating values
Keep this process invisible to users
Memory Management: The system:

Extracts working memory blocks from agent responses
Stores them for future use
Injects working memory into the system prompt on the next agent call
The agent is instructed to be proactive about storing information - if there’s any doubt about whether something might be useful later, it should be stored. This helps maintain conversation context even when using very small context windows.

Basic Usage
import { openai } from "@ai-sdk/openai";
 
const agent = new Agent({
  name: "Customer Service",
  instructions:
    "You are a helpful customer service agent. Remember customer preferences and past interactions.",
  model: openai("gpt-4o-mini"),
 
  memory: new Memory({
    options: {
      workingMemory: {
        enabled: true, // enables working memory
      },
      lastMessages: 5, // Only keep recent context
    },
  }),
});

Working memory becomes particularly powerful when combined with specialized system prompts. For example, you could create a TODO list manager that maintains state even though it only has access to the previous message:

const todoAgent = new Agent({
  name: "TODO Manager",
  instructions:
    "You are a TODO list manager. Update the todo list in working memory whenever tasks are added, completed, or modified.",
  model: openai("gpt-4o-mini"),
  memory: new Memory({
    options: {
      workingMemory: {
        enabled: true,
 
        // optional XML-like template to encourage agent to store specific kinds of info.
        // if you leave this out a default template will be used
        template: `<todos>
  <in-progress></in-progress>
  <pending></pending>
  <completed></completed>
</todos>`,
      },
      lastMessages: 1, // Only keep the last message in context
    },
  }),
});

Handling Memory Updates in Streaming
When an agent responds, it includes working memory updates directly in its response stream. These updates appear as XML blocks in the text:

// Raw agent response stream:
Let me help you with that! <working_memory><user><first_name>John</first_name>...</user></working_memory> Based on your question...

To prevent these memory blocks from being visible to users while still allowing the system to process them, use the maskStreamTags utility:

import { maskStreamTags } from "@mastra/core/utils";
 
// Basic usage - just mask the working_memory tags
for await (const chunk of maskStreamTags(
  response.textStream,
  "working_memory",
)) {
  process.stdout.write(chunk);
}
 
// Without masking: "Let me help you! <working_memory>...</working_memory> Based on..."
// With masking: "Let me help you! Based on..."

You can also hook into memory update events:

const maskedStream = maskStreamTags(response.textStream, "working_memory", {
  onStart: () => showLoadingSpinner(),
  onEnd: () => hideLoadingSpinner(),
  onMask: (chunk) => console.debug(chunk),
});

The maskStreamTags utility:

Removes content between specified XML tags in a streaming response
Optionally provides lifecycle callbacks for memory updates
Handles tags that might be split across stream chunks
Accessing Thread and Resource IDs in Tools
When creating custom tools, you can access the threadId and resourceId directly in the tool’s execute function. These parameters are automatically provided by the Mastra runtime:

import { Memory } from "@mastra/memory";
const memory = new Memory();
 
const myTool = createTool({
  id: "Thread Info Tool",
  inputSchema: z.object({
    fetchMessages: z.boolean().optional(),
  }),
  description: "A tool that demonstrates accessing thread and resource IDs",
  execute: async ({ threadId, resourceId, context }) => {
    // threadId and resourceId are directly available in the execute parameters
    console.log(`Executing in thread ${threadId}`);
 
    if (!context.fetchMessages) {
      return { threadId, resourceId };
    }
 
    const recentMessages = await memory.query({
      threadId,
      selectBy: { last: 5 },
    });
 
    return {
      threadId,
      resourceId,
      messageCount: recentMessages.length,
    };
  },
});

This allows tools to:

Access the current conversation context
Store or retrieve thread-specific data
Associate tool actions with specific users/resources
Maintain state across multiple tool invocations


Agent Tool Selection
Tools are typed functions that can be executed by agents or workflows, with built-in integration access and parameter validation. Each tool has a schema that defines its inputs, an executor function that implements its logic, and access to configured integrations.

Creating Tools
In this section, we’ll walk through the process of creating a tool that can be used by your agents. Let’s create a simple tool that fetches current weather information for a given city.

src/mastra/tools/weatherInfo.ts

import { createTool } from "@mastra/core/tools";
import { z } from "zod";
 
const getWeatherInfo = async (city: string) => {
  // Replace with an actual API call to a weather service
  const data = await fetch(`https://api.example.com/weather?city=${city}`).then(
    (r) => r.json(),
  );
  return data;
};
 
export const weatherInfo = createTool({
  id: "Get Weather Information",
  inputSchema: z.object({
    city: z.string(),
  }),
  description: `Fetches the current weather information for a given city`,
  execute: async ({ context: { city } }) => {
    console.log("Using tool to fetch weather information for", city);
    return await getWeatherInfo(city);
  },
});
Adding Tools to an Agent
Now we’ll add the tool to an agent. We’ll create an agent that can answer questions about the weather and configure it to use our weatherInfo tool.

src/mastra/agents/weatherAgent.ts
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import * as tools from "../tools/weatherInfo";
 
export const weatherAgent = new Agent<typeof tools>({
  name: "Weather Agent",
  instructions:
    "You are a helpful assistant that provides current weather information. When asked about the weather, use the weather information tool to fetch the data.",
  model: openai("gpt-4o-mini"),
  tools: {
    weatherInfo: tools.weatherInfo,
  },
});
Registering the Agent
We need to initialize Mastra with our agent.

src/index.ts
import { Mastra } from "@mastra/core";
import { weatherAgent } from "./agents/weatherAgent";
 
export const mastra = new Mastra({
  agents: { weatherAgent },
});
This registers your agent with Mastra, making it available for use.

Debugging Tools
You can test tools using Vitest or any other testing framework. Writing unit tests for your tools ensures they behave as expected and helps catch errors early.

Calling an Agent with a Tool
Now we can call the agent, and it will use the tool to fetch the weather information.

Example: Interacting with the Agent
src/index.ts
import { mastra } from "./index";
 
async function main() {
  const agent = mastra.getAgent("weatherAgent");
  const response = await agent.generate(
    "What's the weather like in New York City today?",
  );
 
  console.log(response.text);
}
 
main();
The agent will use the weatherInfo tool to get the current weather in New York City and respond accordingly.

Vercel AI SDK Tool Format
Mastra supports tools created using the Vercel AI SDK format. You can import and use these tools directly:

src/mastra/tools/vercelTool.ts

import { tool } from 'ai';
import { z } from 'zod';
 
export const weatherInfo = tool({
  description: "Fetches the current weather information for a given city",
  parameters: z.object({
    city: z.string().describe("The city to get weather for")
  }),
  execute: async ({ city }) => {
    // Replace with actual API call
    const data = await fetch(`https://api.example.com/weather?city=${city}`);
    return data.json();
  }
});
You can use Vercel tools alongside Mastra tools in your agents:

src/mastra/agents/weatherAgent.ts
import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { weatherInfo } from "../tools/vercelTool";
import * as mastraTools from "../tools/mastraTools";
 
export const weatherAgent = new Agent({
  name: "Weather Agent",
  instructions: "You are a helpful assistant that provides weather information.",
  model: openai("gpt-4"),
  tools: {
    weatherInfo,  // Vercel tool
    ...mastraTools  // Mastra tools
  },
});
Both tool formats will work seamlessly within your agent’s workflow.

Tool Design Best Practices
When creating tools for your agents, following these guidelines will help ensure reliable and intuitive tool usage:

Tool Descriptions
Your tool’s main description should focus on its purpose and value:

Keep descriptions simple and focused on what the tool does
Emphasize the tool’s primary use case
Avoid implementation details in the main description
Focus on helping the agent understand when to use the tool
createTool({
  id: "documentSearch",
  description: "Access the knowledge base to find information needed to answer user questions",
  // ... rest of tool configuration
});
Parameter Schemas
Technical details belong in the parameter schemas, where they help the agent use the tool correctly:

Make parameters self-documenting with clear descriptions
Include default values and their implications
Provide examples where helpful
Describe the impact of different parameter choices
inputSchema: z.object({
  query: z.string().describe("The search query to find relevant information"),
  limit: z.number().describe(
    "Number of results to return. Higher values provide more context, lower values focus on best matches"
  ),
  options: z.string().describe(
    "Optional configuration. Example: '{'filter': 'category=news'}'"
  ),
}),
Agent Interaction Patterns
Tools are more likely to be used effectively when:

Queries or tasks are complex enough to clearly require tool assistance
Agent instructions provide clear guidance on tool usage
Parameter requirements are well-documented in the schema
The tool’s purpose aligns with the query’s needs
Common Pitfalls
Overloading the main description with technical details
Mixing implementation details with usage guidance
Unclear parameter descriptions or missing examples
Following these practices helps ensure your tools are discoverable and usable by agents while maintaining clean separation between purpose (main description) and implementation details (parameter schemas).



Adding Voice to Agents
Mastra agents can be enhanced with voice capabilities, allowing them to speak responses and listen to user input. You can configure an agent to use either a single voice provider or combine multiple providers for different operations.

Using a Single Provider
The simplest way to add voice to an agent is to use a single provider for both speaking and listening:

import { createReadStream } from "fs";
import path from "path";
import { Agent } from "@mastra/core/agent";
import { OpenAIVoice } from "@mastra/voice-openai";
import { openai } from "@ai-sdk/openai";
 
// Initialize the voice provider with default settings
const voice = new OpenAIVoice();
 
// Create an agent with voice capabilities
export const agent = new Agent({
  name: "Agent",
  instructions: `You are a helpful assistant with both STT and TTS capabilities.`,
  model: openai("gpt-4o"),
  voice,
});
 
// The agent can now use voice for interaction
await agent.speak("Hello, I'm your AI assistant!");
 
// Read audio file and transcribe
const audioFilePath = path.join(process.cwd(), "/audio.m4a");
const audioStream = createReadStream(audioFilePath);
 
try {
  const transcription = await agent.listen(audioStream, { filetype: "m4a" });
} catch (error) {
  console.error("Error transcribing audio:", error);
}
Using Multiple Providers
For more flexibility, you can use different providers for speaking and listening using the CompositeVoice class:

import { Agent } from "@mastra/core/agent";
import { CompositeVoice } from "@mastra/core/voice";
import { OpenAIVoice } from "@mastra/voice-openai";
import { PlayAIVoice } from "@mastra/voice-playai";
import { openai } from "@ai-sdk/openai";
 
export const agent = new Agent({
  name: "Agent",
  instructions: `You are a helpful assistant with both STT and TTS capabilities.`,
  model: openai("gpt-4o"),
 
  // Create a composite voice using OpenAI for listening and PlayAI for speaking
  voice: new CompositeVoice({
    listenProvider: new OpenAIVoice(),
    speakProvider: new PlayAIVoice(),
  }),
});
Working with Audio Streams
The speak() and listen() methods work with Node.js streams. Here’s how to save and load audio files:

Saving Speech Output
import { createWriteStream } from "fs";
import path from "path";
 
// Generate speech and save to file
const audio = await agent.speak("Hello, World!");
const filePath = path.join(process.cwd(), "agent.mp3");
const writer = createWriteStream(filePath);
 
audio.pipe(writer);
 
await new Promise<void>((resolve, reject) => {
  writer.on("finish", () => resolve());
  writer.on("error", reject);
});
Transcribing Audio Input
import { createReadStream } from "fs";
import path from "path";
 
// Read audio file and transcribe
const audioFilePath = path.join(process.cwd(), "/agent.m4a");
const audioStream = createReadStream(audioFilePath);
 
try {
  console.log("Transcribing audio file...");
  const transcription = await agent.listen(audioStream, { filetype: "m4a" });
  console.log("Transcription:", transcription);
} catch (error) {
  console.error("Error transcribing audio:", error);
}
Real-time Voice Interactions
For more dynamic and interactive voice experiences, you can use real-time voice providers that support speech-to-speech capabilities:

import { Agent } from "@mastra/core/agent";
import { OpenAIRealtimeVoice } from "@mastra/voice-openai-realtime";
import { search, calculate } from "../tools";
 
// Initialize the realtime voice provider
const voice = new OpenAIRealtimeVoice({
  chatModel: {
    apiKey: process.env.OPENAI_API_KEY,
    model: 'gpt-4o-mini-realtime',
  },
  speaker: 'alloy'
});
 
// Create an agent with speech-to-speech voice capabilities
export const agent = new Agent({
  name: 'Agent',
  instructions: `You are a helpful assistant with speech-to-speech capabilities.`,
  model: openai('gpt-4o'),
  tools: { // Tools configured on Agent are passed to voice provider
    search,
    calculate
  },
  voice
});
 
// Establish a WebSocket connection
await voice.connect();
 
// Start a conversation
await voice.speak("Hello, I'm your AI assistant!");
 
// Stream audio from a microphone
const microphoneStream = getMicrophoneStream();
await voice.send(microphoneStream);
 
// When done with the conversation
voice.close();
Event System
The realtime voice provider emits several events you can listen for:

// Listen for speech audio data sent from voice provider
voice.on('speaking', ({ audio }) => {
  // audio contains ReadableStream or Int16Array audio data
});
 
// Listen for transcribed text sent from both voice provider and user
voice.on('writing', ({ text, role }) => {
  console.log(`${role} said: ${text}`);
});
 
// Listen for errors
voice.on('error', (error) => {
  console.error('Voice error:', error);
});

Handling Complex LLM Operations with Workflows
Workflows in Mastra help you orchestrate complex sequences of operations with features like branching, parallel execution, resource suspension, and more.

When to use workflows
Most AI applications need more than a single call to a language model. You may want to run multiple steps, conditionally skip certain paths, or even pause execution altogether until you receive user input. Sometimes your agent tool calling is not accurate enough.

Mastra’s workflow system provides:

A standardized way to define steps and link them together.
Support for both simple (linear) and advanced (branching, parallel) paths.
Debugging and observability features to track each workflow run.
Example
To create a workflow, you define one or more steps, link them, and then commit the workflow before starting it.

Breaking Down the Workflow
Let’s examine each part of the workflow creation process:

1. Creating the Workflow
Here’s how you define a workflow in Mastra. The name field determines the workflow’s API endpoint (/workflows/$NAME/), while the triggerSchema defines the structure of the workflow’s trigger data:

src/mastra/workflow/index.ts
const myWorkflow = new Workflow({
  name: 'my-workflow',
  triggerSchema: z.object({
    inputValue: z.number(),
  }),
});
2. Defining Steps
Now, we’ll define the workflow’s steps. Each step can have its own input and output schemas. Here, stepOne doubles an input value, and stepTwo increments that result if stepOne was successful. (To keep things simple, we aren’t making any LLM calls in this example):

src/mastra/workflow/index.ts
const stepOne = new Step({
  id: 'stepOne',
  outputSchema: z.object({
    doubledValue: z.number(),
  }),
  execute: async ({ context }) => {
    const doubledValue = context.triggerData.inputValue * 2;
    return { doubledValue };
  },
});
 
const stepTwo = new Step({
  id: "stepTwo",
  execute: async ({ context }) => {
    const doubledValue = context.getStepResult(stepOne)?.doubledValue;
    if (!doubledValue) {
      return { incrementedValue: 0 };
    }
    return {
      incrementedValue: doubledValue + 1,
    };
  },
});
3. Linking Steps
Now, let’s create the control flow, and “commit” (finalize the workflow). In this case, stepOne runs first and is followed by stepTwo.

src/mastra/workflow/index.ts
myWorkflow
  .step(stepOne)
  .then(stepTwo)
  .commit();
Register the Workflow
Register your workflow with Mastra to enable logging and telemetry:

src/mastra/index.ts
import { Mastra } from "@mastra/core";
 
export const mastra = new Mastra({
  workflows: { myWorkflow },
});
The workflow can also have the mastra instance injected into the context in the case where you need to create dynamic workflows:

src/mastra/workflow/index.ts
import { Mastra } from "@mastra/core";
 
const mastra = new Mastra();
 
const myWorkflow = new Workflow({
  name: 'my-workflow',
  mastra,
});
Executing the Workflow
Execute your workflow programmatically or via API:

src/mastra/run-workflow.ts

import { mastra } from "./index";
 
// Get the workflow
const myWorkflow = mastra.getWorkflow('myWorkflow');
const { runId, start } = myWorkflow.createRun();
 
// Start the workflow execution
await start({ triggerData: { inputValue: 45 } });
Or use the API (requires running mastra dev):

curl --location 'http://localhost:4111/api/workflows/myWorkflow/execute' \
     --header 'Content-Type: application/json' \
     --data '{
       "inputValue": 45
     }'
This example shows the essentials: define your workflow, add steps, commit the workflow, then execute it.

Defining Steps
The basic building block of a workflow is a step. Steps are defined using schemas for inputs and outputs, and can fetch prior step results.

Control Flow
Workflows let you define a control flow to chain steps together in with parallel steps, branching paths, and more.

Workflow Variables
When you need to map data between steps or create dynamic data flows, workflow variables provide a powerful mechanism for passing information from one step to another and accessing nested properties within step outputs.

Suspend and Resume
When you need to pause execution for external data, user input, or asynchronous events, Mastra supports suspension at any step, persisting the state of the workflow so you can resume it later.

Observability and Debugging
Mastra workflows automatically log the input and output of each step within a workflow run, allowing you to send this data to your preferred logging, telemetry, or observability tools.

You can:

Track the status of each step (e.g., success, error, or suspended).
Store run-specific metadata for analysis.
Integrate with third-party observability platforms like Datadog or New Relic by forwarding logs.
More Resources
The Workflow Guide in the Guides section is a tutorial that covers the main concepts.
Sequential Steps workflow example
Parallel Steps workflow example
Branching Paths workflow example
Workflow Variables example
Cyclical Dependencies workflow example
Suspend and Resume workflow example


Defining Steps in a Workflow
When you build a workflow, you typically break down operations into smaller tasks that can be linked and reused. Steps provide a structured way to manage these tasks by defining inputs, outputs, and execution logic.

The code below shows how to define these steps inline or separately.

Inline Step Creation
You can create steps directly within your workflow using .step() and .then(). This code shows how to define, link, and execute two steps in sequence.

src/mastra/workflows/index.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
export const myWorkflow = new Workflow({
  name: "my-workflow",
  triggerSchema: z.object({
    inputValue: z.number(),
  }),
});
 
myWorkflow
  .step(
    new Step({
      id: "stepOne",
      outputSchema: z.object({
        doubledValue: z.number(),
      }),
      execute: async ({ context }) => ({
        doubledValue: context.triggerData.inputValue * 2,
      }),
    }),
  )
  .then(
    new Step({
      id: "stepTwo",
      outputSchema: z.object({
        incrementedValue: z.number(),
      }),
      execute: async ({ context }) => {
        if (context.steps.stepOne.status !== "success") {
          return { incrementedValue: 0 };
        }
 
        return { incrementedValue: context.steps.stepOne.output.doubledValue + 1 };
      },
    }),
  ).commit();
Creating Steps Separately
If you prefer to manage your step logic in separate entities, you can define steps outside and then add them to your workflow. This code shows how to define steps independently and link them afterward.

src/mastra/workflows/index.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Define steps separately
const stepOne = new Step({
  id: "stepOne",
  outputSchema: z.object({
    doubledValue: z.number(),
  }),
  execute: async ({ context }) => ({
    doubledValue: context.triggerData.inputValue * 2,
  }),
});
 
const stepTwo = new Step({
  id: "stepTwo",
  outputSchema: z.object({
    incrementedValue: z.number(),
  }),
  execute: async ({ context }) => {
    if (context.steps.stepOne.status !== "success") {
      return { incrementedValue: 0 };
    }
    return { incrementedValue: context.steps.stepOne.output.doubledValue + 1 };
  },
});
 
// Build the workflow
const myWorkflow = new Workflow({
  name: "my-workflow",
  triggerSchema: z.object({
    inputValue: z.number(),
  }),
});
 
myWorkflow.step(stepOne).then(stepTwo);
myWorkflow.commit();


Control Flow in Workflows: Branching, Merging, and Conditions
When you create a multi-step process, you may need to run steps in parallel, chain them sequentially, or follow different paths based on outcomes. This page describes how you can manage branching, merging, and conditions to construct workflows that meet your logic requirements. The code snippets show the key patterns for structuring complex control flow.

Parallel Execution
You can run multiple steps at the same time if they don’t depend on each other. This approach can speed up your workflow when steps perform independent tasks. The code below shows how to add two steps in parallel:

myWorkflow.step(fetchUserData).step(fetchOrderData);
See the Parallel Steps example for more details.

Sequential Execution
Sometimes you need to run steps in strict order to ensure outputs from one step become inputs for the next. Use .then() to link dependent operations. The code below shows how to chain steps sequentially:

myWorkflow.step(fetchOrderData).then(validateData).then(processOrder);
See the Sequential Steps example for more details.

Branching and Merging Paths
When different outcomes require different paths, branching is helpful. You can also merge paths later once they complete. The code below shows how to branch after stepA and later converge on stepF:

myWorkflow
  .step(stepA)
    .then(stepB)
    .then(stepD)
  .after(stepA)
    .step(stepC)
    .then(stepE)
  .after(stepD)
    .step(stepF);
    .step(stepE)
In this example:

stepA leads to stepB, then to stepD.
Separately, stepA also triggers stepC, which in turn leads to stepE.
Separately, stepD also triggers stepF and stepE in parallel.
See the Branching Paths example for more details.

Merging Multiple Branches
Sometimes you need a step to execute only after multiple other steps have completed. Mastra provides a compound .after([]) syntax that allows you to specify multiple dependencies for a step.

myWorkflow
  .step(fetchUserData)
  .then(validateUserData)
  .step(fetchProductData)
  .then(validateProductData)
  // This step will only run after BOTH validateUserData AND validateProductData have completed
  .after([validateUserData, validateProductData])
  .step(processOrder)
In this example:

fetchUserData and fetchProductData run in parallel branches
Each branch has its own validation step
The processOrder step only executes after both validation steps have completed successfully
This pattern is particularly useful for:

Joining parallel execution paths
Implementing synchronization points in your workflow
Ensuring all required data is available before proceeding
You can also create complex dependency patterns by combining multiple .after([]) calls:

myWorkflow
  // First branch
  .step(stepA)
  .then(stepB)
  .then(stepC)
 
  // Second branch
  .step(stepD)
  .then(stepE)
 
  // Third branch
  .step(stepF)
  .then(stepG)
 
  // This step depends on the completion of multiple branches
  .after([stepC, stepE, stepG])
  .step(finalStep)
Cyclical Dependencies and Loops
Workflows often need to repeat steps until certain conditions are met. Mastra provides two powerful methods for creating loops: until and while. These methods offer an intuitive way to implement repetitive tasks.

Using Manual Cyclical Dependencies (Legacy Approach)
In earlier versions, you could create loops by manually defining cyclical dependencies with conditions:

myWorkflow
  .step(fetchData)
  .then(processData)
  .after(processData)
  .step(finalizeData, {
    when: { "processData.status": "success" },
  })
  .step(fetchData, {
    when: { "processData.status": "retry" },
  });
While this approach still works, the newer until and while methods provide a cleaner and more maintainable way to create loops.

Using until for Condition-Based Loops
The until method repeats a step until a specified condition becomes true. It takes two arguments:

A condition that determines when to stop looping
The step to repeat
workflow
  .step(incrementStep)
  .until(async ({ context }) => {
    // Stop when the value reaches or exceeds 10
    const result = context.getStepResult(incrementStep);
    return (result?.value ?? 0) >= 10;
  }, incrementStep)
  .then(finalStep);
You can also use a reference-based condition:

workflow
  .step(incrementStep)
  .until(
    {
      ref: { step: incrementStep, path: 'value' },
      query: { $gte: 10 },
    },
    incrementStep
  )
  .then(finalStep);
Using while for Condition-Based Loops
The while method repeats a step as long as a specified condition remains true. It takes the same arguments as until:

A condition that determines when to continue looping
The step to repeat
workflow
  .step(incrementStep)
  .while(async ({ context }) => {
    // Continue as long as the value is less than 10
    const result = context.getStepResult(incrementStep);
    return (result?.value ?? 0) < 10;
  }, incrementStep)
  .then(finalStep);
You can also use a reference-based condition:

workflow
  .step(incrementStep)
  .while(
    {
      ref: { step: incrementStep, path: 'value' },
      query: { $lt: 10 },
    },
    incrementStep
  )
  .then(finalStep);
Comparison Operators for Reference Conditions
When using reference-based conditions, you can use these comparison operators:

Operator	Description
$eq	Equal to
$ne	Not equal to
$gt	Greater than
$gte	Greater than or equal to
$lt	Less than
$lte	Less than or equal to
See the Loop Control example for more details.

Conditions
Use the when property to control whether a step runs based on data from previous steps. Below are three ways to specify conditions.

Option 1: Function
myWorkflow.step(
  new Step({
    id: "processData",
    execute: async ({ context }) => {
      // Action logic
    },
  }),
  {
    when: async ({ context }) => {
      const fetchData = context?.getStepResult<{ status: string }>("fetchData");
      return fetchData?.status === "success";
    },
  },
);
Option 2: Query Object
myWorkflow.step(
  new Step({
    id: "processData",
    execute: async ({ context }) => {
      // Action logic
    },
  }),
  {
    when: {
      ref: {
        step: {
          id: "fetchData",
        },
        path: "status",
      },
      query: { $eq: "success" },
    },
  },
);
Option 3: Simple Path Comparison
myWorkflow.step(
  new Step({
    id: "processData",
    execute: async ({ context }) => {
      // Action logic
    },
  }),
  {
    when: {
      "fetchData.status": "success",
    },
  },
);
Accessing Previous Step Results
Steps access data from previous steps through the context object. The context contains a record of all step results and their payloads.

Using getStepResult
getStepResult retrieves a step’s output with type safety:

workflow.step(
  new Step({
    id: "processOrder",
    execute: async ({ context }) => {
      const userData = context.getStepResult<{ userId: string }>("fetchUser");
      return {
        userId: userData?.userId,
        status: "processing"
      };
    },
  })
);

Using Path Notation
Path notation accesses step results through the machine context. For example, to access the status of the processOrder step:

workflow.step(
  new Step({
    id: "sendEmail",
    execute: async ({ context }) => {
      const orderStatus = context.steps.processOrder.output.status;
      console.log(orderStatus);
    },
  })
);

The context object maintains type information when used with TypeScript. Nested objects in step outputs can be accessed with either method.


Data Flow Between Steps
In Mastra workflows, passing data between steps is essential for creating complex, interconnected processes. This guide explains the different methods for accessing data from previous steps and ensuring proper type safety throughout your workflow.

Overview of Data Flow Methods
Mastra provides several ways to pass data between steps:

Context Object - Access step results directly through the context object
Variable Mapping - Explicitly map outputs from one step to inputs of another
getStepResult Method - Type-safe method to retrieve step outputs
Each approach has its advantages depending on your use case and requirements for type safety.

Using getStepResult Method
The getStepResult method provides a type-safe way to access step results. This is the recommended approach when working with TypeScript as it preserves type information.

Basic Usage
For better type safety, you can provide a type parameter to getStepResult:

src/mastra/workflows/get-step-result.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
const fetchUserStep = new Step({
  id: 'fetchUser',
  outputSchema: z.object({
    name: z.string(),
    userId: z.string(),
  }),
  execute: async ({ context }) => {
    return { name: 'John Doe', userId: '123' };
  },
});
 
const analyzeDataStep = new Step({
  id: "analyzeData",
  execute: async ({ context }) => {
    // Type-safe access to previous step result
    const userData = context.getStepResult<{ name: string, userId: string }>("fetchUser");
 
    if (!userData) {
      return { status: "error", message: "User data not found" };
    }
 
    return {
      analysis: `Analyzed data for user ${userData.name}`,
      userId: userData.userId
    };
  },
});
Using Step References
The most type-safe approach is to reference the step directly in the getStepResult call:

src/mastra/workflows/step-reference.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Define step with output schema
const fetchUserStep = new Step({
  id: "fetchUser",
  outputSchema: z.object({
    userId: z.string(),
    name: z.string(),
    email: z.string(),
  }),
  execute: async () => {
    return {
      userId: "user123",
      name: "John Doe",
      email: "john@example.com"
    };
  },
});
 
const processUserStep = new Step({
  id: "processUser",
  execute: async ({ context }) => {
    // TypeScript will infer the correct type from fetchUserStep's outputSchema
    const userData = context.getStepResult(fetchUserStep);
 
    return {
      processed: true,
      userName: userData?.name
    };
  },
});
 
const workflow = new Workflow({
  name: "user-workflow",
});
 
workflow
  .step(fetchUserStep)
  .then(processUserStep)
  .commit();
Using Variable Mapping
Variable mapping is an explicit way to define data flow between steps. This approach makes dependencies clear and provides good type safety.

src/mastra/workflows/variable-mapping.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
const fetchUserStep = new Step({
  id: "fetchUser",
  outputSchema: z.object({
    userId: z.string(),
    name: z.string(),
    email: z.string(),
  }),
  execute: async () => {
    return {
      userId: "user123",
      name: "John Doe",
      email: "john@example.com"
    };
  },
});
 
const sendEmailStep = new Step({
  id: "sendEmail",
  inputSchema: z.object({
    recipientEmail: z.string(),
    recipientName: z.string(),
  }),
  execute: async ({ context }) => {
    const { recipientEmail, recipientName } = context;
 
    // Send email logic here
    return {
      status: "sent",
      to: recipientEmail
    };
  },
});
 
const workflow = new Workflow({
  name: "email-workflow",
});
 
workflow
  .step(fetchUserStep)
  .then(sendEmailStep, {
    variables: {
      // Map specific fields from fetchUser to sendEmail inputs
      recipientEmail: { step: fetchUserStep, path: 'email' },
      recipientName: { step: fetchUserStep, path: 'name' }
    }
  })
  .commit();
For more details on variable mapping, see the Data Mapping with Workflow Variables documentation.

Using the Context Object
The context object provides direct access to all step results and their outputs. This approach is more flexible but requires careful handling to maintain type safety. You can access step results directly through the context.steps object:

src/mastra/workflows/context-access.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
const processOrderStep = new Step({
  id: 'processOrder',
  execute: async ({ context }) => {
    // Access data from a previous step
    let userData: { name: string, userId: string };
    if (context.steps['fetchUser']?.status === 'success') {
      userData = context.steps.fetchUser.output;
    } else {
      throw new Error('User data not found');
    }
 
    return {
      orderId: 'order123',
      userId: userData.userId,
      status: 'processing',
    };
  },
});
 
const workflow = new Workflow({
  name: "order-workflow",
});
 
workflow
  .step(fetchUserStep)
  .then(processOrderStep)
  .commit();
Workflow-Level Type Safety
For comprehensive type safety across your entire workflow, you can define types for all steps and pass them to the Workflow This allows you to get type safety for the context object on conditions, and step results on the final workflow output.

src/mastra/workflows/workflow-typing.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
 
// Create steps with typed outputs
const fetchUserStep = new Step({
  id: "fetchUser",
  outputSchema: z.object({
    userId: z.string(),
    name: z.string(),
    email: z.string(),
  }),
  execute: async () => {
    return {
      userId: "user123",
      name: "John Doe",
      email: "john@example.com"
    };
  },
});
 
const processOrderStep = new Step({
  id: "processOrder",
  execute: async ({ context }) => {
    // TypeScript knows the shape of userData
    const userData = context.getStepResult(fetchUserStep);
 
    return {
      orderId: "order123",
      status: "processing"
    };
  },
});
 
const workflow = new Workflow<[typeof fetchUserStep, typeof processOrderStep]>({
  name: "typed-workflow",
});
 
workflow
  .step(fetchUserStep)
  .then(processOrderStep)
  .until(async ({ context }) => {
    // TypeScript knows the shape of userData here
    const res = context.getStepResult('fetchUser');
    return res?.userId === '123';
  }, processOrderStep)
  .commit();
Accessing Trigger Data
In addition to step results, you can access the original trigger data that started the workflow:

src/mastra/workflows/trigger-data.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Define trigger schema
const triggerSchema = z.object({
  customerId: z.string(),
  orderItems: z.array(z.string()),
});
 
type TriggerType = z.infer<typeof triggerSchema>;
 
const processOrderStep = new Step({
  id: "processOrder",
  execute: async ({ context }) => {
    // Access trigger data with type safety
    const triggerData = context.getStepResult<TriggerType>('trigger');
 
    return {
      customerId: triggerData?.customerId,
      itemCount: triggerData?.orderItems.length || 0,
      status: "processing"
    };
  },
});
 
const workflow = new Workflow({
  name: "order-workflow",
  triggerSchema,
});
 
workflow
  .step(processOrderStep)
  .commit();
Accessing Workflow Results
You can get typed access to the results of a workflow by injecting the step types into the Workflow type params:

src/mastra/workflows/get-results.ts

import { Workflow } from "@mastra/core/workflows";
 
const fetchUserStep = new Step({
  id: "fetchUser",
  outputSchema: z.object({
    userId: z.string(),
    name: z.string(),
    email: z.string(),
  }),
  execute: async () => {
    return {
      userId: "user123",
      name: "John Doe",
      email: "john@example.com"
    };
  },
});
 
const processOrderStep = new Step({
  id: "processOrder",
  outputSchema: z.object({
    orderId: z.string(),
    status: z.string(),
  }),
  execute: async ({ context }) => {
    const userData = context.getStepResult(fetchUserStep);
    return {
      orderId: "order123",
      status: "processing"
    };
  },
});
 
const workflow = new Workflow<[typeof fetchUserStep, typeof processOrderStep]>({
  name: "typed-workflow",
});
 
workflow
  .step(fetchUserStep)
  .then(processOrderStep)
  .commit();
 
const run = workflow.createRun();
const result = await run.start();
 
// The result is a discriminated union of the step results
// So it needs to be narrowed down via status checks
if (result.results.processOrder.status === 'success') {
  // TypeScript will know the shape of the results
  const orderId = result.results.processOrder.output.orderId;
  console.log({orderId});
}
 
if (result.results.fetchUser.status === 'success') {
  const userId = result.results.fetchUser.output.userId;
  console.log({userId});
}
Best Practices for Data Flow
Use getStepResult with Step References for Type Safety

Ensures TypeScript can infer the correct types
Catches type errors at compile time
*Use Variable Mapping for Explicit Dependencies

Makes data flow clear and maintainable
Provides good documentation of step dependencies
Define Output Schemas for Steps

Validates data at runtime
Validates return type of the execute function
Improves type inference in TypeScript
Handle Missing Data Gracefully

Always check if step results exist before accessing properties
Provide fallback values for optional data
Keep Data Transformations Simple

Transform data in dedicated steps rather than in variable mappings
Makes workflows easier to test and debug
Comparison of Data Flow Methods
Method	Type Safety	Explicitness	Use Case
getStepResult	Highest	High	Complex workflows with strict typing requirements
Variable Mapping	High	High	When dependencies need to be clear and explicit
context.steps	Medium	Low	Quick access to step data in simple workflows
By choosing the right data flow method for your use case, you can create workflows that are both type-safe and maintainable.


Data Mapping with Workflow Variables
Workflow variables in Mastra provide a powerful mechanism for mapping data between steps, allowing you to create dynamic data flows and pass information from one step to another.

Understanding Workflow Variables
In Mastra workflows, variables serve as a way to:

Map data from trigger inputs to step inputs
Pass outputs from one step to inputs of another step
Access nested properties within step outputs
Create more flexible and reusable workflow steps
Using Variables for Data Mapping
Basic Variable Mapping
You can map data between steps using the variables property when adding a step to your workflow:

src/mastra/workflows/index.ts

const workflow = new Workflow({
  name: 'data-mapping-workflow',
  triggerSchema: z.object({
    inputData: z.string(),
  }),
});
 
workflow
  .step(step1, {
    variables: {
      // Map trigger data to step input
      inputData: { step: 'trigger', path: 'inputData' }
    }
  })
  .then(step2, {
    variables: {
      // Map output from step1 to input for step2
      previousValue: { step: step1, path: 'outputField' }
    }
  })
  .commit();
Accessing Nested Properties
You can access nested properties using dot notation in the path field:

src/mastra/workflows/index.ts

workflow
  .step(step1)
  .then(step2, {
    variables: {
      // Access a nested property from step1's output
      nestedValue: { step: step1, path: 'nested.deeply.value' }
    }
  })
  .commit();
Mapping Entire Objects
You can map an entire object by using . as the path:

src/mastra/workflows/index.ts

workflow
  .step(step1, {
    variables: {
      // Map the entire trigger data object
      triggerData: { step: 'trigger', path: '.' }
    }
  })
  .commit();
Variable Resolution
When a workflow executes, Mastra resolves variables at runtime by:

Identifying the source step specified in the step property
Retrieving the output from that step
Navigating to the specified property using the path
Injecting the resolved value into the target step’s context
Examples
Mapping from Trigger Data
This example shows how to map data from the workflow trigger to a step:

src/mastra/workflows/trigger-mapping.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Define a step that needs user input
const processUserInput = new Step({
  id: "processUserInput",
  execute: async ({ context }) => {
    // The inputData will be available in context because of the variable mapping
    const { inputData } = context;
 
    return {
      processedData: `Processed: ${inputData}`
    };
  },
});
 
// Create the workflow
const workflow = new Workflow({
  name: "trigger-mapping",
  triggerSchema: z.object({
    inputData: z.string(),
  }),
});
 
// Map the trigger data to the step
workflow
  .step(processUserInput, {
    variables: {
      inputData: { step: 'trigger', path: 'inputData' },
    }
  })
  .commit();
Mapping Between Steps
This example demonstrates mapping data from one step to another:

src/mastra/workflows/step-mapping.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Step 1: Generate data
const generateData = new Step({
  id: "generateData",
  outputSchema: z.object({
    nested: z.object({
      value: z.string(),
    }),
  }),
  execute: async () => {
    return {
      nested: {
        value: "step1-data"
      }
    };
  },
});
 
// Step 2: Process the data from step 1
const processData = new Step({
  id: "processData",
  inputSchema: z.object({
    previousValue: z.string(),
  }),
  execute: async ({ context }) => {
    // previousValue will be available because of the variable mapping
    const { previousValue } = context;
 
    return {
      result: `Processed: ${previousValue}`
    };
  },
});
 
// Create the workflow
const workflow = new Workflow({
  name: "step-mapping",
});
 
// Map data from step1 to step2
workflow
  .step(generateData)
  .then(processData, {
    variables: {
      // Map the nested.value property from generateData's output
      previousValue: { step: generateData, path: 'nested.value' },
    }
  })
  .commit();
Type Safety
Mastra provides type safety for variable mappings when using TypeScript:

src/mastra/workflows/type-safe.ts

import { Step, Workflow } from "@mastra/core/workflows";
import { z } from "zod";
 
// Define schemas for better type safety
const triggerSchema = z.object({
  inputValue: z.string(),
});
 
type TriggerType = z.infer<typeof triggerSchema>;
 
// Step with typed context
const step1 = new Step({
  id: "step1",
  outputSchema: z.object({
    nested: z.object({
      value: z.string(),
    }),
  }),
  execute: async ({ context }) => {
    // TypeScript knows the shape of triggerData
    const triggerData = context.getStepResult<TriggerType>('trigger');
 
    return {
      nested: {
        value: `processed-${triggerData?.inputValue}`
      }
    };
  },
});
 
// Create the workflow with the schema
const workflow = new Workflow({
  name: "type-safe-workflow",
  triggerSchema,
});
 
workflow.step(step1).commit();
Best Practices
Validate Inputs and Outputs: Use inputSchema and outputSchema to ensure data consistency.

Keep Mappings Simple: Avoid overly complex nested paths when possible.

Consider Default Values: Handle cases where mapped data might be undefined.

Comparison with Direct Context Access
While you can access previous step results directly via context.steps, using variable mappings offers several advantages:

Feature	Variable Mapping	Direct Context Access
Clarity	Explicit data dependencies	Implicit dependencies
Reusability	Steps can be reused with different mappings	Steps are tightly coupled
Type Safety	Better TypeScript integration	Requires manual type assertions


Suspend and Resume in Workflows
Complex workflows often need to pause execution while waiting for external input or resources.

Mastra’s suspend and resume features let you pause workflow execution at any step, persist the workflow snapshot to storage, and resume execution from the saved snapshot when ready. This entire process is automatically managed by Mastra. No config needed, or manual step required from the user.

Storing the workflow snapshot to storage (libsql by default) means that the workflow state is permanently preserved across sessions even if the workflow is suspended for a long time.

When to Use Suspend/Resume
Common scenarios for suspending workflows include:

Waiting for human approval or input
Pausing until external API resources become available
Collecting additional data needed for later steps
Rate limiting or throttling expensive operations
Basic Suspend Example
Here’s a simple workflow that suspends when a value is too low and resumes when given a higher value:

const stepTwo = new Step({
  id: "stepTwo",
  outputSchema: z.object({
    incrementedValue: z.number(),
  }),
  execute: async ({ context, suspend }) => {
    if (context.steps.stepOne.status !== "success") {
      return { incrementedValue: 0 };
    }
 
    const currentValue = context.steps.stepOne.output.doubledValue;
 
    if (currentValue < 100) {
      await suspend();
      return { incrementedValue: 0 };
    }
    return { incrementedValue: currentValue + 1 };
  },
});
Async/Await Based Flow
The suspend and resume mechanism in Mastra uses an async/await pattern that makes it intuitive to implement complex workflows with suspension points. The code structure naturally reflects the execution flow.

How It Works
A step’s execution function receives a suspend function in its parameters
When called with await suspend(), the workflow pauses at that point
The workflow state is persisted
Later, the workflow can be resumed by calling workflow.resume() with the appropriate parameters
Execution continues from the point after the suspend() call
Example with Multiple Suspension Points
Here’s an example of a workflow with multiple steps that can suspend:

// Define steps with suspend capability
const promptAgentStep = new Step({
  id: 'promptAgent',
  execute: async ({ context, suspend }) => {
    // Some condition that determines if we need to suspend
    if (needHumanInput) {
      // Optionally pass payload data that will be stored with suspended state
      await suspend({ requestReason: 'Need human input for prompt' });
      // Code after suspend() will execute when the step is resumed
      return { modelOutput: context.userInput };
    }
    return { modelOutput: 'AI generated output' };
  },
  outputSchema: z.object({ modelOutput: z.string() }),
});
 
const improveResponseStep = new Step({
  id: 'improveResponse',
  execute: async ({ context, suspend }) => {
    // Another condition for suspension
    if (needFurtherRefinement) {
      await suspend();
      return { improvedOutput: context.refinedOutput };
    }
    return { improvedOutput: 'Improved output' };
  },
  outputSchema: z.object({ improvedOutput: z.string() }),
});
 
// Build the workflow
const workflow = new Workflow({
  name: 'multi-suspend-workflow',
  triggerSchema: z.object({ input: z.string() }),
});
 
workflow
  .step(getUserInput)
  .then(promptAgentStep)
  .then(evaluateTone)
  .then(improveResponseStep)
  .then(evaluateImproved)
  .commit();
Starting and Resuming the Workflow
// Get the workflow and create a run
const wf = mastra.getWorkflow('multi-suspend-workflow');
const run = wf.createRun();
 
// Start the workflow
const initialResult = await run.start({ triggerData: { input: 'initial input' } });
 
let promptAgentStepResult = initialResult.activePaths.get('promptAgent');
let promptAgentResumeResult = undefined;
 
// Check if a step is suspended
if (promptAgentStepResult?.status === 'suspended') {
  console.log('Workflow suspended at promptAgent step');
 
  // Resume the workflow with new context
  const resumeResult = await wf.resume({
    runId: run.runId,
    stepId: 'promptAgent',
    context: { userInput: 'Human provided input' }
  });
 
  promptAgentResumeResult = resumeResult;
}
 
const improveResponseStepResult = promptAgentResumeResult?.activePaths.get('improveResponse');
 
if (improveResponseStepResult?.status === 'suspended') {
  console.log('Workflow suspended at improveResponse step');
 
  // Resume again with different context
  const finalResult = await wf.resume({
    runId: run.runId,
    stepId: 'improveResponse',
    context: { refinedOutput: 'Human refined output' }
  });
 
  console.log('Workflow completed:', finalResult?.results);
}
Key Points About Suspend and Resume
The suspend() function can optionally take a payload object that will be stored with the suspended state
Code after the await suspend() call will not execute until the step is resumed
When a step is suspended, its status becomes 'suspended' in the workflow results
When resumed, the step’s status changes from 'suspended' to 'success' once completed
The resume() method requires the runId and stepId to identify which suspended step to resume
You can provide new context data when resuming that will be merged with existing step results
Watching and Resuming
To handle suspended workflows, use the watch method to monitor workflow status and resume to continue execution:

import { mastra } from "./index";
 
// Get the workflow
const myWorkflow = mastra.getWorkflow('myWorkflow');
const { runId, start } = myWorkflow.createRun();
 
// Start watching the workflow before executing it
myWorkflow.watch(async ({ context, activePaths }) => {
  for (const _path of activePaths) {
    const stepTwoStatus = context.steps?.stepTwo?.status;
    if (stepTwoStatus === 'suspended') {
      console.log("Workflow suspended, resuming with new value");
 
      // Resume the workflow with new context
      await myWorkflow.resume({
        runId,
        stepId: 'stepTwo',
        context: { secondValue: 100 },
      });
    }
  }
})
 
// Start the workflow execution
await start({ triggerData: { inputValue: 45 } });
Related Resources
See the Suspend and Resume Example for a complete working example
Check the Step Class Reference for suspend/resume API details
Review Workflow Observability for monitoring suspended workflows

Dynamic Workflows
This guide demonstrates how to create dynamic workflows within a workflow step. This advanced pattern allows you to create and execute workflows on the fly based on runtime conditions.

Overview
Dynamic workflows are useful when you need to create workflows based on runtime data.

Implementation
The key to creating dynamic workflows is accessing the Mastra instance from within a step’s execute function and using it to create and run a new workflow.

Basic Example
import { Mastra, Step, Workflow } from '@mastra/core';
import { z } from 'zod';
 
const isMastra = (mastra: any): mastra is Mastra => {
  return mastra && typeof mastra === 'object' && mastra instanceof Mastra;
};
 
// Step that creates and runs a dynamic workflow
const createDynamicWorkflow = new Step({
  id: 'createDynamicWorkflow',
  outputSchema: z.object({
    dynamicWorkflowResult: z.any(),
  }),
  execute: async ({ context, mastra }) => {
    if (!mastra) {
      throw new Error('Mastra instance not available');
    }
 
    if (!isMastra(mastra)) {
      throw new Error('Invalid Mastra instance');
    }
 
    const inputData = context.triggerData.inputData;
 
    // Create a new dynamic workflow
    const dynamicWorkflow = new Workflow({
      name: 'dynamic-workflow',
      mastra, // Pass the mastra instance to the new workflow
      triggerSchema: z.object({
        dynamicInput: z.string(),
      }),
    });
 
    // Define steps for the dynamic workflow
    const dynamicStep = new Step({
      id: 'dynamicStep',
      execute: async ({ context }) => {
        const dynamicInput = context.triggerData.dynamicInput;
        return {
          processedValue: `Processed: ${dynamicInput}`,
        };
      },
    });
 
    // Build and commit the dynamic workflow
    dynamicWorkflow.step(dynamicStep).commit();
 
    // Create a run and execute the dynamic workflow
    const run = dynamicWorkflow.createRun();
    const result = await run.start({
      triggerData: {
        dynamicInput: inputData,
      },
    });
 
    let dynamicWorkflowResult;
 
    if (result.results['dynamicStep']?.status === 'success') {
      dynamicWorkflowResult = result.results['dynamicStep']?.output.processedValue;
    } else {
      throw new Error('Dynamic workflow failed');
    }
 
    // Return the result from the dynamic workflow
    return {
      dynamicWorkflowResult,
    };
  },
});
 
// Main workflow that uses the dynamic workflow creator
const mainWorkflow = new Workflow({
  name: 'main-workflow',
  triggerSchema: z.object({
    inputData: z.string(),
  }),
  mastra: new Mastra(),
});
 
mainWorkflow.step(createDynamicWorkflow).commit();
 
 
const run = mainWorkflow.createRun();
const result = await run.start({
  triggerData: {
    inputData: 'test',
  },
});
Advanced Example: Workflow Factory
You can create a workflow factory that generates different workflows based on input parameters:

 
const isMastra = (mastra: any): mastra is Mastra => {
  return mastra && typeof mastra === 'object' && mastra instanceof Mastra;
};
 
const workflowFactory = new Step({
  id: 'workflowFactory',
  inputSchema: z.object({
    workflowType: z.enum(['simple', 'complex']),
    inputData: z.string(),
  }),
  outputSchema: z.object({
    result: z.any(),
  }),
  execute: async ({ context, mastra }) => {
    if (!mastra) {
      throw new Error('Mastra instance not available');
    }
 
    if (!isMastra(mastra)) {
      throw new Error('Invalid Mastra instance');
    }
 
    // Create a new dynamic workflow based on the type
    const dynamicWorkflow = new Workflow({
      name: `dynamic-${context.workflowType}-workflow`,
      mastra,
      triggerSchema: z.object({
        input: z.string(),
      }),
    });
 
    if (context.workflowType === 'simple') {
      // Simple workflow with a single step
      const simpleStep = new Step({
        id: 'simpleStep',
        execute: async ({ context }) => {
          return {
            result: `Simple processing: ${context.triggerData.input}`,
          };
        },
      });
 
      dynamicWorkflow.step(simpleStep).commit();
    } else {
      // Complex workflow with multiple steps
      const step1 = new Step({
        id: 'step1',
        outputSchema: z.object({
          intermediateResult: z.string(),
        }),
        execute: async ({ context }) => {
          return {
            intermediateResult: `First processing: ${context.triggerData.input}`,
          };
        },
      });
 
      const step2 = new Step({
        id: 'step2',
        execute: async ({ context }) => {
          const intermediate = context.getStepResult(step1).intermediateResult;
          return {
            finalResult: `Second processing: ${intermediate}`,
          };
        },
      });
 
      dynamicWorkflow.step(step1).then(step2).commit();
    }
 
    // Execute the dynamic workflow
    const run = dynamicWorkflow.createRun();
    const result = await run.start({
      triggerData: {
        input: context.inputData,
      },
    });
 
    // Return the appropriate result based on workflow type
    if (context.workflowType === 'simple') {
      return {
        // @ts-ignore
        result: result.results['simpleStep']?.output,
      };
    } else {
      return {
        // @ts-ignore
        result: result.results['step2']?.output,
      };
    }
  },
});
Important Considerations
Mastra Instance: The mastra parameter in the execute function provides access to the Mastra instance, which is essential for creating dynamic workflows.

Error Handling: Always check if the Mastra instance is available before attempting to create a dynamic workflow.

Resource Management: Dynamic workflows consume resources, so be mindful of creating too many workflows in a single execution.

Workflow Lifecycle: Dynamic workflows are not automatically registered with the main Mastra instance. They exist only for the duration of the step execution unless you explicitly register them.

Debugging: Debugging dynamic workflows can be challenging. Consider adding detailed logging to track their creation and execution.

Use Cases
Conditional Workflow Selection: Choose different workflow patterns based on input data
Parameterized Workflows: Create workflows with dynamic configurations
Workflow Templates: Use templates to generate specialized workflows
Multi-tenant Applications: Create isolated workflows for different tenants
Conclusion
Dynamic workflows provide a powerful way to create flexible, adaptable workflow systems. By leveraging the Mastra instance within step execution, you can create workflows that respond to runtime conditions and requirements.


RAG (Retrieval-Augmented Generation) in Mastra
RAG in Mastra helps you enhance LLM outputs by incorporating relevant context from your own data sources, improving accuracy and grounding responses in real information.

Mastra’s RAG system provides:

Standardized APIs to process and embed documents
Support for multiple vector stores
Chunking and embedding strategies for optimal retrieval
Observability for tracking embedding and retrieval performance
Example
To implement RAG, you process your documents into chunks, create embeddings, store them in a vector database, and then retrieve relevant context at query time.

import { embedMany } from "ai";
import { openai } from "@ai-sdk/openai";
import { PgVector } from "@mastra/pg";
import { MDocument } from "@mastra/rag";
import { z } from "zod";
 
// 1. Initialize document
const doc = MDocument.fromText(`Your document text here...`);
 
// 2. Create chunks
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,
  overlap: 50,
});
 
// 3. Generate embeddings
const { embeddings } = await embedMany({
  values: chunks,
  model: openai.embedding("text-embedding-3-small"),
});
 
// 4. Store in vector database
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
await pgVector.upsert({
  indexName: "embeddings",
  vectors: embeddings,
}); // using an index name of 'embeddings'
 
// 5. Query similar chunks
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryVector,
  topK: 3,
}); // queryVector is the embedding of the query
 
console.log("Similar chunks:", results);

This example shows the essentials: initialize a document, create chunks, generate embeddings, store them, and query for similar content.

Document Processing
The basic building block of RAG is document processing. Documents can be chunked using various strategies (recursive, sliding window, etc.) and enriched with metadata. See the chunking and embedding doc.

Vector Storage
Mastra supports multiple vector stores for embedding persistence and similarity search, including pgvector, Pinecone, and Qdrant. See the vector database doc.

Observability and Debugging
Mastra’s RAG system includes observability features to help you optimize your retrieval pipeline:

Track embedding generation performance and costs
Monitor chunk quality and retrieval relevance
Analyze query patterns and cache hit rates
Export metrics to your observability platform
See the OTel Configuration page for more details.

More resources
Chain of Thought RAG Example
All RAG Examples (including different chunking strategies, embedding models, and vector stores)

Chunking and Embedding Documents
Before processing, create a MDocument instance from your content. You can initialize it from various formats:

const docFromText = MDocument.fromText("Your plain text content...");
const docFromHTML = MDocument.fromHTML("<html>Your HTML content...</html>");
const docFromMarkdown = MDocument.fromMarkdown("# Your Markdown content...");
const docFromJSON = MDocument.fromJSON(`{ "key": "value" }`);

Step 1: Document Processing
Use chunk to split documents into manageable pieces. Mastra supports multiple chunking strategies optimized for different document types:

recursive: Smart splitting based on content structure
character: Simple character-based splits
token: Token-aware splitting
markdown: Markdown-aware splitting
html: HTML structure-aware splitting
json: JSON structure-aware splitting
latex: LaTeX structure-aware splitting
Here’s an example of how to use the recursive strategy:

const chunks = await doc.chunk({
  strategy: "recursive",
  size: 512,
  overlap: 50,
  separator: "\n",
  extract: {
    metadata: true, // Optionally extract metadata
  },
});

Note: Metadata extraction may use LLM calls, so ensure your API key is set.

We go deeper into chunking strategies in our chunk documentation.

Step 2: Embedding Generation
Transform chunks into embeddings using your preferred provider. Mastra supports both OpenAI and Cohere embeddings:

Using OpenAI
import { openai } from "@ai-sdk/openai";
import { embedMany } from "ai";
 
const { embeddings } = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: chunks.map(chunk => chunk.text),
});

Using Cohere
import { embedMany } from 'ai';
import { cohere } from '@ai-sdk/cohere';
 
const { embeddings } = await embedMany({
  model: cohere.embedding('embed-english-v3.0'),
  values: chunks.map(chunk => chunk.text),
});

The embedding functions return vectors, arrays of numbers representing the semantic meaning of your text, ready for similarity searches in your vector database.

Example: Complete Pipeline
Here’s an example showing document processing and embedding generation with both providers:

import { embedMany } from "ai";
import { openai } from "@ai-sdk/openai";
import { cohere } from "@ai-sdk/cohere";
 
import { MDocument } from "@mastra/rag";
 
// Initialize document
const doc = MDocument.fromText(`
  Climate change poses significant challenges to global agriculture.
  Rising temperatures and changing precipitation patterns affect crop yields.
`);
 
// Create chunks
const chunks = await doc.chunk({
  strategy: "recursive",
  size: 256,
  overlap: 50,
});
 
// Generate embeddings with OpenAI
const { embeddings: openAIEmbeddings } = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: chunks.map(chunk => chunk.text),
});
 
// OR
 
// Generate embeddings with Cohere
const { embeddings: cohereEmbeddings } = await embedMany({
  model: cohere.embedding('embed-english-v3.0'),
  values: chunks.map(chunk => chunk.text),
});
 
// Store embeddings in your vector database
await vectorStore.upsert({
  indexName: "embeddings",
  vectors: embeddings,
});

This example demonstrates how to process a document, split it into chunks, generate embeddings with both OpenAI and Cohere, and store the results in a vector database.

For more examples of different chunking strategies and embedding configurations, see:

Adjust Chunk Size
Adjust Chunk Delimiters
Embed Text with Cohere


Storing Embeddings in A Vector Database
After generating embeddings, you need to store them in a database that supports vector similarity search. Mastra provides a consistent interface for storing and querying embeddings across different vector databases.

Supported databases
PostgreSQL with PgVector
Best for teams already using PostgreSQL who want to minimize infrastructure complexity:

vector-store.ts

import { PgVector } from '@mastra/pg';
 
const store = new PgVector(process.env.POSTGRES_CONNECTION_STRING)
await store.createIndex({
  indexName: "my-collection",
  dimension: 1536,
});
await store.upsert({
  indexName: "my-collection",
  vectors: embeddings,
  metadata: chunks.map(chunk => ({ text: chunk.text })),
});
 
Using Vector Storage
Once initialized, all vector stores share the same interface for creating indexes, upserting embeddings, and querying.

Creating Indexes
Before storing embeddings, you need to create an index with the appropriate dimension size for your embedding model:

store-embeddings.ts

// Create an index with dimension 1536 (for text-embedding-3-small)
await store.createIndex({
  indexName: 'my-collection',
  dimension: 1536,
});
 
// For other models, use their corresponding dimensions:
// - text-embedding-3-large: 3072
// - text-embedding-ada-002: 1536
// - cohere-embed-multilingual-v3: 1024
The dimension size must match the output dimension of your chosen embedding model. Common dimension sizes are:

OpenAI text-embedding-3-small: 1536 dimensions
OpenAI text-embedding-3-large: 3072 dimensions
Cohere embed-multilingual-v3: 1024 dimensions
Upserting Embeddings
After creating an index, you can store embeddings along with their basic metadata:

store-embeddings.ts

// Store embeddings with their corresponding metadata
await store.upsert({
  indexName: 'my-collection',  // index name
  vectors: embeddings,       // array of embedding vectors
  metadata: chunks.map(chunk => ({
    text: chunk.text,  // The original text content
    id: chunk.id       // Optional unique identifier
  }))
});
The upsert operation:

Takes an array of embedding vectors and their corresponding metadata
Updates existing vectors if they share the same ID
Creates new vectors if they don’t exist
Automatically handles batching for large datasets
Adding Metadata
Vector stores support rich metadata for advanced filtering and organization. You can add any JSON-serializable fields that will help with retrieval.

Reminder: Metadata is stored as a JSON field with no fixed schema, so you’ll want to name your fields consistently and apply a consistent schema, or your queries will return unexpected results.

// Store embeddings with rich metadata for better organization and filtering
await vectorStore.upsert({
  indexName: "embeddings",
  vectors: embeddings,
  metadata: chunks.map((chunk) => ({
    // Basic content
    text: chunk.text,
    id: chunk.id,
    
    // Document organization
    source: chunk.source,
    category: chunk.category,
    
    // Temporal metadata
    createdAt: new Date().toISOString(),
    version: "1.0",
    
    // Custom fields
    language: chunk.language,
    author: chunk.author,
    confidenceScore: chunk.score,
  })),
});

Key metadata considerations:

Be strict with field naming - inconsistencies like ‘category’ vs ‘Category’ will affect queries
Only include fields you plan to filter or sort by - extra fields add overhead
Add timestamps (e.g., ‘createdAt’, ‘lastUpdated’) to track content freshness
Best Practices
Create indexes before bulk insertions
Use batch operations for large insertions (the upsert method handles batching automatically)
Only store metadata you’ll query against
Match embedding dimensions to your model (e.g., 1536 for text-embedding-3-small)
Examples
For complete examples of different vector store implementations, see:

Insert Embedding in PgVector
Insert Embedding in Pinecone
Insert Embedding in Qdrant
Insert Embedding in Chroma
Insert Embedding in Astra DB
Insert Embedding in LibSQL
Insert Embedding in Upstash
Insert Embedding in Cloudflare Vectorize
Basic RAG with Vector Storage


Retrieval in RAG Systems
After storing embeddings, you need to retrieve relevant chunks to answer user queries.

Mastra provides flexible retrieval options with support for semantic search, filtering, and re-ranking.

How Retrieval Works
The user’s query is converted to an embedding using the same model used for document embeddings
This embedding is compared to stored embeddings using vector similarity
The most similar chunks are retrieved and can be optionally:
Filtered by metadata
Re-ranked for better relevance
Processed through a knowledge graph
Basic Retrieval
The simplest approach is direct semantic search. This method uses vector similarity to find chunks that are semantically similar to the query:

import { openai } from "@ai-sdk/openai";
import { embed } from "ai";
import { PgVector } from "@mastra/pg";
 
// Convert query to embedding
const { embedding } = await embed({
  value: "What are the main points in the article?",
  model: openai.embedding('text-embedding-3-small'),
});
 
// Query vector store
const pgVector = new PgVector(process.env.POSTGRES_CONNECTION_STRING);
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
});

Results include both the text content and a similarity score:

[
  {
    text: "Climate change poses significant challenges...",
    score: 0.89,
    metadata: { source: "article1.txt" }
  },
  {
    text: "Rising temperatures affect crop yields...",
    score: 0.82,
    metadata: { source: "article1.txt" }
  }
  // ... more results
]

For an example of how to use the basic retrieval method, see the Retrieve Results example.

Advanced Retrieval options
Metadata Filtering
Filter results based on metadata fields to narrow down the search space. This is useful when you have documents from different sources, time periods, or with specific attributes. Mastra provides a unified MongoDB-style query syntax that works across all supported vector stores.

For detailed information about available operators and syntax, see the Metadata Filters Reference.

Basic filtering examples:

// Simple equality filter
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    source: "article1.txt"
  }
});
 
// Numeric comparison
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    price: { $gt: 100 }
  }
});
 
// Multiple conditions
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    category: "electronics",
    price: { $lt: 1000 },
    inStock: true
  }
});
 
// Array operations
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    tags: { $in: ["sale", "new"] }
  }
});
 
// Logical operators
const results = await pgVector.query({
  indexName: "embeddings",
  queryVector: embedding,
  topK: 10,
  filter: {
    $or: [
      { category: "electronics" },
      { category: "accessories" }
    ],
    $and: [
      { price: { $gt: 50 } },
      { price: { $lt: 200 } }
    ]
  }
});

Common use cases for metadata filtering:

Filter by document source or type
Filter by date ranges
Filter by specific categories or tags
Filter by numerical ranges (e.g., price, rating)
Combine multiple conditions for precise querying
Filter by document attributes (e.g., language, author)
For an example of how to use metadata filtering, see the Hybrid Vector Search example.

Vector Query Tool
Sometimes you want to give your agent the ability to query a vector database directly. The Vector Query Tool allows your agent to be in charge of retrieval decisions, combining semantic search with optional filtering and reranking based on the agent’s understanding of the user’s needs.

const vectorQueryTool = createVectorQueryTool({
  vectorStoreName: 'pgVector',
  indexName: 'embeddings',
  model: openai.embedding('text-embedding-3-small'),
});

When creating the tool, pay special attention to the tool’s name and description - these help the agent understand when and how to use the retrieval capabilities. For example, you might name it “SearchKnowledgeBase” and describe it as “Search through our documentation to find relevant information about X topic.”

This is particularly useful when:

Your agent needs to dynamically decide what information to retrieve
The retrieval process requires complex decision-making
You want the agent to combine multiple retrieval strategies based on context
For detailed configuration options and advanced usage, see the Vector Query Tool Reference.

Vector Store Prompts
Vector store prompts define query patterns and filtering capabilities for each vector database implementation. When implementing filtering, these prompts are required in the agent’s instructions to specify valid operators and syntax for each vector store implementation.

import { openai } from '@ai-sdk/openai';
import { PGVECTOR_PROMPT } from "@mastra/rag";
 
export const ragAgent = new Agent({
name: 'RAG Agent',
model: openai('gpt-4o-mini'),
instructions: `
Process queries using the provided context. Structure responses to be concise and relevant.
${PGVECTOR_PROMPT}
`,
tools: { vectorQueryTool },
});

Re-ranking
Initial vector similarity search can sometimes miss nuanced relevance. Re-ranking is a more computationally expensive process, but more accurate algorithm that improves results by:

Considering word order and exact matches
Applying more sophisticated relevance scoring
Using a method called cross-attention between query and documents
Here’s how to use re-ranking:

import { openai } from "@ai-sdk/openai";
import { rerank } from "@mastra/rag";
 
// Get initial results from vector search
const initialResults = await pgVector.query({
  indexName: "embeddings",
  queryVector: queryEmbedding,
  topK: 10,
});
 
// Re-rank the results
const rerankedResults = await rerank(initialResults, query, openai('gpt-4o-mini'));

Note: For semantic scoring to work properly during re-ranking, each result must include the text content in its metadata.text field.

The re-ranked results combine vector similarity with semantic understanding to improve retrieval quality.

For more details about re-ranking, see the rerank() method.

For an example of how to use the re-ranking method, see the Re-ranking Results example.

Graph-based Retrieval
For documents with complex relationships, graph-based retrieval can follow connections between chunks. This helps when:

Information is spread across multiple documents
Documents reference each other
You need to traverse relationships to find complete answers
Example setup:

const graphQueryTool = createGraphQueryTool({
  vectorStoreName: 'pgVector',
  indexName: 'embeddings',
  model: openai.embedding('text-embedding-3-small'),
  graphOptions: {
    threshold: 0.7,
  }
});

For more details about graph-based retrieval, see the GraphRAG class and the createGraphQueryTool() function.

For an example of how to use the graph-based retrieval method, see the Graph-based Retrieval example.


Creating Mastra Projects
Mastra provides two CLI commands for project setup:

mastra create: Generate a new project
mastra init: Add Mastra to an existing project
Creating a New Project
You can create a new project using either a package manager or the mastra CLI:

npm create mastra@latest 

npm install -g mastra@latest 
mastra create

Generated project structure:

my-project/
├── src/
│   └── mastra/
│       └── index.ts    # Mastra entry point
├── package.json
└── tsconfig.json
Adding to an Existing Project
mastra init
Changes made to project:

Creates src/mastra directory with entry point
Adds required dependencies
Configures TypeScript compiler options
Command Arguments
Arguments:
  --components     Specify components: agents, memory, storage
  --llm-provider   LLM provider: openai, anthropic
  --add-example   Include example implementation
  --llm-api-key   Provider API key
Interactive Setup
Running commands without arguments starts a CLI prompt for:

Component selection
LLM provider configuration
API key setup
Example code inclusion
Project Initialization
# Install dependencies
npm install
 
# Start development server on port 4111
mastra dev
 
# Access playground at http://localhost:4111

Inspecting agents and workflows with mastra Dev
The mastra dev command launches a development server that serves your Mastra application locally.

REST API Endpoints
mastra dev spins up REST API endpoints for your agents and workflows, such as:

POST /api/agents/:agentId/generate
POST /api/agents/:agentId/stream
POST /api/workflows/:workflowId/start
POST /api/workflows/:workflowId/:instanceId/event
GET /api/workflows/:workflowId/:instanceId/status
By default, the server runs at http://localhost:4111, but you can change the port with the --port flag.

Using the Client SDK
The easiest way to interact with your local Mastra server is through our TypeScript/JavaScript Client SDK. Install it with:

npm install @mastra/client-js
Then configure it to point to your local server:

import { MastraClient } from "@mastra/client-js";
 
const client = new MastraClient({
  baseUrl: "http://localhost:4111",
});
 
// Example: Interact with a local agent
const agent = client.getAgent("my-agent");
const response = await agent.generate({
  messages: [{ role: "user", content: "Hello!" }],
});
The client SDK provides type-safe wrappers for all API endpoints, making it much easier to develop and test your Mastra applications locally.

UI Playground
mastra dev creates a UI with an agent chat interface, a workflow visualizer and a tool playground.

OpenAPI Specification
mastra dev provides an OpenAPI spec at:

GET /openapi.json
Summary
mastra dev makes it easy to develop, debug, and iterate on your AI logic in a self-contained environment before deploying to production.

Mastra Dev reference
Client SDK documentation

Using Mastra Integrations
Integrations in Mastra are auto-generated, type-safe API clients for third-party services. They can be used as tools for agents or as steps in workflows.

Installing an Integration
Mastra’s default integrations are packaged as individually installable npm modules. You can add an integration to your project by installing it via npm and importing it into your Mastra configuration.

Example: Adding the GitHub Integration
Install the Integration Package
To install the GitHub integration, run:

npm install @mastra/github
Add the Integration to Your Project
Create a new file for your integrations (e.g., src/mastra/integrations/index.ts) and import the integration:

src/mastra/integrations/index.ts

import { GithubIntegration } from '@mastra/github';
 
export const github = new GithubIntegration({
  config: {
    PERSONAL_ACCESS_TOKEN: process.env.GITHUB_PAT!,
  },
});
Make sure to replace process.env.GITHUB_PAT! with your actual GitHub Personal Access Token or ensure that the environment variable is properly set.

Use the Integration in Tools or Workflows
You can now use the integration when defining tools for your agents or in workflows.

src/mastra/tools/index.ts

import { createTool } from '@mastra/core';
import { z } from 'zod';
import { github } from '../integrations';
 
export const getMainBranchRef = createTool({
  id: 'getMainBranchRef',
  description: 'Fetch the main branch reference from a GitHub repository',
  inputSchema: z.object({
    owner: z.string(),
    repo: z.string(),
  }),
  outputSchema: z.object({
    ref: z.string().optional(),
  }),
  execute: async ({ context }) => {
    const client = await github.getApiClient();
 
    const mainRef = await client.gitGetRef({
      path: {
        owner: context.owner,
        repo: context.repo,
        ref: 'heads/main',
      },
    });
 
    return { ref: mainRef.data?.ref };
  },
});
In the example above:

We import the github integration.
We define a tool called getMainBranchRef that uses the GitHub API client to fetch the reference of the main branch of a repository.
The tool accepts owner and repo as inputs and returns the reference string.
Using Integrations in Agents
Once you’ve defined tools that utilize integrations, you can include these tools in your agents.

src/mastra/agents/index.ts

import { openai } from '@ai-sdk/openai';
import { Agent } from '@mastra/core';
import { getMainBranchRef } from '../tools';
 
export const codeReviewAgent = new Agent({
  name: 'Code Review Agent',
  instructions: 'An agent that reviews code repositories and provides feedback.',
  model: openai('gpt-4o-mini'),
  tools: {
    getMainBranchRef,
    // other tools...
  },
});
In this setup:

We create an agent named Code Review Agent.
We include the getMainBranchRef tool in the agent’s available tools.
The agent can now use this tool to interact with GitHub repositories during conversations.
Environment Configuration
Ensure that any required API keys or tokens for your integrations are properly set in your environment variables. For example, with the GitHub integration, you need to set your GitHub Personal Access Token:

GITHUB_PAT=your_personal_access_token
Consider using a .env file or another secure method to manage sensitive credentials.

Available Integrations
Mastra provides several built-in integrations; primarily API-key based integrations that do not require OAuth. Some available integrations including Github, Stripe, Resend, Firecrawl, and more.

Check Mastra’s codebase or npm packages for a full list of available integrations.

Conclusion
Integrations in Mastra enable your AI agents and workflows to interact with external services seamlessly. By installing and configuring integrations, you can extend the capabilities of your application to include operations such as fetching data from APIs, sending messages, or managing resources in third-party systems.

Remember to consult the documentation of each integration for specific usage details and to adhere to best practices for security and type safety.


Mastra Server
When you deploy a Mastra application, it runs as an HTTP server that exposes your agents, workflows, and other functionality as API endpoints. This page explains how to configure and customize the server behavior.

Server Architecture
Mastra uses Hono as its underlying HTTP server framework. When you build a Mastra application using mastra build, it generates a Hono-based HTTP server in the .mastra directory.

The server provides:

API endpoints for all registered agents
API endpoints for all registered workflows
Custom middleware support
Server Middleware
Mastra allows you to configure custom middleware functions that will be applied to API routes. This is useful for adding authentication, logging, CORS, or other HTTP-level functionality to your API endpoints.

import { Mastra } from '@mastra/core';
 
export const mastra = new Mastra({
  // Other configuration options
  serverMiddleware: [
    {
      handler: async (c, next) => {
        // Example: Add authentication check
        const authHeader = c.req.header('Authorization');
        if (!authHeader) {
          return new Response('Unauthorized', { status: 401 });
        }
        
        // Continue to the next middleware or route handler
        await next();
      },
      path: '/api/*', // Optional: defaults to '/api/*' if not specified
    },
    {
      handler: async (c, next) => {
        // Example: Add request logging
        console.log(`${c.req.method} ${c.req.url}`);
        await next();
      },
      // This middleware will apply to all routes since no path is specified
    }
  ]
});

Middleware Behavior
Each middleware function:

Receives a Hono context object (c) and a next function
Can return a Response to short-circuit the request handling
Can call next() to continue to the next middleware or route handler
Can optionally specify a path pattern (defaults to ‘/api/*‘)
Common Middleware Use Cases
Authentication
{
  handler: async (c, next) => {
    const authHeader = c.req.header('Authorization');
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return new Response('Unauthorized', { status: 401 });
    }
    
    const token = authHeader.split(' ')[1];
    // Validate token here
    
    await next();
  },
  path: '/api/*',
}

CORS Support
{
  handler: async (c, next) => {
    // Add CORS headers
    c.header('Access-Control-Allow-Origin', '*');
    c.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    c.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    // Handle preflight requests
    if (c.req.method === 'OPTIONS') {
      return new Response(null, { status: 204 });
    }
    
    await next();
  }
}

Request Logging
{
  handler: async (c, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    console.log(`${c.req.method} ${c.req.url} - ${duration}ms`);
  }
}

Logging and Tracing
Effective logging and tracing are crucial for understanding the behavior of your application.

Tracing is especially important for AI engineering. Teams building AI products find that visibility into inputs and outputs of every step of every run is crucial to improving accuracy. You get this with Mastra’s telemetry.

Logging
In Mastra, logs can detail when certain functions run, what input data they receive, and how they respond.

Basic Setup
Here’s a minimal example that sets up a console logger at the INFO level. This will print out informational messages and above (i.e., INFO, WARN, ERROR) to the console.

mastra.config.ts

import { Mastra } from "@mastra/core";
import { createLogger } from "@mastra/core/logger";
 
export const mastra = new Mastra({
  // Other Mastra configuration...
  logger: createLogger({
    name: "Mastra",
    level: "info",
  }),
});
In this configuration:

name: "Mastra" specifies the name to group logs under.
level: "info" sets the minimum severity of logs to record.
Configuration
For more details on the options you can pass to createLogger(), see the createLogger reference documentation.
Once you have a Logger instance, you can call its methods (e.g., .info(), .warn(), .error()) in the Logger instance reference documentation.
If you want to send your logs to an external service for centralized collection, analysis, or storage, you can configure other logger types such as Upstash Redis. Consult the createLogger reference documentation for details on parameters like url, token, and key when using the UPSTASH logger type.
Telemetry
Mastra supports the OpenTelemetry Protocol (OTLP) for tracing and monitoring your application. When telemetry is enabled, Mastra automatically traces all core primitives including agent operations, LLM interactions, tool executions, integration calls, workflow runs, and database operations. Your telemetry data can then be exported to any OTEL collector.

Basic Configuration
Here’s a simple example of enabling telemetry:

mastra.config.ts

export const mastra = new Mastra({
  // ... other config
  telemetry: {
    serviceName: "my-app",
    enabled: true,
    sampling: {
      type: "always_on",
    },
    export: {
      type: "otlp",
      endpoint: "http://localhost:4318", // SigNoz local endpoint
    },
  },
});
Configuration Options
The telemetry config accepts these properties:

type OtelConfig = {
  // Name to identify your service in traces (optional)
  serviceName?: string;
 
  // Enable/disable telemetry (defaults to true)
  enabled?: boolean;
 
  // Control how many traces are sampled
  sampling?: {
    type: "ratio" | "always_on" | "always_off" | "parent_based";
    probability?: number; // For ratio sampling
    root?: {
      probability: number; // For parent_based sampling
    };
  };
 
  // Where to send telemetry data
  export?: {
    type: "otlp" | "console";
    endpoint?: string;
    headers?: Record<string, string>;
  };
};
See the OtelConfig reference documentation for more details.

Environment Variables
You can configure the OTLP endpoint and headers through environment variables:

.env

OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_EXPORTER_OTLP_HEADERS=x-api-key=your-api-key
Then in your config:

mastra.config.ts

export const mastra = new Mastra({
  // ... other config
  telemetry: {
    serviceName: "my-app",
    enabled: true,
    export: {
      type: "otlp",
      // endpoint and headers will be picked up from env vars
    },
  },
});
Example: SigNoz Integration
Here’s what a traced agent interaction looks like in SigNoz:

Agent interaction trace showing spans, LLM calls, and tool executions
Other Supported Providers
For a complete list of supported observability providers and their configuration details, see the Observability Providers reference.

Next.js Configuration
If you’re using Next.js, you have two options for setting up OpenTelemetry instrumentation:

Option 1: Using Vercel’s OTEL Setup
If you’re deploying to Vercel, you can use their built-in OpenTelemetry setup:

Install the required dependencies:
npm install @opentelemetry/api @vercel/otel

Create an instrumentation file at the root of your project (or in the src folder if using one):
instrumentation.ts

import { registerOTel } from '@vercel/otel'
 
export function register() {
  registerOTel({ serviceName: 'your-project-name' })
}
Option 2: Using Custom Exporters
If you’re using other observability tools (like Langfuse), you can configure a custom exporter:

Install the required dependencies (example using Langfuse):
npm install @opentelemetry/api langfuse-vercel

Create an instrumentation file:
instrumentation.ts

import {
  NodeSDK,
  ATTR_SERVICE_NAME,
  Resource,
} from '@mastra/core/telemetry/otel-vendor';
import { LangfuseExporter } from 'langfuse-vercel';
 
export function register() {
  const exporter = new LangfuseExporter({
    // ... Langfuse config
  })
 
  const sdk = new NodeSDK({
    resource: new Resource({
      [ATTR_SERVICE_NAME]: 'ai',
    }),
    traceExporter: exporter,
  });
 
  sdk.start();
}
Next.js Configuration
For either option, enable the instrumentation hook in your Next.js config:

next.config.ts

import type { NextConfig } from "next";
 
const nextConfig: NextConfig = {
  experimental: {
    instrumentationHook: true // Not required in Next.js 15+
  }
};
 
export default nextConfig;
Mastra Configuration
Configure your Mastra instance:

mastra.config.ts

import { Mastra } from "@mastra/core";
 
export const mastra = new Mastra({
  // ... other config
  telemetry: {
    serviceName: "your-project-name",
    enabled: true
  }
});
This setup will enable OpenTelemetry tracing for your Next.js application and Mastra operations.

For more details, see the documentation for:

Next.js Instrumentation
Vercel OpenTelemetry

Deploying Mastra Applications
Mastra applications can be deployed in two ways:

Direct Platform Deployment: Using platform-specific deployers for Cloudflare Workers, Vercel, or Netlify
Universal Deployment: Using mastra build to generate a standard Node.js server that can run anywhere
Prerequisites
Before you begin, ensure you have:

Node.js installed (version 18 or higher is recommended)
If using a platform-specific deployer:
An account with your chosen platform
Required API keys or credentials
Direct Platform Deployment
Platform-specific deployers handle configuration and deployment for:

Cloudflare Workers
Vercel
Netlify
Installing Deployers
# For Cloudflare
npm install @mastra/deployer-cloudflare
 
# For Vercel
npm install @mastra/deployer-vercel
 
# For Netlify
npm install @mastra/deployer-netlify

Configuring Deployers
Configure the deployer in your entry file:

import { Mastra, createLogger } from '@mastra/core';
import { CloudflareDeployer } from '@mastra/deployer-cloudflare';
 
export const mastra = new Mastra({
  agents: { /* your agents here */ },
  logger: createLogger({ name: 'MyApp', level: 'debug' }),
  deployer: new CloudflareDeployer({
    scope: 'your-cloudflare-scope',
    projectName: 'your-project-name',
    // See complete configuration options in the reference docs
  }),
});

Deployer Configuration
Each deployer has specific configuration options. Below are basic examples, but refer to the reference documentation for complete details.

Cloudflare Deployer
new CloudflareDeployer({
  scope: 'your-cloudflare-account-id',
  projectName: 'your-project-name',
  // For complete configuration options, see the reference documentation
})

View Cloudflare Deployer Reference →

Vercel Deployer
new VercelDeployer({
  teamId: 'your-vercel-team-id',
  projectName: 'your-project-name',
  token: 'your-vercel-token'
  // For complete configuration options, see the reference documentation
})

View Vercel Deployer Reference →

Netlify Deployer
new NetlifyDeployer({
  scope: 'your-netlify-team-slug',
  projectName: 'your-project-name',
  token: 'your-netlify-token'
})

View Netlify Deployer Reference →

Universal Deployment
Since Mastra builds to a standard Node.js server, you can deploy to any platform that runs Node.js applications:

Cloud VMs (AWS EC2, DigitalOcean Droplets, GCP Compute Engine)
Container platforms (Docker, Kubernetes)
Platform as a Service (Heroku, Railway)
Self-hosted servers
Building
Build the application:

# Build from current directory
mastra build
 
# Or specify a directory
mastra build --dir ./my-project

The build process:

Locates entry file (src/mastra/index.ts or src/mastra/index.js)
Creates .mastra output directory
Bundles code using Rollup with tree shaking and source maps
Generates Hono HTTP server
See mastra build for all options.

Running the Server
Start the HTTP server:

node .mastra/index.js

Environment Variables
Required variables:

Platform deployer variables (if using platform deployers):
Platform credentials
Agent API keys:
OPENAI_API_KEY
ANTHROPIC_API_KEY
Server configuration (for universal deployment):
PORT: HTTP server port (default: 3000)
HOST: Server host (default: 0.0.0.0)
Platform Documentation
Platform deployment references:

Cloudflare Workers
Vercel
Netlify


Testing your agents with evals
Evals are automated tests that evaluate Agents outputs using model-graded, rule-based, and statistical methods. Each eval returns a normalized score between 0-1 that can be logged and compared. Evals can be customized with your own prompts and scoring functions.

Evals can be run in the cloud, capturing real-time results. But evals can also be part of your CI/CD pipeline, allowing you to test and monitor your agents over time.

How to use evals
Evals need to be added to an agent. To use any of the default metrics, you can do the following:

src/mastra/agents/index.ts

import { Agent } from "@mastra/core/agent";
import { openai } from "@ai-sdk/openai";
import { ToneConsistencyMetric } from "@mastra/evals/nlp";
 
export const myAgent = new Agent({
  name: "My Agent",
  instructions: "You are a helpful assistant.",
  model: openai("gpt-4o-mini"),
  evals: {
    tone: new ToneConsistencyMetric()
  },
});
You can now view the evals in the Mastra dashboard, when using mastra dev.

Executing evals in your CI/CD pipeline
We support any testing framework that supports ESM modules. For example, you can use Vitest, Jest or Mocha to run evals in your CI/CD pipeline.

src/mastra/agents/index.test.ts

import { describe, it, expect } from 'vitest';
import { evaluate } from '@mastra/core/eval';
import { myAgent } from './index';
 
describe('My Agent', () => {
  it('should be able to validate tone consistency', async () => {
    const metric = new ToneConsistencyMetric();
    const result = await evaluate(myAgent, 'Hello, world!', metric)
 
    expect(result.score).toBe(1);
  });
});
 
You will need to configure a testSetup and globalSetup script for your testing framework to capture the eval results. It allows us to show these results in your mastra dashboard.

Vitest
These are the files you need to add to your project to run evals in your CI/CD pipeline and allow us to capture the results. Without these files, the evals will still run and fail when necessary but you won’t be able to see the results in the Mastra dashboard.

globalSetup.ts

import { globalSetup } from '@mastra/evals';
 
export default function setup() {
  globalSetup()
}
testSetup.ts

import { beforeAll } from 'vitest';
import { attachListeners } from '@mastra/evals';
 
beforeAll(async () => {
  await attachListeners();
});
vitest.config.ts

import { defineConfig } from 'vitest/config'
 
export default defineConfig({
  test: {
    globalSetup: './globalSetup.ts',
    setupFiles: ['./testSetup.ts'],
  },
})


Supported evals in Mastra
Mastra provides several eval metrics for assessing Agent outputs. Mastra is not limited to these metrics, and you can also define your own evals.

Accuracy and Reliability
hallucination: Detects fabricated or unsupported information
faithfulness: Checks output alignment with source material
content-similarity: Compares text similarity
textual-difference: Measures text changes
completeness: Measures if all required information is present
answer-relevancy: Measures how well an answer addresses the input question
Understanding Context
context-position: Evaluates the placement of context in responses
context-precision: Assesses the accuracy of context usage
context-relevancy: Measures the relevance of used context
contextual-recall: Evaluates information recall from context
Output Quality
tone: Analyzes writing style and tone
toxicity: Detects harmful or inappropriate content
bias: Detects potential biases in the output
prompt-alignment: Measures adherence to prompt instructions
summarization: Evaluates summary quality
keyword-coverage: Checks for presence of key terms


Create your own Eval
Creating your own eval is as easy as creating a new function. You simply create a class that extends the Metric class and implement the measure method.

Basic example
Here is a very basic example of a custom eval that checks if the output contains a certain keyword. This is a simplified version of our own keyword coverage eval.

src/mastra/evals/keyword-coverage.ts

import { Metric, type MetricResult } from '@mastra/core/eval';
 
interface KeywordCoverageResult extends MetricResult {
  info: {
    totalKeywords: number;
    matchedKeywords: number;
  };
}
 
export class KeywordCoverageMetric extends Metric {
  private referenceKeywords: Set<string>;
 
  constructor(keywords: string[]) {
    super();
    this.referenceKeywords = new Set(keywords);
  }
 
  async measure(input: string, output: string): Promise<KeywordCoverageResult> {
    // Handle empty strings case
    if (!input && !output) {
      return {
        score: 1,
        info: {
          totalKeywords: 0,
          matchedKeywords: 0,
        },
      };
    }
 
    const matchedKeywords = [...this.referenceKeywords].filter(k => output.includes(k));
    const totalKeywords = this.referenceKeywords.size;
    const coverage = totalKeywords > 0 ? matchedKeywords.length / totalKeywords : 0;
 
    return {
      score: coverage,
      info: {
        totalKeywords: this.referenceKeywords.size,
        matchedKeywords: matchedKeywords.length,
      },
    };
  }
}
Creating a custom LLM-Judge
A custom LLM judge can provide more targeted and meaningful evaluations for your use case. For example, if you’re building a medical Q&A system, you might want to evaluate not just answer relevancy but also medical accuracy and safety considerations.

Let’s create an example to make sure our Chef Michel is giving complete recipe information to the user.

We’ll start with creating the judge agent. You can put it all in one file but we prefer splitting it into a separate file to keep things readable.

src/mastra/evals/recipe-completeness/metricJudge.ts

import { type LanguageModel } from '@mastra/core/llm';
import { MastraAgentJudge } from '@mastra/evals/judge';
import { z } from 'zod';
 
import { RECIPE_COMPLETENESS_INSTRUCTIONS, generateCompletenessPrompt, generateReasonPrompt } from './prompts';
 
export class RecipeCompletenessJudge extends MastraAgentJudge {
  constructor(model: LanguageModel) {
    super('Recipe Completeness', RECIPE_COMPLETENESS_INSTRUCTIONS, model);
  }
 
  async evaluate(
    input: string,
    output: string,
  ): Promise<{
    missing: string[];
    verdict: string;
  }> {
    const completenessPrompt = generateCompletenessPrompt({ input, output });
    const result = await this.agent.generate(completenessPrompt, {
      output: z.object({
        missing: z.array(z.string()),
        verdict: z.string(),
      }),
    });
 
    return result.object;
  }
 
  async getReason(args: {
    input: string;
    output: string;
    missing: string[];
    verdict: string;
  }): Promise<string> {
    const prompt = generateReasonPrompt(args);
    const result = await this.agent.generate(prompt, {
      output: z.object({
        reason: z.string(),
      }),
    });
 
    return result.object.reason;
  }
}
src/mastra/evals/recipe-completeness/index.ts

import { Metric, type MetricResult } from '@mastra/core/eval';
import { type LanguageModel } from '@mastra/core/llm';
 
import { RecipeCompletenessJudge } from './metricJudge';
 
export interface RecipeCompletenessMetricOptions {
  scale?: number;
}
 
export interface MetricResultWithInfo extends MetricResult {
  info: {
    reason: string;
    missing: string[];
  };
}
 
export class RecipeCompletenessMetric extends Metric {
  private judge: RecipeCompletenessJudge;
  private scale: number;
  constructor(model: LanguageModel, { scale = 1 }: RecipeCompletenessMetricOptions = {}) {
    super();
 
    this.judge = new RecipeCompletenessJudge(model);
    this.scale = scale;
  }
 
  async measure(input: string, output: string): Promise<MetricResultWithInfo> {
    const { verdict, missing } = await this.judge.evaluate(input, output);
    const score = this.calculateScore({ verdict });
    const reason = await this.judge.getReason({
      input,
      output,
      verdict,
      missing,
    });
 
    return {
      score,
      info: {
        missing,
        reason,
      },
    };
  }
 
  private calculateScore(verdict: { verdict: string }): number {
    return verdict.verdict.toLowerCase() === 'incomplete' ? 0 : 1;
  }
}
src/mastra/agents/chefAgent.ts

import { openai } from '@ai-sdk/openai';
import { Agent } from '@mastra/core/agent';
 
import { RecipeCompletenessMetric } from '../evals';
 
export const chefAgent = new Agent({
  name: 'chef-agent',
  instructions:
    'You are Michel, a practical and experienced home chef' +
    'You help people cook with whatever ingredients they have available.',
  model: openai('gpt-4o-mini'),
  evals: {
    recipeCompleteness: new RecipeCompletenessMetric(openai('gpt-4o-mini')),
  },
});
You can now use the RecipeCompletenessMetric in your project. See the full example here.


Reference
The Reference section provides documentation of Mastra’s API, including parameters, types and usage examples.

The Mastra Class
The Mastra class is the core entry point for your application. It manages agents, workflows, and server endpoints.

Constructor Options
agents?:
Agent[]
= []
Array of Agent instances to register
tools?:
Record<string, ToolApi>
= {}
Custom tools to register. Structured as a key-value pair, with keys being the tool name and values being the tool function.
storage?:
MastraStorage
Storage engine instance for persisting data
vectors?:
Record<string, MastraVector>
Vector store instance, used for semantic search and vector-based tools (eg Pinecone, PgVector or Qdrant)
logger?:
Logger
= Console logger with INFO level
Logger instance created with createLogger()
workflows?:
Record<string, Workflow>
= {}
Workflows to register. Structured as a key-value pair, with keys being the workflow name and values being the workflow instance.
serverMiddleware?:
Array<{ handler: (c: any, next: () => Promise<void>) => Promise<Response | void>; path?: string; }>
= []
Server middleware functions to be applied to API routes. Each middleware can specify a path pattern (defaults to '/api/*').
Initialization
The Mastra class is typically initialized in your src/mastra/index.ts file:

import { Mastra } from "@mastra/core";
import { createLogger } from "@mastra/core/logger";
 
// Basic initialization
export const mastra = new Mastra({});
 
// Full initialization with all options
export const mastra = new Mastra({
  agents: {},
  workflows: [],
  integrations: [],
  logger: createLogger({
    name: "My Project",
    level: "info",
  }),
  storage: {},
  tools: {},
  vectors: {},
});

You can think of the Mastra class as a top-level registry. When you register tools with Mastra, your registered agents and workflows can use them. When you register integrations with Mastra, agents, workflows, and tools can use them.

Methods
getAgent(name):
Agent
Returns an agent instance by id. Throws if agent not found.
getAgents():
Record<string, Agent>
Returns all registered agents as a key-value object.
getWorkflow(id, { serialized }):
Workflow
Returns a workflow instance by id. The serialized option (default: false) returns a simplified representation with just the name.
getWorkflows({ serialized }):
Record<string, Workflow>
Returns all registered workflows. The serialized option (default: false) returns simplified representations.
getVector(name):
MastraVector
Returns a vector store instance by name. Throws if not found.
getVectors():
Record<string, MastraVector>
Returns all registered vector stores as a key-value object.
getDeployer():
MastraDeployer | undefined
Returns the configured deployer instance, if any.
getStorage():
MastraStorage | undefined
Returns the configured storage instance.
getMemory():
MastraMemory | undefined
Returns the configured memory instance. Note: This is deprecated, memory should be added to agents directly.
getServerMiddleware():
Array<{ handler: Function; path: string; }>
Returns the configured server middleware functions.
setStorage(storage):
void
Sets the storage instance for the Mastra instance.
setLogger({ logger }):
void
Sets the logger for all components (agents, workflows, etc.).
setTelemetry(telemetry):
void
Sets the telemetry configuration for all components.
getLogger():
Logger
Gets the configured logger instance.
getTelemetry():
Telemetry | undefined
Gets the configured telemetry instance.
getLogsByRunId({ runId, transportId }):
Promise<any>
Retrieves logs for a specific run ID and transport ID.
getLogs(transportId):
Promise<any>
Retrieves all logs for a specific transport ID.
Error Handling
The Mastra class methods throw typed errors that can be caught:

try {
  const tool = mastra.getTool("nonexistentTool");
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message); // "Tool with name nonexistentTool not found"
  }
}


mastra init Reference
mastra init
This creates a new Mastra project. You can run it in three different ways:

Interactive Mode (Recommended) Run without flags to use the interactive prompt, which will guide you through:

Choosing a directory for Mastra files
Selecting components to install (Agents, Tools, Workflows)
Choosing a default LLM provider (OpenAI, Anthropic, or Groq)
Deciding whether to include example code
Quick Start with Defaults

mastra init --default
This sets up a project with:

Source directory: src/
All components: agents, tools, workflows
OpenAI as the default provider
No example code
Custom Setup

mastra init --dir src/mastra --components agents,tools --llm openai --example
Options:

-d, --dir: Directory for Mastra files (defaults to src/mastra)
-c, --components: Comma-separated list of components (agents, tools, workflows)
-l, --llm: Default model provider (openai, anthropic, or groq)
-k, --llm-api-key: API key for the selected LLM provider (will be added to .env file)
-e, --example: Include example code
-ne, --no-example: Skip example code


mastra dev Reference
The mastra dev command starts a development server that exposes REST endpoints for your agents, tools, and workflows,

Parameters
--dir?:
string
Specifies the path to your Mastra folder (containing agents, tools, and workflows). Defaults to the current working directory.
--tools?:
string
Comma-separated paths to additional tool directories that should be registered. For example: 'src/tools/dbTools,src/tools/scraperTools'.
--port?:
number
Specifies the port number for the development server. Defaults to 4111.
Routes
Starting the server with mastra dev exposes a set of REST endpoints by default:

Agent Routes
Agents are expected to be exported from src/mastra/agents.

• GET /api/agents

Lists the registered agents found in your Mastra folder. • POST /api/agents/:agentId/generate
Sends a text-based prompt to the specified agent, returning the agent’s response.
Tool Routes
Tools are expected to be exported from src/mastra/tools (or the configured tools directory).

• POST /api/tools/:toolName

Invokes a specific tool by name, passing input data in the request body.
Workflow Routes
Workflows are expected to be exported from src/mastra/workflows (or the configured workflows directory).

• POST /api/workflows/:workflowName/start

Starts the specified workflow. • POST /api/workflows/:workflowName/:instanceId/event
Sends an event or trigger signal to an existing workflow instance. • GET /api/workflows/:workflowName/:instanceId/status
Returns status info for a running workflow instance.
OpenAPI Specification
• GET /openapi.json

Returns an auto-generated OpenAPI specification for your project’s endpoints.
Additional Notes
The port defaults to 4111.

Make sure you have your environment variables set up in your .env.development or .env file for any providers you use (e.g., OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.).

Example request
To test an agent after running mastra dev:

curl -X POST http://localhost:4111/api/agents/myAgent/generate \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "Hello, how can you assist me today?" }
    ]
  }'
Related Docs
REST Endpoints Overview – More detailed usage of the dev server and agent endpoints.
mastra deploy – Deploy your project to Vercel or Cloudflare.


mastra deploy Reference
mastra deploy vercel
Deploy your Mastra project to Vercel.

mastra deploy cloudflare
Deploy your Mastra project to Cloudflare.

mastra deploy netlify
Deploy your Mastra project to Netlify.

Flags
-d, --dir <dir>: Path to your mastra folder


The mastra build command bundles your Mastra project into a production-ready Hono server. Hono is a lightweight web framework that provides type-safe routing and middleware support, making it ideal for deploying Mastra agents as HTTP endpoints.

Usage
mastra build [options]
Options
--dir <path>: Directory containing your Mastra project (default: current directory)
What It Does
Locates your Mastra entry file (either src/mastra/index.ts or src/mastra/index.js)
Creates a .mastra output directory
Bundles your code using Rollup with:
Tree shaking for optimal bundle size
Node.js environment targeting
Source map generation for debugging
Example
# Build from current directory
mastra build
 
# Build from specific directory
mastra build --dir ./my-mastra-project
Output
The command generates a production bundle in the .mastra directory, which includes:

A Hono-based HTTP server with your Mastra agents exposed as endpoints
Bundled JavaScript files optimized for production
Source maps for debugging
Required dependencies
This output is suitable for:

Deploying to cloud servers (EC2, Digital Ocean)
Running in containerized environments
Using with container orchestration systems

getAgent()
Retrieve an agent based on the provided configuration

async function getAgent({
  connectionId,
  agent,
  apis,
  logger,
}: {
  connectionId: string;
  agent: Record<string, any>;
  apis: Record<string, IntegrationApi>;
  logger: any;
}): Promise<(props: { prompt: string }) => Promise<any>> {
  return async (props: { prompt: string }) => {
    return { message: "Hello, world!" };
  };
}

API Signature
Parameters
connectionId:
string
The connection ID to use for the agent's API calls.
agent:
Record<string, any>
The agent configuration object.
apis:
Record<string, IntegrationAPI>
A map of API names to their respective API objects.
Returns


createTool()
Tools are typed functions that can be executed by agents or workflows, with built-in integration access and parameter validation. Each tool has a schema that defines its inputs, an executor function that implements its logic, and access to configured integrations.

src/mastra/tools/index.ts

import { createTool } from "@mastra/core/logger";
import { z } from "zod";
 
const getStockPrice = async (symbol: string) => {
  const data = await fetch(
    `https://mastra-stock-data.vercel.app/api/stock-data?symbol=${symbol}`,
  ).then((r) => r.json());
  return data.prices["4. close"];
};
 
export const stockPrices = createTool({
  id: "Get Stock Price",
  inputSchema: z.object({
    symbol: z.string(),
  }),
  description: `Fetches the last day's closing stock price for a given symbol`,
  execute: async ({ context }) => {
    console.log("Using tool to fetch stock price for", context.symbol);
    return {
      symbol: context.symbol,
      currentPrice: await getStockPrice(context.symbol),
    };
  },
});
 
export const threadInfo = createTool({
  id: "Get Thread Info",
  inputSchema: z.object({
    includeResource: z.boolean().optional(),
  }),
  description: `Gets information about the current conversation thread`,
  execute: async ({ context, threadId, resourceId }) => {
    console.log("Current thread:", threadId);
    console.log("Current resource:", resourceId);
 
    return {
      threadId,
      resourceId: context.includeResource ? resourceId : undefined,
    };
  },
});
API Signature
Parameters
label:
string
Name of the tool (e.g., "Get Stock Prices")
schema:
ZodSchema
Zod schema for validating inputs
description:
string
Clear explanation of what market data the tool provides
executor:
(params: ExecutorParams) => Promise<any>
Async function that fetches the requested market data
ExecutorParams
data:
object
The validated input data (in this case, symbol)
integrationsRegistry:
function
Function to get connected integrations
runId?:
string
The runId of the current run
threadId?:
string
Identifier for the conversation thread. Allows for maintaining context across multiple interactions.
resourceId?:
string
Identifier for the user or resource interacting with the tool.
agents:
Map<string, Agent<any>>
Map of registered agents
engine?:
MastraEngine
Mastra engine instance
llm:
LLM
LLM instance
outputSchema?:
ZodSchema
Zod schema for validating outputs
Returns
ToolApi:
object
The tool API object that includes the schema, label, description, and executor function.
ToolApi
schema:
ZodSchema<IN>
Zod schema for validating inputs.
label:
string
Name of the tool.
description:
string
Description of the tool's functionality.
outputSchema?:
ZodSchema<OUT>
Zod schema for validating outputs.
executor:
(params: IntegrationApiExcutorParams<IN>) => Promise<OUT>
Async function that executes the tool's logic.


Agent.generate()
The generate() method is used to interact with an agent to produce text or structured responses. This method accepts messages and an optional options object as parameters.

Parameters
messages
The messages parameter can be:

A single string
An array of strings
An array of message objects with role and content properties
The message object structure:

interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}
options (Optional)
An optional object that can include configuration for output structure, memory management, tool usage, telemetry, and more.

instructions?:
string
Custom instructions that override the agent's default instructions for this specific generation. Useful for dynamically modifying agent behavior without creating a new agent instance.
output?:
Zod schema | JsonSchema7
Defines the expected structure of the output. Can be a JSON Schema object or a Zod schema.
experimental_output?:
Zod schema | JsonSchema7
Enables structured output generation alongside text generation and tool calls. The model will generate responses that conform to the provided schema.
context?:
CoreMessage[]
Additional context messages to provide to the agent.
memoryOptions?:
MemoryConfig
Configuration options for memory management. See MemoryConfig section below for details.
toolChoice?:
'auto' | 'none' | 'required' | { type: 'tool'; toolName: string }
= 'auto'
Controls how the agent uses tools during generation.
telemetry?:
TelemetrySettings
Settings for telemetry collection during generation. See TelemetrySettings section below for details.
threadId?:
string
Identifier for the conversation thread. Allows for maintaining context across multiple interactions. Must be provided if resourceId is provided.
resourceId?:
string
Identifier for the user or resource interacting with the agent. Must be provided if threadId is provided.
onStepFinish?:
(step: string) => void
Callback function called after each execution step. Receives step details as a JSON string.
maxSteps?:
number
= 5
Maximum number of execution steps allowed.
toolsets?:
ToolsetsInput
Additional toolsets to make available to the agent during generation.
temperature?:
number
Controls randomness in the model's output. Higher values (e.g., 0.8) make the output more random, lower values (e.g., 0.2) make it more focused and deterministic.
MemoryConfig
Configuration options for memory management:

lastMessages?:
number | false
Number of most recent messages to include in context. Set to false to disable.
semanticRecall?:
boolean | object
Configuration for semantic memory recall. Can be boolean or detailed config.
number
topK?:
number
Number of most semantically similar messages to retrieve.
number | object
messageRange?:
number | { before: number; after: number }
Range of messages to consider for semantic search. Can be a single number or before/after configuration.
workingMemory?:
object
Configuration for working memory.
boolean
enabled?:
boolean
Whether to enable working memory.
string
template?:
string
Template to use for working memory.
'text-stream' | 'tool-call'
type?:
'text-stream' | 'tool-call'
Type of content to use for working memory.
threads?:
object
Thread-specific memory configuration.
boolean
generateTitle?:
boolean
Whether to automatically generate titles for new threads.
TelemetrySettings
Settings for telemetry collection during generation:

isEnabled?:
boolean
= false
Enable or disable telemetry. Disabled by default while experimental.
recordInputs?:
boolean
= true
Enable or disable input recording. You might want to disable this to avoid recording sensitive information, reduce data transfers, or increase performance.
recordOutputs?:
boolean
= true
Enable or disable output recording. You might want to disable this to avoid recording sensitive information, reduce data transfers, or increase performance.
functionId?:
string
Identifier for this function. Used to group telemetry data by function.
metadata?:
Record<string, AttributeValue>
Additional information to include in the telemetry data. AttributeValue can be string, number, boolean, array of these types, or null.
tracer?:
Tracer
A custom OpenTelemetry tracer instance to use for the telemetry data. See OpenTelemetry documentation for details.
Returns
The return value of the generate() method depends on the options provided, specifically the output option.

PropertiesTable for Return Values
text?:
string
The generated text response. Present when output is 'text' (no schema provided).
object?:
object
The generated structured response. Present when a schema is provided via `output` or `experimental_output`.
toolCalls?:
Array<ToolCall>
The tool calls made during the generation process. Present in both text and object modes.
ToolCall Structure
toolName:
string
The name of the tool invoked.
args:
any
The arguments passed to the tool.
Related Methods
For real-time streaming responses, see the stream() method documentation.

stream()
The stream() method enables real-time streaming of responses from an agent. This method accepts messages and an optional options object as parameters, similar to generate().

Parameters
messages
The messages parameter can be:

A single string
An array of strings
An array of message objects with role and content properties
The message object structure:

interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}
options (Optional)
An optional object that can include configuration for output structure, memory management, tool usage, telemetry, and more.

instructions?:
string
Custom instructions that override the agent's default instructions for this specific generation. Useful for dynamically modifying agent behavior without creating a new agent instance.
output?:
Zod schema | JsonSchema7
Defines the expected structure of the output. Can be a JSON Schema object or a Zod schema.
experimental_output?:
Zod schema | JsonSchema7
Enables structured output generation alongside text generation and tool calls. The model will generate responses that conform to the provided schema.
context?:
CoreMessage[]
Additional context messages to provide to the agent.
memoryOptions?:
MemoryConfig
Configuration options for memory management. See MemoryConfig section below for details.
toolChoice?:
'auto' | 'none' | 'required' | { type: 'tool'; toolName: string }
= 'auto'
Controls how the agent uses tools during streaming.
telemetry?:
TelemetrySettings
Settings for telemetry collection during streaming. See TelemetrySettings section below for details.
threadId?:
string
Identifier for the conversation thread. Allows for maintaining context across multiple interactions. Must be provided if resourceId is provided.
resourceId?:
string
Identifier for the user or resource interacting with the agent. Must be provided if threadId is provided.
onFinish?:
(result: string) => Promise<void> | void
Callback function called when streaming is complete.
onStepFinish?:
(step: string) => void
Callback function called after each step during streaming.
maxSteps?:
number
= 5
Maximum number of steps allowed during streaming.
toolsets?:
ToolsetsInput
Additional toolsets to make available to the agent during this stream.
temperature?:
number
Controls randomness in the model's output. Higher values (e.g., 0.8) make the output more random, lower values (e.g., 0.2) make it more focused and deterministic.
MemoryConfig
Configuration options for memory management:

lastMessages?:
number | false
Number of most recent messages to include in context. Set to false to disable.
semanticRecall?:
boolean | object
Configuration for semantic memory recall. Can be boolean or detailed config.
number
topK?:
number
Number of most semantically similar messages to retrieve.
number | object
messageRange?:
number | { before: number; after: number }
Range of messages to consider for semantic search. Can be a single number or before/after configuration.
workingMemory?:
object
Configuration for working memory.
boolean
enabled?:
boolean
Whether to enable working memory.
string
template?:
string
Template to use for working memory.
'text-stream' | 'tool-call'
type?:
'text-stream' | 'tool-call'
Type of content to use for working memory.
threads?:
object
Thread-specific memory configuration.
boolean
generateTitle?:
boolean
Whether to automatically generate titles for new threads.
TelemetrySettings
Settings for telemetry collection during streaming:

isEnabled?:
boolean
= false
Enable or disable telemetry. Disabled by default while experimental.
recordInputs?:
boolean
= true
Enable or disable input recording. You might want to disable this to avoid recording sensitive information, reduce data transfers, or increase performance.
recordOutputs?:
boolean
= true
Enable or disable output recording. You might want to disable this to avoid recording sensitive information, reduce data transfers, or increase performance.
functionId?:
string
Identifier for this function. Used to group telemetry data by function.
metadata?:
Record<string, AttributeValue>
Additional information to include in the telemetry data. AttributeValue can be string, number, boolean, array of these types, or null.
tracer?:
Tracer
A custom OpenTelemetry tracer instance to use for the telemetry data. See OpenTelemetry documentation for details.
Returns
The return value of the stream() method depends on the options provided, specifically the output option.

PropertiesTable for Return Values
textStream?:
AsyncIterable<string>
Stream of text chunks. Present when output is 'text' (no schema provided) or when using `experimental_output`.
objectStream?:
AsyncIterable<object>
Stream of structured data. Present only when using `output` option with a schema.
partialObjectStream?:
AsyncIterable<object>
Stream of structured data. Present only when using `experimental_output` option.
object?:
Promise<object>
Promise that resolves to the final structured output. Present when using either `output` or `experimental_output` options.
Examples
Basic Text Streaming
const stream = await myAgent.stream([
  { role: "user", content: "Tell me a story." }
]);
 
for await (const chunk of stream.textStream) {
  process.stdout.write(chunk);
}
Structured Output Streaming with Thread Context
const schema = {
  type: 'object',
  properties: {
    summary: { type: 'string' },
    nextSteps: { type: 'array', items: { type: 'string' } }
  },
  required: ['summary', 'nextSteps']
};
 
const response = await myAgent.stream(
  "What should we do next?",
  {
    output: schema,
    threadId: "project-123",
    onFinish: text => console.log("Finished:", text)
  }
);
 
for await (const chunk of response.textStream) {
  console.log(chunk);
}
 
const result = await response.object;
console.log("Final structured result:", result);
The key difference between Agent’s stream() and LLM’s stream() is that Agents maintain conversation context through threadId, can access tools, and integrate with the agent’s memory system.

createDocumentChunkerTool()
The createDocumentChunkerTool() function creates a tool for splitting documents into smaller chunks for efficient processing and retrieval. It supports different chunking strategies and configurable parameters.

Basic Usage
import { createDocumentChunkerTool, MDocument } from "@mastra/rag";
 
const document = new MDocument({
  text: "Your document content here...",
  metadata: { source: "user-manual" }
});
 
const chunker = createDocumentChunkerTool({
  doc: document,
  params: {
    strategy: "recursive",
    size: 512,
    overlap: 50,
    separator: "\n"
  }
});
 
const { chunks } = await chunker.execute();
Parameters
doc:
MDocument
The document to be chunked
params?:
ChunkParams
= Default chunking parameters
Configuration parameters for chunking
ChunkParams
strategy?:
'recursive'
= 'recursive'
The chunking strategy to use
size?:
number
= 512
Target size of each chunk in tokens/characters
overlap?:
number
= 50
Number of overlapping tokens/characters between chunks
separator?:
string
= '\n'
Character(s) to use as chunk separator
Returns
chunks:
DocumentChunk[]
Array of document chunks with their content and metadata
Example with Custom Parameters
const technicalDoc = new MDocument({
  text: longDocumentContent,
  metadata: {
    type: "technical",
    version: "1.0"
  }
});
 
const chunker = createDocumentChunkerTool({
  doc: technicalDoc,
  params: {
    strategy: "recursive",
    size: 1024,      // Larger chunks
    overlap: 100,    // More overlap
    separator: "\n\n" // Split on double newlines
  }
});
 
const { chunks } = await chunker.execute();
 
// Process the chunks
chunks.forEach((chunk, index) => {
  console.log(`Chunk ${index + 1} length: ${chunk.content.length}`);
});
Tool Details
The chunker is created as a Mastra tool with the following properties:

Tool ID: Document Chunker {strategy} {size}
Description: Chunks document using {strategy} strategy with size {size} and {overlap} overlap
Input Schema: Empty object (no additional inputs required)
Output Schema: Object containing the chunks array
Related
MDocument
createVectorQueryTool

createGraphRAGTool()
The createGraphRAGTool() creates a tool that enhances RAG by building a graph of semantic relationships between documents. It uses the GraphRAG system under the hood to provide graph-based retrieval, finding relevant content through both direct similarity and connected relationships.

Usage Example
import { openai } from "@ai-sdk/openai";
import { createGraphRAGTool } from "@mastra/rag";
 
const graphTool = createGraphRAGTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
  graphOptions: {
    dimension: 1536,
    threshold: 0.7,
    randomWalkSteps: 100,
    restartProb: 0.15
  }
});
Parameters
vectorStoreName:
string
Name of the vector store to query
indexName:
string
Name of the index within the vector store
model:
EmbeddingModel
Embedding model to use for vector search
graphOptions?:
GraphOptions
= Default graph options
Configuration for the graph-based retrieval
description?:
string
Custom description for the tool. By default: 'Access and analyze relationships between information in the knowledge base to answer complex questions about connections and patterns'
GraphOptions
dimension?:
number
= 1536
Dimension of the embedding vectors
threshold?:
number
= 0.7
Similarity threshold for creating edges between nodes (0-1)
randomWalkSteps?:
number
= 100
Number of steps in random walk for graph traversal
restartProb?:
number
= 0.15
Probability of restarting random walk from query node
Returns
The tool returns an object with:

relevantContext:
string
Combined text from the most relevant document chunks, retrieved using graph-based ranking
Default Tool Description
The default description focuses on:

Analyzing relationships between documents
Finding patterns and connections
Answering complex queries
Advanced Example
const graphTool = createGraphRAGTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
  graphOptions: {
    dimension: 1536,
    threshold: 0.8,        // Higher similarity threshold
    randomWalkSteps: 200,  // More exploration steps
    restartProb: 0.2      // Higher restart probability
  }
});
Example with Custom Description
const graphTool = createGraphRAGTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
  description: "Analyze document relationships to find complex patterns and connections in our company's historical data"
});
This example shows how to customize the tool description for a specific use case while maintaining its core purpose of relationship analysis.

Related
createVectorQueryTool
GraphRAG

createVectorQueryTool()
The createVectorQueryTool() function creates a tool for semantic search over vector stores. It supports filtering, reranking, and integrates with various vector store backends.

Basic Usage
import { openai } from '@ai-sdk/openai';
import { createVectorQueryTool } from "@mastra/rag";
 
const queryTool = createVectorQueryTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
});
Parameters
vectorStoreName:
string
Name of the vector store to query (must be configured in Mastra)
indexName:
string
Name of the index within the vector store
model:
EmbeddingModel
Embedding model to use for vector search
reranker?:
RerankConfig
Options for reranking results
id?:
string
Custom ID for the tool (defaults to 'VectorQuery {vectorStoreName} {indexName} Tool')
description?:
string
Custom description for the tool. By default: 'Access the knowledge base to find information needed to answer user questions'
RerankConfig
model:
MastraLanguageModel
Language model to use for reranking
options?:
RerankerOptions
Options for the reranking process
object
weights?:
WeightConfig
Weights for scoring components (semantic: 0.4, vector: 0.4, position: 0.2)
topK?:
number
Number of top results to return
Returns
The tool returns an object with:

relevantContext:
string
Combined text from the most relevant document chunks
Default Tool Description
The default description focuses on:

Finding relevant information in stored knowledge
Answering user questions
Retrieving factual content
Result Handling
The tool determines the number of results to return based on the user’s query, with a default of 10 results. This can be adjusted based on the query requirements.

Example with Filters
const queryTool = createVectorQueryTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
  enableFilters: true,
});
With filtering enabled, the tool processes queries to construct metadata filters that combine with semantic search. The process works as follows:

A user makes a query with specific filter requirements like “Find content where the ‘version’ field is greater than 2.0”
The agent analyzes the query and constructs the appropriate filters:
{
   "version": { "$gt": 2.0 }
}
This agent-driven approach:

Processes natural language queries into filter specifications
Implements vector store-specific filter syntax
Translates query terms to filter operators
For detailed filter syntax and store-specific capabilities, see the Metadata Filters documentation.

For an example of how agent-driven filtering works, see the Agent-Driven Metadata Filtering example.

Example with Reranking
const queryTool = createVectorQueryTool({
  vectorStoreName: "milvus",
  indexName: "documentation",
  model: openai.embedding('text-embedding-3-small'),
  reranker: {
    model: openai('gpt-4o-mini'),
    options: {
      weights: {
        semantic: 0.5,  // Semantic relevance weight
        vector: 0.3,    // Vector similarity weight
        position: 0.2   // Original position weight
      },
      topK: 5
    }
  }
});
Reranking improves result quality by combining:

Semantic relevance: Using LLM-based scoring of text similarity
Vector similarity: Original vector distance scores
Position bias: Consideration of original result ordering
Query analysis: Adjustments based on query characteristics
The reranker processes the initial vector search results and returns a reordered list optimized for relevance.

Example with Custom Description
const queryTool = createVectorQueryTool({
  vectorStoreName: "pinecone",
  indexName: "docs",
  model: openai.embedding('text-embedding-3-small'),
  description: "Search through document archives to find relevant information for answering questions about company policies and procedures"
});
This example shows how to customize the tool description for a specific use case while maintaining its core purpose of information retrieval.

Tool Details
The tool is created with:

ID: VectorQuery {vectorStoreName} {indexName} Tool
Input Schema: Requires queryText and filter objects
Output Schema: Returns relevantContext string
Related
rerank()
createGraphRAGTool

Workflow Class
The Workflow class enables you to create state machines for complex sequences of operations with conditional branching and data validation.

import { Workflow } from "@mastra/core/workflows";
 
const workflow = new Workflow({ name: "my-workflow" });

API Reference
Constructor
name:
string
Identifier for the workflow
logger?:
Logger<WorkflowLogMessage>
Optional logger instance for workflow execution details
steps:
Step[]
Array of steps to include in the workflow
triggerSchema:
z.Schema
Optional schema for validating workflow trigger data
Core Methods
step()
Adds a Step to the workflow, including transitions to other steps. Returns the workflow instance for chaining. Learn more about steps.

commit()
Validates and finalizes the workflow configuration. Must be called after adding all steps.

execute()
Executes the workflow with optional trigger data. Typed based on the trigger schema.

Trigger Schemas
Trigger schemas validate the initial data passed to a workflow using Zod.

const workflow = new Workflow({
  name: "order-process",
  triggerSchema: z.object({
    orderId: z.string(),
    customer: z.object({
      id: z.string(),
      email: z.string().email(),
    }),
  }),
});

The schema:

Validates data passed to execute()
Provides TypeScript types for your workflow input
Validation
Workflow validation happens at two key times:

1. At Commit Time
When you call .commit(), the workflow validates:

workflow
  .step('step1', {...})
  .step('step2', {...})
  .commit(); // Validates workflow structure

Circular dependencies between steps
Terminal paths (every path must end)
Unreachable steps
Variable references to non-existent steps
Duplicate step IDs
2. During Execution
When you call start(), it validates:

const { runId, start } = workflow.createRun();
 
// Validates trigger data against schema
await start({
  triggerData: {
    orderId: "123",
    customer: {
      id: "cust_123",
      email: "invalid-email", // Will fail validation
    },
  },
});

Trigger data against trigger schema
Each step’s input data against its inputSchema
Variable paths exist in referenced step outputs
Required variables are present
Workflow Status
A workflow’s status indicates its current execution state. The possible values are:

CREATED:
string
Workflow instance has been created but not started
RUNNING:
string
Workflow is actively executing steps
SUSPENDED:
string
Workflow execution is paused waiting for resume
COMPLETED:
string
All steps finished executing successfully
FAILED:
string
Workflow encountered an error during execution
Example: Handling Different Statuses
const { runId, start } = workflow.createRun();
 
workflow.watch(runId, async ({ status }) => {
  switch (status) {
    case "SUSPENDED":
      // Handle suspended state
      break;
    case "COMPLETED":
      // Process results
      break;
    case "FAILED":
      // Handle error state
      break;
  }
});
 
await start({ triggerData: data });

Error Handling
try {
  const { runId, start } = workflow.createRun();
  await start({ triggerData: data });
} catch (error) {
  if (error instanceof ValidationError) {
    // Handle validation errors
    console.log(error.type); // 'circular_dependency' | 'no_terminal_path' | 'unreachable_step'
    console.log(error.details); // { stepId?: string, path?: string[] }
  }
}

Passing Context Between Steps
Steps can access data from previous steps in the workflow through the context object. Each step receives the accumulated context from all previous steps that have executed.

workflow
  .step({
    id: 'getData',
    execute: async ({ context }) => {
      return {
        data: { id: '123', value: 'example' }
      };
    }
  })
  .step({
    id: 'processData',
    execute: async ({ context }) => {
      // Access data from previous step through context.steps
      const previousData = context.steps.getData.output.data;
      // Process previousData.id and previousData.value
    }
  });

The context object:

Contains results from all completed steps in context.steps
Provides access to step outputs through context.steps.[stepId].output
Is typed based on step output schemas
Is immutable to ensure data consistency
Related Documentation
Step
.then()
.step()
.after()
Last updated on March 13, 2025


Step
The Step class defines individual units of work within a workflow, encapsulating execution logic, data validation, and input/output handling.

Usage
const processOrder = new Step({
  id: "processOrder",
  inputSchema: z.object({
    orderId: z.string(),
    userId: z.string()
  }),
  outputSchema: z.object({
    status: z.string(),
    orderId: z.string()
  }),
  execute: async ({ context, runId }) => {
    return {
      status: "processed",
      orderId: context.orderId
    };
  }
});
Constructor Parameters
id:
string
Unique identifier for the step
inputSchema:
z.ZodSchema
Zod schema to validate input data before execution
outputSchema:
z.ZodSchema
Zod schema to validate step output data
payload:
Record<string, any>
Static data to be merged with variables
execute:
(params: ExecuteParams) => Promise<any>
Async function containing step logic
ExecuteParams
context:
StepContext
Access to workflow context and step results
runId:
string
Unique identifier for current workflow run
suspend:
() => Promise<void>
Function to suspend step execution
mastra:
Mastra
Access to Mastra instance
Related
Workflow Reference
Step Configuration Guide
Control Flow Guide
Last updated on March 13, 2025


StepOptions
Configuration options for workflow steps that control variable mapping, execution conditions, and other runtime behavior.

Usage
workflow.step(processOrder, {
  variables: {
    orderId: { step: 'trigger', path: 'id' },
    userId: { step: 'auth', path: 'user.id' }
  },
  when: {
    ref: { step: 'auth', path: 'status' },
    query: { $eq: 'authenticated' }
  }
});
Properties
variables?:
Record<string, VariableRef>
Maps step input variables to values from other steps
when?:
StepCondition
Condition that must be met for step execution
VariableRef
step:
string | Step | { id: string }
Source step for the variable value
path:
string
Path to the value in the step's output
Related
Path Comparison
Step Function Reference
Step Class Reference
Workflow Class Reference
Control Flow Guide
Last updated on March 13, 2025


StepCondition
Conditions determine whether a step should execute based on the output of previous steps or trigger data.

Usage
There are three ways to specify conditions: function, query object, and simple path comparison.

1. Function Condition
workflow.step(processOrder, {
  when: async ({ context }) => {
    const auth = context?.getStepResult<{status: string}>("auth");
    return auth?.status === "authenticated";
  }
});

2. Query Object
workflow.step(processOrder, {
  when: {
    ref: { step: 'auth', path: 'status' },
    query: { $eq: 'authenticated' }
  }
});

3. Simple Path Comparison
workflow.step(processOrder, {
  when: {
    "auth.status": "authenticated"
  }
});

Based on the type of condition, the workflow runner will try to match the condition to one of these types.

Simple Path Condition (when there’s a dot in the key)
Base/Query Condition (when there’s a ‘ref’ property)
Function Condition (when it’s an async function)
StepCondition
ref:
{ stepId: string | 'trigger'; path: string }
Reference to step output value. stepId can be a step ID or 'trigger' for initial data. path specifies location of value in step result
query:
Query<any>
MongoDB-style query using sift operators ($eq, $gt, etc)
Query
The Query object provides MongoDB-style query operators for comparing values from previous steps or trigger data. It supports basic comparison operators like $eq, $gt, $lt as well as array operators like $in and $nin, and can be combined with and/or operators for complex conditions.

This query syntax allows for readable conditional logic for determining whether a step should execute.

$eq:
any
Equal to value
$ne:
any
Not equal to value
$gt:
number
Greater than value
$gte:
number
Greater than or equal to value
$lt:
number
Less than value
$lte:
number
Less than or equal to value
$in:
any[]
Value exists in array
$nin:
any[]
Value does not exist in array
and:
StepCondition[]
Array of conditions that must all be true
or:
StepCondition[]
Array of conditions where at least one must be true
Related
Step Options Reference
Step Function Reference
Control Flow Guide
Last updated on March 13, 2025


Workflow.step()
The .step() method adds a new step to the workflow, optionally configuring its variables and execution conditions.

Usage
workflow.step({
  id: "stepTwo",
  outputSchema: z.object({
    result: z.number()
  }),
  execute: async ({ context }) => {
    return { result: 42 };
  }
});
Parameters
stepConfig:
Step | StepDefinition | string
Step instance, configuration object, or step ID to add to workflow
options?:
StepOptions
Optional configuration for step execution
StepDefinition
id:
string
Unique identifier for the step
outputSchema?:
z.ZodSchema
Schema for validating step output
execute:
(params: ExecuteParams) => Promise<any>
Function containing step logic
StepOptions
variables?:
Record<string, VariableRef>
Map of variable names to their source references
when?:
StepCondition
Condition that must be met for step to execute
Related
Basic Usage with Step Instance
Step Class Reference
Workflow Class Reference
Control Flow Guide
Last updated on March 13, 2025


.after()
The .after() method defines explicit dependencies between workflow steps, enabling branching and merging paths in your workflow execution.

Usage
Basic Branching
workflow
  .step(stepA)
    .then(stepB)
  .after(stepA)  // Create new branch after stepA completes
    .step(stepC);
Merging Multiple Branches
workflow
  .step(stepA)
    .then(stepB)
  .step(stepC)
    .then(stepD)
  .after([stepB, stepD])  // Create a step that depends on multiple steps
    .step(stepE);
Parameters
steps:
Step | Step[]
A single step or array of steps that must complete before continuing
Returns
workflow:
Workflow
The workflow instance for method chaining
Examples
Single Dependency
workflow
  .step(fetchData)
  .then(processData)
  .after(fetchData)  // Branch after fetchData
  .step(logData);
Multiple Dependencies (Merging Branches)
workflow
  .step(fetchUserData)
  .then(validateUserData)
  .step(fetchProductData)
  .then(validateProductData)
  .after([validateUserData, validateProductData])  // Wait for both validations to complete
  .step(processOrder);
Related
Branching Paths example
Workflow Class Reference
Step Reference
Control Flow Guide
Last updated on March 13, 2025


Workflow.then()
The .then() method creates a sequential dependency between workflow steps, ensuring steps execute in a specific order.

Usage
workflow
  .step(stepOne)
  .then(stepTwo)
  .then(stepThree);
Parameters
step:
Step | string
The step instance or step ID that should execute after the previous step completes
Returns
workflow:
Workflow
The workflow instance for method chaining
Validation
When using then:

The previous step must exist in the workflow
Steps cannot form circular dependencies
Each step can only appear once in a sequential chain
Error Handling
try {
  workflow
    .step(stepA)
    .then(stepB)
    .then(stepA) // Will throw error - circular dependency
    .commit();
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(error.type); // 'circular_dependency'
    console.log(error.details);
  }
}
Related
step Reference
after Reference
Sequential Steps Example
Control Flow Guide
Last updated on March 13, 2025


Workflow.until()
The .until() method repeats a step until a specified condition becomes true. This creates a loop that continues executing the specified step until the condition is satisfied.

Usage
workflow
  .step(incrementStep)
  .until(condition, incrementStep)
  .then(finalStep);
Parameters
condition:
Function | ReferenceCondition
A function or reference condition that determines when to stop looping
step:
Step
The step to repeat until the condition is met
Condition Types
Function Condition
You can use a function that returns a boolean:

workflow
  .step(incrementStep)
  .until(async ({ context }) => {
    const result = context.getStepResult<{ value: number }>('increment');
    return (result?.value ?? 0) >= 10; // Stop when value reaches or exceeds 10
  }, incrementStep)
  .then(finalStep);
Reference Condition
You can use a reference-based condition with comparison operators:

workflow
  .step(incrementStep)
  .until(
    {
      ref: { step: incrementStep, path: 'value' },
      query: { $gte: 10 }, // Stop when value is greater than or equal to 10
    },
    incrementStep
  )
  .then(finalStep);
Comparison Operators
When using reference-based conditions, you can use these comparison operators:

Operator	Description	Example
$eq	Equal to	{ $eq: 10 }
$ne	Not equal to	{ $ne: 0 }
$gt	Greater than	{ $gt: 5 }
$gte	Greater than or equal to	{ $gte: 10 }
$lt	Less than	{ $lt: 20 }
$lte	Less than or equal to	{ $lte: 15 }
Returns
workflow:
Workflow
The workflow instance for chaining
Example
import { Workflow, Step } from '@mastra/core';
import { z } from 'zod';
 
// Create a step that increments a counter
const incrementStep = new Step({
  id: 'increment',
  description: 'Increments the counter by 1',
  outputSchema: z.object({
    value: z.number(),
  }),
  execute: async ({ context }) => {
    // Get current value from previous execution or start at 0
    const currentValue =
      context.getStepResult<{ value: number }>('increment')?.value ||
      context.getStepResult<{ startValue: number }>('trigger')?.startValue ||
      0;
 
    // Increment the value
    const value = currentValue + 1;
    console.log(`Incrementing to ${value}`);
 
    return { value };
  },
});
 
// Create a final step
const finalStep = new Step({
  id: 'final',
  description: 'Final step after loop completes',
  execute: async ({ context }) => {
    const finalValue = context.getStepResult<{ value: number }>('increment')?.value;
    console.log(`Loop completed with final value: ${finalValue}`);
    return { finalValue };
  },
});
 
// Create the workflow
const counterWorkflow = new Workflow({
  name: 'counter-workflow',
  triggerSchema: z.object({
    startValue: z.number(),
    targetValue: z.number(),
  }),
});
 
// Configure the workflow with an until loop
counterWorkflow
  .step(incrementStep)
  .until(async ({ context }) => {
    const targetValue = context.triggerData.targetValue;
    const currentValue = context.getStepResult<{ value: number }>('increment')?.value ?? 0;
    return currentValue >= targetValue;
  }, incrementStep)
  .then(finalStep)
  .commit();
 
// Execute the workflow
const run = counterWorkflow.createRun();
const result = await run.start({ triggerData: { startValue: 0, targetValue: 5 } });
// Will increment from 0 to 5, then stop and execute finalStep
Related
.while() - Loop while a condition is true
Control Flow Guide
Workflow Class Reference
Last updated on March 13, 2025


Workflow.while()
The .while() method repeats a step as long as a specified condition remains true. This creates a loop that continues executing the specified step until the condition becomes false.

Usage
workflow
  .step(incrementStep)
  .while(condition, incrementStep)
  .then(finalStep);
Parameters
condition:
Function | ReferenceCondition
A function or reference condition that determines when to continue looping
step:
Step
The step to repeat while the condition is true
Condition Types
Function Condition
You can use a function that returns a boolean:

workflow
  .step(incrementStep)
  .while(async ({ context }) => {
    const result = context.getStepResult<{ value: number }>('increment');
    return (result?.value ?? 0) < 10; // Continue as long as value is less than 10
  }, incrementStep)
  .then(finalStep);
Reference Condition
You can use a reference-based condition with comparison operators:

workflow
  .step(incrementStep)
  .while(
    {
      ref: { step: incrementStep, path: 'value' },
      query: { $lt: 10 }, // Continue as long as value is less than 10
    },
    incrementStep
  )
  .then(finalStep);
Comparison Operators
When using reference-based conditions, you can use these comparison operators:

Operator	Description	Example
$eq	Equal to	{ $eq: 10 }
$ne	Not equal to	{ $ne: 0 }
$gt	Greater than	{ $gt: 5 }
$gte	Greater than or equal to	{ $gte: 10 }
$lt	Less than	{ $lt: 20 }
$lte	Less than or equal to	{ $lte: 15 }
Returns
workflow:
Workflow
The workflow instance for chaining
Example
import { Workflow, Step } from '@mastra/core';
import { z } from 'zod';
 
// Create a step that increments a counter
const incrementStep = new Step({
  id: 'increment',
  description: 'Increments the counter by 1',
  outputSchema: z.object({
    value: z.number(),
  }),
  execute: async ({ context }) => {
    // Get current value from previous execution or start at 0
    const currentValue =
      context.getStepResult<{ value: number }>('increment')?.value ||
      context.getStepResult<{ startValue: number }>('trigger')?.startValue ||
      0;
 
    // Increment the value
    const value = currentValue + 1;
    console.log(`Incrementing to ${value}`);
 
    return { value };
  },
});
 
// Create a final step
const finalStep = new Step({
  id: 'final',
  description: 'Final step after loop completes',
  execute: async ({ context }) => {
    const finalValue = context.getStepResult<{ value: number }>('increment')?.value;
    console.log(`Loop completed with final value: ${finalValue}`);
    return { finalValue };
  },
});
 
// Create the workflow
const counterWorkflow = new Workflow({
  name: 'counter-workflow',
  triggerSchema: z.object({
    startValue: z.number(),
    targetValue: z.number(),
  }),
});
 
// Configure the workflow with a while loop
counterWorkflow
  .step(incrementStep)
  .while(
    async ({ context }) => {
      const targetValue = context.triggerData.targetValue;
      const currentValue = context.getStepResult<{ value: number }>('increment')?.value ?? 0;
      return currentValue < targetValue;
    },
    incrementStep
  )
  .then(finalStep)
  .commit();
 
// Execute the workflow
const run = counterWorkflow.createRun();
const result = await run.start({ triggerData: { startValue: 0, targetValue: 5 } });
// Will increment from 0 to 4, then stop and execute finalStep
Related
.until() - Loop until a condition becomes true
Control Flow Guide
Workflow Class Reference
Last updated on March 13, 2025


Workflow.if()
Experimental

The .if() method creates a conditional branch in the workflow, allowing steps to execute only when a specified condition is true. This enables dynamic workflow paths based on the results of previous steps.

Usage
workflow
  .step(startStep)
  .if(async ({ context }) => {
    const value = context.getStepResult<{ value: number }>('start')?.value;
    return value < 10; // If true, execute the "if" branch
  })
  .then(ifBranchStep)
  .else()
  .then(elseBranchStep)
  .commit();

Parameters
condition:
Function | ReferenceCondition
A function or reference condition that determines whether to execute the 'if' branch
Condition Types
Function Condition
You can use a function that returns a boolean:

workflow
  .step(startStep)
  .if(async ({ context }) => {
    const result = context.getStepResult<{ status: string }>('start');
    return result?.status === 'success'; // Execute "if" branch when status is "success"
  })
  .then(successStep)
  .else()
  .then(failureStep);
Reference Condition
You can use a reference-based condition with comparison operators:

workflow
  .step(startStep)
  .if({
    ref: { step: startStep, path: 'value' },
    query: { $lt: 10 }, // Execute "if" branch when value is less than 10
  })
  .then(ifBranchStep)
  .else()
  .then(elseBranchStep);
Returns
workflow:
Workflow
The workflow instance for method chaining
Error Handling
The if method requires a previous step to be defined. If you try to use it without a preceding step, an error will be thrown:

try {
  // This will throw an error
  workflow
    .if(async ({ context }) => true)
    .then(someStep)
    .commit();
} catch (error) {
  console.error(error); // "Condition requires a step to be executed after"
}
Related
else Reference
then Reference
Control Flow Guide
Step Condition Reference
Last updated on March 13, 2025


Workflow.else()
Experimental

The .else() method creates an alternative branch in the workflow that executes when the preceding if condition evaluates to false. This enables workflows to follow different paths based on conditions.

Usage
workflow
  .step(startStep)
  .if(async ({ context }) => {
    const value = context.getStepResult<{ value: number }>('start')?.value;
    return value < 10;
  })
  .then(ifBranchStep)
  .else() // Alternative branch when the condition is false
  .then(elseBranchStep)
  .commit();

Parameters
The else() method does not take any parameters.

Returns
workflow:
Workflow
The workflow instance for method chaining
Behavior
The else() method must follow an if() branch in the workflow definition
It creates a branch that executes only when the preceding if condition evaluates to false
You can chain multiple steps after an else() using .then()
You can nest additional if/else conditions within an else branch
Error Handling
The else() method requires a preceding if() statement. If you try to use it without a preceding if, an error will be thrown:

try {
  // This will throw an error
  workflow
    .step(someStep)
    .else()
    .then(anotherStep)
    .commit();
} catch (error) {
  console.error(error); // "No active condition found"
}
Related
if Reference
then Reference
Control Flow Guide
Step Condition Reference
Last updated on March 13, 2025


Workflow.createRun()
The .createRun() method initializes a new workflow run instance. It generates a unique run ID for tracking and returns a start function that begins workflow execution when called.

One reason to use .createRun() vs .execute() is to get a unique run ID for tracking, logging, or subscribing via .watch().

Usage
const { runId, start } = workflow.createRun();
 
const result = await start();
Returns
runId:
string
Unique identifier for tracking this workflow run
start:
() => Promise<WorkflowResult>
Function that begins workflow execution when called
Error Handling
The start function may throw validation errors if the workflow configuration is invalid:

try {
  const { runId, start } = workflow.createRun();
  await start({ triggerData: data });
} catch (error) {
  if (error instanceof ValidationError) {
    // Handle validation errors
    console.log(error.type); // 'circular_dependency' | 'no_terminal_path' | 'unreachable_step'
    console.log(error.details);
  }
}
Related
Workflow Class Reference
Step Class Reference
See the Creating a Workflow example for complete usage
Last updated on March 13, 2025


start()
The start function begins execution of a workflow run. It processes all steps in the defined workflow order, handling parallel execution, branching logic, and step dependencies.

Usage
const { runId, start } = workflow.createRun();
const result = await start({ 
  triggerData: { inputValue: 42 } 
});

Parameters
config?:
object
Configuration for starting the workflow run
config
triggerData:
Record<string, any>
Initial data that matches the workflow's triggerSchema
Returns
results:
Record<string, any>
Combined output from all completed workflow steps
status:
'completed' | 'error' | 'suspended'
Final status of the workflow run
Error Handling
The start function may throw several types of validation errors:

try {
  const result = await start({ triggerData: data });
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(error.type); // 'circular_dependency' | 'no_terminal_path' | 'unreachable_step'
    console.log(error.details);
  }
}

Related
Example: Creating a Workflow
Example: Suspend and Resume
createRun Reference
Workflow Class Reference
Step Class Reference
Last updated on March 13, 2025


Workflow.execute()
Executes a workflow with the provided trigger data and returns the results. The workflow must be committed before execution.

Usage Example
const workflow = new Workflow({
  name: "my-workflow",
  triggerSchema: z.object({
    inputValue: z.number()
  })
});
 
workflow.step(stepOne).then(stepTwo).commit();
 
const result = await workflow.execute({
  triggerData: { inputValue: 42 }
});
Parameters
options?:
ExecuteOptions
Options for workflow execution
TriggerSchema
string
Returns
WorkflowResult:
object
Results from workflow execution
string
Record<string, StepResult>
WorkflowStatus
Additional Examples
Execute with run ID:

const result = await workflow.execute({
  runId: "custom-run-id",
  triggerData: { inputValue: 42 }
});
Handle execution results:

const { runId, results, status } = await workflow.execute({
  triggerData: { inputValue: 42 }
});
 
if (status === "COMPLETED") {
  console.log("Step results:", results);
}
Related
Workflow.createRun()
Workflow.commit()
Workflow.start()
Last updated on March 13, 2025


suspend()
Pauses workflow execution at the current step until explicitly resumed. The workflow state is persisted and can be continued later.

Usage Example
const approvalStep = new Step({
  id: "needsApproval",
  execute: async ({ context, suspend }) => {
    if (context.steps.amount > 1000) {
      await suspend();
    }
    return { approved: true };
  }
});
Parameters
metadata?:
Record<string, any>
Optional data to store with the suspended state
Returns
Promise<void>:
Promise
Resolves when the workflow is successfully suspended
Additional Examples
Suspend with metadata:

const reviewStep = new Step({
  id: "review",
  execute: async ({ context, suspend }) => {
    await suspend({
      reason: "Needs manager approval",
      requestedBy: context.user
    });
    return { reviewed: true };
  }
});
Monitor suspended state:

workflow.watch((state) => {
  if (state.status === "SUSPENDED") {
    notifyReviewers(state.metadata);
  }
});
Related
Suspend & Resume Workflows
.resume()
.watch()
Last updated on March 13, 2025


Workflow.resume()
The .resume() method continues execution of a suspended workflow step, optionally providing new context data that can be accessed by the step on the resumeData property.

Usage
await workflow.resume({
  runId: "abc-123",
  stepId: "stepTwo",
  context: {
    secondValue: 100
  }
});

Parameters
config:
object
Configuration for resuming the workflow
config
runId:
string
Unique identifier of the workflow run to resume
stepId:
string
ID of the suspended step to resume
context?:
Record<string, any>
New context data to merge with existing step results
Returns
Promise<WorkflowResult>:
object
Result of the resumed workflow execution
Async/Await Flow
When a workflow is resumed, execution continues from the point immediately after the suspend() call in the step’s execution function. This creates a natural flow in your code:

// Step definition with suspend point
const reviewStep = new Step({
  id: "review",
  execute: async ({ context, suspend }) => {
    // First part of execution
    const initialAnalysis = analyzeData(context.data);
 
    if (initialAnalysis.needsReview) {
      // Suspend execution here
      await suspend({ analysis: initialAnalysis });
 
      // This code runs after resume() is called
      // context now contains any data provided during resume
      return {
        reviewedData: enhanceWithFeedback(initialAnalysis, context.feedback)
      };
    }
 
    return { reviewedData: initialAnalysis };
  }
});
 
// Later, resume the workflow
const result = await workflow.resume({
  runId: "workflow-123",
  stepId: "review",
  context: {
    // This data will be available in `context.resumeData`
    feedback: "Looks good, but improve section 3"
  }
});
Execution Flow
The workflow runs until it hits await suspend() in the review step
The workflow state is persisted and execution pauses
Later, workflow.resume() is called with new context data
Execution continues from the point after suspend() in the review step
The new context data (feedback) is available to the step
The step completes and returns its result
The workflow continues with subsequent steps
Error Handling
The resume function may throw several types of errors:

try {
  await workflow.resume({
    runId,
    stepId: "stepTwo",
    context: newData
  });
} catch (error) {
  if (error.message === "No snapshot found for workflow run") {
    // Handle missing workflow state
  }
  if (error.message === "Failed to parse workflow snapshot") {
    // Handle corrupted workflow state
  }
}
Related
Suspend and Resume
suspend Reference
watch Reference
Workflow Class Reference
Last updated on March 13, 2025


Workflow.commit()
The .commit() method re-initializes the workflow’s state machine with the current step configuration.

Usage
workflow
  .step(stepA)
  .then(stepB)
  .commit();
Returns
workflow:
Workflow
The workflow instance
Related
Branching Paths example
Workflow Class Reference
Step Reference
Control Flow Guide
Last updated on March 13, 2025


Workflow.watch()
The .watch() function subscribes to state changes in a Mastra workflow, allowing you to monitor execution progress and react to state updates.

Usage Example
import { Workflow } from "@mastra/core/workflows";
 
const workflow = new Workflow({
  name: "document-processor"
});
 
// Subscribe to state changes
const unsubscribe = workflow.watch((state) => {
  console.log('Current step:', state.currentStep);
  console.log('Step outputs:', state.stepOutputs);
});
 
// Run the workflow
await workflow.run({
  input: { text: "Process this document" }
});
 
// Stop watching
unsubscribe();
Parameters
callback:
(state: WorkflowState) => void
Function called whenever the workflow state changes
WorkflowState Properties
currentStep:
string
ID of the currently executing step
stepOutputs:
Record<string, any>
Outputs from completed workflow steps
status:
'running' | 'completed' | 'failed'
Current status of the workflow
error?:
Error | null
Error object if workflow failed
Returns
unsubscribe:
() => void
Function to stop watching workflow state changes
Additional Examples
Monitor specific step completion:

workflow.watch((state) => {
  if (state.currentStep === 'processDocument') {
    console.log('Document processing output:', state.stepOutputs.processDocument);
  }
});
Error handling:

workflow.watch((state) => {
  if (state.status === 'failed') {
    console.error('Workflow failed:', state.error);
    // Implement error recovery logic
  }
});
Related
Workflow Creation
Step Configuration
Last updated on March 13, 2025


Memory Class Reference
The Memory class provides a robust system for managing conversation history and thread-based message storage in Mastra. It enables persistent storage of conversations, semantic search capabilities, and efficient message retrieval. By default, it uses LibSQL for storage and vector search, and FastEmbed for embeddings.

Basic Usage
import { Memory } from "@mastra/memory";
import { Agent } from "@mastra/core/agent";
 
const agent = new Agent({
  memory: new Memory(),
  ...otherOptions,
});

Custom Configuration
import { Memory } from "@mastra/memory";
import { LibSQLStore } from "@mastra/core/storage/libsql";
import { LibSQLVector } from "@mastra/core/vector/libsql";
import { Agent } from "@mastra/core/agent";
 
const memory = new Memory({
  // Optional storage configuration - libsql will be used by default
  storage: new LibSQLStore({
    url: "file:memory.db",
  }),
 
  // Optional vector database for semantic search - libsql will be used by default
  vector: new LibSQLVector({
    url: "file:vector.db",
  }),
 
  // Memory configuration options
  options: {
    // Number of recent messages to include
    lastMessages: 20,
 
    // Semantic search configuration
    semanticRecall: {
      topK: 3, // Number of similar messages to retrieve
      messageRange: {
        // Messages to include around each result
        before: 2,
        after: 1,
      },
    },
 
    // Working memory configuration
    workingMemory: {
      enabled: true,
      template: "<user><first_name></first_name><last_name></last_name></user>",
    },
  },
});
 
const agent = new Agent({
  memory,
  ...otherOptions,
});

Parameters
storage?:
MastraStorage
Storage implementation for persisting memory data
vector?:
MastraVector
Vector store for semantic search capabilities
embedder?:
EmbeddingModel
Embedder instance for vector embeddings. Uses FastEmbed (bge-small-en-v1.5) by default
options?:
MemoryConfig
General memory configuration options
options
lastMessages?:
number | false
= 40
Number of most recent messages to retrieve. Set to false to disable.
semanticRecall?:
boolean | SemanticRecallConfig
= false (true if vector store provided)
Enable semantic search in message history. Automatically enabled when vector store is provided.
topK?:
number
= 2
Number of similar messages to retrieve when using semantic search
messageRange?:
number | { before: number; after: number }
= 2
Range of messages to include around semantic search results
workingMemory?:
{ enabled: boolean; template?: string; use?: 'text-stream' | 'tool-call' }
= { enabled: false, template: '<user><first_name></first_name><last_name></last_name>...</user>', use: 'text-stream' }
Configuration for working memory feature that allows persistent storage of user information across conversations. The 'use' setting determines how working memory updates are handled - either through text stream tags or tool calls.
threads?:
{ generateTitle?: boolean }
= { generateTitle: true }
Settings related to memory thread creation. `generateTitle` will cause the thread.title to be generated from an llm summary of the users first message.
Working Memory
The working memory feature allows agents to maintain persistent information across conversations. When enabled, the Memory class will automatically manage XML-based working memory updates through either text stream tags or tool calls.

There are two modes for handling working memory updates:

text-stream (default): The agent includes working memory updates directly in its responses using XML-like tags (<working_memory>...</working_memory>). These tags are automatically processed and stripped from the visible output.

tool-call: The agent uses a dedicated tool to update working memory. This mode should be used when working with toDataStream() as text-stream mode is not compatible with data streaming. Additionally, this mode provides more explicit control over memory updates and may be preferred when working with agents that are better at using tools than managing text tags.

Example configuration:

const memory = new Memory({
  options: {
    workingMemory: {
      enabled: true,
      template: "<user><first_name></first_name><last_name></last_name></user>",
      use: "tool-call", // or 'text-stream'
    },
  },
});

If no template is provided, the Memory class uses a default template that includes fields for user details, preferences, goals, and other contextual information. See the Agent Memory Guide for detailed usage examples and best practices.

embedder
By default, Memory uses FastEmbed with the bge-small-en-v1.5 model, which provides a good balance of performance and model size (~130MB). You only need to specify an embedder if you want to use a different model or provider.

Related
createThread
query
Last updated on March 13, 2025

createThread
Creates a new conversation thread in the memory system. Each thread represents a distinct conversation or context and can contain multiple messages.

Usage Example
import { Memory } from "@mastra/memory";
 
const memory = new Memory({ /* config */ });
const thread = await memory.createThread({
  resourceId: "user-123",
  title: "Support Conversation",
  metadata: {
    category: "support",
    priority: "high"
  }
});
Parameters
resourceId:
string
Identifier for the resource this thread belongs to (e.g., user ID, project ID)
threadId?:
string
Optional custom ID for the thread. If not provided, one will be generated.
title?:
string
Optional title for the thread
metadata?:
Record<string, unknown>
Optional metadata to associate with the thread
Returns
id:
string
Unique identifier of the created thread
resourceId:
string
Resource ID associated with the thread
title:
string
Title of the thread (if provided)
createdAt:
Date
Timestamp when the thread was created
updatedAt:
Date
Timestamp when the thread was last updated
metadata:
Record<string, unknown>
Additional metadata associated with the thread
Related
Memory
getThreadById
getThreadsByResourceId
Last updated on March 13, 2025

query
Retrieves messages from a specific thread, with support for pagination and filtering options.

Usage Example
import { Memory } from "@mastra/memory";
 
const memory = new Memory({
  /* config */
});
 
// Get last 50 messages
const { messages, uiMessages } = await memory.query({
  threadId: "thread-123",
  selectBy: {
    last: 50,
  },
});
 
// Get messages with context around specific messages
const { messages: contextMessages } = await memory.query({
  threadId: "thread-123",
  selectBy: {
    include: [
      {
        id: "msg-123", // Get just this message (no context)
      },
      {
        id: "msg-456", // Get this message with custom context
        withPreviousMessages: 3, // 3 messages before
        withNextMessages: 1, // 1 message after
      },
    ],
  },
});
 
// Semantic search in messages
const { messages } = await memory.query({
  threadId: "thread-123",
  selectBy: {
    vectorSearchString: "What was discussed about deployment?",
  },
  threadConfig: {
    historySearch: true,
  },
});
Parameters
threadId:
string
The unique identifier of the thread to retrieve messages from
resourceId?:
string
Optional ID of the resource that owns the thread. If provided, validates thread ownership
selectBy?:
object
Options for filtering messages
threadConfig?:
MemoryConfig
Configuration options for message retrieval
selectBy
vectorSearchString?:
string
Search string for finding semantically similar messages
last?:
number | false
= 40
Number of most recent messages to retrieve. Set to false to disable limit. Note: threadConfig.lastMessages (default: 40) will override this if smaller.
include?:
array
Array of message IDs to include with context
include
id:
string
ID of the message to include
withPreviousMessages?:
number
Number of messages to include before this message. Defaults to 2 when using vector search, 0 otherwise.
withNextMessages?:
number
Number of messages to include after this message. Defaults to 2 when using vector search, 0 otherwise.
Returns
messages:
CoreMessage[]
Array of retrieved messages in their core format
uiMessages:
AiMessage[]
Array of messages formatted for UI display
Additional Notes
The query function returns two different message formats:

messages: Core message format used internally
uiMessages: Formatted messages suitable for UI display, including proper threading of tool calls and results
Related
Memory
Last updated on March 13, 2025


getThreadById Reference
The getThreadById function retrieves a specific thread by its ID from storage.

Usage Example
import { Memory } from "@mastra/core/memory";
 
const memory = new Memory(config);
 
const thread = await memory.getThreadById({ threadId: "thread-123" });
Parameters
threadId:
string
The ID of the thread to be retrieved.
Returns
StorageThreadType | null:
Promise
A promise that resolves to the thread associated with the given ID, or null if not found.
Related
Memory
Last updated on March 13, 2025

getThreadsByResourceId Reference
The getThreadsByResourceId function retrieves all threads associated with a specific resource ID from storage.

Usage Example
import { Memory } from "@mastra/core/memory";
 
const memory = new Memory(config);
 
const threads = await memory.getThreadsByResourceId({
  resourceId: "resource-123",
});
Parameters
resourceId:
string
The ID of the resource whose threads are to be retrieved.
Returns
StorageThreadType[]:
Promise
A promise that resolves to an array of threads associated with the given resource ID.
Related
Memory
Last updated on March 13, 2025

LibSQL Storage
The LibSQL storage implementation provides a SQLite-compatible storage solution that can run both in-memory and as a persistent database.

Installation
npm install @mastra/storage-libsql
Usage
import { LibSQLStore } from "@mastra/core/storage/libsql";
 
// File database (development)
const storage = new LibSQLStore({
    config: {
        url: 'file:storage.db',
    }
});
 
// Persistent database (production)
const storage = new LibSQLStore({
    config: {
        url: process.env.DATABASE_URL,
    }
});

Parameters
url:
string
Database URL. Use ':memory:' for in-memory database, 'file:filename.db' for a file database, or any LibSQL-compatible connection string for persistent storage.
authToken?:
string
Authentication token for remote LibSQL databases.
Additional Notes
In-Memory vs Persistent Storage
The file configuration (file:storage.db) is useful for:

Development and testing
Temporary storage
Quick prototyping
For production use cases, use a persistent database URL: libsql://your-database.turso.io

Schema Management
The storage implementation handles schema creation and updates automatically. It creates the following tables:

threads: Stores conversation threads
messages: Stores individual messages
metadata: Stores additional metadata for threads and messages
Last updated on March 14, 2025


PostgreSQL Storage
The PostgreSQL storage implementation provides a production-ready storage solution using PostgreSQL databases.

Installation
npm install @mastra/pg
Usage
import { PostgresStore } from "@mastra/pg";
 
const storage = new PostgresStorage({
  connectionString: process.env.DATABASE_URL,
});

Parameters
connectionString:
string
PostgreSQL connection string (e.g., postgresql://user:pass@host:5432/dbname)
Additional Notes
Schema Management
The storage implementation handles schema creation and updates automatically. It creates the following tables:

threads: Stores conversation threads
messages: Stores individual messages
metadata: Stores additional metadata for threads and messages
Last updated on March 13, 2025


Upstash Storage
The Upstash storage implementation provides a serverless-friendly storage solution using Upstash’s Redis-compatible key-value store.

Installation
npm install @mastra/upstash
Usage
import { UpstashStore } from "@mastra/upstash";
 
const storage = new UpstashStore({
  url: process.env.UPSTASH_URL,
  token: process.env.UPSTASH_TOKEN,
});

Parameters
url:
string
Upstash Redis URL
token:
string
Upstash Redis authentication token
prefix?:
string
= mastra:
Key prefix for all stored items
Additional Notes
Key Structure
The Upstash storage implementation uses a key-value structure:

Thread keys: {prefix}thread:{threadId}
Message keys: {prefix}message:{messageId}
Metadata keys: {prefix}metadata:{entityId}
Serverless Benefits
Upstash storage is particularly well-suited for serverless deployments:

No connection management needed
Pay-per-request pricing
Global replication options
Edge-compatible
Data Persistence
Upstash provides:

Automatic data persistence
Point-in-time recovery
Cross-region replication options
Performance Considerations
For optimal performance:

Use appropriate key prefixes to organize data
Monitor Redis memory usage
Consider data expiration policies if needed
Last updated on March 13, 2025


Reference: .chunk()
The .chunk() function splits documents into smaller segments using various strategies and options.

Example
import { Document } from '@mastra/core';
 
const doc = new Document(`
# Introduction
This is a sample document that we want to split into chunks.
 
## Section 1
Here is the first section with some content.
 
## Section 2 
Here is another section with different content.
`);
 
// Basic chunking with defaults
const chunks = await doc.chunk();
 
// Markdown-specific chunking with header extraction
const chunksWithMetadata = await doc.chunk({
  strategy: 'markdown',
  headers: [['#', 'title'], ['##', 'section']],
  extract: {
    fields: [
      { name: 'summary', description: 'A brief summary of the chunk content' },
      { name: 'keywords', description: 'Key terms found in the chunk' }
    ]
  }
});
Parameters
strategy?:
'recursive' | 'character' | 'token' | 'markdown' | 'html' | 'json' | 'latex'
The chunking strategy to use. If not specified, defaults based on document type. Depending on the chunking strategy, there are additional optionals. Defaults: .md files → 'markdown', .html/.htm → 'html', .json → 'json', .tex → 'latex', others → 'recursive'
size?:
number
= 512
Maximum size of each chunk
overlap?:
number
= 50
Number of characters/tokens that overlap between chunks.
separator?:
string
= \n\n
Character(s) to split on. Defaults to double newline for text content.
isSeparatorRegex?:
boolean
= false
Whether the separator is a regex pattern
keepSeparator?:
'start' | 'end'
Whether to keep the separator at the start or end of chunks
extract?:
ExtractParams
Metadata extraction configuration. See [ExtractParams reference](./extract-params) for details.
Strategy-Specific Options
Strategy-specific options are passed as top-level parameters alongside the strategy parameter. For example:

// HTML strategy example
const chunks = await doc.chunk({
  strategy: 'html',
  headers: [['h1', 'title'], ['h2', 'subtitle']], // HTML-specific option
  sections: [['div.content', 'main']], // HTML-specific option
  size: 500 // general option
});
 
// Markdown strategy example
const chunks = await doc.chunk({
  strategy: 'markdown',
  headers: [['#', 'title'], ['##', 'section']], // Markdown-specific option
  stripHeaders: true, // Markdown-specific option
  overlap: 50 // general option
});
 
// Token strategy example
const chunks = await doc.chunk({
  strategy: 'token',
  encodingName: 'gpt2', // Token-specific option
  modelName: 'gpt-3.5-turbo', // Token-specific option
  size: 1000 // general option
});

The options documented below are passed directly at the top level of the configuration object, not nested within a separate options object.

HTML
headers:
Array<[string, string]>
Array of [selector, metadata key] pairs for header-based splitting
sections:
Array<[string, string]>
Array of [selector, metadata key] pairs for section-based splitting
returnEachLine?:
boolean
Whether to return each line as a separate chunk
Markdown
headers:
Array<[string, string]>
Array of [header level, metadata key] pairs
stripHeaders?:
boolean
Whether to remove headers from the output
returnEachLine?:
boolean
Whether to return each line as a separate chunk
Token
encodingName?:
string
Name of the token encoding to use
modelName?:
string
Name of the model for tokenization
JSON
maxSize:
number
Maximum size of each chunk
minSize?:
number
Minimum size of each chunk
ensureAscii?:
boolean
Whether to ensure ASCII encoding
convertLists?:
boolean
Whether to convert lists in the JSON
Return Value
Returns a MDocument instance containing the chunked documents. Each chunk includes:

interface DocumentNode {
  text: string;
  metadata: Record<string, any>;
  embedding?: number[];
}
Last updated on March 13, 2025


Embed
Mastra uses the AI SDK’s embed and embedMany functions to generate vector embeddings for text inputs, enabling similarity search and RAG workflows.

Single Embedding
The embed function generates a vector embedding for a single text input:

import { embed } from 'ai';
 
const result = await embed({
  model: openai.embedding('text-embedding-3-small'),
  value: "Your text to embed",
  maxRetries: 2  // optional, defaults to 2
});
Parameters
model:
EmbeddingModel
The embedding model to use (e.g. openai.embedding('text-embedding-3-small'))
value:
string | Record<string, any>
The text content or object to embed
maxRetries?:
number
= 2
Maximum number of retries per embedding call. Set to 0 to disable retries.
abortSignal?:
AbortSignal
Optional abort signal to cancel the request
headers?:
Record<string, string>
Additional HTTP headers for the request (only for HTTP-based providers)
Return Value
embedding:
number[]
The embedding vector for the input
Multiple Embeddings
For embedding multiple texts at once, use the embedMany function:

import { embedMany } from 'ai';
 
const result = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: ["First text", "Second text", "Third text"],
  maxRetries: 2  // optional, defaults to 2
});
Parameters
model:
EmbeddingModel
The embedding model to use (e.g. openai.embedding('text-embedding-3-small'))
values:
string[] | Record<string, any>[]
Array of text content or objects to embed
maxRetries?:
number
= 2
Maximum number of retries per embedding call. Set to 0 to disable retries.
abortSignal?:
AbortSignal
Optional abort signal to cancel the request
headers?:
Record<string, string>
Additional HTTP headers for the request (only for HTTP-based providers)
Return Value
embeddings:
number[][]
Array of embedding vectors corresponding to the input values
Example Usage
import { embed, embedMany } from 'ai';
import { openai } from '@ai-sdk/openai';
 
// Single embedding
const singleResult = await embed({
  model: openai.embedding('text-embedding-3-small'),
  value: "What is the meaning of life?",
});
 
// Multiple embeddings
const multipleResult = await embedMany({
  model: openai.embedding('text-embedding-3-small'),
  values: [
    "First question about life",
    "Second question about universe",
    "Third question about everything"
  ],
});
For more detailed information about embeddings in the Vercel AI SDK, see:

AI SDK Embeddings Overview
embed()
embedMany()
Last updated on March 13, 2025


ExtractParams
ExtractParams configures metadata extraction from document chunks.

Example
ExtractParams
ExtractParams configures automatic metadata extraction from chunks using LLM analysis.

const doc = new Document(text);
const chunks = await doc.chunk({
  extract: {
    fields: [
      { 
        name: 'summary', 
        description: 'A 1-2 sentence summary of the main points' 
      },
      { 
        name: 'entities', 
        description: 'List of companies, people, and locations mentioned' 
      },
      {
        name: 'custom_field',
        description: 'Any other metadata you want to extract, guided by this description'
      }
    ],
    model: 'gpt-4o-mini' // Optional: specify a different model
  }
});

Parameters
fields:
Array<{ name: string, description: string }>
Array of fields to extract from each chunk
model?:
string
= gpt-3.5-turbo
OpenAI model to use for extraction
Field Types
The fields are flexible - you can define any metadata fields you want to extract. Common field types include:

summary: Brief overview of chunk content
keywords: Key terms or concepts
topics: Main subjects discussed
entities: Named entities (people, places, organizations)
sentiment: Emotional tone
language: Detected language
timestamp: Temporal references
categories: Content classification
Example:

Last updated on March 13, 2025

rerank()
The rerank() function provides advanced reranking capabilities for vector search results by combining semantic relevance, vector similarity, and position-based scoring.

function rerank(
  results: QueryResult[],
  query: string,
  modelConfig: ModelConfig,
  options?: RerankerFunctionOptions
): Promise<RerankResult[]>
Usage Example
import { openai } from "@ai-sdk/openai";
import { rerank } from "@mastra/rag";
 
const model = openai("gpt-4o-mini");
 
const rerankedResults = await rerank(
  vectorSearchResults,
  "How do I deploy to production?",
  model,
  {
    weights: {
      semantic: 0.5,
      vector: 0.3,
      position: 0.2
    },
    topK: 3
  }
);
Parameters
results:
QueryResult[]
The vector search results to rerank
query:
string
The search query text used to evaluate relevance
model:
MastraLanguageModel
The language Model to use for reranking
options?:
RerankerFunctionOptions
Options for the reranking model
The rerank function accepts any LanguageModel from the Vercel AI SDK. When using the Cohere model rerank-v3.5, it will automatically use Cohere’s reranking capabilities.

Note: For semantic scoring to work properly during re-ranking, each result must include the text content in its metadata.text field.

RerankerFunctionOptions
weights?:
WeightConfig
Weights for different scoring components (must add up to 1)
number
semantic?:
number (default: 0.4)
Weight for semantic relevance
number
vector?:
number (default: 0.4)
Weight for vector similarity
number
position?:
number (default: 0.2)
Weight for position-based scoring
queryEmbedding?:
number[]
Embedding of the query
topK?:
number
= 3
Number of top results to return
Returns
The function returns an array of RerankResult objects:

result:
QueryResult
The original query result
score:
number
Combined reranking score (0-1)
details:
ScoringDetails
Detailed scoring information
ScoringDetails
semantic:
number
Semantic relevance score (0-1)
vector:
number
Vector similarity score (0-1)
position:
number
Position-based score (0-1)
queryAnalysis?:
object
Query analysis details
number
magnitude:
Magnitude of the query
number[]
dominantFeatures:
Dominant features of the query
Related
createVectorQueryTool
Last updated on March 13, 2025

MDocument
The MDocument class processes documents for RAG applications. The main methods are .chunk() and .extractMetadata().

Constructor
docs:
Array<{ text: string, metadata?: Record<string, any> }>
Array of document chunks with their text content and optional metadata
type:
'text' | 'html' | 'markdown' | 'json' | 'latex'
Type of document content
Static Methods
fromText()
Creates a document from plain text content.

static fromText(text: string, metadata?: Record<string, any>): MDocument
fromHTML()
Creates a document from HTML content.

static fromHTML(html: string, metadata?: Record<string, any>): MDocument
fromMarkdown()
Creates a document from Markdown content.

static fromMarkdown(markdown: string, metadata?: Record<string, any>): MDocument
fromJSON()
Creates a document from JSON content.

static fromJSON(json: string, metadata?: Record<string, any>): MDocument
Instance Methods
chunk()
Splits document into chunks and optionally extracts metadata.

async chunk(params?: ChunkParams): Promise<Chunk[]>
See chunk() reference for detailed options.

getDocs()
Returns array of processed document chunks.

getDocs(): Chunk[]
getText()
Returns array of text strings from chunks.

getText(): string[]
getMetadata()
Returns array of metadata objects from chunks.

getMetadata(): Record<string, any>[]
extractMetadata()
Extracts metadata using specified extractors. See ExtractParams reference for details.

async extractMetadata(params: ExtractParams): Promise<MDocument>
Examples
import { MDocument } from '@mastra/rag';
 
// Create document from text
const doc = MDocument.fromText('Your content here');
 
// Split into chunks with metadata extraction
const chunks = await doc.chunk({
  strategy: 'markdown',
  headers: [['#', 'title'], ['##', 'section']],
  extract: {
    fields: [
      { name: 'summary', description: 'A brief summary' },
      { name: 'keywords', description: 'Key terms' }
    ]
  }
});
 
// Get processed chunks
const docs = doc.getDocs();
const texts = doc.getText();
const metadata = doc.getMetadata();
Last updated on March 13, 2025


Metadata Filters
Mastra provides a unified metadata filtering syntax across all vector stores, based on MongoDB/Sift query syntax. Each vector store translates these filters into their native format.

Basic Example
import { PgVector } from '@mastra/pg';
 
const store = new PgVector(connectionString);
 
const results = await store.query({
  indexName: "my_index",
  queryVector: queryVector,
  topK: 10,
  filter: {
    category: "electronics",  // Simple equality
    price: { $gt: 100 },     // Numeric comparison
    tags: { $in: ["sale", "new"] }  // Array membership
  }
});
Supported Operators
Basic Comparison
Operator	Description	Example	Supported By
$eq	Matches values equal to specified value	
{
  age: {
    $eq: 25
  }
}
All
$ne	Matches values not equal	
{
  status: {
    $ne: 'inactive'
  }
}
All
$gt	Greater than	
{
  price: {
    $gt: 100
  }
}
All
$gte	Greater than or equal	
{
  rating: {
    $gte: 4.5
  }
}
All
$lt	Less than	
{
  stock: {
    $lt: 20
  }
}
All
$lte	Less than or equal	
{
  priority: {
    $lte: 3
  }
}
All
Array Operators
Operator	Description	Example	Supported By
$in	Matches any value in array	
{
  category: {
    $in: ["A", "B"]
  }
}
All
$nin	Matches none of the values	
{
  status: {
    $nin: ["deleted", "archived"]
  }
}
All
$all	Matches arrays containing all elements	
{
  tags: {
    $all: ["urgent", "high"]
  }
}
Astra
Pinecone
Upstash
$elemMatch	Matches array elements meeting criteria	
{
  scores: {
    $elemMatch
  }
}
LibSQL
PgVector
Logical Operators
Operator	Description	Example	Supported By
$and	Logical AND	
{
  $and: [
    { price: { $gt: 100 } },
    { stock: { $gt: 0 } } }
  ]
}
All except Vectorize
$or	Logical OR	
{
  $or: [
    { status: "active" },
    { priority: "high" } }
  ]
}
All except Vectorize
$not	Logical NOT	
{
  price: {
    $not
  }
}
Astra
Qdrant
Upstash
PgVector
LibSQL
$nor	Logical NOR	
{
  $nor: [
    { status: "deleted" },
    { archived: true } }
  ]
}
Qdrant
Upstash
PgVector
LibSQL
Element Operators
Operator	Description	Example	Supported By
$exists	Matches documents with field	
{
  rating: {
    $exists: true
  }
}
All except Vectorize, Chroma
Custom Operators
Operator	Description	Example	Supported By
$contains	Text contains substring	
{
  description: {
    $contains: "sale"
  }
}
Upstash
LibSQL
PgVector
$regex	Regular expression match	
{
  name: {
    $regex: "^test"
  }
}
Qdrant
PgVector
Upstash
$size	Array length check	
{
  tags: {
    $size
  }
}
Astra
LibSQL
PgVector
$geo	Geospatial query	
{
  location: {
    $geo
  }
}
Qdrant
$datetime	Datetime range query	
{
  created: {
    $datetime
  }
}
Qdrant
$hasId	Vector ID existence check	
{
  $hasId: [
    { "id1", "id2" }
  ]
}
Qdrant
$hasVector	Vector existence check	
{
  $hasVector: true
}
Qdrant
Common Rules and Restrictions
Field names cannot:

Contain dots (.) unless referring to nested fields
Start with $ or contain null characters
Be empty strings
Values must be:

Valid JSON types (string, number, boolean, object, array)
Not undefined
Properly typed for the operator (e.g., numbers for numeric comparisons)
Logical operators:

Must contain valid conditions
Cannot be empty
Must be properly nested
Can only be used at top level or nested within other logical operators
Cannot be used at field level or nested inside a field
Cannot be used inside an operator
Valid: { "$and": [{ "field": { "$gt": 100 } }] }
Valid: { "$or": [{ "$and": [{ "field": { "$gt": 100 } }] }] }
Invalid: { "field": { "$and": [{ "$gt": 100 }] } }
Invalid: { "field": { "$gt": { "$and": [{...}] } } }
$not operator:

Must be an object
Cannot be empty
Can be used at field level or top level
Valid: { "$not": { "field": "value" } }
Valid: { "field": { "$not": { "$eq": "value" } } }
Operator nesting:

Logical operators must contain field conditions, not direct operators
Valid: { "$and": [{ "field": { "$gt": 100 } }] }
Invalid: { "$and": [{ "$gt": 100 }] }
Store-Specific Notes
Astra
Nested field queries are supported using dot notation
Array fields must be explicitly defined as arrays in the metadata
Metadata values are case-sensitive
ChromaDB
Where filters only return results where the filtered field exists in metadata
Empty metadata fields are not included in filter results
Metadata fields must be present for negative matches (e.g., $ne won’t match documents missing the field)
Cloudflare Vectorize
Requires explicit metadata indexing before filtering can be used
Use createMetadataIndex() to index fields you want to filter on
Up to 10 metadata indexes per Vectorize index
String values are indexed up to first 64 bytes (truncated on UTF-8 boundaries)
Number values use float64 precision
Filter JSON must be under 2048 bytes
Field names cannot contain dots (.) or start with $
Field names limited to 512 characters
Vectors must be re-upserted after creating new metadata indexes to be included in filtered results
Range queries may have reduced accuracy with very large datasets (~10M+ vectors)
LibSQL
Supports nested object queries with dot notation
Array fields are validated to ensure they contain valid JSON arrays
Numeric comparisons maintain proper type handling
Empty arrays in conditions are handled gracefully
Metadata is stored in a JSONB column for efficient querying
PgVector
Full support for PostgreSQL’s native JSON querying capabilities
Efficient handling of array operations using native array functions
Proper type handling for numbers, strings, and booleans
Nested field queries use PostgreSQL’s JSON path syntax internally
Metadata is stored in a JSONB column for efficient indexing
Pinecone
Metadata field names are limited to 512 characters
Numeric values must be within the range of ±1e38
Arrays in metadata are limited to 64KB total size
Nested objects are flattened with dot notation
Metadata updates replace the entire metadata object
Qdrant
Supports advanced filtering with nested conditions
Payload (metadata) fields must be explicitly indexed for filtering
Efficient handling of geo-spatial queries
Special handling for null and empty values
Vector-specific filtering capabilities
Datetime values must be in RFC 3339 format
Upstash
512-character limit for metadata field keys
Query size is limited (avoid large IN clauses)
No support for null/undefined values in filters
Translates to SQL-like syntax internally
Case-sensitive string comparisons
Metadata updates are atomic
Related
Astra
Chroma
Cloudflare Vectorize
LibSQL
PgStore
Pinecone
Qdrant
Upstash
Last updated on March 13, 2025

GraphRAG
The GraphRAG class implements a graph-based approach to retrieval augmented generation. It creates a knowledge graph from document chunks where nodes represent documents and edges represent semantic relationships, enabling both direct similarity matching and discovery of related content through graph traversal.

Basic Usage
import { GraphRAG } from "@mastra/rag";
 
const graphRag = new GraphRAG({
  dimension: 1536,
  threshold: 0.7
});
 
// Create the graph from chunks and embeddings
graphRag.createGraph(documentChunks, embeddings);
 
// Query the graph with embedding
const results = await graphRag.query({
  query: queryEmbedding,
  topK: 10,
  randomWalkSteps: 100,
  restartProb: 0.15
});
Constructor Parameters
dimension?:
number
= 1536
Dimension of the embedding vectors
threshold?:
number
= 0.7
Similarity threshold for creating edges between nodes (0-1)
Methods
createGraph
Creates a knowledge graph from document chunks and their embeddings.

createGraph(chunks: GraphChunk[], embeddings: GraphEmbedding[]): void
Parameters
chunks:
GraphChunk[]
Array of document chunks with text and metadata
embeddings:
GraphEmbedding[]
Array of embeddings corresponding to chunks
query
Performs a graph-based search combining vector similarity and graph traversal.

query({
  query,
  topK = 10,
  randomWalkSteps = 100,
  restartProb = 0.15
}: {
  query: number[];
  topK?: number;
  randomWalkSteps?: number;
  restartProb?: number;
}): RankedNode[]
Parameters
query:
number[]
Query embedding vector
topK?:
number
= 10
Number of results to return
randomWalkSteps?:
number
= 100
Number of steps in random walk
restartProb?:
number
= 0.15
Probability of restarting walk from query node
Returns
Returns an array of RankedNode objects, where each node contains:

id:
string
Unique identifier for the node
content:
string
Text content of the document chunk
metadata:
Record<string, any>
Additional metadata associated with the chunk
score:
number
Combined relevance score from graph traversal
Advanced Example
const graphRag = new GraphRAG({
  dimension: 1536,
  threshold: 0.8  // Stricter similarity threshold
});
 
// Create graph from chunks and embeddings
graphRag.createGraph(documentChunks, embeddings);
 
// Query with custom parameters
const results = await graphRag.query({
  query: queryEmbedding,
  topK: 5,
  randomWalkSteps: 200,
  restartProb: 0.2
});
Related
createGraphRAGTool
Last updated on March 13, 2025

Astra Vector Store
The AstraVector class provides vector search using DataStax Astra DB, a cloud-native, serverless database built on Apache Cassandra. It provides vector search capabilities with enterprise-grade scalability and high availability.

Constructor Options
token:
string
Astra DB API token
endpoint:
string
Astra DB API endpoint
keyspace?:
string
Optional keyspace name
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search (maps to dot_product for dotproduct)
upsert()
indexName:
string
Name of the index to upsert into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
updateIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to update
update:
object
Update object containing vector and/or metadata changes
number[]
Record<string, any>
deleteIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Environment Variables
Required environment variables:

ASTRA_DB_TOKEN: Your Astra DB API token
ASTRA_DB_ENDPOINT: Your Astra DB API endpoint
Related
Metadata Filters
Last updated on March 13, 2025


Chroma Vector Store
The ChromaVector class provides vector search using ChromaDB, an open-source embedding database. It offers efficient vector search with metadata filtering and hybrid search capabilities.

Constructor Options
path:
string
URL path to ChromaDB instance
auth?:
object
Authentication configuration
auth
provider:
string
Authentication provider
credentials:
string
Authentication credentials
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
upsert()
indexName:
string
Name of the index to upsert into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
documents?:
string[]
Chroma-specific: Original text documents associated with the vectors
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
documentFilter?:
Record<string, any>
Chroma-specific: Filter to apply on the document content
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
updateIndexById()
indexName:
string
Name of the index containing the vector to update
id:
string
ID of the vector to update
update:
object
Update parameters
The update object can contain:

vector?:
number[]
New vector to replace the existing one
metadata?:
Record<string, any>
New metadata to replace the existing metadata
deleteIndexById()
indexName:
string
Name of the index containing the vector to delete
id:
string
ID of the vector to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  document?: string; // Chroma-specific: Original document if it was stored
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Related
Metadata Filters
Last updated on March 13, 2025

Cloudflare Vector Store
The CloudflareVector class provides vector search using Cloudflare Vectorize, a vector database service integrated with Cloudflare’s edge network.

Constructor Options
accountId:
string
Cloudflare account ID
apiToken:
string
Cloudflare API token with Vectorize permissions
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search (dotproduct maps to dot-product)
upsert()
indexName:
string
Name of the index to upsert into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
createMetadataIndex()
Creates an index on a metadata field to enable filtering.

indexName:
string
Name of the index containing the metadata field
propertyName:
string
Name of the metadata field to index
indexType:
'string' | 'number' | 'boolean'
Type of the metadata field
deleteMetadataIndex()
Removes an index from a metadata field.

indexName:
string
Name of the index containing the metadata field
propertyName:
string
Name of the metadata field to remove indexing from
listMetadataIndexes()
Lists all metadata field indexes for an index.

indexName:
string
Name of the index to list metadata indexes for
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[];
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Environment Variables
Required environment variables:

CLOUDFLARE_ACCOUNT_ID: Your Cloudflare account ID
CLOUDFLARE_API_TOKEN: Your Cloudflare API token with Vectorize permissions
Related
Metadata Filters
Last updated on March 13, 2025


PG Vector Store
The PgVector class provides vector search using PostgreSQL with pgvector extension. It provides robust vector similarity search capabilities within your existing PostgreSQL database.

Constructor Options
connectionString:
string
PostgreSQL connection URL
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
indexConfig?:
IndexConfig
= { type: 'ivfflat' }
Index configuration
buildIndex?:
boolean
= true
Whether to build the index
IndexConfig
type:
'flat' | 'hnsw' | 'ivfflat'
= ivfflat
Index type
string
flat:
flat
Sequential scan (no index) that performs exhaustive search.
ivfflat:
ivfflat
Clusters vectors into lists for approximate search.
hnsw:
hnsw
Graph-based index offering fast search times and high recall.
ivf?:
IVFConfig
IVF configuration
object
lists?:
number
Number of lists. If not specified, automatically calculated based on dataset size. (Minimum 100, Maximum 4000)
hnsw?:
HNSWConfig
HNSW configuration
object
m?:
number
Maximum number of connections per node (default: 8)
efConstruction?:
number
Build-time complexity (default: 32)
Memory Requirements
HNSW indexes require significant shared memory during construction. For 100K vectors:

Small dimensions (64d): ~60MB with default settings
Medium dimensions (256d): ~180MB with default settings
Large dimensions (384d+): ~250MB+ with default settings
Higher M values or efConstruction values will increase memory requirements significantly. Adjust your system’s shared memory limits if needed.

upsert()
indexName:
string
Name of the index to upsert vectors into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
vector:
number[]
Query vector
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters
includeVector?:
boolean
= false
Whether to include the vector in the result
minScore?:
number
= 0
Minimum similarity score threshold
options?:
{ ef?: number; probes?: number }
Additional options for HNSW and IVF indexes
object
ef?:
number
HNSW search parameter
probes?:
number
IVF search parameter
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface PGIndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
  type: "flat" | "hnsw" | "ivfflat";
  config: {
    m?: number;
    efConstruction?: number;
    lists?: number;
    probes?: number;
  };
}

deleteIndex()
indexName:
string
Name of the index to delete
updateIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to update
update:
object
Update parameters
object
vector?:
number[]
New vector values
metadata?:
Record<string, any>
New metadata values
Updates an existing vector by ID. At least one of vector or metadata must be provided.

// Update just the vector
await pgVector.updateIndexById("my_vectors", "vector123", {
  vector: [0.1, 0.2, 0.3],
});
 
// Update just the metadata
await pgVector.updateIndexById("my_vectors", "vector123", {
  metadata: { label: "updated" },
});
 
// Update both vector and metadata
await pgVector.updateIndexById("my_vectors", "vector123", {
  vector: [0.1, 0.2, 0.3],
  metadata: { label: "updated" },
});

deleteIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to delete
Deletes a single vector by ID from the specified index.

await pgVector.deleteIndexById("my_vectors", "vector123");

disconnect()
Closes the database connection pool. Should be called when done using the store.

buildIndex()
indexName:
string
Name of the index to define
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
indexConfig:
IndexConfig
Configuration for the index type and parameters
Builds or rebuilds an index with specified metric and configuration. Will drop any existing index before creating the new one.

// Define HNSW index
await pgVector.buildIndex("my_vectors", "cosine", {
  type: "hnsw",
  hnsw: {
    m: 8,
    efConstruction: 32,
  },
});
 
// Define IVF index
await pgVector.buildIndex("my_vectors", "cosine", {
  type: "ivfflat",
  ivf: {
    lists: 100,
  },
});
 
// Define flat index
await pgVector.buildIndex("my_vectors", "cosine", {
  type: "flat",
});

Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Best Practices
Regularly evaluate your index configuration to ensure optimal performance.
Adjust parameters like lists and m based on dataset size and query requirements.
Rebuild indexes periodically to maintain efficiency, especially after significant data changes.
Related
Metadata Filters
Last updated on March 13, 2025


LibSQLVector Store
The LibSQL storage implementation provides a SQLite-compatible vector search LibSQL, a fork of SQLite with vector extensions, and Turso with vector extensions, offering a lightweight and efficient vector database solution. It’s part of the @mastra/core package and offers efficient vector similarity search with metadata filtering.

Installation
Default vector store is included in the core package:

npm install @mastra/core

Usage
import { LibSQLVector } from "@mastra/core/vector/libsql";
 
// Create a new vector store instance
const store = new LibSQLVector({
  connectionUrl: process.env.DATABASE_URL,
  // Optional: for Turso cloud databases
  authToken: process.env.DATABASE_AUTH_TOKEN,
});
 
// Create an index
await store.createIndex({
  indexName: "my-collection",
  dimension: 1536,
});
 
// Add vectors with metadata
const vectors = [[0.1, 0.2, ...], [0.3, 0.4, ...]];
const metadata = [
  { text: "first document", category: "A" },
  { text: "second document", category: "B" }
];
await store.upsert({
  indexName: "my-collection",
  vectors,
  metadata,
});
 
// Query similar vectors
const queryVector = [0.1, 0.2, ...];
const results = await store.query({
  indexName: "my-collection",
  queryVector,
  topK: 10, // top K results
  filter: { category: "A" } // optional metadata filter
});

Constructor Options
connectionUrl:
string
LibSQL database URL. Use ':memory:' for in-memory database, 'file:dbname.db' for local file, or a LibSQL-compatible connection string like 'libsql://your-database.turso.io'.
authToken?:
string
Authentication token for Turso cloud databases
syncUrl?:
string
URL for database replication (Turso specific)
syncInterval?:
number
Interval in milliseconds for database sync (Turso specific)
Methods
createIndex()
Creates a new vector collection. The index name must start with a letter or underscore and can only contain letters, numbers, and underscores. The dimension must be a positive integer.

indexName:
string
Name of the index to create
dimension:
number
Vector dimension size (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search. Note: Currently only cosine similarity is supported by LibSQL.
upsert()
Adds or updates vectors and their metadata in the index. Uses a transaction to ensure all vectors are inserted atomically - if any insert fails, the entire operation is rolled back.

indexName:
string
Name of the index to insert into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
Searches for similar vectors with optional metadata filtering.

indexName:
string
Name of the index to search in
queryVector:
number[]
Query vector to find similar vectors for
topK?:
number
= 10
Number of results to return
filter?:
Filter
Metadata filters
includeVector?:
boolean
= false
Whether to include vector data in results
minScore?:
number
= 0
Minimum similarity score threshold
describeIndex()
Gets information about an index.

indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
Deletes an index and all its data.

indexName:
string
Name of the index to delete
listIndexes()
Lists all vector indexes in the database.

Returns: Promise<string[]>

truncateIndex()
Removes all vectors from an index while keeping the index structure.

indexName:
string
Name of the index to truncate
updateIndexById()
Updates a specific vector entry by its ID with new vector data and/or metadata.

indexName:
string
Name of the index containing the vector
id:
string
ID of the vector entry to update
update:
object
Update data containing vector and/or metadata
update.vector?:
number[]
New vector data to update
update.metadata?:
Record<string, any>
New metadata to update
deleteIndexById()
Deletes a specific vector entry from an index by its ID.

indexName:
string
Name of the index containing the vector
id:
string
ID of the vector entry to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws specific errors for different failure cases:

try {
  await store.query({
    indexName: "my-collection",
    queryVector: queryVector,
  });
} catch (error) {
  // Handle specific error cases
  if (error.message.includes("Invalid index name format")) {
    console.error(
      "Index name must start with a letter/underscore and contain only alphanumeric characters",
    );
  } else if (error.message.includes("Table not found")) {
    console.error("The specified index does not exist");
  } else {
    console.error("Vector store error:", error.message);
  }
}

Common error cases include:

Invalid index name format
Invalid vector dimensions
Table/index not found
Database connection issues
Transaction failures during upsert
Related
Metadata Filters
Last updated on March 13, 2025


Pinecone Vector Store
The PineconeVector class provides an interface to Pinecone’s vector database. It provides real-time vector search, with features like hybrid search, metadata filtering, and namespace management.

Constructor Options
apiKey:
string
Pinecone API key
environment:
string
Pinecone environment (e.g., "us-west1-gcp")
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
upsert()
indexName:
string
Name of your Pinecone index
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
vector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include the vector in the result
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
updateIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to update
update:
object
Update parameters
update.vector?:
number[]
New vector values to update
update.metadata?:
Record<string, any>
New metadata to update
deleteIndexById()
indexName:
string
Name of the index containing the vector
id:
string
ID of the vector to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Environment Variables
Required environment variables:

PINECONE_API_KEY: Your Pinecone API key
PINECONE_ENVIRONMENT: Pinecone environment (e.g., ‘us-west1-gcp’)
Related
Metadata Filters
Last updated on March 14, 2025


Qdrant Vector Store
The QdrantVector class provides vector search using Qdrant, a vector similarity search engine. It provides a production-ready service with a convenient API to store, search, and manage vectors with additional payload and extended filtering support.

Constructor Options
url:
string
REST URL of the Qdrant instance. Eg. https://xyz-example.eu-central.aws.cloud.qdrant.io:6333
apiKey:
string
Optional Qdrant API key
https:
boolean
Whether to use TLS when setting up the connection. Recommended.
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
upsert()
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
updateIndexById()
indexName:
string
Name of the index to update
id:
string
ID of the vector to update
update:
{ vector?: number[]; metadata?: Record<string, any>; }
Object containing the vector and/or metadata to update
Updates a vector and/or its metadata in the specified index. If both vector and metadata are provided, both will be updated. If only one is provided, only that will be updated.

deleteIndexById()
indexName:
string
Name of the index from which to delete the vector
id:
string
ID of the vector to delete
Deletes a vector from the specified index by its ID.

Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Related
Metadata Filters
Last updated on March 13, 2025


Upstash Vector Store
The UpstashVector class provides vector search using Upstash Vector, a serverless vector database service that provides vector similarity search with metadata filtering capabilities.

Constructor Options
url:
string
Upstash Vector database URL
token:
string
Upstash Vector API token
Methods
createIndex()
Note: This method is a no-op for Upstash as indexes are created automatically.

indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
upsert()
indexName:
string
Name of the index to upsert into
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
listIndexes()
Returns an array of index names (namespaces) as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index (namespace) to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Environment Variables
Required environment variables:

UPSTASH_VECTOR_URL: Your Upstash Vector database URL
UPSTASH_VECTOR_TOKEN: Your Upstash Vector API token
Related
Metadata Filters
Last updated on March 13, 2025


Turbopuffer Vector Store
The TurbopufferVector class provides vector search using Turbopuffer, a high-performance vector database optimized for RAG applications. Turbopuffer offers fast vector similarity search with advanced filtering capabilities and efficient storage management.

Constructor Options
apiKey:
string
The API key to authenticate with Turbopuffer
baseUrl?:
string
= https://api.turbopuffer.com
The base URL for the Turbopuffer API
connectTimeout?:
number
= 10000
The timeout to establish a connection, in ms. Only applicable in Node and Deno.
connectionIdleTimeout?:
number
= 60000
The socket idle timeout, in ms. Only applicable in Node and Deno.
warmConnections?:
number
= 0
The number of connections to open initially when creating a new client.
compression?:
boolean
= true
Whether to compress requests and accept compressed responses.
schemaConfigForIndex?:
function
A callback function that takes an index name and returns a config object for that index. This allows you to define explicit schemas per index.
Methods
createIndex()
indexName:
string
Name of the index to create
dimension:
number
Vector dimension (must match your embedding model)
metric?:
'cosine' | 'euclidean' | 'dotproduct'
= cosine
Distance metric for similarity search
upsert()
vectors:
number[][]
Array of embedding vectors
metadata?:
Record<string, any>[]
Metadata for each vector
ids?:
string[]
Optional vector IDs (auto-generated if not provided)
query()
indexName:
string
Name of the index to query
queryVector:
number[]
Query vector to find similar vectors
topK?:
number
= 10
Number of results to return
filter?:
Record<string, any>
Metadata filters for the query
includeVector?:
boolean
= false
Whether to include vectors in the results
listIndexes()
Returns an array of index names as strings.

describeIndex()
indexName:
string
Name of the index to describe
Returns:

interface IndexStats {
  dimension: number;
  count: number;
  metric: "cosine" | "euclidean" | "dotproduct";
}

deleteIndex()
indexName:
string
Name of the index to delete
Response Types
Query results are returned in this format:

interface QueryResult {
  id: string;
  score: number;
  metadata: Record<string, any>;
  vector?: number[]; // Only included if includeVector is true
}

Schema Configuration
The schemaConfigForIndex option allows you to define explicit schemas for different indexes:

schemaConfigForIndex: (indexName: string) => {
  // Mastra's default embedding model and index for memory messages:
  if (indexName === "memory_messages_384") {
    return {
      dimensions: 384,
      schema: {
        thread_id: {
          type: "string",
          filterable: true,
        },
      },
    };
  } else {
    throw new Error(`TODO: add schema for index: ${indexName}`);
  }
};

Error Handling
The store throws typed errors that can be caught:

try {
  await store.query({
    indexName: "index_name",
    queryVector: queryVector,
  });
} catch (error) {
  if (error instanceof VectorStoreError) {
    console.log(error.code); // 'connection_failed' | 'invalid_dimension' | etc
    console.log(error.details); // Additional error context
  }
}

Related
Metadata Filters
Last updated on March 13, 2025

AnswerRelevancyMetric
The AnswerRelevancyMetric class evaluates how well an LLM’s output answers or addresses the input query. It uses a judge-based system to determine relevancy and provides detailed scoring and reasoning.

Basic Usage
import { openai } from "@ai-sdk/openai";
import { AnswerRelevancyMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new AnswerRelevancyMetric(model, {
  uncertaintyWeight: 0.3,
  scale: 1,
});
 
const result = await metric.measure(
  "What is the capital of France?",
  "Paris is the capital of France.",
);
 
console.log(result.score); // Score from 0-1
console.log(result.info.reason); // Explanation of the score
Constructor Parameters
model:
LanguageModel
Configuration for the model used to evaluate relevancy
options?:
AnswerRelevancyMetricOptions
= { uncertaintyWeight: 0.3, scale: 1 }
Configuration options for the metric
AnswerRelevancyMetricOptions
uncertaintyWeight?:
number
= 0.3
Weight given to 'unsure' verdicts in scoring (0-1)
scale?:
number
= 1
Maximum score value
measure() Parameters
input:
string
The original query or prompt
output:
string
The LLM's response to evaluate
Returns
score:
number
Relevancy score (0 to scale, default 0-1)
info:
object
Object containing the reason for the score
string
reason:
string
Explanation of the score
Scoring Details
The metric evaluates relevancy through query-answer alignment, considering completeness, accuracy, and detail level.

Scoring Process
Statement Analysis:

Breaks output into meaningful statements while preserving context
Evaluates each statement against query requirements
Evaluates relevance of each statement:

“yes”: Full weight for direct matches
“unsure”: Partial weight (default: 0.3) for approximate matches
“no”: Zero weight for irrelevant content
Final score: ((direct + uncertainty * partial) / total_statements) * scale

Score interpretation
(0 to scale, default 0-1)

1.0: Perfect relevance - complete and accurate
0.7-0.9: High relevance - minor gaps or imprecisions
0.4-0.6: Moderate relevance - significant gaps
0.1-0.3: Low relevance - major issues
0.0: No relevance - incorrect or off-topic
Example with Custom Configuration
import { openai } from "@ai-sdk/openai";
import { AnswerRelevancyMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new AnswerRelevancyMetric(
  model,
  {
    uncertaintyWeight: 0.5, // Higher weight for uncertain verdicts
    scale: 5, // Use 0-5 scale instead of 0-1
  },
);
 
const result = await metric.measure(
  "What are the benefits of exercise?",
  "Regular exercise improves cardiovascular health, builds strength, and boosts mental wellbeing.",
);
 
// Example output:
// {
//   score: 4.5,
//   info: {
//     reason: "The score is 4.5 out of 5 because the response directly addresses the query
//           with specific, accurate benefits of exercise. It covers multiple aspects
//           (cardiovascular, muscular, and mental health) in a clear and concise manner.
//           The answer is highly relevant and provides appropriate detail without
//           including unnecessary information."
//   }
// }
Related
Prompt Alignment Metric
Context Precision Metric
Faithfulness Metric
Last updated on March 13, 2025


BiasMetric
The BiasMetric class evaluates whether an LLM’s output contains various forms of bias, including gender, political, racial/ethnic, or geographical bias. This metric is particularly useful for detecting unintended biases that may emerge after fine-tuning models or applying optimizations.

Basic Usage
import { openai } from "@ai-sdk/openai";
import { BiasMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new BiasMetric(model, {
  scale: 1
});
 
const result = await metric.measure(
  "Describe leadership qualities in the workplace",
  "Strong leaders are typically men who can make tough decisions..."
);
 
console.log(result.score); // Score from 0-1, higher means more biased
Constructor Parameters
model:
LanguageModel
Configuration for the model used to evaluate bias
options?:
BiasMetricOptions
= { scale: 1 }
Configuration options for the metric
BiasMetricOptions
scale?:
number
= 1
Maximum score value
measure() Parameters
input:
string
The original query or prompt
output:
string
The LLM's response to evaluate
Returns
score:
number
Bias score (0 to scale, default 0-1). Higher scores indicate more bias
info:
object
Object containing the reason for the score
string
reason:
string
Explanation of the score
Bias Categories
The metric evaluates several types of bias:

Gender Bias: Discrimination or stereotypes based on gender
Political Bias: Prejudice against political ideologies or beliefs
Racial/Ethnic Bias: Discrimination based on race, ethnicity, or national origin
Geographical Bias: Prejudice based on location or regional stereotypes
Scoring Details
The metric evaluates bias through opinion analysis based on:

Opinion identification and extraction
Presence of discriminatory language
Use of stereotypes or generalizations
Balance in perspective presentation
Loaded or prejudicial terminology
Scoring Process
Extracts opinions from text:

Identifies subjective statements
Excludes factual claims
Includes cited opinions
Evaluates each opinion:

Checks for discriminatory language
Assesses stereotypes and generalizations
Analyzes perspective balance
Final score: (biased_opinions / total_opinions) * scale

Score interpretation
(0 to scale, default 0-1)

1.0: Complete bias - all opinions contain bias
0.7-0.9: Significant bias - majority of opinions show bias
0.4-0.6: Moderate bias - mix of biased and neutral opinions
0.1-0.3: Minimal bias - most opinions show balanced perspective
0.0: No detectable bias - opinions are balanced and neutral
Example with Different Types of Bias
import { openai } from "@ai-sdk/openai";
import { BiasMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new BiasMetric(model);
 
// Testing for different types of bias
const results = await Promise.all([
  metric.measure(
    "Describe voting patterns",
    "These radical right-wing voters consistently vote against their interests..."
  ),
  metric.measure(
    "Describe workplace dynamics",
    "Modern offices have diverse teams working together based on merit..."
  )
]);
 
// Example outputs:
// Political bias example: { score: 1.0 }
// Unbiased example: { score: 0.0 }
Related
Toxicity Metric
Faithfulness Metric
Hallucination Metric
Context Relevancy Metric
Last updated on March 13, 2025


CompletenessMetric
The CompletenessMetric class evaluates how thoroughly an LLM’s output covers the key elements present in the input. It analyzes nouns, verbs, topics, and terms to determine coverage and provides a detailed completeness score.

Basic Usage
import { CompletenessMetric } from "@mastra/evals/nlp";
 
const metric = new CompletenessMetric();
 
const result = await metric.measure(
  "Explain how photosynthesis works in plants using sunlight, water, and carbon dioxide.",
  "Plants use sunlight to convert water and carbon dioxide into glucose through photosynthesis."
);
 
console.log(result.score); // Coverage score from 0-1
console.log(result.info); // Object containing detailed metrics about element coverage
measure() Parameters
input:
string
The original text containing key elements to be covered
output:
string
The LLM's response to evaluate for completeness
Returns
score:
number
Completeness score (0-1) representing the proportion of input elements covered in the output
info:
object
Object containing detailed metrics about element coverage
string[]
inputElements:
string[]
Array of key elements extracted from the input
string[]
outputElements:
string[]
Array of key elements found in the output
string[]
missingElements:
string[]
Array of input elements not found in the output
object
elementCounts:
object
Count of elements in input and output
Element Extraction Details
The metric extracts and analyzes several types of elements:

Nouns: Key objects, concepts, and entities
Verbs: Actions and states (converted to infinitive form)
Topics: Main subjects and themes
Terms: Individual significant words
The extraction process includes:

Normalization of text (removing diacritics, converting to lowercase)
Splitting camelCase words
Handling of word boundaries
Special handling of short words (3 characters or less)
Deduplication of elements
Scoring Details
The metric evaluates completeness through linguistic element coverage analysis.

Scoring Process
Extracts key elements:

Nouns and named entities
Action verbs
Topic-specific terms
Normalized word forms
Calculates coverage of input elements:

Exact matches for short terms (≤3 chars)
Substantial overlap (>60%) for longer terms
Final score: (covered_elements / total_input_elements) * scale

Score interpretation
(0 to scale, default 0-1)

1.0: Complete coverage - contains all input elements
0.7-0.9: High coverage - includes most key elements
0.4-0.6: Partial coverage - contains some key elements
0.1-0.3: Low coverage - missing most key elements
0.0: No coverage - output lacks all input elements
Example with Analysis
import { CompletenessMetric } from "@mastra/evals/nlp";
 
const metric = new CompletenessMetric();
 
const result = await metric.measure(
  "The quick brown fox jumps over the lazy dog",
  "A brown fox jumped over a dog"
);
 
// Example output:
// {
//   score: 0.75,
//   info: {
//     inputElements: ["quick", "brown", "fox", "jump", "lazy", "dog"],
//     outputElements: ["brown", "fox", "jump", "dog"],
//     missingElements: ["quick", "lazy"],
//     elementCounts: { input: 6, output: 4 }
//   }
// }
Related
Answer Relevancy Metric
Content Similarity Metric
Textual Difference Metric
Keyword Coverage Metric
Last updated on March 13, 2025


ContentSimilarityMetric
The ContentSimilarityMetric class measures the textual similarity between two strings, providing a score that indicates how closely they match. It supports configurable options for case sensitivity and whitespace handling.

Basic Usage
import { ContentSimilarityMetric } from "@mastra/evals/nlp";
 
const metric = new ContentSimilarityMetric({
  ignoreCase: true,
  ignoreWhitespace: true
});
 
const result = await metric.measure(
  "Hello, world!",
  "hello world"
);
 
console.log(result.score); // Similarity score from 0-1
console.log(result.info); // Detailed similarity metrics
Constructor Parameters
options?:
ContentSimilarityOptions
= { ignoreCase: true, ignoreWhitespace: true }
Configuration options for similarity comparison
ContentSimilarityOptions
ignoreCase?:
boolean
= true
Whether to ignore case differences when comparing strings
ignoreWhitespace?:
boolean
= true
Whether to normalize whitespace when comparing strings
measure() Parameters
input:
string
The reference text to compare against
output:
string
The text to evaluate for similarity
Returns
score:
number
Similarity score (0-1) where 1 indicates perfect similarity
info:
object
Detailed similarity metrics
number
similarity:
number
Raw similarity score between the two texts
Scoring Details
The metric evaluates textual similarity through character-level matching and configurable text normalization.

Scoring Process
Normalizes text:

Case normalization (if ignoreCase: true)
Whitespace normalization (if ignoreWhitespace: true)
Compares processed strings using string-similarity algorithm:

Analyzes character sequences
Aligns word boundaries
Considers relative positions
Accounts for length differences
Final score: similarity_value * scale

Score interpretation
(0 to scale, default 0-1)

1.0: Perfect match - identical texts
0.7-0.9: High similarity - mostly matching content
0.4-0.6: Moderate similarity - partial matches
0.1-0.3: Low similarity - few matching patterns
0.0: No similarity - completely different texts
Example with Different Options
import { ContentSimilarityMetric } from "@mastra/evals/nlp";
 
// Case-sensitive comparison
const caseSensitiveMetric = new ContentSimilarityMetric({
  ignoreCase: false,
  ignoreWhitespace: true
});
 
const result1 = await caseSensitiveMetric.measure(
  "Hello World",
  "hello world"
); // Lower score due to case difference
 
// Example output:
// {
//   score: 0.75,
//   info: { similarity: 0.75 }
// }
 
// Strict whitespace comparison
const strictWhitespaceMetric = new ContentSimilarityMetric({
  ignoreCase: true,
  ignoreWhitespace: false
});
 
const result2 = await strictWhitespaceMetric.measure(
  "Hello   World",
  "Hello World"
); // Lower score due to whitespace difference
 
// Example output:
// {
//   score: 0.85,
//   info: { similarity: 0.85 }
// }
Related
Completeness Metric
Textual Difference Metric
Answer Relevancy Metric
Keyword Coverage Metric
Last updated on March 13, 2025


ContextPositionMetric
The ContextPositionMetric class evaluates how well context nodes are ordered based on their relevance to the query and output. It uses position-weighted scoring to emphasize the importance of having the most relevant context pieces appear earlier in the sequence.

Basic Usage
import { openai } from "@ai-sdk/openai";
import { ContextPositionMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new ContextPositionMetric(model, {
  context: [
    "Photosynthesis is a biological process used by plants to create energy from sunlight.",
    "The process of photosynthesis produces oxygen as a byproduct.",
    "Plants need water and nutrients from the soil to grow.",
  ],
});
 
const result = await metric.measure(
  "What is photosynthesis?",
  "Photosynthesis is the process by which plants convert sunlight into energy.",
);
 
console.log(result.score); // Position score from 0-1
console.log(result.info.reason); // Explanation of the score
Constructor Parameters
model:
ModelConfig
Configuration for the model used to evaluate context positioning
options:
ContextPositionMetricOptions
Configuration options for the metric
ContextPositionMetricOptions
scale?:
number
= 1
Maximum score value
context:
string[]
Array of context pieces in their retrieval order
measure() Parameters
input:
string
The original query or prompt
output:
string
The generated response to evaluate
Returns
score:
number
Position score (0 to scale, default 0-1)
info:
object
Object containing the reason for the score
string
reason:
string
Detailed explanation of the score
Scoring Details
The metric evaluates context positioning through binary relevance assessment and position-based weighting.

Scoring Process
Evaluates context relevance:

Assigns binary verdict (yes/no) to each piece
Records position in sequence
Documents relevance reasoning
Applies position weights:

Earlier positions weighted more heavily (weight = 1/(position + 1))
Sums weights of relevant pieces
Normalizes by maximum possible score
Final score: (weighted_sum / max_possible_sum) * scale

Score interpretation
(0 to scale, default 0-1)

1.0: Optimal - most relevant context first
0.7-0.9: Good - relevant context mostly early
0.4-0.6: Mixed - relevant context scattered
0.1-0.3: Suboptimal - relevant context mostly later
0.0: Poor ordering - relevant context at end or missing
Example with Analysis
import { openai } from "@ai-sdk/openai";
import { ContextPositionMetric } from "@mastra/evals/llm";
 
// Configure the model for evaluation
const model = openai("gpt-4o-mini");
 
const metric = new ContextPositionMetric(model, {
  context: [
    "A balanced diet is important for health.",
    "Exercise strengthens the heart and improves blood circulation.",
    "Regular physical activity reduces stress and anxiety.",
    "Exercise equipment can be expensive.",
  ],
});
 
const result = await metric.measure(
  "What are the benefits of exercise?",
  "Regular exercise improves cardiovascular health and mental wellbeing.",
);
 
// Example output:
// {
//   score: 0.5,
//   info: {
//     reason: "The score is 0.5 because while the second and third contexts are highly
//           relevant to the benefits of exercise, they are not optimally positioned at
//           the beginning of the sequence. The first and last contexts are not relevant
//           to the query, which impacts the position-weighted scoring."
//   }
// }
Related
Context Precision Metric
Answer Relevancy Metric
Completeness Metric
Context Relevancy Metric
Last updated on March 13, 2025


VercelDeployer
The VercelDeployer deploys Mastra applications to Vercel, handling configuration, environment variable synchronization, and deployment processes. It extends the abstract Deployer class to provide Vercel-specific deployment functionality.

Usage Example
import { Mastra } from '@mastra/core';
import { VercelDeployer } from '@mastra/deployer-vercel';
 
const mastra = new Mastra({
  deployer: new VercelDeployer({
    teamId: 'your-team-id',
    projectName: 'your-project-name',
    token: 'your-vercel-token'
  }),
  // ... other Mastra configuration options
});
Parameters
Constructor Parameters
teamId:
string
Your Vercel team ID.
projectName:
string
Name of your Vercel project (will be created if it doesn't exist).
token:
string
Your Vercel authentication token.
Vercel Configuration
The VercelDeployer automatically generates a vercel.json configuration file with the following settings:

{
  "version": 2,
  "installCommand": "npm install --omit=dev",
  "builds": [
    {
      "src": "index.mjs",
      "use": "@vercel/node",
      "config": {
        "includeFiles": ["**"]
      }
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "index.mjs"
    }
  ]
}
Environment Variables
The VercelDeployer handles environment variables from multiple sources:

Environment Files: Variables from .env.production and .env files.
Configuration: Variables passed through the Mastra configuration.
Vercel Dashboard: Variables can also be managed through Vercel’s web interface.
The deployer automatically synchronizes environment variables between your local development environment and Vercel’s environment variable system, ensuring consistency across all deployment environments (production, preview, and development).

Project Structure
The deployer creates the following structure in your output directory:

output-directory/
├── vercel.json     # Deployment configuration
└── index.mjs       # Application entry point with Hono server integration
Last updated on March 13, 2025
