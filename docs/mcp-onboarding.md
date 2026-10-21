# Model Context Protocol (MCP) — Onboarding Notes

Welcome to the team! This note answers the two questions every new joiner asks in week one: *what is MCP, and why does everyone keep talking about it?* Read it top to bottom in about 20 minutes, then skim the videos in Section 4 over your first week.

---

## 1. What is MCP? (Plain-language primer)

The **Model Context Protocol (MCP)** is an open-source standard for connecting AI applications to external systems. It lets apps like Claude or ChatGPT connect to **data sources** (local files, databases), **tools** (search engines, calculators, APIs) and **workflows** (specialized prompts) so the model can access key information and actually perform tasks.

The docs use an analogy that is worth memorizing: **MCP is like a USB-C port for AI applications**. Just as USB-C gives you one standardized way to plug devices into a computer, MCP gives you one standardized way to plug external systems into an AI application.

### The problem it solves

Before MCP, connecting a model to the outside world meant a custom integration for every (model × system) pair — the "N×M problem". Every service exposes a differently-shaped API, and every client had to write its own glue code for each one. MCP replaces that with a single protocol: implement once on the client side, expose once on the server side, and everything speaks the same language.

### Why it matters to you, concretely

- **Developers** get less integration code and less complexity when building agents.
- **AI applications and agents** get an instant ecosystem of data sources, tools and apps to draw on.
- **End users** get assistants that can access their data and take actions on their behalf — reading a calendar, booking a trip, generating a web app from a Figma design, or printing a 3D design on a 3D printer.

### Origins, in one paragraph

MCP was introduced by Anthropic in November 2024 as an open standard. In December 2025 Anthropic donated the protocol to the **Agentic AI Foundation (AAIF)**, a directed fund under the Linux Foundation co-founded with Block and OpenAI, with support from Google, Microsoft, AWS, Cloudflare and Bloomberg — so MCP is now a vendor-neutral open standard, not a single company's product.

### Terms you'll hear in the hallways

- **MCP host** — the AI application the user actually interacts with (Claude Desktop, an IDE, your agent app).
- **MCP client** — the protocol-level component inside the host that maintains one connection to one server.
- **MCP server** — the program exposing capabilities to the host.
- **Tools, resources, prompts** — the three building blocks a server can expose (see Section 2).

---

## 2. How it works: architecture (clients, servers, tools, resources)

MCP follows a **client–server architecture**: an **MCP host** (an AI application like Claude Code or Claude Desktop) establishes connections to one or more **MCP servers**. Crucially, the host creates **one MCP client per server**, and each client maintains a dedicated connection with its corresponding server. A VS Code host that connects to the Sentry server and a local filesystem server spins up two separate client objects, one for each.

```
┌─────────────────────────────────────┐
│          MCP Host (AI app)           │
│  [Client 1]  [Client 2]  [Client 3] │
└─────┬──────────────┬──────────┬─────┘
      │              │          │
      ▼              ▼          ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server A │  │ Server B │  │ Server C │
│ local    │  │ local    │  │ remote   │
│ (files)  │  │ (db)     │  │ (Sentry) │
└──────────┘  └──────────┘  └──────────┘
```

### Participants

| Participant | What it does |
| --- | --- |
| **MCP host** | The AI application that coordinates and manages one or multiple MCP clients (e.g. Claude Desktop, VS Code) |
| **MCP client** | A component that maintains a connection to an MCP server and obtains context from it for the host to use |
| **MCP server** | A program that provides context and capabilities to MCP clients |

### Local vs. remote servers

An MCP server can run **locally** (e.g. Claude Desktop launching a filesystem server over the **stdio** transport, since stdio serves one client at a time) or **remotely** (e.g. the official Sentry MCP server running on the Sentry platform over **Streamable HTTP**, which can serve many clients). "Local" and "remote" describe where the server runs, not a different kind of server.

### Two layers

- **Data layer** — the JSON-RPC 2.0 based protocol for client–server communication: capability/version discovery, and the core primitives (tools, resources, prompts) plus notifications and progress tracking.
- **Transport layer** — how messages actually travel: **stdio** (standard input/output streams between local processes, no network overhead) or **Streamable HTTP** (HTTP POST with optional Server-Sent Events, supporting OAuth and standard HTTP auth).

### The three server building blocks

| Feature | What it is | Example | Who controls it |
| --- | --- | --- | --- |
| **Tools** | Functions the LLM can actively call | `get_weather(location)` — querying databases, calling APIs, computations | **Model-controlled**: the LLM discovers and invokes them automatically |
| **Resources** | Context data the model can read | Files, database schemas, application-specific info — each identified by a **URI** | **Application-driven**: the host app decides how to include it (tree/list UI, search, auto-include) |
| **Prompts** | Reusable interaction templates | A specialized prompt for a recurring task | **User-controlled**: explicitly selected by the user |

The distinction that trips people up: **tools are model-controlled** (the LLM decides when to call them, based on context), while **resources are application-driven** (the host decides how to surface them). The protocol mandates no specific UI pattern for either — but for trust & safety there should **always be a human in the loop** who can deny tool invocations; clients should show which tools are exposed, indicate when they run, and prompt for confirmation on consequential operations.

### A typical tool call, end to end

1. The client sends `tools/list` to the server and receives the available tools with their JSON schemas.
2. The LLM selects a tool and fills in arguments.
3. The client sends `tools/call` with those arguments.
4. The server executes and returns the result (e.g. weather text) back through the client to the model.

Resources work analogously with `resources/list` / `resources/read`, each resource addressed by URI, with optional subscriptions so the host can be notified when the list changes.

---

## 3. The ecosystem: which clients support MCP

MCP is an **open protocol supported across a wide range of clients and servers**. The official docs highlight AI assistants like **Claude and ChatGPT**, and development tools like **Visual Studio Code, Cursor, and MCPJam**, among many others — making it possible to build once and integrate everywhere.

In practice, the support landscape groups like this:

| Category | Clients |
| --- | --- |
| **Anthropic's own** | Claude Desktop, Claude Code, Claude.ai — the protocol's originators, with the deepest support |
| **AI chat assistants** | ChatGPT (via Developer Mode / Apps), Gemini, Perplexity, LibreChat, Cherry Studio |
| **Code editors & IDEs** | Visual Studio Code (GitHub Copilot), Cursor, Windsurf, Zed, Cline, Continue |
| **Agent & CLI tools** | Codex CLI, Gemini CLI, Goose, MCPJam and the broader CLI ecosystem |

For onboarding, the practical takeaway is simple: **the tools you already use almost certainly speak MCP**, and any MCP server you build can be used by all of them. There's also a community [**MCP Inspector**](https://modelcontextprotocol.io/docs/tools/inspector) for testing and debugging servers, official SDKs in Python, TypeScript/JavaScript and other languages, and a growing public registry of thousands of servers (10,000+ active servers as of the AAIF donation announcement).

---

## 4. Video walkthroughs (our picks)

A curated shortlist of solid introductions — watch the first one if you only watch one.

1. **[MCP In 26 Minutes (Model Context Protocol)](https://www.youtube.com/watch?v=kOhLoixrJXo)** — Tina Huang — 26:23
   The go-to beginner explainer: what MCP is, how it works, how to use existing MCP servers, and how to build your own with both code and no-code approaches.

2. **[The Model Context Protocol: Connecting AI to Everything](https://www.youtube.com/watch?v=T4-BdZQUCSE)** — PyTorch (speaker: Adam Jones, Anthropic) — 32:53
   A conference talk from someone on the MCP team itself: what MCP is, why it exists, use cases for building a server or client, a walkthrough of building a first MCP server with Claude Code, and where the ecosystem is headed (computer use, programmatic tool calling, agent-to-agent communication).

3. **[How Model Context Protocol (MCP) actually works](https://www.youtube.com/watch?v=cGuyrANVi4A)** — Google Cloud Tech — 7:58
   A tight, 8-minute deep-dive from a non-Anthropic perspective: what MCP is, how it compares to traditional APIs, and why APIs designed for deterministic programs don't fit probabilistic models that need to discover their capabilities.

4. **[Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w)** — Shaw Talebi — 19:49
   A structured 20-minute intro from an AI-agents series: the concepts, how clients and servers connect, then a live demo building a Python server and connecting it to Claude Desktop.

### Suggested viewing order

Watch #1 for the conceptual foundation → #3 if you want the "why" in 8 minutes → #2 for the ecosystem's direction from the Anthropic team → #4 when you're ready to type code along.

---

## Sources

Official documentation (read while writing this note):

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/) — official overview page
- [Getting started: What is MCP?](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources](https://modelcontextprotocol.io/docs/concepts/resources)
- [Understanding MCP clients](https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts.md)
- [Understanding MCP servers](https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md)
- [Clients](https://modelcontextprotocol.io/clients) — client ecosystem overview
- [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector)

Background reading:

- [Anthropic — Donating MCP to the Agentic AI Foundation](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation)
- [Linux Foundation — Formation of the Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
- [Model Context Protocol — Wikipedia](https://en.wikipedia.org/wiki/Model_Context_Protocol)
