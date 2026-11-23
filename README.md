# MCP Agent Harness

A lightweight, experimental Python harness for building a tool-using AI assistant. It connects an LLM to local Python functions and external tools exposed through the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/).

The project is being developed incrementally to explore the core mechanics behind AI agents: model interaction, function calling, iterative tool execution, MCP discovery, conversation history, and simple persistent memory.

> **Project status:** Early-stage prototype. The current implementation is a command-line application intended for learning and experimentation, not production deployment.

## Current capabilities

- **LLM integration:** Uses the OpenAI Python SDK's chat-completions interface with configurable OpenAI-compatible endpoints.
- **Model configuration:** Keeps model endpoint, model name, and API key configuration in a small registry. The current registry includes local Ollama and cloud-provider examples.
- **Agent execution loop:** Sends tool schemas to the model, executes requested tools, returns tool results, and continues until the model produces a normal response.
- **Local tools:** Includes workspace file listing, reading, and writing, plus a simple memory-save function.
- **MCP integration:** Starts configured MCP servers over stdio, discovers their tools, and makes those tools available to the agent.
- **Conversation context:** Retains the conversation in memory while the CLI process is running.
- **Simple persistent memory:** Loads facts from and appends facts to a Markdown file.
- **CLI commands:** Supports listing configured models and tools, switching models, displaying memory, and exiting.

## Architecture at a glance

```text
User
 |
 v
Command-line interface
 |
 v
Agent loop <----------------------> LLM provider
 |
 | model requests tool call
 v
Tool dispatcher
 |                         |
 v                         v
Local Python tools       MCP client sessions
 |                         |
 v                         v
Workspace / memory       Configured MCP servers
```

At startup, the application reads `mcp_servers.json`, launches each configured MCP server, initializes its session, and discovers the available tools. The discovered tool schemas are added to the schemas sent to the model. When the model requests a tool, the harness routes the call to either a local Python function or the corresponding MCP session and sends the result back into the conversation.

## Repository structure

```text
mcp-agent-harness/
├── harness.py           # Current integrated async harness
├── mcp_servers.json     # MCP server process configuration
├── memory.md            # Example persistent-memory file
├── v0.py                # Basic LLM conversation
├── v1.py                # Initial local function calling
├── v2.py                # Iterative agent/tool execution loop
├── v3.py                # Asynchronous MCP integration
├── v4.py                # Conversation history and simple memory
├── pyproject.toml       # Project metadata and dependencies
└── README.md
```

The numbered `v0.py`–`v4.py` files are learning milestones showing how the implementation evolved. **`harness.py` is the latest integrated version** and the recommended entry point.

## Requirements

- Python 3.13 or newer (as declared in `pyproject.toml`)
- [uv](https://docs.astral.sh/uv/) for managing the project and running MCP server tools
- [Ollama](https://ollama.com/) for the default local model configuration, or credentials and a compatible endpoint for another configured provider
- Network access to download the configured MCP servers the first time they are run

The default local configuration expects Ollama at `http://localhost:11434/v1` and the model `qwen3.5:4b`. Make sure the model is available in your Ollama installation before starting the harness.

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/AshishRaj97/mcp-agent-harness.git
cd mcp-agent-harness
```

### 2. Install dependencies

With uv:

```bash
uv sync
```

Alternatively, install the declared dependencies in a Python 3.13+ environment:

```bash
pip install "mcp>=2.2.0" "openai>=3.10.0" "pydantic>=2.13.5"
```

### 3. Configure a model

The default model is the local Ollama entry named `local`. Start Ollama and ensure `qwen3.5:4b` is available.

For a cloud provider, set the relevant environment variable before running the harness:

**PowerShell**
```powershell
$env:OPENAI_API_KEY="your-api-key"
```

**macOS / Linux**
```bash
export OPENAI_API_KEY="your-api-key"
```

The current registry also contains an Anthropic-compatible endpoint example using `ANTHROPIC_API_KEY`. Verify provider endpoint and model compatibility with your account before selecting that entry. Never commit real API keys.

### 4. Configure MCP servers

Edit `mcp_servers.json` to add or remove MCP server processes. The checked-in example configures:

- `mcp-server-time`
- `mcp-server-fetch`

The harness launches these commands through `uvx`, so uv must be installed and available on your PATH. Only configure MCP servers you trust: an MCP server runs as a local process with the permissions of the user who launched it.

### 5. Run the harness

```bash
uv run python harness.py
```

Or, from an activated environment:

```bash
python harness.py
```

## CLI commands

| Command | Purpose |
| --- | --- |
| `/models` | List configured model entries |
| `/model <name>` | Switch to a configured model entry |
| `/tools` | List available local and discovered MCP tools |
| `/memory` | Display the contents of the memory file |
| `/quit` | Exit the application |

For example, enter `/model local` to use the default Ollama configuration.

## Development history

| Version | Focus |
| --- | --- |
| `v0.py` | Basic conversation with an OpenAI-compatible chat endpoint |
| `v1.py` | First local tools and function-calling integration |
| `v2.py` | Iterative loop for handling tool calls until the model finishes |
| `v3.py` | Async execution and MCP server discovery/invocation |
| `v4.py` | Persistent conversation context and Markdown-based memory |
| `harness.py` | Integration of model configuration, MCP configuration, memory, CLI commands, and async agent execution |

## Current limitations

This repository is an experiment and has not yet been hardened for untrusted or multi-user environments. In particular:

- Workspace file operations currently need path-containment checks and stronger access controls.
- The agent loop does not yet enforce a maximum iteration count or comprehensive execution timeouts.
- Tool argument parsing and tool failures need more robust validation and error handling.
- MCP tool-name collisions across servers need explicit namespacing.
- Conversation history is held in process memory and does not have session isolation or context-window management.
- The Markdown memory store is intentionally simple; it does not yet provide structured retrieval, editing, or deletion.
- Automated tests, observability, and a web/API interface are not included yet.


## Contributing

Suggestions, bug reports, and contributions are welcome. For substantial changes, open an issue first to discuss the approach.

## License

No license has been specified yet. Until a license is added, the repository should be treated as **all rights reserved**; reuse and redistribution are not automatically permitted.
