# MCP Onboarding Notes

This note gives new teammates a plain-language introduction to the Model Context Protocol (MCP): what it is, how its client/server architecture works, which AI applications support it today, and a short list of videos for further learning.

## 1. What MCP Is (Primer)

MCP (Model Context Protocol) is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts) — enabling them to access key information and perform tasks.

A helpful mental model: **MCP is like a USB-C port for AI applications**. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. Instead of building a custom integration for every model-tool pairing, you expose your data and tools once as an MCP server, and any MCP-compatible client can use them.

Why it matters:
- **Developers**: MCP reduces development time and complexity when building or integrating with AI applications or agents.
- **AI applications/agents**: MCP gives them access to an ecosystem of data sources, tools and apps, enhancing capabilities and improving the end-user experience.
- **End-users**: MCP results in more capable AI applications that can access user data and take actions on the user's behalf.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**:

- **MCP Host**: The AI application (e.g. Claude Code, Claude Desktop, Visual Studio Code) that coordinates and manages one or multiple MCP clients.
- **MCP Client**: A component that maintains a connection to an MCP server and obtains context from that server for the host to use. The host creates one client per server, and each client maintains a dedicated connection to its corresponding server.
- **MCP Server**: A program that provides context to MCP clients. Servers can execute locally (e.g. the official filesystem server launched by Claude Desktop via the STDIO transport — a "local" server) or remotely (e.g. the official Sentry MCP server on the Sentry platform via the Streamable HTTP transport — a "remote" server). Local servers typically serve a single client; remote servers typically serve many clients.

Core primitives a server can expose:

- **Tools**: Servers expose tools that can be invoked by language models. Tools enable models to interact with external systems such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are designed to be model-controlled — the language model can discover and invoke them automatically based on its contextual understanding and the user's prompts.
- **Resources**: Servers provide resources to share data that gives context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are designed to be application-driven, with host applications determining how to incorporate context (e.g. exposing resources through UI elements for explicit selection, allowing the user to search/filter, or automatic context inclusion based on heuristics).
- **Prompts** (related core primitive): reusable prompt templates describing model behavior for a task (mentioned alongside tools and resources as core protocol primitives).

The protocol has two layers:
- **Data layer**: Defines the JSON-RPC-based protocol for client-server communication, including capability and version discovery and core primitives such as tools, resources, prompts and notifications.
- **Transport layer**: Defines how messages are transmitted — STDIO for local servers and Streamable HTTP for remote servers.

MCP focuses solely on the protocol for context exchange; it does not dictate how AI applications use LLMs or manage the provided context.

## 3. Ecosystem: Supporting AI Applications

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, development tools like Visual Studio Code, Cursor, and MCPJam, and many others all support MCP — making it easy to build once and integrate everywhere.

Notable supporting clients include:
- **Claude / Claude Desktop / Claude Code** (Anthropic) — the reference implementation where MCP originated.
- **ChatGPT** (OpenAI) — supports MCP connects.
- **Visual Studio Code** — acts as an MCP host, instantiating one client per connected server (e.g. the Sentry MCP server or the local filesystem server).
- **Cursor** — AI-powered code editor supporting MCP.
- **MCPJam** — collaborative AI coding environment supporting MCP.

The MCP ecosystem also includes reference server implementations, MCP development tools such as the MCP Inspector, and MCP SDKs for different programming languages, so teams can build servers to expose their data and tools and clients to connect to MCP servers.

## 4. Videos

Picked introduction videos for getting started with MCP:

1. **[The Model Context Protocol (MCP)](https://www.youtube.com/watch?v=CQywdSdi5iA)** — Anthropic (Theo Chu, David Soria Parra, Alex Albert). Duration: 00:19:35. A panel discussion with MCP co-creators and Claude relations lead diving deep into what MCP is and what's next.
2. **[How Model Context Protocol (MCP) actually works](https://www.youtube.com/watch?v=cGuyrANVi4A)** — Google Cloud Tech. Duration: 07:58. Explains MCP's client-server architecture, capability discovery, and how any compliant model can interact without custom integration code.
3. **[Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w)** — Shaw Hin Talabi. Duration: 19:49. A beginner-friendly walkthrough of MCP concept and architecture in plain English.
4. **[Model Context Protocol (MCP) Explained in 90 Seconds](https://www.youtube.com/watch?v=mMWWU4PambM)** — Simplilearn. Duration: 01:51. A quick enterprise-oriented explainer on how MCP works and why it matters for deploying AI agents.

## Sources

- MCP official documentation — "What is the Model Context Protocol (MCP)?": https://modelcontextprotocol.io/docs/getting-started/intro
- MCP official documentation — "Architecture overview": https://modelcontextprotocol.io/docs/concepts/architecture
- MCP official documentation — "Tools": https://modelcontextprotocol.io/docs/concepts/tools
- MCP official documentation — "Resources": https://modelcontextprotocol.io/docs/concepts/resources
- MCP official documentation — clients list: https://modelcontextprotocol.io/clients
- YouTube — "The Model Context Protocol (MCP)" (Anthropic): https://www.youtube.com/watch?v=CQywdSdi5iA
- YouTube — "How Model Context Protocol (MCP) actually works" (Google Cloud Tech): https://www.youtube.com/watch?v=cGuyrANVi4A
- YouTube — "Model Context Protocol (MCP) Explained in 20 Minutes" (Shaw Hin Talabi): https://www.youtube.com/watch?v=N3vHJcHBS-w
- YouTube — "Model Context Protocol (MCP) Explained in 90 Seconds" (Simplilearn): https://www.youtube.com/watch?v=mMWWU4PambM
