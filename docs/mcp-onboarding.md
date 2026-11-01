# MCP Onboarding Notes

Welcome to the team! This document gives new teammates a plain-language primer on the Model Context Protocol (MCP), a run-through of how its client/server architecture works, an overview of the AI applications that support it today, and a short selection of introduction videos for going deeper.

## 1. What MCP Is (Plain-Language Primer)

The Model Context Protocol (MCP) is an open-standard protocol for connecting AI applications to external systems. Think of it as a **USB-C port for AI applications**: just as USB-C gives you a standardized, plug-and-play way to connect devices, MCP gives AI applications a standardized way to connect to data sources (local files, databases), tools (search engines, calculators), and workflows (specialized prompts).

Key points:

- MCP was introduced by Anthropic and released as an open standard in November 2024; it has since been donated to the Linux Foundation.
- It follows a client-server architecture: an MCP host (the AI application) creates one MCP client per MCP server it connects to.
- Servers expose capabilities through primitives such as **tools** (actions the model can invoke), **resources** (data/context the model can read), and **prompts** (reusable interaction templates).
- Communication is JSON-RPC 2.0 based, over either the STDIO transport (local servers) or Streamable HTTP (remote servers).
- Benefits: developers reduce integration complexity; AI applications gain access to a growing ecosystem of data sources and tools; end users get more capable assistants that can act on their behalf.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a client-server architecture with three key participant types:

- **MCP Host** — the AI application (e.g., Claude Desktop, Claude Code, Visual Studio Code) that coordinates and manages one or multiple MCP clients.
- **MCP Client** — a component inside the host that maintains a dedicated connection to an MCP server and obtains context from it for the host to use. For each server, the host creates one client, giving a 1:1 relationship between client and server.
- **MCP Server** — a program that provides context and capabilities to clients. Servers can run locally (e.g., the official filesystem server launched by Claude Desktop via STDIO) or remotely (e.g., the official Sentry MCP server over Streamable HTTP).

### Core Primitives

- **Tools** — functions the language model can invoke to interact with external systems, such as querying a database. Tools have named parameters and return structured results that are injected back into the model's conversation.
- **Resources** — read-only data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI and can be listed (`resources/list`) and read (`resources/read`). Servers may optionally support `listChanged` notifications and resource subscriptions.
- **Prompts** — reusable, user-customizable templates and workflows for LLM-server communication, with arguments that the client can provide (e.g., a `code_review` prompt taking code as an argument).
- **Elicitation** — a client feature that enables servers to ask the user for input during a conversation.

### Layers

- **Data layer** — defines the JSON-RPC 2.0-based protocol for client-server communication, including capability and version discovery and core primitives (tools, resources, prompts, notifications).
- **Transport layer** — defines the communication mechanisms, including connection establishment, message framing, and authorization. Local servers typically use the STDIO transport; remote servers typically use Streamable HTTP.

## 3. Ecosystem: Supporting AI Applications

MCP is an open protocol supported across a wide range of clients and servers, so you can build once and integrate everywhere:

- **AI assistants**: Claude (Claude Desktop, Claude Code) and ChatGPT both support MCP.
- **Development tools**: Visual Studio Code (with the MCP servers docs), Cursor, Windsurf Editor, and MCPJam are among the supported clients.
- **Other clients**: IBM BeeAI, Microsoft Copilot Studio, Postman, and Replit have also adopted MCP for agentic and coding workflows.
- **Reference implementations**: The MCP project provides reference server implementations (e.g., filesystem, git, GitHub, Postgres, Puppeteer,and more) and the MCP Inspector tool for developing and debugging servers.
- **SDKs**: Official SDKs exist for popular languages including Python, TypeScript, C#, and Java.

## 4. Video Section

A short selection of introduction videos covering MCP concepts, how it works, and hands-on setup:

- [How Model Context Protocol (MCP) actually works](https://www.youtube.com/watch?v=cGuyrANVi4A) — Google Cloud Tech — 7:58
- [MCP Complete Explanation](https://www.youtube.com/watch?v=_fzpnqt39jQ) — Aishwarya Srinivasan — 15:22
- [Get Started With The Model Context Protocol // 2-Minute Tutorial](https://www.youtube.com/watch?v=MC2BwMGFRx4) — Matt Pocock — 2:24
- [Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w) — Shaw Hin Talibi — 19:49
- [Model Context Protocol Clearly Explained | MCP Beyond the Hype](https://www.youtube.com/watch?v=tzrwxLNHtRY) — 15:04

## Sources

- Model Context Protocol documentation — [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/docs/getting-started/intro)
- Model Context Protocol documentation — [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture)
- Model Context Protocol documentation — [Tools](https://modelcontextprotocol.io/docs/concepts/tools)
- Model Context Protocol documentation — [Resources](https://modelcontextprotocol.io/docs/concepts/resources)
- Model Context Protocol documentation — [Prompts](https://modelcontextprotocol.io/docs/concepts/prompts)
- Model Context Protocol documentation — [Clients](https://modelcontextprotocol.io/clients)
- Wikipedia — [Model Context Protocol](https://en.wikipedia.org/wiki/Model_Context_Protocol)
