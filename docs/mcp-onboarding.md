# MCP Onboarding Notes

A plain-language guide for new teammates joining the team: what the Model Context Protocol (MCP) is, how its client/server architecture works, which AI applications support it today, and a short set of video walkthroughs to watch first.

## 1. Primer: What Is the Model Context Protocol?

MCP (Model Context Protocol) is an **open-source standard for connecting AI applications to external systems**. Instead of every AI application building a bespoke integration for every data source, tool, or workflow, MCP provides one common, standardized way for an AI application to reach out and connect to external systems.

The official documentation uses a helpful analogy: **MCP is like a USB-C port for AI applications**. Just as USB-C gives you one standard connector that works with many different peripherals, MCP gives AI applications one standard way to connect to local files, databases, search engines, calendars, and the rest of your tooling ecosystem.

In practice, MCP lets AI applications like Claude or ChatGPT:

- Connect to **data sources** (e.g. local files, databases)
- Connect to **tools** (e.g. search engines, calculators)
- Connect to **workflows** (e.g. specialized prompts)

…so the model can access key information and actually perform tasks rather than just talking about it. For developers, MCP reduces development time and complexity when building or integrating with AI applications; for end users it means more capable assistants that can act on your behalf.

*(Sources: [What is the Model Context Protocol? — modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction); [MCP clients overview](https://modelcontextprotocol.io/clients))*

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture**. The key participants are:

- **MCP Host** — the AI application (e.g. Claude Code, Claude Desktop, Visual Studio Code) that coordinates and manages one or multiple MCP clients.
- **MCP Client** — a component that maintains a dedicated connection to an MCP server and obtains context from that server for the host to use. The host creates one client per server.
- **MCP Server** — a program that provides context to MCP clients.

For example, Visual Studio Code acts as an MCP host; when it connects to the Sentry MCP server, the VS Code runtime instantiates an MCP client object maintaining the connection. Local servers using the STDIO transport typically serve a single client, while remote servers using the Streamable HTTP transport typically serve many clients.

The protocol has two layers:

- **Data layer** — a JSON-RPC based protocol for client-server communication, including capability and version discovery and core primitives such as **tools**, **resources**, **prompts**, and notifications.
- **Transport layer** — defines how messages are transported (STDIO for local, Streamable HTTP for remote).

The core primitives a server can expose:

- **Tools** — model-controlled actions the AI can invoke automatically (e.g. querying a database, calling an API, performing a computation). Each tool has a name and a schema describing its inputs and outputs. Discovered via `tools/list`, invoked via `tools/call`. A human should always be able to deny tool invocations.
- **Resources** — application-controlled data that provides context to the model, identified by a URI (e.g. files, database schemas, application-specific information). Discovered via `resources/list`; servers may optionally support subscription notifications for updates.
- **Prompts** — user-controlled, predefined prompt templates the user can explicitly select and parameterization.

MCP focuses solely on the protocol for context exchange; it does not dictate how AI applications use LLMs or manage the provided context.

*(Sources: [Architecture overview — modelcontextprotocol.io/docs/concepts/architecture](https://modelcontextprotocol.io/docs/concepts/architecture); [Tools](https://modelcontextprotocol.io/docs/concepts/tools); [Resources](https://modelcontextprotocol.io/docs/concepts/resources); [Prompts](https://modelcontextprotocol.io/docs/concepts/prompts))*

## 3. Ecosystem: Supporting AI Applications

MCP is an open protocol with broad ecosystem support. AI assistants like **Claude** (including Claude Desktop and Claude Code) and **ChatGPT** support MCP, as do development tools such as **Visual Studio Code, Cursor, MCPJam**, and many others. This "build once, integrate everywhere" model makes it easy to build an MCP server once and have it adopted across many different AI applications.

Server-side there is a large ecosystem of reference and community servers wrapping databases, file systems, Git, GitHub, Google Drive, Suõ? — in short, anything worth exposing as a tool or data source can be packaged as an MCP server.

 прак??? For new teammates: if you need data or a capability in an AI assistant, first check whether an MCP server already exists for it before writing one.

*(Sources: [What is the Model Context Protocol? — ecosystem support section](https://modelcontextprotocol.io/introduction); [MCP clients overview](https://modelcontextprotocol.io/clients))*

## 4. Video Walkthroughs

A short selection of introduction material — a conference talk, a vendor tutorial, and a son of the official conference workshop. Each entry is a link with its title, publisher, and duration where available.

1. **30 minutes to understand MCP (Model Context Protocol)** — Sébastien Blanc, Devoxx UK 2025 (conference talk). Duration: 32m. [Watch on YouTube](https://youtu.be/hICGtUH7K-4). A hands-on talk covering the MCP protocol, architecture, and a live demo building a bot with an MCP server.
2. **Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic** — AI Engineer channel (live workshop by Mahesh Murag of Anthropic's Applied AI team). Duration: 1:44:12. [Watch on YouTube](https://www.youtube.com/watch?v=kQmXtrmQ5Zg). The authoritative deep-dive: MCP philosophy, the N Times M integration problem, the three primitives (tools/resources/prompts), sampling, composability, remote servers, and the roadmap.
3. **MCP vs API: Simplifying AI Agent Integration with External Data** — IBM Technology (tutorial). Duration: not listed in the listing. [Watch on YouTube](https://www.youtube.com/watch?v=7j1t3UZA1TY). A clear explainer contrasting MCP with traditional REST APIs, using the USB-C analogy and covering tools, resources, and prompt templates.

*(Sources: video listings as cited above; background research drew on the official MCP documentation pages listed in Sections 1–3.)*

## Sources

- [What is the Model Context Protocol? — modelcontextprotocol.io](https://modelcontextprotocol.io/introduction)
- [Architecture overview — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/resources)
- [Prompts — modelcontextprotocol.io](https://modelcontextprotocol.io/docs/concepts/prompts)
- [MCP clients overview — modelcontextprotocol.io](https://modelcontextprotocol.io/clients)
- [Devoxx UK 2025 — "30 minutes to understand MCP (Model Context Protocol)" (Sébastien Blanc)](https://youtu.be/hICGtUH7K-4)
- [YouTube — "Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic" (AI Engineer)](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)
- [YouTube — "MCP vs API: Simplifying AI Agent Integration with External Data" (IBM Technology)](https://www.youtube.com/watch?v=7j1t3UZA1TY)
