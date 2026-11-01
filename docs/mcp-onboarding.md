# MCP Onboarding Notes

A plain-language onboarding note for the Model Context Protocol (MCP), suitable for new teammates joining an AI-features team. It covers what MCP is, how the client/server architecture works, which applications support it today, and a short list of intro videos.

## 1. What MCP Is (Primer)

MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts) — enabling them to access key information and perform tasks.

The official documentation uses a helpful analogy: think of MCP like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems.

What can MCP enable in practice?

- Agents can access your Google Calendar and Notion, acting as a more personalized AI assistant.
- Claude Code can generate an entire web app using a Figma design.
- Enterprise chatbots can connect to multiple databases across an organization, empowering users to analyze data using chat.
- AI models can create 3D designs on Blender and print them out using a 3D printer.

Why does MCP matter?

- **Developers:** MCP reduces development time and complexity when building or integrating with an AI application or agent.
- **AI applications or agents:** MCP gives them access to an ecosystem of data sources, tools and apps, which enhances their capabilities and improves the end-user experience.
- **End-users:** MCP results in more capable AI applications or agents that can access user data and take actions on the user's behalf when necessary.

MCP was introduced by Anthropic in November 2024 and is now an open standard adopted across a broad ecosystem. See the [official intro page](https://modelcontextprotocol.io/docs/getting-started/intro) and [Wikipedia](https://en.wikipedia.org/wiki/Model_Context_Protocol) for background.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a client-server architecture. An **MCP host** — an AI application such as Claude Code or Claude Desktop — establishes connections to one or more MCP servers. For each server it connects to, the host creates one **MCP client**, and each client maintains a dedicated connection with its corresponding server.

Key participants:

- **MCP Host:** the AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client:** a component that maintains a connection to an MCP server and obtains context from that server for the host to use.
- **MCP Server:** a program that provides context to MCP clients.

Local MCP servers that use the STDIO transport typically serve a single MCP client, whereas remote MCP servers that use the Streamable HTTP transport will typically serve many MCP clients. For example, when Visual Studio Code establishes a connection to the Sentry MCP server, the VS Code runtime instantiates an MCP client object maintaining the connection; connecting to a second server such as the local filesystem server instantiates an additional client object.

MCP consists of two layers:

- **Data layer:** defines the JSON-RPC based protocol for client-server communication, including capability and version discovery, and core primitives such as tools, resources, prompts and notifications.
- **Transport layer:** defines how messages are transmitted (e.g. STDIO for local servers, Streamable HTTP for remote servers).

Note that MCP servers can execute locally or remotely; "local" refers to a server running on the same machine as the client (typically via STDIO), while "remote" refers to a server running elsewhere (typically via Streamable HTTP).

### Tools

Tools allow servers to expose functionality that can be invoked by language models. Tools enable models to interact with external systems, such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools in MCP are designed to be **model-controlled**: the language model can discover and invoke tools automatically based on its contextual understanding and the user's prompts. Servers that support tools must declare the `tools` capability and respond to `tools/list` requests with the set of available tools. When a model wants to use a tool, it sends a `tools/call` request. For trust and safety, there should always be a human in the loop with the ability to deny tool invocations; applications should provide UI making clear which tools are exposed and insert visual indicators when tools are invoked.

### Resources

Resources provide a standardized way for servers to expose data that gives context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources in MCP are designed to be **application-driven**: host applications determine how to incorporate context based on their needs — for example, exposing resources through UI elements for explicit selection, allowing the user to search through and filter available resources, or implementing automatic context inclusion based on heuristics or the model's selection. Servers that support resources must declare the `resources` capability, which supports two optional features: `listChanged` (whether the server emits notifications when the resource list changes) and `subscribe` (whether the server supports resource-specific update notifications). Servers that declare the resources capability must respond to `resources/list` requests with the set of currently available resources.

### Prompts (brief mention)

Beyond tools and resources, MCP also defines **prompts**: server-defined prompt templates that the model can invoke, optionally with arguments. These complete the core conceptually primitive surface of the protocol alongside tools and resources.

For the full architecture overview, see [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture), [Tools](https://modelcontextprotocol.io/docs/concepts/tools), and [Resources](https://modelcontextprotocol.io/docs/concepts/resources).

## 3. Ecosystem: Supporting Clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, development tools like Visual Studio Code, Cursor, and MCPJam, and many others all support MCP — making it easy to build once and integrate everywhere.

The MCP specification includes:

- **MCP Specification:** outlines the implementation requirements for clients and servers.
- **MCP SDKs:** SDKs for different programming languages that implement MCP (including Python, TypeScript, C#, and Java per Wikipedia).
- **MCP Development Tools:** tools for developing MCP servers and clients, including the MCP Inspector.
- **MCP Reference Server Implementations:** reference implementations of MCP servers.

MCP focuses solely on the protocol for context exchange — it does not dictate how AI applications use LLMs or manage the provided context.

## 4. Video Walkthroughs

A short selection of intro videos, each with title, publisher, and duration where the listing shows one.

1. **Claude's Model Context Protocol is here... Let's test it** — Fireship — https://www.youtube.com/watch?v=HyzlYwjoXOQ — published 2025-03-31; duration not listed in the search results. A fast, hands-on tutorial that builds an MCP server connecting a storage bucket, a Postgres database, and a REST API so Claude can access data and execute code.
2. **MCP In 26 Minutes (Model Context Protocol)** — Tina Huang — https://www.youtube.com/watch?v=kOhLoixrJXo — 26:23. Explains MCP fundamentals, how to use MCP servers, and how to build your own MCP servers both with code and no code, with little assessments along the way.
3. **How Model Context Protocol (MCP) actually works** — Google Cloud Tech — https://www.youtube.com/watch?v=cGuyrANVi4A — 07:58. Walks through what MCP is, why it exists, and how it compares to traditional APIs, aimed at developers building agents or AI-powered applications.
4. **Model Context Protocol (MCP) Explained in 20 Minutes** — Shaw Talebi — https://www.youtube.com/watch?v=N3vHJcHBS-w — 19:49. Covers key concepts and shows how to build a custom MCP server with Python as the fifth video in a series on AI agents.
5. **The Model Context Protocol (MCP)** — Anthropic (Theo Chu, David Soria Parra, Alex Albert) — https://www.youtube.com/watch?v=CQywdSdi5iA — 00:19:35. A panel discussion with three key members of Anthropic about MCP's development, what it is, and its implications for AI applications.

## Sources

- [What is the Model Context Protocol (MCP)? — official docs](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Architecture overview — official docs](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools — official docs](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources — official docs](https://modelcontextprotocol.io/docs/concepts/resources)
- [Clients — official docs](https://modelcontextprotocol.io/clients)
- [Model Context Protocol — Wikipedia](https://en.wikipedia.org/wiki/Model_Context_Protocol)
- [You need to learn MCP RIGHT NOW!! (Model Context Protocol) — Docker](https://www.youtube.com/watch?v=GuTcle5edjk) — 38:40 (backup pick)
