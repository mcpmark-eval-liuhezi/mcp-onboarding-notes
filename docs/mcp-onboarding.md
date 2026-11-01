# MCP Onboarding Notes

A plain-language onboarding note on the Model Context Protocol (MCP) for new team members. It covers what MCP is, how its client/server architecture works, which applications support it, and a short selection of introduction videos.

## 1. What is MCP? (Primer)

MCP (Model Context Protocol) is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts), enabling them to access key information and perform tasks.

The official documentation describes it with a simple analogy: **MCP is like a USB-C port for AI applications**. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems.

Why it matters:

- **Developers**: MCP reduces development time and complexity when building or integrating with an AI application or agent, replacing fragmented one-off integrations with a single protocol.
- **AI applications / agents**: MCP gives them access to an ecosystem of data sources, tools and apps, enhancing capabilities and the end-user experience.
- **End-users**: MCP results in more capable AI applications or agents that can access user data and take actions on the user's behalf.

MCP was introduced and open-sourced by Anthropic in November 2024. It builds on existing concepts like tool use and function calling but standardizes them, reducing the need for custom connections for each new AI model and external system.

## 2. Architecture: Clients, Servers, Tools, Resources

MCP follows a **client-server architecture**. An MCP host — an AI application such as Claude Code or Claude Desktop — establishes connections to one or more MCP servers. The host does this by creating one MCP client for each MCP server, and each client maintains a dedicated connection with its corresponding server.

Key participants:

- **MCP Host**: The AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client**: A component that maintains a connection to an MCP server and obtains context from that server for the host to use.
- **MCP Server**: A program that provides context to MCP clients.

Transport: local MCP servers that use the STDIO transport typically serve a single MCP client, whereas remote MCP servers that use the Streamable HTTP transport will typically serve many MCP clients.

The three core primitives a server can expose:

- **Tools** (`tools/list`, `tools/call`): Tools enable models to interact with external systems, such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are **model-controlled** — the language model can discover and invoke them automatically based on its contextual understanding and the user's prompts. A human should generally remain in the loop for tool invocations.
- **Resources** (`resources/list`, `resources/read`): Resources allow servers to share data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are **application-driven** — the host application determines how to incorporate context based on its needs. The capability also supports optional `listChanged` and `subscribe` notifications for update announcements.
- **Prompts** (`prompts/list`, `prompts/get`): Prompts allow servers to provide structured messages and instructions for interacting with language models. Clients can discover available prompts, retrieve their contents, and provide arguments to customize them. Prompts are **user-controlled** — exposed with the intention that the user explicitly selects them for use (e.g. as slash commands).

Notes from the specification: MCP focuses solely on the protocol for context exchange — it does not dictate how AI applications use LLMs or manage the provided context. The protocol uses JSON-RPC 2.0 messages, and servers/clients negotiate capabilities during initialization.

## 3. Ecosystem: Supporting Clients

MCP is an open protocol supported across a wide range of clients and servers, which is what makes it easy to "build once and integrate everywhere."

Per the official documentation, supported clients include:

- **AI assistants**: Claude and ChatGPT.
- **Development tools**: Visual Studio Code, Cursor, MCPJam, and many others.

The architecture overview进一步 notes client examples such as first-party Anthropic applications, Perplexity, Windsurf, and the Goose agent launched by Block — all of which are MCP clients able to connect to any MCP server with zero additional work.

For providers and builders, the ecosystem includes a growing set of reference server implementations and MCP SDKs for different programming languages, plus development tooling such as the MCP Inspector for debugging servers.

## 4. Videos

A short selection of introduction videos, each as a link with title, publisher, and duration where the listing shows one.

- [Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg) — published by AI Engineer on YouTube (talk/workshop style; duration not listed in the listing; published 2025-03-01).covers the philosophy behind MCP, the N×M integration problem, the three interfaces (prompts, tools, resources), and adoption patterns.
- [Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w) — publisher: Shaw Talebi; duration: 19:49. A tutorial-style deep dive that explains MCP's key concepts and builds a custom MCP server with Python.
- [What is MCP (Model Context Protocol)? | BEST ANIMATION | Explained Simply for Beginners](https://www.youtube.com/watch?v=OMydblSSv58) — publisher: KnowledgeCatch; duration: 07:39. A beginner-friendly animated explainer of what MCP is, why it was invented, and how it works step by side, including the N×M problem and the USB-C analogy.

## Sources

Documentation pages actually used for this note:

- [What is the Model Context Protocol (MCP)? — modelcontextprotocol.io](https://modelcontextprotocol.io/) (official documentation home / introduction)
- [What is the Model Context Protocol (MCP)? — Getting Started Intro](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Architecture overview — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/resources)
- [Prompts — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/prompts)
- [Clients — modelcontextprotocol.io](https://modelcontextprotocol.io/clients)
- [Introducing the Model Context Protocol — Anthropic](https://www.anthropic.com/news/model-context-protocol)
- [What is Model Context Protocol (MCP)? A guide — Google Cloud](https://cloud.google.com/discover/what-is-model-context-protocol)
