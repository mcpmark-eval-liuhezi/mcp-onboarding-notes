# MCP Onboarding Notes

A plain-language onboarding document for new teammates joining the team. It covers what the Model Context Protocol (MCP) is, how its client/server architecture works, which AI applications support it today, and a short list of videos to watch first.

## 1. What MCP is (plain-language primer)

The Model Context Protocol (MCP) is an open-source standard for connecting AI applications to external systems. The official documentation describes it as a standardized way for AI applications like Claude or ChatGPT to connect to data sources (local files, databases), tools (search engines, calculators), and workflows (specialized prompts) — so they can access key information and perform tasks.

The most popular analogy is that MCP is a **USB-C port for AI applications**: just as USB-C standardizes how devices connect to peripherals, MCP standardizes how AI applications connect to external systems. Before MCP, every new integration between a model and a data source or tool required custom code; MCP replaces those many fragmented one-off integrations with a single, universal protocol.

**Why it matters:**

- **Developers:** less development time and complexity when building or integrating with AI applications or agents.
- **AI applications/agents:** access to an ecosystem of data sources, tools, and apps that extends their capabilities and improves the end-user experience.
- **End users:** more capable AI applications that can access user data and take actions on the user's behalf.

Available use cases include personal assistants connected to Google Calendar and Notion, Claude Code generating a web app from a Figma design, enterprise chatbots querying multiple databases, and AI models creating 3D designs in Blender and sending them to a 3D printer.

## 2. Architecture: clients, servers, tools, resources

MCP follows a **client-server architecture**. The key participants are:

- **MCP Host** — the AI application (e.g. Claude Code, Claude Desktop, or Visual Studio Code) that coordinates and manages one or multiple MCP clients.
- **MCP Client** — a component that maintains a connection to an MCP server and obtains context from it for the host to use. The host creates one client per server, and each client maintains a dedicated connection to its corresponding server.
- **MCP Server** — a program that provides context to MCP clients. Servers can run locally (e.g. the filesystem server launched by Claude Desktop over STDIO transport) or remotely (e.g. the official Sentry MCP server over Streamable HTTP transport).

Transport notes: local servers using the STDIO transport typically serve a single client, while remote servers using the Streamable HTTP transport typically serve many clients.

**Two layers make up the protocol:**

- **Data layer:** a JSON-RPC-based protocol for client-server communication, including capability and version discovery and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer:** defines how messages are carried (STDIO, Streamable HTTP, and others).

**Core primitives:**

- **Tools** — server-exposed executables the language model can invoke, each identified by a name and a schema describing its inputs. Tools are model-controlled: the model discovers and invokes them based on its understanding of the prompt. For trust and safety, there should always be a human in the loop able to deny tool invocations; applications should show visual indicators and confirmation prompts for operations.
- **Resources** — server-exposed data that provides context to language models (files, database schemas, application-specific information), each uniquely identified by a URI. Resources are application-driven: the host decides how to incorporate context, e.g. through UI elements, search/filter, or automatic inclusion. Servers may advertise list-change notifications and resource-specific update notifications (subscriptions).
- **Prompts** (bonus) — server-exposed prompt templates that provide structured messages and instructions; clients discover them, retrieve their contents, and supply arguments to customize them. Prompts are user-controlled and typically triggered as slash commands in the UI.

Source pages: https://modelcontextprotocol.io/introduction and https://modelcontextprotocol.io/docs/concepts/architecture (plus https://modelcontextprotocol.io/docs/concepts/tools and https://modelcontextprotocol.io/docs/concepts/resources for the tools/resource details).

## 3. Ecosystem: supporting clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants such as **Claude** and **ChatGPT**, and development tools such as **Visual Studio Code**, **Cursor**, and **MCPJam**, all support MCP — making it easy to build once and integrate everywhere.

The MCP ecosystem also includes:

- **MCP Specification** — the specification outlining implementation requirements for clients and servers.
- **MCP SDKs** — official SDKs for multiple programming languages that implement MCP.
- **MCP Development Tools** — tools for developing MCP servers and clients, including the MCP Inspector.
- **MCP Reference Server Implementations** — reference implementations of MCP servers.

Source page: https://modelcontextprotocol.io/introduction

## 4. Video walkthroughs

These are the picks for new teammates who want a visual introduction before reading further. Each entry is a link with its title, publisher, and duration where the listing shows one.

1. **Why Everyone's Talking About MCP?** — ByteByteGo. Length shown in listing: 05:03. https://www.youtube.com/watch?v=_d0duu3dED4
2. **Claude's Model Context Protocol is here... Let's test it** — Fireship (published 2025-03-31; duration not shown in the listing). https://www.youtube.com/watch?v=HyzlYwjoXOQ
3. **Power up your LLMs: Gemini CLI and Model Context Protocol (MCP)** — Google for Developers (duration not shown in the listing). https://www.youtube.com/watch?v=IYpt25GHB1U
4. **Model Context Protocol (MCP) Explained (+ Example Code)** — Stream Developers (duration not shown in the listing). https://www.youtube.com/watch?v=Dkmfvfr1L4I

## Sources

- MCP official introduction: https://modelcontextprotocol.io/introduction
- MCP architecture overview: https://modelcontextprotocol.io/docs/concepts/architecture
- MCP tools concept page: https://modelcontextprotocol.io/docs/concepts/tools
- MCP resources concept page: https://modelcontextprotocol.io/docs/concepts/resources
- MCP prompts concept page: https://modelcontextprotocol.io/docs/concepts/prompts
