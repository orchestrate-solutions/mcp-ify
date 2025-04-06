# 🧱 Turn Any Library Into a Model Context Protocol (MCP) Server
## Instantly turn any library into a Model Context Protocol server — tools, resources, prompts, and Docker-ready.

Cursor TL;DR: Go to your repo, define what you want to mcp-ify and give the model this prompt. 
```
# 🧱 Turn Any Library Into a Model Context Protocol (MCP) Server ## Instantly turn any library into a Model Context Protocol server — tools, resources, prompts, and Docker-ready. > **Purpose:** This guide helps you transform *any* function, script, module, or API into an **MCP-compliant server** — making it accessible to LLM-powered clients like Claude, Continue, fast-agent, Genkit, and others. Whether it’s a Python package, a CLI tool, or a business logic module, you’ll learn how to wrap it using the Model Context Protocol so that models can *discover*, *understand*, and *invoke* it using a standard interface. --- ## 🧠 Why MCP? (And Why Wrap Your Library) LLMs don’t operate in a vacuum — they need to: - Access real-world data (`resources`) - Run actions or business logic (`tools`) - Follow workflows or playbooks (`prompts`) But everyone implements this differently. The **Model Context Protocol** solves this by defining **a universal schema and transport layer** to connect models to capabilities and context. > Think of MCP as the *“USB for LLM integrations”* — plug in once, and any compliant AI agent can use it. By wrapping your library as an MCP server, you gain: - ✅ **Standardized interface** for any client - ✅ **Modular reuse** across teams or tools - ✅ **LLM discoverability** and auto-documentation - ✅ **Scoped, secure access** via protocols and URIs --- ## 🔧 What Can You Wrap? | You Have… | Wrap It As… | Example Outcome | |---------------------------|---------------|---------------------------------------------| | A Python/Node function | Tool | Model can call `calculate()` with arguments | | A config or log file | Resource | Model can read and analyze it | | A prompt-based template | Prompt | Model can run “Summarize this bug” flow | | A REST or internal API | Tool + Resource | Model can query or inspect endpoints | | A shell script | Tool | Trigger CLI logic via LLM | --- ## 📦 Step-by-Step: Wrapping Your Code with MCP ### 🥇 Step 1: Choose Your SDK Install your language SDK: ```bash # Python pip install mcp-sdk ``` ```bash # TypeScript npm install @modelcontextprotocol/sdk ``` --- ### 🥈 Step 2: Scaffold Your Server Here's an example in Python: ```python from mcp_sdk.server import Server from mcp_sdk.types import ListToolsRequest, CallToolRequest def calculate_sum(x, y): return x + y server = Server(name="calc-server", version="0.1", capabilities={"tools": {}}) @server.on(ListToolsRequest) async def list_tools(_): return { "tools": [{ "name": "calculate_sum", "description": "Adds two numbers", "inputSchema": { "type": "object", "properties": {"x": {"type": "number"}, "y": {"type": "number"}}, "required": ["x", "y"] } }] } @server.on(CallToolRequest) async def call_tool(req): args = req.params["arguments"] result = calculate_sum(args["x"], args["y"]) return {"content": [{"type": "text", "text": str(result)}]} ``` --- ### 🥉 Step 3: Run the Server (stdio) ```bash python server.py ``` This starts your server over stdin/stdout — which works perfectly with local MCP clients like Claude Desktop, Continue, or fast-agent. --- ### 🧪 Step 4: Inspect with MCP Inspector ```bash npx -y @modelcontextprotocol/inspector python server.py ``` Use this to verify your: - Tool schema - Response shape - Errors, logs, and inputs --- ### 🧩 Step 5: Add to Your Client (Claude, Continue, etc.) For Claude Desktop, in `claude_desktop_config.json`: ```json { "mcpServers": { "calc": { "command": "python", "args": ["C:/Users/yourname/server.py"] } } } ``` For macOS/Linux: ```json { "mcpServers": { "calc": { "command": "python", "args": ["/Users/you/dev/server.py"] } } } ``` --- ## 🐳 Docker + MCP: Clean, Portable Wrapping ### Why Docker? Docker is ideal when: - Your code has native dependencies - You want cross-platform reproducibility - You need to ship remote SSE servers ### Example Dockerfile: ```Dockerfile FROM python:3.11-slim WORKDIR /app COPY server.py . RUN pip install mcp-sdk CMD ["python", "server.py"] ``` ```bash docker build -t mcp-calc . docker run -i mcp-calc ``` > Claude can use this via `"command": "docker", "args": ["run", "-i", "mcp-calc"]` --- ## 💻 OS Differences: macOS / Windows / Linux ### Working Directory | OS | Behavior | Solution | |----------|--------------------------------|-------------------------------| | macOS | GUI apps launch from `/` | Use absolute paths | | Windows | GUI apps or shell may vary | Use full `C:/...` paths | | Linux | Launcher sets working dir | Still use absolute paths | ### File Paths - macOS/Linux: `file:///Users/you/path.txt` - Windows: `file:///C:/Users/you/path.txt` Use this to normalize: ```python from urllib.parse import pathname2url from pathlib import Path def to_uri(path): return "file://" + pathname2url(str(Path(path).absolute())) ``` ### Environment Variables Use `"env"` block in your config: ```json { "env": { "API_KEY": "abc123", "ROOT_PATH": "/Users/you/dev" } } ``` ### Docker Quirks | Platform | Quirk | Fix | |----------|-------------------------------|----------------------------------------| | macOS | Docker runs inside VM | Avoid local volume mounts by default | | Windows | Paths inside container = WSL | Use `/mnt/c/...` or use COPY instead | | Linux | Native support | ✅ No changes needed | --- ## 🧠 Prompts & Resources (Optional Features) ### 📄 Expose a Resource (e.g., File or API) ```python @server.on(ListResourcesRequest) async def list_resources(_): return {"resources": [{ "uri": "file:///tmp/output.txt", "name": "Output File", "mimeType": "text/plain" }]} @server.on(ReadResourceRequest) async def read_resource(req): return {"contents": [{ "uri": req.params["uri"], "text": open("/tmp/output.txt").read() }]} ``` ### ✨ Provide a Prompt Template ```python @server.on(ListPromptsRequest) async def list_prompts(_): return {"prompts": [{"name": "summarize", "description": "Summarize text"}]} @server.on(GetPromptRequest) async def get_prompt(req): return { "messages": [{ "role": "user", "content": {"type": "text", "text": f"Summarize: {req.params['arguments']['text']}"} }] } ``` --- ## 🧭 Design Principles | Principle | Why it matters | |------------------------------|----------------| | ✅ Composable | Start with one tool or resource — keep it small | | ✅ Self-describing | Use names, descriptions, schemas clearly | | ✅ Transport-agnostic | stdio for local, SSE for remote | | ✅ Stateless if possible | Easier to scale, test, reuse | | ✅ Validate inputs | Schema required — helps model + safety | | ✅ Log to stderr | stdout is for JSON-RPC only | | ✅ Use Inspector early | Fast feedback loop | --- ## 🔐 Security & Good Practices - Validate tool inputs - Avoid directory traversal in resource URIs - Don’t expose stack traces to clients - Log security-relevant events - Rate limit long-running operations --- ## 🧰 Troubleshooting | Problem | Solution | |--------------------------------|------------------------------------------------| | Claude shows "not responding" | Check stdout isn't polluted with logs | | Tool not showing up | Verify `tools/list` works via Inspector | | Paths break on Windows | Use `file:///C:/...` format | | Server launches but does nothing | Missing `@server.on(...)` or bad entrypoint | | “Tool not found” error | Check `name` matches in `tools/call` | --- ## 🚀 What’s Next? - ✅ Wrap a real-world CLI or API call - ✅ Dockerize it for reuse - ✅ Add it to your Claude, Continue, or fast-agent stack - ✅ Share it on GitHub or submit to Awesome MCP Servers --- ## 📚 References - [📘 MCP Specification](https://modelcontextprotocol.io/spec) - [🛠 Python SDK](https://modelcontextprotocol.io/docs/python) - [🧪 MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) - [🎒 Example Servers](https://modelcontextprotocol.io/examples) - [🌍 Awesome MCP Servers](https://github.com/modelcontextprotocol/awesome-mcp-servers) clarify what the user wants to convert and begin collaberating for a solution.  
```

> **Purpose:**  
This guide helps you transform *any* function, script, module, or API into an **MCP-compliant server** — making it accessible to LLM-powered clients like Claude, Continue, fast-agent, Genkit, and others.

Whether it’s a Python package, a CLI tool, or a business logic module, you’ll learn how to wrap it using the Model Context Protocol so that models can *discover*, *understand*, and *invoke* it using a standard interface.

---

## 🧠 Why MCP? (And Why Wrap Your Library)

LLMs don’t operate in a vacuum — they need to:
- Access real-world data (`resources`)
- Run actions or business logic (`tools`)
- Follow workflows or playbooks (`prompts`)

But everyone implements this differently. The **Model Context Protocol** solves this by defining **a universal schema and transport layer** to connect models to capabilities and context.

> Think of MCP as the *“USB for LLM integrations”* — plug in once, and any compliant AI agent can use it.

By wrapping your library as an MCP server, you gain:
- ✅ **Standardized interface** for any client
- ✅ **Modular reuse** across teams or tools
- ✅ **LLM discoverability** and auto-documentation
- ✅ **Scoped, secure access** via protocols and URIs

---

## 🔧 What Can You Wrap?

| You Have…                 | Wrap It As…   | Example Outcome                             |
|---------------------------|---------------|---------------------------------------------|
| A Python/Node function    | Tool          | Model can call `calculate()` with arguments |
| A config or log file      | Resource      | Model can read and analyze it               |
| A prompt-based template   | Prompt        | Model can run “Summarize this bug” flow     |
| A REST or internal API    | Tool + Resource | Model can query or inspect endpoints      |
| A shell script            | Tool          | Trigger CLI logic via LLM                   |

---

## 📦 Step-by-Step: Wrapping Your Code with MCP

### 🥇 Step 1: Choose Your SDK

Install your language SDK:

```bash
# Python
pip install mcp-sdk
```

```bash
# TypeScript
npm install @modelcontextprotocol/sdk
```

---

### 🥈 Step 2: Scaffold Your Server

Here's an example in Python:

```python
from mcp_sdk.server import Server
from mcp_sdk.types import ListToolsRequest, CallToolRequest

def calculate_sum(x, y):
    return x + y

server = Server(name="calc-server", version="0.1", capabilities={"tools": {}})

@server.on(ListToolsRequest)
async def list_tools(_):
    return {
        "tools": [{
            "name": "calculate_sum",
            "description": "Adds two numbers",
            "inputSchema": {
                "type": "object",
                "properties": {"x": {"type": "number"}, "y": {"type": "number"}},
                "required": ["x", "y"]
            }
        }]
    }

@server.on(CallToolRequest)
async def call_tool(req):
    args = req.params["arguments"]
    result = calculate_sum(args["x"], args["y"])
    return {"content": [{"type": "text", "text": str(result)}]}
```

---

### 🥉 Step 3: Run the Server (stdio)

```bash
python server.py
```

This starts your server over stdin/stdout — which works perfectly with local MCP clients like Claude Desktop, Continue, or fast-agent.

---

### 🧪 Step 4: Inspect with MCP Inspector

```bash
npx -y @modelcontextprotocol/inspector python server.py
```

Use this to verify your:
- Tool schema
- Response shape
- Errors, logs, and inputs

---

### 🧩 Step 5: Add to Your Client (Claude, Continue, etc.)

For Claude Desktop, in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "calc": {
      "command": "python",
      "args": ["C:/Users/yourname/server.py"]  // Windows
    }
  }
}
```

For macOS/Linux:

```json
{
  "mcpServers": {
    "calc": {
      "command": "python",
      "args": ["/Users/you/dev/server.py"]
    }
  }
}
```

---

## 🐳 Docker + MCP: Clean, Portable Wrapping

### Why Docker?

Docker is ideal when:
- Your code has native dependencies
- You want cross-platform reproducibility
- You need to ship remote SSE servers

### Example Dockerfile:

```Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY server.py .
RUN pip install mcp-sdk
CMD ["python", "server.py"]
```

```bash
docker build -t mcp-calc .
docker run -i mcp-calc
```

> Claude can use this via `"command": "docker", "args": ["run", "-i", "mcp-calc"]`

---

## 💻 OS Differences: macOS / Windows / Linux

### Working Directory

| OS       | Behavior                       | Solution                      |
|----------|--------------------------------|-------------------------------|
| macOS    | GUI apps launch from `/`       | Use absolute paths            |
| Windows  | GUI apps or shell may vary     | Use full `C:/...` paths       |
| Linux    | Launcher sets working dir      | Still use absolute paths      |

### File Paths

- macOS/Linux: `file:///Users/you/path.txt`
- Windows: `file:///C:/Users/you/path.txt`

Use this to normalize:

```python
from urllib.parse import pathname2url
from pathlib import Path

def to_uri(path):
    return "file://" + pathname2url(str(Path(path).absolute()))
```

### Environment Variables

Use `"env"` block in your config:

```json
{
  "env": {
    "API_KEY": "abc123",
    "ROOT_PATH": "/Users/you/dev"
  }
}
```

### Docker Quirks

| Platform | Quirk                          | Fix                                   |
|----------|--------------------------------|----------------------------------------|
| macOS    | Docker runs inside VM          | Avoid local volume mounts by default   |
| Windows  | Paths inside container = WSL   | Use `/mnt/c/...` or use COPY instead   |
| Linux    | Native support                 | ✅ No changes needed                    |

---

## 🧠 Prompts & Resources (Optional Features)

### 📄 Expose a Resource (e.g., File or API)

```python
@server.on(ListResourcesRequest)
async def list_resources(_):
    return {"resources": [{
        "uri": "file:///tmp/output.txt",
        "name": "Output File",
        "mimeType": "text/plain"
    }]}

@server.on(ReadResourceRequest)
async def read_resource(req):
    return {"contents": [{
        "uri": req.params["uri"],
        "text": open("/tmp/output.txt").read()
    }]}
```

### ✨ Provide a Prompt Template

```python
@server.on(ListPromptsRequest)
async def list_prompts(_):
    return {"prompts": [{"name": "summarize", "description": "Summarize text"}]}

@server.on(GetPromptRequest)
async def get_prompt(req):
    return {
        "messages": [{
            "role": "user",
            "content": {"type": "text", "text": f"Summarize: {req.params['arguments']['text']}"}
        }]
    }
```

---

## 🧭 Design Principles

| Principle                     | Why it matters |
|------------------------------|----------------|
| ✅ Composable                | Start with one tool or resource — keep it small |
| ✅ Self-describing           | Use names, descriptions, schemas clearly |
| ✅ Transport-agnostic        | stdio for local, SSE for remote |
| ✅ Stateless if possible     | Easier to scale, test, reuse |
| ✅ Validate inputs           | Schema required — helps model + safety |
| ✅ Log to stderr             | stdout is for JSON-RPC only |
| ✅ Use Inspector early       | Fast feedback loop |

---

## 🔐 Security & Good Practices

- Validate tool inputs
- Avoid directory traversal in resource URIs
- Don’t expose stack traces to clients
- Log security-relevant events
- Rate limit long-running operations

---

## 🧰 Troubleshooting

| Problem                         | Solution                                       |
|--------------------------------|------------------------------------------------|
| Claude shows "not responding"  | Check stdout isn't polluted with logs         |
| Tool not showing up            | Verify `tools/list` works via Inspector       |
| Paths break on Windows         | Use `file:///C:/...` format                   |
| Server launches but does nothing | Missing `@server.on(...)` or bad entrypoint |
| “Tool not found” error         | Check `name` matches in `tools/call`          |

---

## 🚀 What’s Next?

- ✅ Wrap a real-world CLI or API call
- ✅ Dockerize it for reuse
- ✅ Add it to your Claude, Continue, or fast-agent stack
- ✅ Share it on GitHub or submit to Awesome MCP Servers

---

## 📚 References

- [📘 MCP Specification](https://modelcontextprotocol.io/spec)
- [🛠 Python SDK](https://modelcontextprotocol.io/docs/python)
- [🧪 MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector)
- [🎒 Example Servers](https://modelcontextprotocol.io/examples)
- [🌍 Awesome MCP Servers](https://github.com/modelcontextprotocol/awesome-mcp-servers)
```
