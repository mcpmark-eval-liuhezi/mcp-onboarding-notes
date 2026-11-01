# MCP Onboarding Notes

A plain-language onboarding note for new teammates joining the team. It covers what the Model Context Protocol (MCP) is, how the client/server architecture works, which AI applications support it today, and a selection of introductory videos.

## 1. What MCP Is (Primer)

The Model Context Protocol (MCP) is an **open-source standard for connecting AI applications to external systems**. Think of it as a "USB-C port for AI applications": just as USB-C standardizes how electronic devices connect, MCP standardizes how AI applications connect to data sources, tools, and workflows.

Using MCP, AI applications like Claude or ChatGPT can connect to:

- **Data sources** — e.g. local files, databases
- **Tools** — e.g. search engines, calculators
- **Workflows** — e.g. specialized prompts

This enables agents to access key information and perform tasks. For example, agents can read your Google Calendar and Notion, Claude Code can generate a web app from a Figma design, enterprise chatbots can query multiple databases via chat, and AI models can create 3D designs in Blender and send them to a 3D printer.

### Why MCP Matters

- **Developers:** reduces development time and complexity when building or integrating with AI applications or agents.
- **AI applications/agents:** gives them access to an ecosystem of data sources, tools, and apps, improving capabilities and end-user experience.
- **End-users:** results in more capable AI applications that can access user data and take actions on the user's behalf.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**:

- **MCP Host:** the AI application (e.g. Claude Code, Claude Desktop, Visual Studio Code) that coordinates and manages one or multiple MCP clients.
- **MCP Client:** a component that maintains a connection to an MCP server and obtains context from it for the host to use. The host creates one client per server, and each client maintains a dedicated connection to its corresponding server.
- **MCP Server:** a program that provides context to MCP clients. Servers can run locally (e.g. the filesystem server launched by Claude Desktop using the STDIO transport) or remotely (e.g. the official Sentry MCP server using the Streamable HTTP transport).

### Layers

MCP consists of two layers:

- **Data layer:** defines the JSON-RPC-based protocol for client-server communication, including capability and version discovery, and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer:** defines how messages are transmitted (e.g. STDIO for local servers, Streamable HTTP for remote servers).

### Core Primitives

- **Tools:** servers expose tools that can be invoked by language models. Tools enable models to interact with external systems such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are model-controlled: the language model can discover and invoke them automatically based on its understanding of the context and the user's prompts (though implementations are free to expose tools through any interface pattern).
- **Resources:** servers provide resources to share data that gives context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are application-driven: host applications determine how to incorporate context based on their needs (e.g. exposing resources through UI elements for explicit selection, allowing the user to search/filter available resources, or implementing automatic context inclusion).
- **Prompts:** reusable prompt templates (part of the core primitives alongside tools and resources).

For more detail, see the [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture), [Tools](https://modelcontextprotocol.io/docs/concepts/tools), and [Resources](https://modelcontextprotocol.io/docs/concepts/resources) documentation pages.

## 3. Ecosystem: Supporting Clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like **Claude** and **ChatGPT**, and development tools like **Visual Studio Code**, **Cursor**, and **MCPJam**, all support MCP — making it easy to build once and integrate everywhere.

Additional client applications known to support MCP include IDEs and agentic tools such as Windsurf, Zed, and Block's Goose, as well as various MCP SDK-based hosts. The official specification, SDKs for multiple languages, development tools (including the MCP Inspector), and reference server implementations are all part of the MCP project.

## 4. Video Walkthroughs

Picked introductory videos to get up to speed quickly:

1. **[MCP In 26 Minutes (Model Context Protocol)](https://www.youtube.com/watch?v=kOhLoixrJXo)** — Tina Huang — 26:23 — A deep dive explaining what MCP is, how it works, how to use MCP servers, and how to build your own MCP servers with both code and no-code approaches.
2. **[How Model Context Protocol (MCP) actually works](https://www.youtube.com/watch?v=cGuyrANVi4A)** — Google for Developers — 7:58 — Explains how MCP works with Google MCPs, building an ADK agent with Google MCPs, and connecting an MCP server to an AI agent.
3. **[you need to learn MCP RIGHT NOW!! (Model Context Protocol)](https://www.youtube.com/watch?v=GuTcle5edjk)** — Docker — 38:40 — A teardown of MCP covering the Docker MCP Catalog and practical MCP usage.

## Sources

- [What is the Model Context Protocol (MCP)? — Introduction](https://modelcontextprotocol.io/introduction)
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources](https://modelcontextprotocol.io/docs/concepts/resources)
- [Clients](https://modelcontextprotocol.io/clients)
