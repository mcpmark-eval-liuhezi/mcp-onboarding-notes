# MCP Onboarding Notes

> A plain-language primer, architecture walkthrough, ecosystem map, and video references for new teammates joining a team that uses the **Model Context Protocol**.

MCP is an **open-source standard for connecting AI applications to external systems** — data sources, tools, and workflows. Think of it as a **USB-C port for AI applications**: a single standardized connection that lets an AI app reach files, databases, APIs, and services without a bespoke integration for every pairing.

A new revealing quote: **"MCP reduces development time and complexity when building or integrating with an AI application or agent. It gives AI applications or agents access to an ecosystem of data sources, tools and apps, which enhances their capabilities and improves the end-user experience. It results in more capable AI applications or agents that can access user data and take actions on the user's behalf when necessary."**

---

## 1. What is MCP? (Plain-Language Primer)

MCP = **Model Context Protocol**, launched open-source by Anthropic in November 2024. It was donated to the Linux Foundation and is now governed as an open standard, so it is not tied to any single vendor (Google and Microsoft have both adopted it).

### The problem it solves
Before MCP, every AI app–external-system pairing needed its own custom integration. AI applications like Claude or ChatGPT had to connect to data sources (local files, databases), tools (search engines, calculators) and workflows (specialized prompts) — one bespoke bridge for each combination.

### What it actually does
Working through the MCP specification, it defines:
- **封闭 montre... (oops, drafting):**

MCP defines three interfaces (primitives) through which servers expose what AI applications can use — **tools** (executable functions the model can invoke), **resources** (context data the application can read), and **prompts** (reusable templates for structuring interactions with the language model).

### Why it matters, in one breath
- **Developers:** build one connection (an MCP server) and it works with every MCP-compatible AI application — "build once, integrate everywhere."
- **AI applications:** gain access to an ecosystem of data sources, tools and apps, improving their capabilities and end-user experience.
- **End users:** get more capable AI applications that can access your data and act on your behalf.

### Worked examples from the docs
- Agents accessing your Google Calendar and Notion to act as a more personalized AI assistant.
- Claude Code generating an entire web app from a Figma design.
- Enterprise chatbots connecting to multiple databases across the organization.
- AI models creating 3D designs on Blender and printing them on a 3D printer.

---

## 2. Architecture: Clients, Servers, Tools, Resources

### Participants

MCP follows a **client-host-server architecture**:

- **MCP Host** — the AI application that coordinates and manages one or multiple MCP clients (e.g., Claude Code, Claude Desktop, IDEs, AI tools).
- **MCP Client** — a component that maintains a connection to a specific MCP server and obtains context from it for the host to use. For each server, the host creates one dedicated client (a 1:1 connection).
- **MCP Server** — a program that provides context to MCP clients (e.g., a local filesystem server, a GitHub server, the Sentry server running remotely).

Memory aid: **One Host → many Clients → many Servers**, each client holding one dedicated connection to its server. Local servers typically serve a single client via STDIO transport; remote servers commonly serve many clients via the Streamable HTTP transport.

### Two layers

- **Data layer:** the JSON-RPC 2.0 protocol for client-server communication — lifecycle management, capability and version discovery, and core primitives (tools, resources, prompts, notifications, progress).
- **Transport layer:** the communication mechanisms (connection establishment, message framing, authorization), currently **STDIO** for local processes and **Streamable HTTP** for remote servers, including OAuth support.

NOTE: MCP focuses solely on the protocol for context exchange. It does not dictate how AI applications use LLMs or manage the provided context.

### Server primitives

| Primitive | What it is | Owner | Example methods |
| --- | --- | --- | --- |
| **Tools** | Executable functions the model can invoke (file operations, API calls, database queries) | The Model | `tools/list` (discover), `tools/call` (execute) |
| **Resources** | Passive, read-only data sources that provide context (file contents, database schemas, API docs) | The Application | `resources/list`, `resources/templates/list`, `resources/read` |
| **Prompts** | Pre-built instruction templates that structure interactions with language models | The User | — |

JSON Schema is used to validate tool inputs and outputs; tools may require user consent prior to execution. The model decides when to discover and invoke tools; each server exercises a known set of protocol operations (e.g., `tools/call`).

Server capabilities include **sampling** (clients can offer the host LLM), **elicitation** (servers can ask the client for user input), and **logging** (message logging to the client).

### How to build with the concepts
A practical build flows: build an MCP **server** that exposes data and tools; build an MCP **client** that connects to those servers; optionally use **MCP Apps** for interactive apps that run inside AI clients.

---

## 3. Ecosystem: Where MCP is Supported Today

MCP is an open protocol supported across a wide range of clients and servers — the goal being “build once and integrate everywhere.”

### AI assistants & chat clients
- **Claude / Claude Desktop** — full support for resources, prompts and tools (desktop app only; Claude.ai web does not currently support MCP).
- **ChatGPT (OpenAI)** — MCP support for remote servers via the connections UI in settings (partial OAuth client-credentials support in the current matrix).

### IDE & coding-assistant clients
- **Visual Studio Code** (GitHub Copilot) — agent mode with tools, roots, dynamic discovery, secure secret configuration, one-click installation, STDIO + SSE transports.
- **Cursor** — MCP tools in Composer; both STDIO and SSE transports.
- **Zed** — tool integration with resource support; prompts surface as slash commands (no resources).
- **Continue** — open-source assistant with built-in support for all MCP features.
- **Cline** — autonomous coding agent that supports tools and resources.
- **5ire, BeeAI Framework, fast-agent, Genkit, GenAIScript, Goose, LibreChat, Roo Code, Witsy, Roo Code / Windsurf, mcp-agent, Apify MCP Tester, Sourcegraph Cody, Theia AI / Theia IDE, OpenSumi, SpinAI** — each contribute tools, resources, prompts or elicitation in various combinations.

### Enterprise platforms
- **Microsoft Copilot Studio** — MCP tools; **Microsoft 365 Copilot** — enterprise-grade security and compliance features.
- **Google** and **Microsoft** have adopted MCP (Google and Microsoft adoption is documented in the ecosystem materials).

### Key feature-support matrix highlights
| Client | Resources | Prompts | Tools |
| --- | --- | --- | --- |
| Claude Desktop App | ✅ | ✅ | ✅ |
| Claude Code | ❌ | ✅ | ✅ |
| VS Code GitHub Copilot | ❌ | ✅ | ✅ |
| Cursor | ❌ | ❌ | ✅ |
| Zed | ❌ | ✅ | ❌ |
| OpenAI ChatGPT | ❓ | ❓ | ✅ (remote) |

The client-matrix page also lists贯通ng for enterprise auth, MCP Apps, tasks and skills availability, so check it for the latest per-client details.

### Server-side ecosystem
Common reference server types include filesystem servers, database servers, GitHub servers, Slack servers, and calendar servers. The official repository ships reference server implementations, and the `modelcontextprotocol/servers` repo publishes pre-built servers for popular platforms (Google Drive, Slack, Postgres, Puppeteer are common examples).

---

## 4. Videos: Introduction Picks

Hands-on, with details where the listing shows them.

1. **Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic** — AI Engineer (video                                            ,感受到 er of the workforce; full walkthrough of MCP philosophy, adoption patterns, and roadmap. — https://www.youtube.com/watch?v=kQmXtrmQ5Zg
2. **The Model Context Protocol (MCP)** — Anthropic (panel discussion with Theo Chu, Alex Albert, and David Soria Parra; covers MCP basics, history, launch and reception). — https://www.youtube.com/watch?v=CQywdSdi5iA
3. **Claude MCP Tutorial: Give Claude Superpowers in 30 Seconds** — Teacher's Tech (practical walkthrough using Claude connectors for Gmail, Canva, Notion, Slack and over 150 apps). — https://www.youtube.com/watch?v=Lxznc91wlTk
4. **Anthropic's New Model Context Protocol in 10 Minutes** — Developers Digest (walks through the official announcement, SDKs and pre-built servers). — https://www.youtube.com/watch?v=bpb45HVwwD8
5. **Minimum viable AI company** — Anthropic (survey of the early agent ecosystem and the role MCP plays in unglueing context). — https://www.youtube.com/watch?v=b7l5lO2gfDk

| # | Title | Publisher | Duration (where listed) |
| --- | --- | --- | --- |
| 1 | Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic | AI Engineer | 1:44:12 |
| 2 | The Model Context Protocol (MCP) | Anthropic | 19:35 |
| 3 | Claude MCP Tutorial: Give Claude Superpowers in 30 Seconds | Teacher's Tech | (not shown in listing) |
| 4 | Anthropic's New Model Context Protocol in 10 Minutes | Developers Digest | 10:01 |
| 5 | Minimum viable AI company | Anthropic | (not shown in listing) |

Primary pick for a first watch: #2 on the official channel, since it is a conversation-driven overview that moves from core components through history and what's next — a tidy linear introduction rather than a single demo.

---

## Sources

Official documentation used for this document:

1. **What is the Model Context Protocol (MCP)?** — https://modelcontextprotocol.io/introduction
2. **Architecture overview** — https://modelcontextprotocol.io/docs/learn/architecture
3. **Understanding MCP servers (core server features: tools, resources, prompts)** — https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md
4. **Clients page: applications that support the Model Context Protocol (feature-support matrix)** — https://modelcontextprotocol.io/clients
5. **Client support matrix for enterprise clients** — https://modelcontextprotocol.io/extensions/client-matrix
6. **`docs/clients.mdx` (open-source docs repo `modelcontextprotocol/docs`)** — https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx
7. **Client feature-support matrix (`docs/clients.mdx`, `modelcontextprotocol/modelcontextprotocol`)** — https://github.com/modelcontextprotocol/modelcontextprotocol/blob/ff960c9e/docs/clients.mdx

Video origins are listed in Section 4 (YouTube channels/publishers are cited with each entry).

---

## Suggested usage for the Monday onboarding

1. Read **Section 1 (Primer)** first for the mental model.
2. Watch **video 2** (19:35) for a paced overview, or **video 1** (1:44:12) if you want the deep-dive.
3. Review **Sections 2–3** to understand the architecture and which tools you can integrate with.
4. Explore **video 4** (10:01) for a quick SDK-level tour.
