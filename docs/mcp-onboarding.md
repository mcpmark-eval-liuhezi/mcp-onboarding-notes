# MCP Onboarding Notes

This note gathers the essentials of the **Model Context Protocol (MCP)** for new teammates: what it is, how the client-server architecture works, who supports it today, and where to read and watch more. Use it as a first stop before the official documentation and specification.

---

## 1. What MCP Is — A Plain-Language Primer

The Model Context Protocol (MCP) is an **open-source standard for connecting AI applications to external systems** — data sources (e.g. local files, databases), tools (e.g. search engines, calculators), and workflows (e.g. specialized prompts). Using MCP, AI applications like Claude or ChatGPT can connect to those systems and act on their behalf, enabling them to access key information and perform tasks.

A helpful analogy from the official documentation: **MCP is like a USB-C port for AI applications.** Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. Devices that use a common physical interface can plug into any device with that interface, and a single USB-C cable works across cell phones, laptops, and peripherals.

**Why it matters:**

- **Developers** — MCP reduces development time and complexity when building, or integrating with, an AI application or agent.
- **AI applications or agents** — MCP gives them access to an ecosystem of data sources, tools, and apps, which enhances their capabilities and improves the end-user experience.
- **End users** — MCP results in more capable AI applications or agents that can access user data and take actions on the user's behalf when necessary.

MCP replaces the classic N×M integration problem (a custom integration for every AI application combined with every tool or data source) with a single shared protocol. Build a server once, connect any MCP client — the interface is consistent, not fragmented.

### Sources

- Official introduction: <https://modelcontextprotocol.io/docs/getting-started/intro>
- Anthropic's original announcement (background for the primer): <https://www.anthropic.com/news/model-context-protocol>
- Specification overview: <https://modelcontextprotocol.io/specification/draft/server/index>

---

## 2. Architecture — Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**, coordinated by an MCP host application. Understanding these roles is the key to everything else in MCP.

### Participants

- **MCP Host** — the AI application (e.g. Claude Code, Claude Desktop, Visual Studio Code). It coordinates and manages one or multiple MCP clients.
- **MCP Client** — a component that maintains a connection to an MCP server and obtains context from it for the host to use. Each client maintains a dedicated connection with its corresponding server.
- **MCP Server** — a program that provides context to MCP clients.

> Example from the documentation: Visual Studio Code acts as an MCP host. When VS Code establishes a connection to an MCP server, such as the Sentry MCP server, the VS Code runtime instantiates an MCP client object that maintains the connection to the Sentry MCP server. When VS Code additionally connects to another server, such as the local filesystem server, the runtime instantiates a second client object.

**Transports:**

- **Stdio transport** — uses standard input/output streams for direct process communication between local processes on the same machine.
- **Streamable HTTP transport** — used for remote servers (e.g. the official Sentry MCP server, which runs on the Sentry platform).

> Note: a local MCP server using the STDIO transport typically serves a single client, while a remote server using the Streamable HTTP transport typically serves many clients. MCP focuses solely on the protocol for context exchange — it does not dictate how AI applications use LLMs or manage the provided context.

### Two layers

- **Data layer** — a JSON-RPC 2.0-based exchange protocol defining the message structure and semantics: capability and version discovery, plus core primitives such as tools, resources, prompts, and notifications.
- **Transport layer** — defines the communication mechanisms and channels enabling data exchange between clients and servers, including transport-specific connection establishment, message framing, and authorization.

### Server primitives: Tools, Resources, and Prompts

Servers expose functionality through three building blocks. The control hierarchy is a useful shortcut for onboarding — it tells you who decides whether a capability is shown or invoked:

| Primitive | Description | Who controls it | Protocol operations |
|-----------|-------------|-----------------|---------------------|
| **Tools** | Functions the LLM can actively call and decides, based on user requests, when to use. Schema-defined, validated with JSON Schema; may require user consent before execution. | **Model-controlled** | `tools/list` (discover tools), `tools/call` (execute) |
| **Resources** | Passive, read-only data sources that provide access to information as context for the model (e.g. file contents, database schemas, API documentation). Each resource has a unique URI and a MIME type; supports direct resources and resource templates with dynamic URIs. | **Application-controlled** | `resources/list`, `resources/templates/list`, `resources/read`, `resources/subscribe` |
| **Prompts** | Pre-built, parameterized instruction templates that tell the model to work with specific tools and resources; user-controlled and invoked explicitly. | **User-controlled** | `prompts/list`, `prompts/get` |

The spec's control hierarchy: **Prompts = user-controlled**, **Resources = application-controlled**, **Tools = model-controlled**.

### Getting started

- Build a server: <https://modelcontextprotocol.io/docs/getting-started/quickstart/server>
- MCP apps that run inside AI clients: <https://modelcontextprotocol.io/docs/getting-started/mcp-apps>

### Sources

- Architecture overview: <https://modelcontextprotocol.io/docs/concepts/architecture>
- Understanding MCP servers: <https://modelcontextprotocol.io/docs/2025-06-18/learn/server-concepts>
- MCP specification overview: <https://modelcontextprotocol.io/specification/draft/server/index>
- Data layer and transport details: see <https://modelcontextprotocol.io/docs/concepts/architecture>

---

## 3. Ecosystem — Supporting Clients Today

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, development tools like Visual Studio Code, Cursor, MCPJam, and many others all support MCP — so it is easy to build once and integrate everywhere. The exact feature set varies by client; some support resources, prompts, and tool calls fully, while others focus on tools.

### Feature-breadth snapshot

| Client | Resources | Prompts | Tools | Notes |
|--------|-----------|---------|-------|-------|
| Claude Desktop | ✅ | ✅ | ✅ | Full support for all MCP features |
| Continue | ✅ | ✅ | ✅ | Full support for all MCP features |
| fast-agent | ✅ | ✅ | ✅ | Full multimodal MCP support with end-to-end tests |
| Claude Code | ✅ | ❌ | ✅ | Supports prompts, tools, and roots |
| Cline | ✅ | ❌ | ✅ | Supports tools and resources |
| Roo Code | ✅ | ❌ | ✅ | Supports tools and resources |
| Zed | ✅ | ❌ | ✅ | Prompts appear as slash commands |
| Cursor | ❌ | ✅ | ✅ | Supports tools |
| VS Code GitHub Copilot | ❌ | ✅ | ✅ | One-click installation of servers, tools, roots, dynamic discovery, secure secret configuration |
| Windsurf Editor | ❌ | ✅ | ✅ | Supports tools with AI Flow for collaborative development |
| Roo Code | ✅ | ❌ | ✅ | Supports tools and resources |
| Sourcegraph Cody | ✅ | ❌ | ❌ | Supports resources through OpenCTX |
| Microsoft Copilot Studio | ❌ | ✅ | ✅ | Supports tools |
| ChatGPT | ❌ | ❌ | ❌ | MCP support shown in the extension matrix |
| Gemini CLI | ✅ | ✅ | ✅ | The MCP ecosystem includes AI assistants such as Claude and ChatGPT, plus development tools like Visual Studio Code, Cursor, and MCPJam |
| Microsoft 365 Copilot | ❌ | ✅ | ✅ | MCP support shown in the extension matrix |

> Note: MCP clients and servers vary in the features they implement, which determines depth of integration with MCP servers. For the full per-client feature matrix and for pairing MCP clients with pre-built servers, see the clients page. MCP inspectors exist as well — for example the MCP Inspector tool, a development aid for debugging servers and clients.

### Where to check support

- Official client list and feature support matrix: <https://modelcontextprotocol.io/docs/clients>
- Client support matrix (GitHub source of the official clients page): <https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx>
- Extension support matrix (which clients implement which official extensions): <https://modelcontextprotocol.io/extensions/client-matrix>

### Sources

- Official intro: <https://modelcontextprotocol.io/docs/getting-started/intro>
- Client list / feature matrix: <https://modelcontextprotocol.io/docs/clients>
- Client matrix (GitHub source): <https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx>
- Extension support matrix: <https://modelcontextprotocol.io/extensions/client-matrix>

---

## 4. Video Section — Introduction Walkthroughs

Four introduction walkthroughs are recommended for onboarding: a live demonstration video, several third-party sessions covering fundamentals and structure, and an extended workshop run for depth. Each entry is a link with its title, publisher, and duration where the listing shows one.

1. **[Power up your LLMs: Gemini CLI and Model Context Protocol (MCP)](https://www.youtube.com/watch?v=IYpt25GHB1U)** — published by **Google for Developers** (Lukе Schlangen, Luke Schlangen). A hands-on tutorial-style video: it explains what MCP is, builds a JavaScript MCP server with addition and subtraction tools, and runs that server from Gemini CLI via its `/mcp` command — noting that MCP uses OAuth for user authentication. Good for seeing server + client + OAuth in one walkthrough.

2. **[Building Agents with Model Context Protocol — Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)** — published by **AI Engineer**. This full workshop, from March 2025, is the reference induction presentation and is delivered live at an AI engineering conference. It argues for MCP from the integration problem (N×M bespoke connectors) and walks the three primitives and who controls each of them, plus sampling and composability (a server acting as a client). **Duration: 1 hour 44 minutes.**

3. **[Anthropic's New Model Context Protocol in 10 Minutes](https://www.youtube.com/watch?v=bpb45HVwwD8)** — published by **Developers Digest**. A plain-language deep dive of Anthropic's open-source announcement: the three core components (servers, clients, tools), the concept of a universal data/tool connector, plus the pre-built servers for platforms like Google Drive, Slack, Postgres, and Puppeteer and the early adopters (Block, Apollo, Zed, Replit, Sourcegraph). **Duration: 10:01.**

4. **[MCP Explained](https://clyep.io/series/mcp-explained/)** — published by **Clyep (Abhyuday Desai)**. An animated series of short videos, one idea each, covering the model context protocol from first principles for developers hearing about MCP for the first time: what is MCP and why we need it, how host/client/server work together, and tools, resources, and prompts as server capabilities.

### Where to watch more

- **[All You Need to Know About Model Context Protocol (MCP)](https://www.youtube.com/watch?v=1uG8Uvm_SA8)** — Krish Naik. A fundamentals-style overview of the protocol (USB-C analogy, the client–server connection, tools) on the Krįsh platform — suitable as a 25-minute second read.
- **[Introduction to Model Context Protocol (MCP)](https://www.youtube.com/watch?v=0xFndfMNxVo)** — AI SDK. An introductory walkthrough of what MCP is and how it works for building applications with AI.

### Sources

- Official introduction: <https://modelcontextprotocol.io/docs/getting-started/intro>
- Architecture overview: <https://modelcontextprotocol.io/docs/concepts/architecture>
- Understanding MCP servers: <https://modelcontextprotocol.io/docs/2025-06-18/learn/server-concepts>
- Client list / support matrix: <https://modelcontextprotocol.io/docs/clients>
- Feature matrix (GitHub source): <https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx>
- Extension support matrix: <https://modelcontextprotocol.io/extensions/client-matrix>
- MCP documentation index: <https://modelcontextprotocol.io/llms.txt>
