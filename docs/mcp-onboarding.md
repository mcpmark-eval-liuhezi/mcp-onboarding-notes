# MCP Onboarding Notes

> A plain-language primer, architecture walkthrough, ecosystem map, and video references for new teammates joining a team that uses the Model Context Protocol.

This note answers one question: **what is MCP, how does it work, who supports it, and where should I start watching?** It is intentionally short enough to read before two new teammates arrive on Monday.

---

## 1. What is MCP? (Plain-Language Primer)

**MCP stands for Model Context Protocol.** It is an **open-source standard for connecting AI applications to external systems** — data sources (local files, databases), tools (search engines, calculators), and workflows (specialized prompts). The docs are explicit about the analogy: **think of MCP like a USB-C port for AI applications.** Just as USB-C provides one standardized way to connect devices, MCP provides one standardized way to connect AI applications to external systems.

### What MCP enables
- Agents can access your Google Calendar and Notion, acting as a more personalized AI assistant.
- Claude Code can generate an entire web app using a Figma design.
- Enterprise chatbots can connect to multiple databases across an organization, letting users analyze data using chat.
- AI models can create 3D designs on Blender and print them on a 3D printer.

### Why it matters
**Developers:** MCP reduces development time and complexity when building, or integrating with, an AI application or agent.

**AI applications or agents:** MCP gives them access to an ecosystem of data sources, tools and apps, which enhances their capabilities and improves the end-user experience.

**End-users:** MCP results in more capable AI applications that can access user data and take actions on the user's behalf when necessary.

### The problem it replaced
Before MCP, every AI application—Claude or ChatGPT—had to build a custom integration for every combination of app and system. MCP replaces that N-times-M sprawl of bespoke bridges with a single standard interface. You build one MCP server and it works with Claude, Cursor, Gemini, ChatGPT and any other MCP-supporting AI tool.

### Status of the standard
Anthropic open-sourced MCP in November 2024 and subsequently donated it to the Linux Foundation, in part to make sure the standard could not be undone by a single vendor. Google and Microsoft have both adopted it, and the protocol continues to evolve through open participation.

---

## 2. Architecture: Clients, Servers, Tools, Resources

### Participants
MCP follows a **client-host-server architecture**. An MCP host—a program such as Claude Code, Claude Desktop, Visual Studio Code, or an AI tool—establishes connections to one or more MCP servers, and does so by creating one MCP client per server. Each client maintains a dedicated connection with its corresponding server.

- **MCP Host:** the AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client:** a component that maintains a connection to an MCP server and obtains context from it for the host to use.
- **MCP Server:** a program that provides context to MCP clients. Servers can execute locally or remotely.

**Memory aid:** one Host → many Clients → many Servers. Local servers that use the STDIO transport typically serve a single client, whereas remote servers that use the Streamable HTTP transport typically serve many clients. Visual Studio Code, for example, instantiates a separate client object each time it connects to a new server—so one IDE can talk to the filesystem server, the Sentry server and any others simultaneously.

### Two layers
MCP consists of two layers:

- **Data layer:** defines the JSON-RPC 2.0 based protocol for client-server communication, including lifecycle management, capability negotiation, and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer:** defines the communication mechanisms and channels between clients and servers, including connection establishment, message framing, and authorization.

The data layer is the inner layer; the transport layer is the outer layer. MCP supports two transports: **STDIO** for local process-to-process communication on the same machine (optimal performance, no network overhead), and **Streamable HTTP** (`HTTP POST` with optional Server-Sent Events) for remote servers, which supports standard HTTP authentication including bearer tokens and OAuth.

> NOTE: MCP focuses solely on the protocol for context exchange. It does not dictate how AI applications use language models or manage the provided context.

### Server primitives
The most important concept in MCP is its **primitives**—the building blocks servers expose, and the build server/client docs describe in full. A server exposes three building blocks through three features (guided by JSON Schema for validation):

| Feature | What it is | Who controls it | Example |
| --- | --- | --- | --- |
| **Tools** | Executable functions the model can invoke to perform actions—file operations, API calls, database queries | The Model | `searchFlights`, `createCalendarEvent`, `sendEmail` |
| **Resources** | Passive, read-only data sources that supply context, such as file contents, database schemas, or API documentation | The Application | Retrieve documents, access knowledge bases, read calendars |
| **Prompts** | Pre-built instruction templates that structure interactions with language models | The User | Plan a vacation, summarize meetings, draft an email |

*Prim: `食物`*

**Tools** are model-controlled: the model decides when to discover and invoke them, then calls the server with a typed request. Each tool has a `name`, `description`, and `inputSchema`. Tools may require user consent prior to execution, and MCP emphasizes human oversight through UI controls, approval dialogs, permission settings, and activity logs. Discovery is dynamic—servers can advertise new tools at `tools/list`, changes, and effects, so clients re-fetch rather than hard-coding.

**Resources** expose data from files, APIs, databases, or anything an AI needs to understand context. Each resource has a unique URI (for example `file:///path/to/document.md`) declaring its MIME type, supporting both fixed/direct resources and dynamic resource templates with parameters. Applications can read resources directly and decide how to inject them—selecting portions, searching with embeddings, or passing them to the model.

**Prompts** are reusable templates that the user selects when they want a standard interaction structure with tools and resources.

The data layer also supports **client features** (sampling and elicitation) and **utility features** (notifications for real-time updates and progress tracking for long-running operations), plus lifecycle management via `server/discover` in the current spec version.

### Building with MCP
The practical build order is: build an MCP **server** that exposes your data and tools, build an **MCP client** that connects to those servers, and optionally use **MCP Apps**—interactive apps that run inside AI clients—for richer interfaces.

---

## 3. Ecosystem: Where MCP Is Supported Today

MCP is an open protocol supported across a wide range of clients and servers. The official **clients matrix** lists feature support for the main applications: Claude and ChatGPT, development tools like Visual Studio Code, Cursor, MCPJam, and many others, so it is "easy to build once and integrate everywhere."

### AI assistants & chat clients
- **Claude / Claude Desktop:** comprehensive desktop support for resources, prompt templates, tool execution, and local server connections. (Note: the Claude.ai web application does not currently support MCP.)
- **ChatGPT (OpenAI):** MCP support for remote servers via a connections UI in settings; partial OAuth client-credentials support is flagged in the current matrix.

### Coding-assistant and IDE clients
- **Visual Studio Code** (GitHub Copilot): agent mode with direct interaction with MCP-provided tools; supports stdio and server-sent events transports, dynamic tool discovery, secure secret configuration, and one-click installation.
- **Cursor:** MCP tools in Cursor Composer, supporting both STDIO and SSE transports.
- **Zed:** built-in MCP support focused on prompt templates (surfacing as slash commands) and tool integration; does not support MCP resources.
- **Continue:** open-source AI code assistant with built-in support for all MCP features, including `@` mentions of resources and both built-in and MCP tools.
- **Cline:** autonomous coding agent in VS Code that supports tools and resources.
- **Cody (Sourcegraph):** tools via OpenCTX, context passing into AI agents.
- **Roo Code, Windsurf, Cursor:** tools-first integrations, often agent-mode.
- **5ire, BeeAI Framework, fast-agent, Genkit, GenAIScript, Goose, LibreChat, mcp-agent, Theia AI / Theia IDE, OpenSumi, Apify MCP Tester, SpinAI, mcp-agent:** tool support with varying combinations of resources, prompts, and elicitation.

### Enterprise platforms
- **Microsoft 365 Copilot** and **Microsoft Copilot Studio**: enterprise-grade security and compliance features around MCP connections.
- Google and Microsoft have both adopted the protocol.

### Feature-support matrix highlights
| Client | Resources | Prompts | Tools |
| --- | --- | --- | --- |
| Claude Desktop App | ✅ | ✅ | ✅ |
| Claude Code | ❌ | ✅ | ✅ |
| VS Code GitHub Copilot | ❌ | ✅ | ✅ |
| Cursor | ❌ | ❌ | ✅ |
| Zed | ❌ | ✅ | ❌ |
| OpenAI ChatGPT | ❓ | ❓ | ✅ (remote) |

Check the official **client support matrix for enterprise clients** for the current per-client details, including OAuth client credentials, enterprise auth, MCP Apps, skills, and tasks columns.

### The server side
Common reference server types include filesystem servers, database servers, GitHub servers, Slack servers, and calendar servers. The official repositories ship reference server implementations, and the open-source server list publishes pre-built servers for popular platforms—Google Drive, Slack, Postgres, and Puppeteer are common examples.

---

## 4. Videos: Introduction Picks

Each entry below links to the video, names its publisher, and gives the duration where the listing shows one.

1. **The Model Context Protocol (MCP)** — **Anthropic**. A panel discussion with Theo Chu, David Soria Parra, and Alex Albert covering the protocol's core components, how it started, how it was received, how it was made open source, and what comes next. (The listed duration is 19:35.) — https://www.youtube.com/watch?v=CQywdSdi5iA
2. **Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic** — **AI Engineer** (SUMMIT talk). A deep-dive workshop walk-through of MCP philosophy, adoption patterns, and the roadmap. (Listed duration: 1:44:12.) — https://www.youtube.com/watch?v=kQmXtrmQ5Zg
3. **Anthropic's New Model Context Protocol in 10 Minutes** — **Developers Digest**. A narration of the official MCP announcement, the SDK approach, and pre-built servers for Google Drive, Slack, Postgres, and Puppeteer. (Listed duration: 10:01.) — https://www.youtube.com/watch?v=bpb45HVwwD8
4. **Claude MCP Tutorial: Give Claude Superpowers in 30 Seconds** — **Teacher's Tech**. A practical tutorial walking through Claude connectors to Gmail, Canva, Notion, Slack, and over 150 apps, plus the Many reasons MCP matters and what's next. (Duration: not shown in the listing.) — https://www.youtube.com/watch?v=Lxznc91wlTk
5. **Minimum viable AI company** — **Anthropic**. A survey of the early agent ecosystem and where MCP fits among tools, APIs, and agent frameworks. (Duration: not shown in the listing.) — https://www.youtube.com/watch?v=b7l5lO2gfDk

### Quick reference table

| # | Title | Publisher | Duration (where listed) |
| --- | --- | --- | --- |
| 1 | The Model Context Protocol (MCP) | Anthropic | 19:35 |
| 2 | Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic | AI Engineer (SUMMIT) | 1:44:12 |
| 3 | Anthropic's New Model Context Protocol in 10 Minutes | Developers Digest | 10:01 |
| 4 | Claude MCP Tutorial: Give Claude Superpowers in 30 Seconds | Teacher's Tech | not shown |
| 5 | Minimum viable AI company | Anthropic | not shown |

**Recommended first watch:** entry #1—it is a 19-minute, conversation-driven overview that moves from core components through history and outlook, which is the shortest path from zero to confident.

---

## Sources

### Official MCP documentation
1. **What is the Model Context Protocol (MCP)?** https://modelcontextprotocol.io/introduction
2. **Architecture overview** https://modelcontextprotocol.io/docs/learn/architecture
3. **Understanding MCP servers** (core server features: tools, resources, prompts) https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md
4. **Clients** https://modelcontextprotocol.io/clients
5. **Client support matrix (enterprise clients)** https://modelcontextprotocol.io/extensions/client-matrix

### Open-source docs source
6. **`docs/clients.mdx` in the `modelcontextprotocol/docs` repository** — feature-support matrix and per-client notes https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx

### Client repository
7. **`docs/clients.mdx` in the `modelcontextprotocol/modelcontextprotocol` repository** — build-server/build-client client guides and client notes https://github.com/modelcontextprotocol/modelcontextprotocol/blob/ff960c9e/docs/clients.mdx

### Videos
8–12. Listed in Section 4 above; each entry cites its YouTube channel/publisher.

---

## Suggested usage for Monday onboarding

1. Read **Section 1 (Primer)** first to internalize the USB-C framing and the problem MCP replaces.
2. Watch entry **#1** (Anthropic, 19:35) for a paced, conversation-driven overview.
3. Review **Sections 2–3** to understand host/client/server roles and which clients you might integrate with.
4. Watch entry **#3** (Developers Digest, 10:01) for a tight SDK-focused tour if you want something to build along with.
