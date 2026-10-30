# Model Context Protocol (MCP) — Onboarding Notes

Welcome to the team! This note answers the question every new joiner asks in week one: **what is MCP, and why does everyone keep talking about it?** Read it top to bottom in about 20 minutes, then skim the videos in Section 4 over your first week. The full document lives at [docs/mcp-onboarding.md](docs/mcp-onboarding.md) in this repository.

---

## 1. What is MCP? (Plain-language primer)

The **Model Context Protocol (MCP)** is an open-source standard for connecting AI applications to external systems. Using MCP, AI applications like Claude or ChatGPT can connect to **data sources** (local files, databases), **tools** (search engines, calculators, APIs) and **workflows** (specialized prompts) so the model can access key information and perform tasks.

A helpful analogy from the official documentation: **MCP is like a USB-C port for AI applications**. Just as USB-C gives you one standardized way to plug devices into a computer, MCP gives you one standardized way to plug external systems into an AI application.

### The problem it solves

Before MCP, connecting a model to the outside world meant a custom integration for every (model × system) pair — the **"N×M problem"**. Every service exposes a differently-shaped API, and every client had to write its own glue code for each pairing. MCP replaces that with a single protocol: implement once on the client side, expose once on the server side, and everything speaks the same language.

### What it enables in practice

The official documentation points to several concrete uses:

- Agents can access a user's Google Calendar and Notion, acting as a more personalized assistant.
- Claude Code can generate an entire web app from a Figma design.
- Enterprise chatbots can connect to multiple databases across an organization and analyze data using chat.
- AI models can create 3D designs in Blender and print them on a 3D printer.

### Why it matters, by role

| Role | Benefit |
| --- | --- |
| **Developers** | MCP reduces development time and complexity when building or integrating with AI applications and agents. |
| **AI applications / agents** | MCP gives them access to an ecosystem of data sources, tools, and apps, improving capabilities and the end-user experience. |
| **End users** | More capable AI applications or agents that can access user data and take actions on the user's behalf. |

### Origins

Anthropic introduced MCP in November 2024 as a way to plug AI applications into data and tools without a bespoke integration for each combination. The protocol was subsequently donated to the **Agentic AI Foundation (AAIF)**, a directed fund under the Linux Foundation, so it is now a vendor-neutral, open standard rather than a single company's product.

---

## 2. How it works: architecture (clients, servers, tools, resources)

MCP follows a **client–server architecture**. An MCP host — an AI application like Claude Code or Claude Desktop — establishes connections to one or more MCP servers, and does so by creating **one MCP client per server**. Each client maintains a dedicated connection with its corresponding server. (Local servers using the stdio transport typically serve one client; remote servers using Streamable HTTP can serve many.)

```
┌─────────────────────────────────────┐
│         MCP Host (AI app)           │
│   [Client 1]   [Client 2]   [Client 3]│
└───────┬────────────┬────────────┬────┘
        │            │            │
        ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server A │  │ Server B │  │ Server C │
│ local    │  │ local    │  │ remote   │
│ (files)  │  │ (db)     │  │ (Sentry) │
└──────────┘  └──────────┘  └──────────┘
```

### Participants

| Participant | What it does |
| --- | --- |
| **MCP host** | The AI application that coordinates and manages one or multiple MCP clients (e.g. Claude Desktop, VS Code, IDEs, AI tools). |
| **MCP client** | A component that maintains a connection to an MCP server and obtains context from that server for the host to use. |
| **MCP server** | A program that provides context and capabilities to MCP clients. The term refers to the program regardless of where it runs — local or remote. |

Local vs. remote: a local server runs on the same machine as the host (e.g. Claude Desktop launching a filesystem server over stdio), while a remote server runs somewhere else and is reached over the network (e.g. the official Sentry MCP server over Streamable HTTP).

### Two layers

- **Data layer** — defines the JSON-RPC 2.0 based protocol for client–server communication, including lifecycle management (connection initialization, capability negotiation, termination) and the core primitives: tools, resources, prompts, plus notifications and progress tracking.
- **Transport layer** — defines how messages travel: **stdio** (standard input/output streams between local processes, optimal for local servers) or **Streamable HTTP** (HTTP POST with optional Server-Sent Events, enabling remote servers with OAuth and standard HTTP authentication).

### The three server primitives

| Primitive | What it is | Example | Who controls it |
| --- | --- | --- | --- |
| **Tools** | Executable functions the LLM can call to perform actions | `get_weather(location)`, database queries, API calls | **Model-controlled** — the model decides when to call them |
| **Resources** | Data sources that provide contextual information, each addressed by a URI | Files, database schemas, API responses | **Application-driven** — the host decides how to surface them |
| **Prompts** | Reusable templates that structure interactions with the model | A templated prompt for a recurring task | **User-controlled** — explicitly selected by the user |

The distinction that trips people up: **tools are model-controlled** (the LLM discovers and invokes them automatically based on context), while **resources are application-driven** (the host decides how to include them). The protocol mandates no specific UI pattern for either, but for trust and safety there should always be a **human in the loop** who can deny tool invocations; clients should show which tools are exposed, indicate when they run, and prompt for confirmation on consequential operations.

### A typical tool call, end to end

1. The client sends `tools/list` to the server and receives the available tools with their JSON schemas.
2. The LLM selects a tool and fills in its arguments.
3. The client sends `tools/call` with those arguments.
4. The server executes and returns the result through the client to the model.

Resources work analogously with `resources/list` / `resources/read`, each resource addressed by a URI, with optional subscriptions so the host can be notified when the list changes.

---

## 3. The ecosystem: which clients support MCP

MCP is an open protocol supported across a wide range of clients and servers. The official docs highlight AI assistants like **Claude and ChatGPT**, and development tools like **Visual Studio Code, Cursor, and MCPJam**. The practical group is:

| Category | Clients |
| --- | --- |
| **Anthropic's own** | Claude Desktop, Claude Code, Claude.ai — the protocol's originators with the deepest support |
| **AI chat assistants** | ChatGPT, Gemini, Perplexity, LibreChat, Cherry Studio |
| **Code editors & IDEs** | Visual Studio Code (with GitHub Copilot), Cursor, Zed, Windsurf, Cline, Continue |
| **Agent & CLI tools** | Codex CLI, Gemini CLI, Goose, MCPJam, and the broader CLI ecosystem |

OpenAI adopted MCP in its Agents SDK in March 2025, making the official interface a first-class standard there as well. For onboarding, the practical takeaway is simple: whatever AI application your team uses, there is a good chance it already speaks MCP — so servers built once work everywhere.

---

## 4. Videos

Pick your style: a quick 4-minute primer, a long hands-on build course, a live workspace demo, or a conference/workshop talk by an Anthropic engineer.

- [MCP Explained in 4 Minutes](https://www.youtube.com/watch?v=xczmoDpuDyQ) — Codesh — ~4 minutes. A compact, clear primer covering the N×M integration problem, the three primitives (resources, tools, prompts), transports, and human-in-the-loop safety.
- [Full Course (Lessons 1-11) MCP for Beginners](https://www.youtube.com/watch?v=VfZlglOWWZw) — Microsoft Developer — duration not listed on the video page. A full beginners course from the Microsoft Developer channel, walking through client/server architecture, all three server primitives, JSON-RPC communication, security, and deploying servers on Azure with MCP Inspector.
- [Claude's Model Context Protocol is here... Let's test it](https://www.youtube.com/watch?v=HyzlYwjoXOQ) — Fireship — duration not listed on the video page. A fast, energetic live demo inside Claude Desktop: building an MCP server from a PostgreSQL database and an S3-like storage bucket, connecting it via stdio, and using tools and resources directly from the assistant.
- [Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg) — AI Engineer — duration not listed on the video page. A conference-style workshop by an Anthropic engineer covering the philosophy behind MCP, adoption traction, the "N times M" flattening, the roadmap, and real agent demos alongside apps like Cursor, Windsurf, and Goose.

> Durations above are taken from the video listing or title where shown; the Microsoft Developer, Fireship, and AI Engineer listings do not display a duration (the page metadata is unavailable to the authorship tooling used here), so they are marked accordingly. Also: the Fireship and AI Engineer talks each include live coding, which makes them great mailing-list stand-ins for a pair review of this document.

---

## Sources

Primary documentation (read for this note):

- [What is the Model Context Protocol (MCP)? — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Architecture overview — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/2025-11-25/learn/architecture)
- [MCP clients list — modelcontextprotocol.io](https://modelcontextprotocol.io/clients)

Supplementary:

- [Introduction.mdx — MCP docs GitHub repo](https://github.com/modelcontextprotocol/docs/blob/main/introduction.mdx) (the source of the USB-C analogy and the client/tools/resources intro)
- [Brennan on MCP adoption — Claude Academy](https://academy.claude.com/courses/introduction-to-model-context-protocol/introducing-mcp) (background on why clients adopt MCP)
- [MCP clients list — third-party comparison](https://fenbs.ai/blog/mcp-clients) (used to cross-check client support: Claude, ChatGPT, Codex, Cursor, VS Code/GitHub Copilot, Gemini CLI, Devin, Zed)

Videos selected (see Section 4 for links, titles, publishers, and durations).
