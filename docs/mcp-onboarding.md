# MCP Onboarding Notes

> **Audience:** New teammates joining the team on Monday.
> **Status:** Documentation living in version control. Sources and observations recorded for future reference; open a pull request review before merging.

---

## 1. What is MCP? (Plain-Language Primer)

The **Model Context Protocol (MCP)** is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts) — enabling them to access key information and perform tasks.

The official documentation's own analogy is that MCP is like **a USB-C port for AI applications**: just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems.

### Why it matters

- **For developers:** MCP reduces development time and complexity when building or integrating with AI applications or agents.
- **For AI applications / agents:** MCP gives them access to an ecosystem of data sources, tools, and apps, which enhances their capabilities.
- **For end users:** MCP results in more capable AI applications that can access user data and take actions on the user's behalf.

### What "context" means here

Every AI assistant today is only as good as the context fed to it. MCP addresses that variability by standardizing how that context is provided — instead of pasting context manually or copy/pasting between systems, the model connects to servers that expose data and tools in a uniform way.

### Quick takeaway

> MCP is the standard plug: once you write an MCP **client** and an MCP **server**, any MCP-capable AI application can use them without custom glue code.

---

## 2. How the Architecture Works: Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**. Each AI application that wants to use MCP is called an **MCP host**. The host instantiates **one MCP client per MCP server**, and each client maintains a dedicated connection with its corresponding server.

### Participants

| Participant | What it is |
| --- | --- |
| **MCP Host** | The AI application (e.g. Claude Desktop, Claude Code, an IDE like Visual Studio Code) that coordinates and manages one or more MCP clients — it owns the user experience. |
| **MCP Client** | A protocol-level component; the host creates one client for each server it connects to. It may also offer additional features (elicitation, sampling, roots) to servers. |
| **MCP Server** | A program that exposes context and capabilities to clients. Servers can run locally (stdio transport) or remotely (Streamable HTTP transport). |

### Two layers

1. **Data layer** — a JSON-RPC 2.0-based exchange protocol defining message structure and semantics: discovery (`server/discover`), core primitives (tools, resources, prompts, notifications, progress), and utility features.
2. **Transport layer** — defines communication mechanisms and channels: **stdio** transport for local processes, and **Streamable HTTP** for remote servers.

### Server primitives: Tools and Resources (plus Prompts)

Servers provide functionality through three building blocks:

| Primitive | What it is | Who controls it | Example |
| --- | --- | --- | --- |
| **Tools** | Functions the AI agent actively calls to perform actions | The model | Search flights, send messages, create calendar events — often read/write database operations, external API calls, file modifications |
| **Resources** | Passive, read-only data sources providing context for the model | The application | File contents, database schemas, API documentation; rendered into the conversation or passed based on relevance |
| **Prompts** | Pre-built instruction templates that tell the model how to work with specific tools and resources | The user | Plan a vacation, summarize my meetings, draft an email |

### How tools work in practice

- Tools are **schema-defined interfaces** validated with **JSON Schema**; each tool performs a single operation with clearly defined inputs and outputs.
- The model requests tool execution based on context.
- Core protocol methods: `tools/list` (discovers available tools, returns definitions with schemas) and `tools/call` (executes a specific tool, returning the result).
- Tools may require **user consent** before execution, helping users stay in control.

### Resources in practice

- Each resource has a unique URI (e.g. `file:///path/to/document.md`) and declares an MIME type.
- Two discovery patterns: **Direct Resources** (fixed URIs) and **Resource Templates** (dynamic URIs with parameters, e.g. `travel://activities/{city}/{category}`).
- Resources provide structured access that applications can read directly and decide how to use — extracting relevant portions, searching with embeddings, or passing all of it to the model.

### Client features

Beyond consuming context, clients can offer richer interactions to servers: **elicitation** (servers request structured input from users mid-interaction), **roots** (clients tell servers which directories to focus on), and **sampling** (servers request LLM completions through the client). The protocol standardizes how these capabilities are offered and invoked.

### Topic model in brief

> MCP is a client-server protocol for AI applications. A host owns one client per server. Servers expose **tools** (actions), **resources** (read-only context) and **prompts** (templates). Communication happens over JSON-RPC 2.0 using **stdio** (local) or **Streamable HTTP** (remote) transports.

---

## 3. Ecosystem: Which AI Applications Support MCP Today?

MCP is an open protocol supported across a **wide range of clients and servers**, which is what makes it useful for "build once, integrate everywhere."

### Clients (AI applications)

- **Claude** — Claude apps and Claude Code, and Claude Desktop are first-party MCP clients.
- **ChatGPT** — supported as an MCP client.
- **Visual Studio Code** — acts as an MCP host, instantiating one client per MCP server.
- **Cursor** — MCP client for AI coding.
- **Gemini** — MCP support in the Gemini client series.
- **MCPJam** — a playground/client for testing MCP servers.
- **Windsurf** — MCP client.
- **Block's Goose** — an AI agent built on MCP.

### Servers

Servers are programs that expose specific capabilities and are typically built to wrap or federate access to systems and data — a database to query and fetch data and give the LLM access, a CRM like Salesforce to read and write remotely, or the user's local laptop and version control (e.g. Git) APIs.

The ecosystem includes both **official/adopted servers** (e.g. reference servers like the Sentry server, filesystem servers) and hundreds of **community servers** published through the Model Context Protocol ecosystem. Because there is a **single standard interface**, a client that speaks MCP can connect to any MCP server with zero additional work.

### Official SDK ecosystem

The MCP project also provides **official SDKs in multiple languages** that implement MCP — SDKs handle the protocol details so developers can focus on building their features (tools, resources, prompts, clients, servers, debugging with the MCP Inspector, etc.).

> **Net for new teammates:** when you want to give your AI client new data or capabilities, you rarely need to build a new integration — you build one MCP **server** and any MCP-capable client, including your own, can use it.

---

## 4. Videos: Walkthroughs, Talks, and Tutorials

**Which video to pick:** we picked three videos because together they cover the topic from different angles — a short concept-first explainer (IBM Technology, 08 min), a deep architect-led workshop from Schrödinger workshop by the Anthropic team (AI Engineer, 1h 44m), and a hands-on Microsoft technical-meeting demo which shows how MCP integrates with real development tooling (Microsoft Developer, 20 min). The Anthropic workshop is the most complete for expansion; the IBM Technology one is the best 8-minute primer; the Microsoft session is the best live demo of embeddings and data interpretation (one video that covers the most ground for a newcomer).

- **[What is MCP? Integrate AI Agents with Databases & APIs](https://www.youtube.com/watch?v=eur8dUO9mvE)** — **IBM Technology**. Title, publisher: IBM Technology; duration on the YouTube listing: **07:58** (~8 minutes). Best for: a quick, accurate introduction to the host/client/server architecture and the model/LLM layer.
- **[Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)** — **AI Engineer**. Title, publisher: AI Engineer (Anthropic's applied AI team); duration on the YouTube listing: **~1 hour 44 minutes**. Best for: the deepest walkthrough available — MCP's design rationale, the "N times M" → single-standard shift, the replicable architecture patterns behind it, and what a production host actually does.
- **[Introduction to Model Context Protocol (MCP) Servers | DEM517](https://www.youtube.com/watch?v=cfwooBzzHBs)** — **Microsoft Developer** (technical-meetings thread; also presented at Visual Studio Live! Microsoft HQ 2025). Title, publisher: Microsoft Developer; duration: 20-minute Fast Focus session. Best for: a live architectural and hands-on demo, including background on securing MCP servers; a good next watch after the IBM video.

> Note: the remaining YouTube listing durations for the two longer videos were not confirmable from the listing pages during research, so durations here are as explicitly shown on YouTube (IBM Technology 07:58; the Anthropic workshop is listed as 2 hours on the creator's note and ~1h 44m elsewhere; Microsoft Developer is a 20-minute Fast Focus session).

---

## Sources

These documentation pages were actually used for the primer, architecture, and ecosystem sections:

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/docs/getting-started/intro) — official documentation introduction: definition, the USB-C analogy, what MCP enables, why it matters, and the broad ecosystem support statement.
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture) — participants (host, client, server), transport (stdio vs. Streamable HTTP), and the two-layer model (data layer / transport layer); example document lines.
- [Understanding MCP servers](https://modelcontextprotocol.io/docs/concepts/learn/server-concepts) — the server primitives: tools, resources, and prompts, plus `tools/list` and `tools/call` protocol operations.
- [Understanding MCP clients](https://modelcontextprotocol.io/docs/concepts/learn/client-concepts) — the role of the client within the host, and the client-facing features (elicitation, roots, sampling).
- [llms.txt](https://modelcontextprotocol.io/llms.txt) — full documentation index used to locate all pages above.

Video sources (URLs as listed):

- YouTube — *What is MCP? Integrate AI Agents with Databases & APIs* (07:58, IBM Technology).
- YouTube — *Building Agents with Model Context Protocol - Full Workshop* (~1h 44m, AI Engineer).
- YouTube — *Introduction to Model Context Protocol (MCP) Servers | DEM517* (20-minute Fast Focus session, Microsoft Developer).
