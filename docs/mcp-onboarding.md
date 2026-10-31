# MCP Onboarding Notes

A plain-language reference for new teammates joining the team: what the Model Context Protocol (MCP) is, how its client/server architecture works, which AI applications support it today, and a short selection of videos for getting oriented.

## 1. What MCP Is (Primer)

The Model Context Protocol (MCP) is an open-source standard for connecting AI applications to external systems. Using MCP, AI applications like Claude or ChatGPT can connect to data sources (e.g. local files, databases), tools (e.g. search engines, calculators) and workflows (e.g. specialized prompts), enabling them to access key information and perform tasks.

A useful mental model: MCP is like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems. Instead of building a custom integration for every model-to-tool combination, a service builds one MCP server and every MCP-compatible AI application can use it — "build once, integrate everywhere."

Why it matters by audience:

- **Developers:** MCP reduces development time and complexity when building or integrating with an AI application or agent.
- **AI applications / agents:** MCP gives them access to an ecosystem of data sources, tools and apps, which enhances their capabilities and improves the end-user experience.
- **End users:** MCP results in more capable AI applications or agents that can access user data and take actions on the user's behalf when necessary.

### What MCP can enable

- Agents can access your Google Calendar and Notion, acting as a more personalized AI assistant.
- Claude Code can generate an entire web app using a Figma design.
- Enterprise chatbots can connect to multiple databases across an organization, empowering users to analyze data using chat.
- AI models can create 3D designs on Blender and print them out using a 3D printer.

## 2. Architecture: Clients, Servers, Tools, and Resources

MCP follows a client-server architecture. An **MCP host** — an AI application such as Claude Code, Claude Desktop, or Visual Studio Code — establishes connections to one or more **MCP servers**. The host creates one **MCP client** for each MCP server it connects to, and each client maintains a dedicated connection with its corresponding server.

- **MCP Host:** the AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client:** a component that maintains a connection to an MCP server and obtains context from that server for the host to use. Local MCP servers using the STDIO transport typically serve a single client; remote MCP servers using the Streamable HTTP transport typically serve many clients.
- **MCP Server:** a program that provides context to MCP clients. Servers can execute locally (e.g. the filesystem server launched by Claude Desktop via STDIO) or remotely (e.g. the official Sentry MCP server via Streamable HTTP).

MCP consists of two layers:

- **Data layer:** defines the JSON-RPC-based protocol for client-server communication, including capability and version discovery and core primitives such as tools, resources, prompts and notifications.
- **Transport layer:** defines how messages are carried — typically STDIO for local servers and Streamable HTTP for remote ones.

### Core primitives

- **Tools:** servers expose tools that can be invoked by language models. Tools enable models to interact with external systems, such as querying databases, calling APIs, or performing computations. Each tool is uniquely identified by a name and includes metadata describing its schema. Tools are designed to be model-controlled — the language model can discover and invoke them automatically based on its understanding of context and the user's prompt. For trust and safety, there should always be a human in the loop with the ability to deny tool invocations.
- **Resources:** servers expose resources to share data that provides context to language models, such as files, database schemas, or application-specific information. Each resource is uniquely identified by a URI. Resources are designed to be application-driven — host applications determine how to incorporate context based on their needs (e.g. exposing resources through UI elements, allowing search/filter, or automatic context inclusion).
- **Prompts:** a third core primitive (per the architecture docs) — prompts are reusable template prompts that servers can expose for models to use.

## 3. Ecosystem: Supporting Clients

MCP is an open protocol supported across a wide range of clients and servers. AI assistants like Claude and ChatGPT, and development tools like Visual Studio Code, Cursor, and MCPJam, all support MCP. This broad ecosystem support makes it easy to build once and integrate everywhere.

Notable examples cited in the official documentation:

- **Claude / Claude Desktop / Claude Code** — first-party MCP clients; Claude Desktop can launch local MCP servers such as the filesystem server.
- **ChatGPT** — an AI assistant that can connect to data sources, tools, and workflows via MCP.
- **Visual Studio Code** — acts as an MCP host; when it connects to a server such as the Sentry MCP server or a local filesystem server, the runtime instantiates a dedicated MCP client object for that connection.
- **Cursor, Windsurf, Goose, Zed, and others** — other development tools and agents that connect to MCP servers.
- **8,000+ MCP servers** — as of recent talks, there are over 8,000 MCP servers available (e.g. for GitHub, Slack, filesystem access, Figma, Stripe, Supabase), covering most common services.

For developers: MCP SDKs exist for multiple programming languages (Python, JavaScript, Java, C#, Go, and others), so you can start building your own MCP servers in a language you already know.

## 4. Video Walkthroughs

A short selection of introduction videos — conference talks, tutorials, and live demos — to get oriented with MCP. Durations are noted where the video listing shows one; otherwise the listing did not display a duration.

1. [MCP Explained | Model Context Protocol Tutorial | How Model Context Protocol Works?](https://www.youtube.com/watch?v=znNuYwtMryY) — Simplilearn — 12:14 (length shown on the listing).
2. [Full Course (Lessons 1-11) MCP for Beginners](https://www.youtube.com/watch?v=VfZlglOWWZw) — Microsoft Developer — duration not shown on the listing; a full beginner course covering host/client/server, tools, resources, and prompts.
3. [What Are MCP Servers? Complete Beginner Guide + Real Example](https://www.youtube.com/watch?v=3cKwANLhOo8) — Noah Does Coding (NDC talk) — duration not shown on the listing; a hands-on live demo connecting the GitHub MCP server to Visual Studio Code and Claude Code.
4. [Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg) — AI Engineer (talk by Mahesh Murag of Anthropic's applied AI team) — duration not shown on the listing; a conference talk covering the philosophy behind MCP, adoption, and the roadmap.

## Sources

Documentation pages actually used for this note:

- [What is the Model Context Protocol (MCP)?](https://modelcontextprotocol.io/introduction) — official MCP introduction (what MCP is, why it matters, ecosystem support).
- [Architecture overview](https://modelcontextprotocol.io/docs/concepts/architecture) — official architecture docs (host/client/server participants, layers, transports, primitives).
- [Tools](https://modelcontextprotocol.io/docs/concepts/tools) — official tools concept page (model-controlled invocation, human-in-the-loop guidance).
- [Resources](https://modelcontextprotocol.io/docs/concepts/resources) — official resources concept page (URI-identified context, application-driven interaction, listChanged/subscribe capabilities).
- [What is the Model Context Protocol (MCP)? (clients page)](https://modelcontextprotocol.io/clients) — official clients/ecosystem page (Claude, ChatGPT, VS Code, Cursor, MCPJam).

Video metadata was gathered from the respective YouTube listings (Simplilearn listing showed length 12:14; other listings did not display a duration).
