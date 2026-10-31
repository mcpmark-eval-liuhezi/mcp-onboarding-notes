# MCP Onboarding Notes

This note is for new teammates joining the team. It covers what the Model Context Protocol (MCP) is, how its client/server architecture works, which AI applications support it today, and a short selection of videos to get up to speed. The document lives at `docs/mcp-onboarding.md` in this repository.

## 1. What MCP is (plain-language primer)

MCP (Model Context Protocol) is an **open-source standard for connecting AI applications to external systems**. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts) — enabling them to access key information and perform tasks.

The官方 analogy from the documentation is that MCP is like **"a USB-C port for AI applications"**: just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. Instead of building a custom integration for every model-to-tool pairing, you build once against the standard and integrate everywhere.

### What MCP can enable in practice

- Agents can access your Google Calendar and Notion, acting as a more personalized AI assistant.
- Claude Code can generate an entire web app using a Figma design.
- Enterprise chatbots can connect to multiple databases across an organization, empowering users to analyze data using chat.
- AI models can create 3D designs on Blender and print them out using a 3D printer.

### Why it matters for our team

- **Developers:** MCP reduces development time and complexity when building or integrating with an AI application or agent.
- **AI applications or agents:** MCP gives them access to an ecosystem of data sources, tools and apps, enhancing capabilities and improving the end-user experience.
- **End-users:** MCP results in more capable AI applications that can access user data and take actions on the user's behalf when necessary.

## 2. Architecture: clients, servers, tools, and resources

MCP follows a **client-server architecture**. An MCP host — an AI application like Claude Code or Claude Desktop — establishes connections to one or more MCP servers, by creating one MCP client for each MCP server. Each client maintains a dedicated connection with its corresponding server.

### Key participants

- **MCP Host:** The AI application that coordinates and manages one or multiple MCP clients (e.g. Visual Studio Code, Claude Code, Claude Desktop).
- **MCP Client:** A component that maintains a connection to an MCP server and obtains context from that server for the host to use. Local servers that use the STDIO transport typically serve a single client; remote servers using the Streamable HTTP transport typically serve many clients.
- **MCP Server:** A program that provides context to MCP clients. Servers can run locally (e.g. the filesystem server launched by Claude Desktop via STDIO) or remotely (e.g. the official Sentry MCP server via Streamable HTTP).

### Two layers

- **Data layer:** Defines the JSON-RPC based protocol for client-server communication, including capability and version discovery and core primitives such as tools, resources, prompts and notifications.
- **Transport layer:** Defines how messages are transported (STDIO for local, Streamable HTTP for remote).

### Core primitives: tools and resources

- **Tools:** Servers expose tools that can be invoked by language models. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are designed to be model-controlled — the model discovers and invokes them based on its contextual understanding and the user's prompts — though implementations may expose them through any suitable interface. Servers that support tools must declare the `tools` capability and respond to `tools/list` requests. For security, a human should always be able to deny tool invocations; applications should show visual indicators when tools are invoked and present confirmation prompts.
- **Resources:** Servers provide resources — a standardized way to share data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are designed to be application-driven (the host decides how to incorporate context, e.g. via UI selection, search/filter, or automatic inclusion). Servers that support resources must declare the `resources` capability and respond to `resources/list` requests; the capability supports optional `listChanged` and `subscribe` features for update notifications.

> Note: MCP focuses solely on the protocol for context exchange — it does not dictate how AI applications use LLMs or manage the provided context.

## 3. Ecosystem: supporting clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, development tools like Visual Studio Code, Cursor, and MCPJam, and many others all support MCP — making it easy to build once and integrate everywhere.

Selected clients from the official MCP client support matrix (with their feature support for resources, prompts, tools, sampling, and roots):

| Client | Resources | Prompts | Tools | Sampling | Roots |
| --- | --- | --- | --- | --- | --- |
| Claude Desktop App | ✅ | ✅ | ✅ | ❌ | ❌ |
| Claude Code | ❌ | ✅ | ✅ | ❌ | ✅ |
| Cursor | ❌ | ❌ | ✅ | ❌ | ❌ |
| VS Code (GitHub Copilot) | ❌ | ❌ | ✅ | ❌ | ✅ |
| Windsurf Editor | ❌ | ❌ | ✅ | ❌ | ❌ |
| Zed | ❌ | ✅ | ❌ | ❌ | ❌ |
| Cline | ✅ | ❌ | ✅ | ❌ | ❌ |
| Continue | ✅ | ✅ | ✅ | ❌ | ❌ |
| Microsoft Copilot Studio | ❌ | ❌ | ✅ | ❌ | ❌ |
| Roo Code | ✅ | ❌ | ✅ | ❌ | ❌ |

Other notable MCP clients include Copilot-MCP, Goose, Sourcegraph Cody, LibreChat, mcp-agent, oterm, Superinterface, TheiaAI/TheiaIDE, Genkit, GenAIScript, Emacs Mcp, fast-agent, Daydreams Agents, SpinAI, Witsy, sling, plus CLI/agent tools such as Codex CLI, Gemini CLI, and Open WebUI. The full current matrix lives in the official MCP docs repository (`clients.mdx`).

## 4. Video walkthroughs

Picks for onboarding, mixing conference talks, workshops, and beginner tutorials. Durations are noted where the listing shows one.

1. **Analytics Vidhya — "Model Context Protocol(MCP) Explained for Beginners in Python (with multiple examples)!"** ([youtube.com/watch?v=LYfr7qusVSs](https://www.youtube.com/watch?v=LYfr7qusVSs)) — Tutorial with hands-on examples; length shown as **21:19**.
2. **AI Engineer — "Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic"** ([youtube.com/watch?v=kQmXtrmQ5Zg](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)) — Conference-style workshop talk by Mahesh Murag of Anthropic's applied AI team on the philosophy and adoption of MCP; duration not listed in the search result.
3. **PyCon AU 2025 — "MCP for dummies: Feed more context to LLM" (Tarun Jain)** ([youtube.com/watch?v=1uZl3e8GdsU](https://www.youtube.com/watch?v=1uZl3e8GdsU)) — Conference talk covering what MCP is, the architecture and transport mechanism, and security, with code demos; duration not listed in the search result.
4. **Hugging Face — "Model Context Protocol Course from Hugging Face - MCP Course Introduction"** ([youtube.com/watch?v=p4q6LI-2yZ8](https://www.youtube.com/watch?v=p4q6LI-2yZ8)) — Onboarding video for Hugging Face's beginners-to-advanced MCP course; duration not listed in the search result.

## Sources

- [What is the Model Context Protocol (MCP)? — official introduction](https://modelcontextprotocol.io/introduction)
- [Architecture overview — official docs](https://modelcontextprotocol.io/docs/concepts/architecture)
- [Tools — official docs](https://modelcontextprotocol.io/docs/concepts/tools)
- [Resources — official docs](https://modelcontextprotocol.io/docs/concepts/resources)
- [MCP clients overview (feature support matrix) — modelcontextprotocol/docs repo](https://github.com/modelcontextprotocol/docs/blob/main/clients.mdx)
- [Analytics Vidhya video listing](https://www.youtube.com/watch?v=LYfr7qusVSs) (runtime 21:19 shown on the listing)
