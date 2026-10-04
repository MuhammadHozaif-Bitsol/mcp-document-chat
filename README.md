# MCP Document Chat

A command-line chat app that lets Claude read and edit a set of documents through the [Model Context Protocol (MCP)](https://modelcontextprotocol.io). It includes both sides of MCP: a small **MCP server** that exposes documents as tools, resources and prompts, and an **MCP client** that connects the server to Claude.

Built as a hands-on project for learning how MCP servers and clients fit together.

## Features

- **Chat with Claude** in your terminal.
- **`@` mentions**: type `@` to pick a document. Its contents are fetched and sent to Claude with your question.
- **`/` commands**: run a prompt defined by the server, such as `/format plan.md`.
- **Tool use**: Claude can read and edit documents on its own by calling the server's tools. Each tool call is printed as it happens.
- **Extra servers**: pass more MCP server scripts on the command line and their tools become available to Claude too.

## What the MCP server exposes

`mcp_server.py` holds a few in-memory sample documents and exposes them in all three MCP forms:

| Type | Name | What it does |
|---|---|---|
| Tool | `read_doc_contents` | Returns the text of a document |
| Tool | `edit_document` | Replaces a piece of text in a document |
| Resource | `docs://documents` | Lists all document ids (JSON) |
| Resource | `docs://documents/{doc_id}` | Returns one document's text |
| Prompt | `format` | Asks Claude to rewrite a document in Markdown |

## How it works

```
You ──> CLI (prompt_toolkit) ──> Chat loop ──> Claude API
                                   │   ▲
                                   ▼   │
                               MCP client ──stdio──> MCP server (documents)
```

- `@plan.md`: the client reads the `docs://documents/plan.md` resource and adds the text to your message.
- `/format plan.md`: the client gets the `format` prompt from the server, filled in for `plan.md`, and adds it to the conversation.
- When Claude asks for a tool, the client calls it on the server and sends the result back, looping until Claude has a final answer.

## Project structure

```
main.py           Entry point: loads config, starts MCP clients and the CLI
mcp_server.py     MCP server: documents, tools, resources and prompts
mcp_client.py     MCP client: wraps a session to one server
core/
  cli.py          Terminal UI, with autocomplete for @ and /
  cli_chat.py     Handles @ mentions and / commands
  chat.py         Chat loop with Claude, including tool calls
  claude.py       Thin wrapper around the Anthropic API
  tools.py        Finds and runs tools across all connected servers
```

## Requirements

- Python 3.10+
- [uv](https://github.com/astral-sh/uv) (recommended)
- An [Anthropic API key](https://console.anthropic.com/)

## Setup

1. Clone the repo:

   ```bash
   git clone https://github.com/MuhammadHozaif-Bitsol/mcp-document-chat.git
   cd mcp-document-chat
   ```

2. Create a `.env` file in the project root:

   ```
   ANTHROPIC_API_KEY="your-api-key"
   CLAUDE_MODEL="claude-sonnet-5-5"
   USE_UV=1
   ```

   `USE_UV=1` starts the MCP server with `uv run`. Leave it out to start it with plain `python`.

3. Install dependencies:

   ```bash
   uv sync
   ```

   Without uv:

   ```bash
   python -m venv .venv
   .venv\Scripts\activate          # macOS/Linux: source .venv/bin/activate
   pip install -e .
   ```

## Usage

Run the app in a real terminal, such as PowerShell, cmd or Windows Terminal on Windows. The interactive prompt needs a console window.

```bash
uv run main.py
```

Ask a question:

```
> What is in the report?
```

Mention a document with `@`:

```
> Tell me about @deposition.md
```

Run a server prompt with `/` (press Tab to autocomplete):

```
> /format plan.md
```

Connect extra MCP servers by passing their scripts:

```bash
uv run main.py other_server.py
```

### Inspecting the server

Use the MCP Inspector to try the server's tools, resources and prompts in your browser:

```bash
uv run mcp dev mcp_server.py
```

## Adding documents

Add entries to the `docs` dictionary in `mcp_server.py`. The key is the document id and the value is its text. Documents live in memory, so edits are lost when the app exits.
