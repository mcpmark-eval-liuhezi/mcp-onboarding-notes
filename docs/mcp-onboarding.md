# MCP Onboarding Notes

Onboarding reference for new team members joining the AI applications practice. This document covers what the Model Context Protocol (MCP) is, how its client/server architecture works, which applications support it today, and a curated set of video walkthroughs.

The document lives here: `docs/mcp-onboarding.md`.

---

## 1. Plain-Language Primer: What Is MCP?

The Model Context Protocol (MCP) is an **open-source standard for connecting AI applications to external systems**.

Think of it as a **USB-C port for AI applications**: just as USB-C gives you one standardized, widely supported way to plug an electronic device into your phone, charger, or monitor, MCP gives AI applications one standardized way to plug into data sources, tools, and workflows — instead of every integration being built from scratch as a bespoke connector.

In practice, MCP lets an AI application like Claude, ChatGPT, or a coding agent connect to:

- **Data sources** — local files, databases, calendars, knowledge bases, APIs
- **Tools** — search engines, calculators, code interpreters, CRM systems, design tools
- **Workflows** — pre-defined prompts and templates that drive specific tasks

The official documentation summarizes the value proposition as triple-benef.edge for developers, applications/agents, and end users: developers reduce development time and complexity; AI applications and agents gain access to an ecosystem of data, tools, and apps; and end users get more capable applications that can act on their behalf when needed.

### Key takeaways

- MCP is an **open protocol** and standard, not an Anthropic-only API (the spec and SDKs are open; Anthropic open-sourced the original protocol in 2024).
- It follows the **client-server architecture** pattern: an MCP host application owns one or more MCP clients, and each client connects to one MCP server.
- Most of the value comes from **standardization**: build an integration once, expose it as an MCP server, and any MCP client can use it. Connect to any server with zero additional work.

---

## 2. Architecture: Clients, Servers, Tools, and Resources

### Participants

MCP follows a client-server architecture with three key participant roles.

- **MCP Host** — The AI application that coordinates and manages one or more MCP clients (for example, Claude Code, Claude Desktop, or Visual Studio Code).
- **MCP Client** — A protocol-level component created by the host. The host instantiates one client **per server**, and each client maintains a dedicated, one-to-one connection with its corresponding MCP server. That's why the server list can include one client per server even when a single host connects to several servers.
- **MCP Server** — A program that provides context and capabilities to MCP clients. MCP servers can run **locally** (using the STDIO transport, typically serving a single client) or **remotely** (using the Streamable HTTP transport, typically serving many clients).

Example flow: Visual Studio Code acts as an MCP host. When it connects to a local filesystem server, the VS Code runtime instantiates a dedicated client object connected to that server; connecting to a remote server such as Sentry instantiates another client object with its own connection.

### Layers

MCP consists of two layers:

- **Data layer** — A JSON-RPC 2.0 based exchange protocol defining the message structure and semantics: capability and version discovery, and core primitives such as tools, resources, prompts, and notifications.
- **Transport layer** — Defines the communication channels that carry data between clients and servers: connection establishment, message framing, authorization, and secure communication. Two transports are supported:
  - **Stdio transport** — standard input/output streams for direct process communication between local processes on the same machine.
  - **Streamable HTTP transport** — for remote servers, serving many clients over HTTP.

### Core server primitives

Servers expose capabilities, and the three core building blocks are:

| Primitive | Who controls it | What it is | Examples |
| --- | --- | --- | --- |
| **Tools** | The model | Functions the language model can actively call to perform actions — query databases, call APIs, write files, trigger logic | Search flights, send messages, create calendar events |
| **Resources** | The application | Read-only data sources that provide context to the model, identified by URIs | File contents, database schemas, API documentation, knowledge bases |
| **Prompts** | The user | Pre-built instruction templates that tell the model how to work with specific tools and resources | "Plan a vacation", "Summarize my meetings" |

**Tools** are the protocol's most-used primitives: servers declare the `tools` capability and respond to `tools/list` with name and metadata (including a JSON Schema `inputSchema`), then are invoked via `tools/call`. Tools are *model-controlled* — the model decides when to invoke them based on context — and the protocol expects a human in the loop with the ability to deny invocations (confirmations, visible indicators, etc.).

**Resources** are *application-controlled* read-only context: servers declare the `resources` capability (with optional `listChanged` and `subscribe` behaviors), identify data by URI, and servers may notify clients when a resource changes.

**Prompts** are per-user templates. Servers may also request information from users during interactions via **elicitation** (`elicitation/create`), letting a server ask for structured input on demand.

> Note: capability negotiation happens at connection startup; servers and clients declare what they support, and the resulting interface drives how tools, resources, prompts, and notifications are exchanged.

---

## 3. Ecosystem: Clients That Support MCP Today

According to the official introduction documentation, MCP is supported across a broad ecosystem of clients and servers. Three ecosystem categories are worth knowing as a new team member;

1. **AI assistants** — applications like **Claude** and **ChatGPT** can act as MCP hosts/clients and connect to MCP servers for tools and data.
2. **Development tools** — IDEs and coding environments such as **Visual Studio Code** and **Cursor** support MCP, letting developers connect servers for file system access, debugging, and other specialized triggers.
3. **Third-party and emerging clients** — the protocol's open, standardized nature means new clients and servers ship frequently; the official documentation points to the MCP Inspector and reference server implementations as development tooling.

### How to verify currently-supported clients

Client support changes frequently as the ecosystem matures. The authoritative current list is the official docs' **clients** page and the **SDKs page**, which list supported clients and language SDKs (Python, JavaScript/TypeScript, Java, Kotlin, C#, Ruby, Swift, Go, Rust, and more). When in doubt, check the latest versioned docs at `https://modelcontextprotocol.io()` to confirm what your team's tooling currently supports before assuming.

### Developer tooling worth knowing

- **MCP Inspector** — interactive developer tooling (browser, CLI, and TUI tabs) for testing and debugging MCP servers; includes CI recipes and configuration reference.
- **Reference server implementations** — official examples (filesystem, GitHub, etc.) you can connect to locally over STDIO or remotely over Streamable HTTP.
- **SDKs** — language-specific bindists so teams write servers/clients in their preferred language rather than hand-rolling JSON-RPC plumbing.

---

## 4. Video Walkthroughs

A curated selection for different learning styles and depths. Each entry is a link to the YouTube listing, with title, publisher, and duration where the listing shows one.

### Beginner overview (short walkthroughs)

- **[Model Context Protocol (MCP) Explained in 20 Minutes](https://www.youtube.com/watch?v=9SnF7fSw1aE)** — **Shaw Talebi**. ~20 minutes (per the listing title). A compact introduction to what MCP is, how its client/server model works, and a hands-on demo building a custom server with Python and connecting it to Claude Desktop.
  - Note: the "20 minutes" figure comes from the listing title; the full video is the channel's free video course.

- **[MCP Explained | Model Context Protocol Tutorial | How Model Context Protocol Works?](https://www.youtube.com/watch?v=znNuYwtMryY)** — **Simplilearn**. Duration not listed in the listing I retrieved; the related free course (simplilearn.com/mcp-free-course-skillup) totals 2 hours of self-paced lessons, and its "Understanding MCP Architecture" lesson alone runs 26:51. The intro video covers the USB-C analogy, the client/server model, and what tools, resources, and prompts look like in practice.

### Deep dives and conference talks

- **[Building Agents with Model Context Protocol — Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)** — **AI Engineer** (Symposium / conference workshop), delivered by **Mahesh Murag of Anthropic's Applied AI team**. **1:44:12**. The definitive deep-dive from the protocol's own team: the philosophy behind MCP, the N×M integration problem, the three primitives (tools, resources, prompts) and who controls each, sampling and elicitation, remote servers with OAuth, composability, and the roadmap then in flight (MCP Registry API, etc.). Also available via Class Central and AIE Talks with a detailed chapter breakdown.

- **[MCP Architecture | Model Context Protocol Architecture (MCP Trilogy, Part 2)](https://learnwith.campusx.in/courses/Model-Context-Protocol-6914562196552f5a888a3acc)** — **CampusX**. **77:00**. Part of the free "MCP Trilogy" playlist; the architecture walkthrough covers servers, clients, and hosts and how they interact, followed by build-and-deploy examples (making it a good follow-up to the Shaw Talebi intro).

- **[Full Course (Lessons 1-11) MCP for Beginners](https://www.youtube.com/watch?v=VfZlglOWWZw)** — **Microsoft Developer**. Duration not listed in the listing I retrieved. An official vendor full course covering client-server architecture, containers (MCP and containers), debugging, and building an MCP server, with code tracked across Java, JavaScript, and Python.

### Remarks on choosing a starting point

- New to the protocol? Start with the **MCP Explained (Simplilearn)** or **Explained in 20 Minutes (Shaw Talebi)** video for the mental model, then the **Mahesh Murag (Anthropic)** workshop for the authoritative deep-dive.
- If you want a hands-on build from a vendor perspective, the **Microsoft Developer** series and the **CampusX (77:00)** architecture lecture pair well together.

---

## Sources

The content in this document was drawn from the official Model Context Protocol documentation, which I read during preparation:

- **What is the Model Context Protocol (MCP)?** — https://modelcontextprotocol.io/introduction
- **Architecture overview** — https://modelcontextprotocol.io/docs/concepts/architecture (JSON-RPC 2.0 data/transport layering, host/client/server roles, STDIO vs. Streamable HTTP, tools, resources, prompting, elicitation).
- **Tools** — https://modelcontextprotocol.io/docs/concepts/tools (`tools/list`, `tools/call`, JSON Schema validation, human-in-the-loop expectations).
- **Resources** — https://modelcontextprotocol.io/docs/concepts/resources (`resources/list`, URI-addressable context, read-only application-controlled data).
- **Understanding MCP servers** — https://modelcontextprotocol.io/docs/2025-11-25/learn/server-concepts.md (tools, resources, prompts mapping and examples).
- **Understanding MCP clients** — https://modelcontextprotocol.io/docs/2025-11-25/learn/client-concepts.md (host vs. client distinction, elicitation, roots, sampling).
- **Documentation index** — https://modelcontextprotocol.io/llms.txt (used to discover the current docs structure and pages).

Video listings were verified against YouTube listing metadata and the Class Central / AI Talks metadata aggregators noted above. Where a listing did not show a duration, the document says so rather than estimating.
