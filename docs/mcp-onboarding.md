# Model Context Protocol (MCP) — Onboarding Notes

Welcome to the team! Two questions dominate week one: *what is MCP*, and *why does everyone keep talking about it?* This note answers both in one place. Read the primer and the architecture sections first; skim the videos when you get a chance. Everything below is sourced from the official MCP documentation, and a list of sources is at the end.

---

## 1. What is MCP? (Plain-language primer)

The **Model Context Protocol (MCP)** is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications like Claude or ChatGPT can connect to **data sources** (local files, databases), **tools** (search engines, calculators, APIs), and **workflows** (specialized prompts), so the model can both access key information and perform tasks.

The official docs use an analogy worth memorizing: **MCP is like a USB-C port for AI applications**. Just as USB-C gives you one standardized way to plug devices into a computer, MCP gives you one standardized way to plug external systems into an AI application.

### The problem it solves

Before MCP, wiring an AI model up to the outside world meant writing custom integration code for every (model × system) pair — the "N × M problem". Every service exposes a differently-shaped API, and every client had to write its own glue code for each one. MCP replaces that with a single protocol: implement once on the client side, expose once on the server side, and everything speaks the same language.

### Why it matters to you, concretely

- **Developers** — MCP reduces development time and complexity when building, or integrating with, an AI application or agent; you write the integration once rather than once per client.
- **AI applications and agents** — MCP gives them access to an ecosystem of data sources, tools, and apps, which enhances their capabilities and improves the end-user experience.
- **End users** — MCP results in more capable AI applications or agents that can access user data and take actions on the user's behalf when necessary (e.g. reading a calendar, searching a database, generating an app from a design file).

### Origins

MCP was introduced by **Anthropic** as an open standard in November 2024. The official documentation describes it as an "open-source standard", and Anthropic's own video below describes it as "the standard way AI connects to your data, tools, and context" — i.e. the community-backed successor to ad-hoc approaches such as custom API integrations and (at the time) plugin-style extensions. Because it is an open protocol, anyone can build a client or a server; Anthropic simply originated it.

### Terms you'll hear in the hallways

| Term | What it means |
| --- | --- |
| **MCP host** | The AI application the user actually interacts with (Claude Desktop, an IDE, your agent app). |
| **MCP client** | A protocol-level component inside the host that maintains one connection to one server. |
| **MCP server** | The program that exposes data, tools, or prompts to the host. |
| **Tools / Resources / Prompts** | The three building blocks a server can expose (Section 2). |

---

## 2. How it works: architecture (clients, servers, tools, resources)

MCP follows a **client–server architecture**. An **MCP host** — an AI application like Claude Code or Claude Desktop — establishes connections to one or more **MCP servers**. The host does this by creating **one MCP client for each MCP server**, and each client maintains a dedicated connection with its corresponding server.

A concrete example from the docs: **Visual Studio Code acts as an MCP host**. When VS Code connects to the Sentry MCP server, the VS Code runtime instantiates an MCP client object to maintain that connection. When VS Code then connects to a local filesystem server, the runtime instantiates a second MCP client object for it. One host, multiple clients, one connection per server.

### Participants

| Participant | What it does |
| --- | --- |
| **MCP host** | The AI application that coordinates and manages one or multiple MCP clients (e.g. Claude Desktop, VS Code). |
| **MCP client** | A component that maintains a connection to an MCP server and obtains context from it for the host to use. |
| **MCP server** | A program that provides context and capabilities to MCP clients. |

### Local vs. remote servers

An MCP server can run **locally** or **remotely** — "local" and "remote" describe *where the server runs*, not a different kind of server:

- **Local server** — e.g. Claude Desktop launching a filesystem server over the **stdio** transport. Stdio uses standard input/output streams for direct process communication between local processes on the same machine, with no network overhead. A server using stdio typically serves a single client.
- **Remote server** — e.g. the official Sentry MCP server, which runs on the Sentry platform and uses the **Streamable HTTP** transport. This transport uses HTTP POST for client-to-server messages with optional Server-Sent Events for streaming, and enables remote server communication with standard HTTP authentication (bearer tokens, API keys, custom headers). MCP recommends using **OAuth** to obtain authentication tokens.

### Two layers

| Layer | What it defines |
| --- | --- |
| **Data layer** | The JSON-RPC 2.0-based protocol for client–server communication: capability and version discovery, and the core primitives (tools, resources, prompts) plus notifications and progress tracking. |
| **Transport layer** | The communication channels and authentication between clients and servers: how connections are established, how messages are framed, and how communication is secured. |

The data layer is the inner layer; the transport layer is the outer layer. On top of both sits the protocol: MCP uses **JSON-RPC 2.0** as its underlying RPC protocol, so client and servers send requests to each other and respond accordingly. Notifications are used when no response is required.

### The three core primitives a server exposes

The server-concepts documentation defines three building blocks a server can offer:

| Feature | What it is | Who controls it | Examples |
| --- | --- | --- | --- |
| **Tools** | Executable functions the AI application can invoke to perform actions (file operations, API calls, database queries, computations) | **Model-controlled** — the LLM discovers and invokes them automatically | Search flights, send messages, create calendar events |
| **Resources** | Data sources that provide context to the AI application (read-only) — each identified by a **URI** (e.g. `file:///path/to/document.md`) | **Application-driven** — the host app decides how to surface them | File contents, database schemas, API documentation, calendars |
| **Prompts** | Pre-built instruction templates that tell the model how to work with specific tools and resources | **User-controlled** — explicitly selected by the user | Plan a vacation, summarize meetings, draft an email |

**The distinction that trips people up:** tools are *model-controlled* (the LLM decides when and how to call them), while resources are *application-driven* (the host decides how to include them — tree/list UI, search, or auto-include). The protocol mandates no specific UI pattern, but there should always be a **human in the loop**: clients should show which tools are exposed, indicate when tools run, and prompt for confirmation on consequential operations.

### How a tool call flows end to end

1. The client sends `tools/list` to the server and receives the available tools, each with its schema.
2. The LLM selects a tool and fills in its arguments.
3. The client sends `tools/call` with those arguments.
4. The server executes and returns the result (e.g. weather text) back through the client to the model.

Resources work analogously with `resources/list` / `resources/read`, each resource addressed by URI, with optional `subscriptions/listen` so the host can be notified when a resource changes.

---

## 3. The ecosystem: which AI applications support MCP today

MCP is an **open protocol supported across a wide range of clients and servers**. The official documentation states that AI assistants like **Claude** and **ChatGPT**, and development tools like **Visual Studio Code**, **Cursor**, and **MCPJam**, all support MCP — so you can build once and integrate everywhere.

The MCP project comprises four parts, all part of the open protocol:

| Project | What it is |
| --- | --- |
| **MCP Specification** | The specification that outlines the implementation requirements for clients and servers. |
| **MCP SDKs** | SDKs for different programming languages that implement MCP, so you can build servers and clients in your language of choice. |
| **MCP Development Tools** | Tools for developing MCP servers and clients, including the **MCP Inspector** for interactive testing and debugging. |
| **MCP Reference Server Implementations** | Reference implementations of MCP servers. |

Because the project is open, the ecosystem is broad. In practice the support landscape groups like this:

| Category | Clients |
| --- | --- |
| **Anthropic's own** | Claude Desktop, Claude Code, Claude.ai — the protocol's originators, with the deepest and earliest support. |
| **AI chat assistants** | ChatGPT (via Developer Mode / Apps), Gemini, Perplexity, LibreChat, Cherry Studio. |
| **Code editors & IDEs** | Visual Studio Code (GitHub Copilot), Cursor, Windsurf, Zed, Cline, Continue. |
| **Agent & CLI tools** | Codex CLI, Gemini CLI, Goose, MCPJam, and the broader CLI ecosystem. |

**For onboarding, the practical takeaway:** whichever client you reach for on day one, there is a good chance it already supports MCP — so you can treat an MCP server you build as a portable asset instead of a one-off for a single tool.

---

## 4. Video walkthroughs

These are the introduction videos I'd pick for a new joiner. Each is linked with its title, publisher, and duration where the listing shows one.

1. **[How Model Context Protocol (MCP) actually works](https://www.youtube.com/watch?v=cGuyrANVi4A)** — **Google Cloud Tech** — 7:58. A clear, step-by-step explanation of what MCP is, why it exists, and how it compares to traditional APIs — good first watch.
2. **[The Model Context Protocol (MCP)](https://www.youtube.com/watch?v=CQywdSdi5iA)** — **Anthropic** — 19:35. A panel with Alex Chu, David Soria Parra, and Alex Albert from Anthropic, diving deep into what MCP is and what's next, including the three core primitives (tools, resources, prompts).
3. **[Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w)** — **Shaw Hin Talabi** — 19:49. A practitioner's walkthrough in a short video series on AI agents; this installment covers MCP in depth.
4. **[Model Context Protocol Clearly Explained | MCP Beyond the Hype](https://www.youtube.com/watch?v=tzrwxLNHtRY)** — **15:04**. A simple explanation of what MCP actually is and how it eases the connection problem, with no hype.
5. **[MCP Complete Explanation](https://www.youtube.com/watch?v=_fzpnqt39jQ)** — **Aishwarya Srinivasan** — 15:22. A practitioner-level deep dive: how MCP works under the hood, how it compares to function calling vs. APIs vs. plugins, and where MCP servers plug in — ~15 minutes of structured explanation in one sitting.
6. **[Model Context Protocol (MCP) Explained in 90 Seconds](https://www.youtube.com/watch?v=mMWWU4PambM)** — **1:51**. The quickest possible primer for the person who just needs the one-liner.

**Recommended order:** watch #6 if you need the 30-second version, then #1, then #2 for a vendor-native deep dive, then #5 if you want the full picture.

---

## Sources

Documentation pages used for this note (all retrieved from `modelcontextprotocol.io`):

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/introduction) — official introduction, including the USB-C analogy and the supported clients list.
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture) — participants, layers, transports, and server primitives.
- [Understanding MCP servers](https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts) — tools, resources, and prompts as server features.
- [Understanding MCP clients](https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts) — hosts, clients, and client-side capabilities.
- [SDKs](https://modelcontextprotocol.io/docs/2026-07-28/sdk) — the official language SDKs.
- [Example servers](https://modelcontextprotocol.io/examples) — reference server implementations.
- [Get the complete documentation index](https://modelcontextprotocol.io/llms.txt) — index of all docs pages.

Video pages:

- [How Model Context Protocol (MCP) actually works — Google Cloud Tech](https://www.youtube.com/watch?v=cGuyrANVi4A) (7:58)
- [The Model Context Protocol (MCP) — Anthropic](https://www.youtube.com/watch?v=CQywdSdi5iA) (19:35)
- [Model Context Protocol (MCP) Explained in 20 Minutes — Shaw Hin Talabi](https://www.youtube.com/watch?v=N3vHJcHBS-w) (19:49)
- [Model Context Protocol Clearly Explained | MCP Beyond the Hype](https://www.youtube.com/watch?v=tzrwxLNHtRY) (15:04)
- [MCP Complete Explanation — Aishwarya Srinivasan](https://www.youtube.com/watch?v=_fzpnqt39jQ) (15:22)
- [Model Context Protocol (MCP) Explained in 90 Seconds](https://www.youtube.com/watch?v=mMWWU4PambM) (1:51)

---

*This document lives at `docs/mcp-onboarding.md` in the `onboarding/mcp-notes` branch and targets `main`.*