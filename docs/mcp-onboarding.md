# MCP Onboarding Notes

A plain-language guide to the Model Context Protocol (MCP) for new teammates.

## 1. What MCP Is — A Plain-Language Primer

MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts)—enabling them to access key information and perform tasks.

The official documentation captures it with a simple analogy: think of MCP like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. Before MCP, every AI app had to build custom integrations for every data source and tool—N×M integration complexity. MCP flattens that into a single protocol.

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, development tools like Visual Studio Code, Cursor, MCPJam, and many others all support MCP — making it easy to build once and integrate everywhere.

## 2. Architecture: Clients, Servers, Tools, and Resources

### Participants

MCP follows a client-server architecture. An **MCP host** — an AI application like Claude Code or Claude Desktop — establishes connections to one or more MCP servers. The host creates **one MCP client for each MCP server**, and each client maintains a dedicated connection with its corresponding server.

- **MCP Host**: the AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client**: a component that maintains a connection to an MCP server and obtains context from that server for the host to use.
- **MCP Server**: a program that provides context to MCP clients.

For example, Visual Studio Code acts as an MCP host. When it connects to the Sentry MCP server, the runtime instantiates an MCP client object for that connection; when it connects to a local filesystem server, it instantiates a separate client for that connection. Local MCP servers using the STDIO transport typically serve a single client, while remote servers using the Streamable HTTP transport typically serve many clients.

### Core Primitives Served by Servers

Servers can expose three core primitives, plus tools:

- **Tools**: Servers expose tools that can be invoked by language models to let models interact with external systems, such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are model-controlled—the language model can discover and invoke them automatically based on context; the protocol itself does not mandate a specific user interaction model. A human should always remain in the loop with the ability to deny tool invocations.
- **Resources**: Servers expose resources to share data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are application-driven; host applications determine how to incorporate context based on their needs (e.g., exposing resources through UI elements, allowing search/filter, or automatic inclusion based on heuristics).
- **Prompts**: Servers expose prompt templates that provide structured messages and instructions for interacting with language models. Clients can discover available prompts, retrieve their contents, and provide arguments to customize them. Prompts are user-controlled and typically triggered through user-initiated commands in the UI, such as slash commands.

The protocol itself consists of two layers:

- **Data layer**: defines the JSON-RPC-based protocol for client-server communication, including capability and version discovery, and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer**: defines how messages are transmitted (e.g., STDIO for local servers, Streamable HTTP for remote servers).

MCP focuses solely on the protocol for context exchange—it does not dictate how AI applications use LLMs or manage the provided context.

## 3. Ecosystem: Supporting Clients

MCP has broad ecosystem support. Per the official client support documentation, notable clients include:

- **Claude Desktop App** — full support for all MCP features (resources, prompts, tools).
- **Claude Code** — supports prompts, tools, and roots.
- **Cursor** — supports tools (in Cursor Composer).
- **VS Code (GitHub Copilot)** — supports tools, roots, dynamic discovery, secure secret configuration, and one-click installation; integrates MCP with GitHub Copilot through agent mode.
- **Windsurf Editor** — supports tools with AI Flow for collaborative development.
- **Zed** — prompts appear as slash commands.
- **Cline**, **Continue**, **Roo Code** — support tools and/or resources.
- **Microsoft Copilot Studio** — supports tools.
- **Goose** (launched by Block) — supports tools.
- **5ire**, **BeeAI Framework**, **Genkit**, **GenAIScript**, **LibreChat**, **Sourcegraph Cody**, **Superinterface**, **TheiaAI/TheiaIDE**, **oervet**, **SpinAI**, **OpenSumi**, **Daydreams Agents**, **Apify MCP Tester** — additional clients with varying feature support.

The MCP client ecosystem also includes official SDKs for multiple programming languages, MCP development tools such as the MCP Inspector, and reference server implementations. On the Chatbot side, ChatGPT custom MCP servers (developer mode, deep research) also support MCP connections.

## 4. Video Walkthroughs

Picked introductions to help new teammates get up to speed:

- **[The Model Context Protocol (MCP)](https://www.youtube.com/watch?v=CQywdSdi5iA)** — published by Anthropic, duration 19:35. A panel discussion with Theo Chu, David Soria Parra, and Alex Albert diving into MCP, the standard changing how AI applications connect with external data and tools.
- **[MCP In 26 Minutes (Model Context Protocol)](https://www.youtube.com/watch?v=kOhLoixrJXo)** — published by Tina Huang, duration 26:00. A hands-on tutorial covering MCP fundamentals, using MCP servers, and building your own servers both with code and no-code.
- **[5 things you need to know about MCP](https://www.youtube.com/watch?v=une3Ba19EQM)** — published by Visual Studio Code, duration 01:20. A short overview covering host/client/server architecture and the tools, resources, and prompts servers expose.
- **[Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)** — published by AI Engineer, duration not listed. A full workshop from an Anthropic applied AI team member on the philosophy behind MCP and adoption patterns for agents.

## Sources

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/docs/getting-started/intro) — official MCP documentation, "What is MCP" overview.
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture) — official MCP documentation, client-server architecture, participants, and layers.
- [Tools](https://modelcontextprotocol.io/docs/concepts/tools) — official MCP documentation, server tools primitive.
- [Resources](https://modelcontextprotocol.io/docs/concepts/resources) — official MCP documentation, server resources primitive.
- [Prompts](https://modelcontextprotocol.io/docs/concepts/prompts) — official MCP documentation, server prompts primitive.
- [clients.mdx](https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx) — official MCP client support matrix.
- [The Model Context Protocol (MCP)](https://www.youtube.com/watch?v=CQywdSdi5iA) — Anthropic video used in the primer/architecture notes.
- [MCP In 26 Minutes (Model Context Protocol)](https://www.youtube.com/watch?v=kOhLoixrJXo) — Tina Huang video used in the video walkthroughs section.
- [5 things you need to know about MCP](https://www.youtube.com/watch?v=une3Ba19EQM) — Visual Studio Code video used in the video walkthroughs section.
