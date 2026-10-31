# MCP Onboarding Notes

A plain-language note on the **Model Context Protocol (MCP)** for new teammates. It covers what MCP is, how its client/server architecture works, which AI applications support it, and a short picks list of introduction videos.

## 1. What MCP Is — A Primer

The Model Context Protocol (MCP) is an **open-source standard for connecting AI applications to external systems**. In practice, it lets an AI application such as Claude or ChatGPT reach data sources (local files, databases), tools (search engines, calculators), and workflows (specialized prompts) so it can retrieve key information and perform tasks.

A useful mental model: **MCP is like a USB-C port for AI applications**. Just as USB-C standardizes how devices connect, MCP standardizes how AI applications connect to external systems — so that whatever AI app you use, you can point it at the same set of data sources, tools, and apps.

Why it matters by audience:

- **Developers**: MCP reduces development time and complexity when building or integrating with AI applications or agents, because you write one integration per system rather than one per AI app.
- **AI applications / agents**: gain access to an ecosystem of data sources, tools, and apps, which enhances their capabilities and the end-user experience.
- **End users**: get more capable AI applications that can access their data and take actions on their behalf.

Real-world capabilities already enabled include agents reading your Google Calendar and Notion, Claude Code generating a web app from a Figma design, enterprise chatbots querying multiple databases, and AI models creating 3D designs in Blender and sending them to a 3D printer.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a **client-server architecture** with three key participants:

- **MCP Host** — the AI application that coordinates and manages one or multiple MCP clients (e.g., Claude Code, Claude Desktop, Visual Studio Code).
- **MCP Client** — a component that maintains a connection to an MCP server and obtains context from it for the host to use. The host creates one client per server, and each client keeps a dedicated connection with its server.
- **MCP Server** — a program that provides context to MCP clients. Servers can run locally (using the STDIO transport, typically serving a single client) or remotely (using the Streamable HTTP transport, typically serving many clients).

Example from the docs: when Visual Studio Code connects to the Sentry MCP server, the runtime instantiates an MCP client for that connection; connecting to a local filesystem server instantiates another client. Each client and server pair has a dedicated connection.

**Two layers make up the protocol:**

- **Data layer** — defines the JSON-RPC-based protocol for client-server communication, including capability and version discovery and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer** — defines how messages travel; STDIO for local servers, Streamable HTTP for remote servers.

The core primitives servers expose:

- **Tools** — capabilities that let models interact with external systems (querying databases, calling APIs, performing computations). Each tool has a name and schema metadata. Tools are model-controlled: the language model discovers and invokes them based on context and the user's prompt, though implementations may expose them through any interface pattern. A human should always stay in the loop so users can deny tool invocations.
- **Resources** — standardized way for servers to share data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are application-driven: the host decides how to incorporate context (UI selection, search/filter, or automatic inclusion).
- **Prompts** (bonus) — structured message templates a server provides; users can explicitly select and invoke them, typically as slash commands.

## 3. Ecosystem: Supporting Clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like **Claude** and **ChatGPT** support it, as do development tools such as **Visual Studio Code**, **Cursor**, and **MCPJam**, among many others.

The broader MCP ecosystem includes:

- **MCP specification** — the standard outlining implementation requirements for clients and servers.
- **MCP SDKs** — language SDKs that implement MCP (e.g., Python and JavaScript SDKs).
- **MCP development tools** — including the MCP Inspector for debugging servers.
- **MCP reference server implementations** — reference servers for common systems.

The payoff: build once, integrate everywhere. Once a client is MCP-compatible, it can connect to any MCP server with zero additional work, and server providers can build one server and see adoption across all MCP clients.

## 4. Video Picks

Introduction videos that are genuinely useful for onboarding — each entry is a link with its title, publisher, and duration where the listing shows one.

1. **[Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=N3vHJcHBS-w)** — Published by Shaw Talebi (ShawhinT). Duration listed on the video's own title as ~20 minutes; the video is labelled "Explained in 20 Minutes" with timestamps covering what MCP is, how it works, MCP clients and servers, a Python MCP server example, and connecting to Claude Desktop. Good all-round primer with a hands-on section.
2. **[Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)** — Published by the AI Engineer channel. Mahesh Murag of Anthropic's applied AI team gives a workshop-style talk on the philosophy behind MCP, the N-times-M integration problem it solves, the role of clients/servers/tools/resources, and the roadmap. Published 2025-03-01; no duration shown on the listing.
3. **[Model Context Protocol (MCP) Explained for Beginners: AI Flight Booking Demo!](https://www.youtube.com/watch?v=E2DEHOEbzks)** — Published by KodeKloud. A beginner-friendly visual explainer of agents, tools, and the MCP protocol, ending with a flight-booking demo. Published 2025-12-25; no duration shown on the listing.
4. **[Model Content Protocol Course from Hugging Face - MCP Course Introduction](https://www.youtube.com/watch?v=p4q6LI-2yZ8)** — Published by Hugging Face. An onboarding video for the Hugging Face MCP course, covering the tools and platforms used to build MCP applications end to end. No duration shown on the listing.

## Sources

Documentation consulted for this note (all modelcontextprotocol.io):

- **Introduction — What is the Model Context Protocol (MCP)?**: https://modelcontextprotocol.io/
- **Architecture overview** (host/client/server participants, layers, transports): https://modelcontextprotocol.io/docs/concepts/architecture
- **Tools** (model-controlled invocation, schema, human-in-the-loop): https://modelcontextprotocol.io/docs/concepts/tools
- **Resources** (URI-based context, application-driven): https://modelcontextprotocol.io/docs/concepts/resources
- **Prompts** (server-exposed template messages): https://modelcontextprotocol.io/docs/concepts/prompts

Video sources are linked inline in Section 4; publication dates and durations were taken from the respective YouTube listings where available.
