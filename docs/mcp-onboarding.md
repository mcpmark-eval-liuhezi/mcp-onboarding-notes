# MCP Onboarding Notes

A plain-language onboarding note on the **Model Context Protocol (MCP)** for new team members.

Contents:

1. [What MCP is](#1-what-mcp-is)
2. [Architecture: hosts, clients, servers, tools, and resources](#2-architecture-hosts-clients-servers-tools-and-resources)
3. [Ecosystem: AI applications that support MCP](#3-ecosystem-ai-applications-that-support-mcp)
4. [Videos: talks, tutorials, and demos](#4-videos-talks-tutorials-and-demos)
5. [Sources](#5-sources)

---

## 1. What MCP is

MCP (Model Context Protocol) is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications such as Claude or ChatGPT can connect to data sources (local files, databases), tools (search engines, calculators), and workflows (specialized prompts), enabling them to access key information and perform tasks.

A useful mental model: **MCP is like a USB-C port for AI applications**. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. This standardization means you build an integration once and it works across many AI clients — “build once, integrate everywhere.”

What MCP can enable in practice:

- Agents accessing your Google Calendar and Notion, acting as a more personalized AI assistant.
- Claude Code generating an entire web app from a Figma design.
- Enterprise chatbots connecting to multiple databases across an organization so users can analyze data through chat.
- AI models creating 3D designs in Blender and printing them on a 3D printer.

Why it matters, per audience:

- **Developers:** MCP reduces development time and complexity when building or integrating with an AI application or agent.
- **AI applications/agents:** MCP gives them access to an ecosystem of data sources, tools, and apps, enhancing capabilities and end-user experience.
- **End users:** MCP results in more capable AI applications that can access user data and take actions on the user’s behalf.

MCP focuses solely on the protocol for context exchange — it does not dictate how AI applications use LLMs or manage the provided context.

---

## 2. Architecture: hosts, clients, servers, tools, and resources

MCP follows a **client-server architecture**. An MCP host — an AI application like Claude Code or Claude Desktop — establishes connections to one or more MCP servers. The host accomplishes this by creating one MCP client for each MCP server; each client maintains a dedicated connection with its corresponding server.

Key participants:

- **MCP Host:** the AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client:** a component that maintains a connection to an MCP server and obtains context from it for the host to use. Local MCP servers using the STDIO transport typically serve a single client; remote servers using the Streamable HTTP transport typically serve many clients.
- **MCP Server:** a program that provides context to MCP clients. Servers can run locally (e.g., the filesystem server launched by Claude Desktop using STDIO) or remotely (e.g., the official Sentry MCP server on the Sentry platform using Streamable HTTP).

Example: Visual Studio Code acts as an MCP host. When VS Code connects to the Sentry MCP server, the runtime instantiates an MCP client object for that server; connecting to a second server (e.g., the local filesystem server) instantiates another client.

Two layers make up the protocol:

- **Data layer:** a JSON-RPC 2.0-based exchange protocol defining message structure and semantics, including discovery of versions/capabilities, and core primitives such as tools, resources, and prompts, plus notifications and progress tracking.
- **Transport layer:** defines communication mechanisms and channels, handling connection establishment, message framing, and secure communication between participants.

MCP supports two transports:

- **Stdio transport:** uses standard input/output streams for direct process communication between local processes on the same machine (optimal performance for local servers).
- **Streamable HTTP transport:** used by remote servers, typically serving many clients.

Core server features exposed by servers:

- **Tools** — enable models to interact with external systems (querying databases, calling APIs, performing computations). Each tool is uniquely identified by a name and includes metadata describing its schema; the model can discover and invoke tools automatically. A human-in-the-loop confirmation is recommended for trust and safety.
- **Resources** — a standardized way for servers to expose data that provides context to language models (files, database schemas, application-specific information), each uniquely identified by a URI. Resources are designed to be application-driven.
- **Prompts** — reusable interaction templates offered by servers (covered in the protocol’s prompts concept; mentioned here as the third core primitive alongside tools and resources).

Client features include elicitation (servers can ask the user for input), plus notifications and progress tracking for long-running operations.

---

## 3. Ecosystem: AI applications that support MCP

MCP is an open protocol with broad ecosystem support across clients and servers. AI assistants like **Claude** and **ChatGPT**, and development tools like **Visual Studio Code**, **Cursor**, and **MCPJam**, all support MCP, making it easy to build once and integrate everywhere.

Supporting clients include (per the official documentation):

- **Claude / Claude Desktop, Claude Code** — AI applications that act as MCP hosts and connect to MCP servers.
- **ChatGPT** — AI assistant supporting MCP connections.
- **Visual Studio Code** — development tool acting as an MCP host; connects to servers such as the Sentry MCP server and the local filesystem server.
- **Cursor** — AI coding assistant; an MCP client that connects to servers.
- **MCPJam** — another client supporting MCP connections.

The MCP ecosystem also includes language SDKs for different programming languages, MCP development tools (including the MCP Inspector), and reference server implementations, so servers can be built in Python, TypeScript, and other languages.

---

## 4. Videos: talks, tutorials, and demos

The following videos are recommended for getting up to speed with MCP. Each entry is a link with its title, publisher, and duration where the listing shows one.

1. [**Model Context Protocol (MCP) Explained in 20 Minutes**](https://www.classcentral.com/course/youtube-model-context-protocol-mcp-explained-in-20-minutes-447743) — Shaw Talebi. Duration: 20 minutes (per the listing). Introduces MCP, explains clients and servers, and demonstrates building a custom server with Python and connecting it to Claude Desktop.
2. [**MCP for dummies: Feed more context to LLM**](https://www.youtube.com/watch?v=1uZl3e8GdsU) — Tarun Jain, presented at PyCon AU 2025 (conference talk). Duration: not listed on the listing. Covers what MCP is and its pain points with LLMs, the architecture, transport mechanism, security, plus a live code demo connecting any MCP server with an agentic framework.
3. [**WebMCP is MCP for Single Page Apps!**](https://www.youtube.com/watch?v=IAfrzel524s) — Jack Herrington. Duration: 05:32. A live demo/tutorial showing how WebMCP lets Single Page Applications join the AI ecosystem by exposing MCP tools in the browser.
4. [**Full Course (Lessons 1-11) MCP for Beginners**](https://www.youtube.com/watch?v=VfZlglOWWZw) — Microsoft Developer. Duration: not listed on the listing. A tutorial series covering MCP fundamentals, architecture, and core concepts, with SDKs in Python and JavaScript.

---

## 5. Sources

Documentation pages actually used for this note:

- [What is the Model Context Protocol (MCP)? — official intro](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Architecture overview — official concepts](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools — official concepts](https://modelcontextprotocol.io/docs/concepts/tools)
- [ Resources — official concepts](https://modelcontextprotocol.io/docs/concepts/resources)
- [Prompts — official concepts](https://modelcontextprotocol.io/docs/concepts/prompts)
- [Clients — official ecosystem page](https://modelcontextprotocol.io/clients)

Video sources:

- [Class Central listing: Model Context Protocol (MCP) Explained in 20 Minutes](https://www.classcentral.com/course/youtube-model-context-protocol-mcp-explained-in-20-minutes-447743)
- [PyCon AU 2025: MCP for dummies — Feed more context to LLM](https://www.youtube.com/watch?v=1uZl3e8GdsU)
- [Stack Manifest: MCP in 60 seconds — a real session, message by message](https://stackmanifest.com/videos/ai/mcp-in-60-seconds/)
- [YouTube: WebMCP is MCP for Single Page Apps!](https://www.youtube.com/watch?v=IAfrzel524s)
