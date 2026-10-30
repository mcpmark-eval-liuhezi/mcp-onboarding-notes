# MCP Onboarding Notes

A plain-language reference about the Model Context Protocol (MCP), what it is, how
its architecture works, which clients support it today, and a curated selection
of videos for learners.

## 1. What is MCP?

MCP (Model Context Protocol) is an **open-source standard for connecting AI
applications to external systems**. It was created by Anthropic and released in
November 2024. Using MCP, an AI application like Claude, ChatGPT, or an IDE can
connect to:

- **Data sources** — e.g. local files, databases, calendars;
- **Tools** — e.g. search engines, calculators, chatbots;
- **Workflows** — e.g. pre-built prompts or agent flows.

The official documentation describes MCP with a memorable analogy: it is like a
**USB-C port for AI applications**. Just as USB-C provides one standard connector
for a large variety of devices, MCP provides one standard way for AI
applications to connect to a large variety of data sources, tools, and apps,
instead of every AI app building and maintaining its own bespoke connectors.

Concretely, MCP exists because connecting AI models to the outside world used to
be a fragmented, custom-integration problem. MCP standardizes this so that a
model (or an agent) can whatever it needs — query a database, read files,
call an API — through one common protocol, without the application and the
tool/data provider each having to invent a new integration for every pairing
(the well-known "N x M problem").

Two points that matter for a new teammate:

1. **MCP is protocol-layer, not application-layer.** MCP defines *how* context
   is exchanged between AI applications and external systems. It says
   nothing about how the application uses an LLM or manages the provided
   context. See the docs [Scope and concept](#scope-and-concept).
2. **It is not just for Anthropic.** Since launch the protocol has been adopted
   across a wide industry, with support from both AI assistants and
   development tools, and it continues to evolve (multiple protocol versions now
   ship, e.g. the 2026-07-28 revision).

## 2. Architecture: hosts, clients, servers, tools, resources

MCP uses a **client-server architecture**. Understanding who says what to whom
is the single most useful skill when reading the MCP docs or debugging an
integration.

### Participants in the protocol

| Participant | Role |
| --- | --- |
| **MCP Host** | The AI application users interact with (e.g. Claude Desktop, Claude Code, an IDE like VS Code). It coordinates one or more MCP clients and decides how to use the context the clients get back. |
| **MCP Client** | A protocol-level component created by the host. One client connects to one server, and each client maintains its own dedicated connection. The host forks a new client for each server it talks to. |
| **MCP Server** | A program that provides context (data, tools) to MCP clients. Servers can run locally or remotely. |

**Transport matters for layout:**

- **Local servers** use the **STDIO** transport and typically serve a single
  client (e.g. the local filesystem server launched by Claude Desktop).
- **Remote servers** use **Streamable HTTP** and typically serve many clients
  (e.g. the official Sentry MCP server hosted on Sentry's platform).

### Core protocol concepts

The data layer is a **JSON-RPC-based protocol** defining capability and version
discovery plus the core primitives:

- **Tools** — functions the *model* decides when to use, based on what it is
  asked to do. Each tool is identified by a name and includes metadata describing
  its schema (MCP uses JSON Schema for validation). The model invokes them
  through `tools/call` after discovering them via `tools/list`. Tools write to
  databases, call external APIs, or perform computations; users should always
  have the ability to deny their invocation — there is a human in the loop.
- **Resources** — *passive*, application-controlled data that provides
  context to the language model (e.g. a file's contents, a database schema,
  application-specific information, calendars). Each resource is uniquely
  identified by a URI. They are meant to be read-only for the model, but the
  host application decides how to incorporate them (exposing them through UI
  elements, auto-inclusion, etc.).
- **Prompts** — pre-built instruction templates that tell the model how to work
  with specific tools and resources; users invoke them.

### The example from the docs

> An MCP host such as VS Code instantiates one MCP client for each MCP server it
> connects to: one client for a local filesystem server, another for a database
> server, and so on. Each client keeps a dedicated connection to its server, so
> an AI application can holistically harness many servers simultaneously.

### A concrete mental model for a new teammate

```
AI app (user asks: "what's the weather in Tokyo?")
   |
   +-- MCP Client #1 --(devices: get_weather tool -- tools/list + tools/call)--> MCP Server (local, STDIO transport) --> weather API
   |
   +-- MCP Client #2 --(files: project docs)-----------------------------------> MCP Server (local, STDIO transport) --> local file system
   |
   +-- MCP Client #3 --(backlog: issues)---------------------------------------> MCP Server (remote, Streamable HTTP) --> GitHub
```

For reference, the full participant diagram from the official docs pours
concrete examples in: local servers such as Filesystem and remote servers such
as Sentry.

## 3. Ecosystem: MCP clients today

MCP is an **open protocol supported across a wide range of clients and servers**,
making it easy to build once and integrate everywhere.

### AI applications / hosts that implement MCP

- **Claude and ChatGPT** — Anthropic's own applications; ChatGPT added MCP
  support, making MCP the default integration layer for the ChatGPT ecosystem.
- **Visual Studio Code** — the IDE as MCP host; it instantiates a client per
  server (e.g. a local filesystem server, the Sentry MCP server, etc.).
- **Cursor** — an IDE-style coding agent that acts as an MCP client.
- Other promoted client examples in the documentation include Perplexity and
  Windsurf, plus agents such as Goose (launched by Block).

### Development tooling and SDKs

Official SDKs for popular languages are all Tier 1:

| Language | SDK repo | Tier |
| --- | --- | --- |
| TypeScript | modelcontextprotocol/typescript-sdk | Tier 1 |
| Python | modelcontextprotocol/python-sdk | Tier 1 |
| C# | modelcontextprotocol/csharp-sdk | Tier 1 |
| Go | modelcontextprotocol/go-sdk | Tier 1 |
| Rust | modelcontextprotocol/rust-sdk | Tier 1 |
| Ruby | modelcontextprotocol/ruby-sdk | Tier 1 |
| Java | modelcontextprotocol/java-sdk | Tier 2 |
| Swift | modelcontextprotocol/swift-sdk | Tier 3 |
| PHP | modelcontextprotocol/php-sdk | Tier 3 |
| Kotlin | modelcontextprotocol/kotlin-sdk | Tier 3 |

Official development tooling includes the **MCP Inspector**, an interactive
tool for testing and debugging MCP servers in the browser, CLI, or TUI, so any
teammate can inspect what tools, resources, and prompts a server exposes
before wiring it into an application.

The ecosystem's tokens of fidelity: **MCP follows well-known predecessor
protocols in spirit** (API as the web's standard inter-service layer; the Language
Server Protocol for how IDEs talk to language tooling), so the mental models
carry over directly.

## 4. Videos to watch first

Picks are biased toward what a new teammate can handle on day one, with a
couple of deeper-dive talks because MCP is most interesting when you build a
little yourself.

1. **Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic** — *AI Engineer* (YouTube), **1:44:12** (1 h 44 min), published 2025-03-01.
   https://www.youtube.com/watch?v=kQmXtrmQ5Zg
   A full workshop from Mahesh Murag of Anthropic's Applied AI team at the AI
   Engineer Summit 2025. Covers the philosophy behind MCP, why the
   N-times-M integration problem was a pain, the three protocol primitives
   (tools/rRessources/prompts) and who controls each, then a live MCP demo
   (Claude for Desktop + a GitHub integration, plus research/report writing
   agents). Best single "you need to understand MCP" starting point for anyone
   joining a team that builds MCP servers.

2. **What is MCP? Integrate AI Agents with Databases & APIs** — *IBM Technology* (YouTube), **7:58**, published 2025-02-19.
   https://www.youtube.com/watch?v=eur8dUO9mvE
   A short, well-structured teaching video from an enterprise technology
   publisher. Walks the host → client → server → protocol tiers and shows a
   working LLM + MCP server flow end-to-end. Ideal day-1 primer if you want
   only ~8 minutes and a corporate-sounding reference.

3. **30 minutes to understand MCP (Model Context Protocol)** — *Sébastien Blanc* (Devoxx UK 2025 talk), **32m**, 2025-05-07..05-09.
   Conference talk at Devoxx UK 2025, aimed at a large LL[model] / AI-assisted
   development audience; includes a live demo building a remote MCP server and
   invoking tools over HTTP SSE.
   https://confpad.io/conferences/devoxx-uk-2025/30-minutes-to-understand-mcp-model-context-protocol

4. **you need to learn MCP RIGHT NOW!! (Model Context Protocol)** — *NetworkChuck* (YouTube), **38:40**, published 2025-09-12.
   https://www.youtube.com/watch?v=GuTcle5edjk
   A hands-on, high-energy live demo. Uses Docker's MCP toolkit to connect
   Claude Desktop, LM Studio (local models), and Cursor to services, then walks
   through building a custom Kali Linux MCP server from scratch. Great for
   teammates who learn by running things.

### Where to go next

- Official docs: [What is the Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro.md) and [Architecture overview](https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture.md).
- Try it yourself with the [MCP Inspector](https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector.md).

## Sources

- Model Context Protocol — What is the Model Context Protocol (MCP)?
  <https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro.md>
- Model Context Protocol — Architecture overview
  <https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture.md>
- Model Context Protocol — Understanding MCP servers (core server features: tools / resources / prompts)
  <https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md>
- Model Context Protocol — Understanding MCP clients (elicitation, sampling, etc.)
  <https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts.md>
- Model Context Protocol — Resources (URI-identified context provided by servers)
  <https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md>
- Model Context Protocol — SDK tiers (official language SDKs)
  <https://modelcontextprotocol.io/docs/2026-07-28/sdk.md>
- YouTube — "Building Agents with Model Context Protocol - Full Workshop" (AI Engineer, Mahesh Murag of Anthropic)
  <https://www.youtube.com/watch?v=kQmXtrmQ5Zg>
- YouTube — "What is MCP? Integrate AI Agents with Databases & APIs" (IBM Technology)
  <https://www.youtube.com/watch?v=eur8dUO9mvE>
- YouTube — "30 minutes to understand MCP (Model Context Protocol)" (Sébastien Blanc, Devoxx UK 2025)
  https://confpad.io/conferences/devoxx-uk-2025/30-minutes-to-understand-mcp-model-context-protocol
- YouTube — "you need to learn MCP RIGHT NOW!! (Model Context Protocol)" (NetworkChuck)
  <https://www.youtube.com/watch?v=GuTcle5edjk>
