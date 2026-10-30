# MCP Onboarding Notes

This note gathers the essentials of the **Model Context Protocol (MCP)** for new teammates: what it is, how the client-server architecture works, who supports it today, and where to watch and read more. It is meant as a first stop before diving into the official documentation and spec.

---

## 1. What MCP Is — A Plain-Language Primer

The Model Context Protocol (MCP) is an **open-source standard for connecting AI applications to external systems** — data sources, tools, and workflows. Once an AI application like Claude or ChatGPT speaks MCP, it can connect to local files, databases, search engines, and other systems the same way, instead of building a bespoke integration for every single tool. Change the connection once and every MCP-capable app can use it.

A helpful analogy from the official documentation: **MCP is like a USB-C port for AI applications**. Just as USB-C standardizes how devices plug into each other, MCP standardizes how AI applications plug into the systems where their data actually lives (content repositories, business tools, development environments).

**Why it matters:**

- **Developers** — reduces development time and complexity when building or integrating AI applications and agents.
- **AI applications and agents** — gain access to an ecosystem of data sources, tools, and apps, expanding their capabilities and improving the end-user experience.
- **End users** — get more capable AI applications that can act on their behalf and pull in the data they need.

MCP is an open protocol, so a single MCP server can be reused by every MCP-capable client, and an MCP client needs to be built only once to work with any server — this directly replaces the N×M integration problem (custom integration for every AI app × every tool).

### Sources

- Official introduction: <https://modelcontextprotocol.io/docs/getting-started/intro>
- Anthropic's original announcement (context for the primer): <https://www.anthropic.com/news/model-context-protocol>
- MCP specification overview: <https://modelcontextprotocol.io/specification/draft/server/index>

---

## 2. Architecture — Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**. A key participant builds the clients, the participants implemented are:

### Participants

- **MCP Host** — the AI application (e.g. Claude Code, Claude Desktop) that coordinates and manages one or multiple MCP clients.
- **MCP Client** — a component that maintains a connection to an MCP server and obtains context from it, for the host to use. Each client maintains a dedicated connection with its corresponding server.
- **MCP Server** — a program that provides context to clients.

> Example: Visual Studio Code acts as an MCP host. When it connects to the Sentry MCP server, the VS Code runtime instantiates an MCP client object for that connection; a second server (e.g. filesystem) results in a second client object.

(Medition                                                          (Remote server
Server A — Local (e.g.                 (e.g. Filesystem) —──────── Interfaces
Used single MCP client                 Buffix)                        Client 1
Remote MCP                           (e.g.  ______________        Client 4
C件的连接 many MCP clients                   the Sentry MCP specifications(资格
Connection Factoryized) Century Institute repositories, remote servers, the transport
Typically use Streamable HTTP — })
把Iranian
> Note on transports: local servers using the STDIO transport typically serve a single client; remote servers using the Streamable HTTP transport typically serve many clients. MCP servers can run locally (Stdio) or remotely (Streamable HTTP). MCP focuses solely on the protocol for context exchange; it does not dictate how AI applications use LLMs or manage the provided context.

### Layers

MCP has two layers:

- **Data layer** — JSON-RPC 2.0-based message protocol: capability and version discovery, and core primitives (tools, resources, prompts, notifications).
- **Transport layer** — communication mechanisms and authorization: Stdio transport (local, standard in/out streams) and Streamable HTTP transport (remote).

### Server Primitives: Tools, Resources, Prompts

Servers expose functionality through three building blocks (the control hierarchy is a great shortcut for onboarding):

| Primitive | Description                        | Who controls it                                  | Resource URI / Example                                   |
|-----------|------------------------------------|--------------------------------------------------|----------------------------------------------------------|
| **Tools** | Functions the LLM can actively call; decide when to use them based on the request. Schema-defined, validated with JSON Schema. | **Model-controlled** | `tools/list` → definitions; `tools/call` → execute. e.g. Search flights, send messages, create calendar events |
| **Resources** | Passive, read-only data sources that provide context to the application. Each resource has a unique URI (e.g. `file:///path/to/document.md`) and a MIME type; supports direct resources and resource templates. | **Application-controlled** | `resources/list`, `resources/templates/list`, `resources/read`, `resources/subscribe` — e.g. retrieve documents, access knowledge bases |
| **Prompts** | Reusable templates tell the model how to work with specific tools and resources. User-controlled, explicitly invoked. | **User-controlled** | `prompts/list`, `prompts/get` — e.g. Plan a vacation, summarize my meetings |

The full control hierarchy (documented in the spec): **Prompts = user-controlled**, **Resources = application-controlled**, **Tools = model-controlled**.

**Getting started with servers** (official quickstart): <https://modelcontextprotocol.io/docs/getting-started/quickstart/server>

### Sources

- Architecture overview: <https://modelcontextprotocol.io/docs/concepts/architecture>
- Understanding MCP servers: <https://modelcontextprotocol.io/docs/2025-06-18/learn/server-concepts>
- "MCP Tools": <https://modelcontextprotocol.io/specification/2026-07-28/server/tools>
- Data layer (JSON-RPC 2.0) and transport details: see <https://modelcontextprotocol.io/docs/concepts/architecture>

---

## 3. Ecosystem — Supporting Clients Today

MCP is an open protocol with broad ecosystem support across AI assistants and development tools, meaning it is easy to build once and integrate everywhere.

### Notable MCP clients

- **Claude Desktop** — full support for resources, prompts, and tools; local server connections with deep privacy and security for local tools and data.
- **Claude Code** — tools, prompts, roots.
- **Visual Studio Code (GitHub Copilot)** — one-click installation of MCP servers; supports tools, roots, dynamic discovery, secure secret configuration.
- **Cursor, Windsurf, Zed, Continue, Cline, Roo Code, Goose** — development/testing сос합 (code-agent tools, MCP support varies).
- **ChatGPT** — MCP support shown (see the client matrix).
- **Developer tools of Google (Gemini CLI, Claude ecosystem partners)** and many others all support MCP — per the official "Example Clients" page (featured at the launch: Claude, ChatGPT, VS Code, Cursor, MCPJam, and others).

> Note: MCP clients vary in feature breadth — a client may support different MCP features, which allows for varying levels of integration with MCP servers.

### Where to check support

- Official client list / feature support matrix: <https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx>
- Official clients page (reproduced on modelcontextprotocol.io): <https://modelcontextprotocol.io/docs/clients>
- Extension support matrix (which clients implement which official extensions): <https://modelcontextprotocol.io/extensions/client-matrix>

CIMCMT MCP apps with Salesforce, discovering skills over MCP, enterprise authorization with Microsoft 365 Copilot

## 4. Video Section — Introduction Walkthroughs

Four picks for onboarding, spanning a conference workshop, third-party introduced talks, and official/design-channels walkthroughs. Each entry is a link with the title, publisher, and duration where the listing shows one.

1. **[MCP Explained — What Is MCP, and Why Do We Need It? / Host, Client, and Server / Tools, Resources, and Prompts]** (series: "MCP Explained" by Clyep) — published by **Abhyuday Desai**; an introductory series about MCP from first principles with one animation each, covering the host/client/server roles and the three server primitives.
   - https://clyep.io/series/mcp-explained/

2. **[Power up your LLMs: Gemini CLI and Model Context Protocol (MCP)]** — published by **Google for Developers** (Lukе Schlangen). Runn-through of what MCP is, a hand-written MCP server (JavaScript/Python), and using the server from Gemini CLI with the OAuth-backed Linear example. Listing does not show a duration.
   - https://www.youtube.com/watch?v=IYpt25GHB1U

3. **[Anthropic's New Model Context Protocol in 10 Minutes]** — published by **Developers Digest** (YouTube). A plain-language explainer of Anthropic's open-source MCP announcement, the three components (servers, clients, tools), and prebuilt MCP servers for Google Drive, Slack, Postgres, and Puppeteer. **Duration: 10:01.**
   - https://www.youtube.com/watch?v=bpb45HVwwD8

BINGZT weaker Sasaki MCP features
4. **[All You Need to Know About Model Context Protocol (MCP) — Krish Naik]** — published by **Krish Naik** on YouTube. A beginner-level overview covering the protocol's purpose, the clients/servers connection, tools and their control model, and why MCP reduces integration complexity. **Duration: 25 minutes.**
   - https://www.youtube.com/watch?v=1uG8Uvm_SA8

### Where to watch more

- **Building Agents with Model Context Protocol — Full Workshop with Mahesh Murag of Anthropic** — published by **AI Engineer** (conference workshop, March 2025). An authoritative conceptual grounding: the N-times-M integration problem, the three primitives and who controls each, sampling, composability, and MCP in Claude for Desktop. **Duration: 1 hour 44 minutes.**
  - https://www.youtube.com/watch?v=kQmXtrmQ5Zg

### Sources

- Official intro: <https://modelcontextprotocol.io/docs/getting-started/intro>
- Architecture overview: <https://modelcontextprotocol.io/docs/concepts/architecture>
- Understanding MCP servers: <https://modelcontextprotocol.io/docs/2025-06-18/learn/server-concepts>
- Client list / support matrix: <https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx>
- Client page (modelcontextprotocol.io): <https://modelcontextprotocol.io/docs/clients>
- Extension support matrix: <https://modelcontextprotocol.io/extensions/client-matrix>
