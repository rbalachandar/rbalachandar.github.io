---
layout: post
title: "Building a Code Execution Tool with Monty and AgentFS"
date: 2026-02-21 10:00:00 +0000
categories: python ai-agents sandboxing
excerpt: "I built a sandboxed code execution tool for AI agents using Monty and AgentFS. Here's what I learned about Python package management, version compatibility, and the challenges of building agentic tooling."
permalink: /blog/code-execution-tool/
---

Claude's code execution tool is one of its most powerful features — it can run Python code in a sandbox and return results. I wanted to build something similar, but with better filesystem support and multi-agent coordination.

This is the story of building that tool, and the lessons learned along the way.

---

## The Goal

Create a code execution tool that:
1. **Runs Python code safely** - Sandboxed execution with timeout and resource limits
2. **Has filesystem access** - Agents can read/write files for data persistence
3. **Supports multi-agent coordination** - Multiple agents can share data via files
4. **Tracks provenance** - Every file operation is logged (who, what, when, why)

I chose **Monty** (pydantic-monty) for sandboxing and built an **AgentFS** layer for filesystem operations with provenance tracking.

---

## Tech Stack Decisions

### Why Monty?

Monty is a Python interpreter written in Rust, designed specifically for AI/LLM code execution:

| Feature | Monty | Docker/VM |
|---------|-------|-----------|
| Startup time | ~milliseconds | ~seconds |
| Security | AST-based validation | Kernel isolation |
| Memory | Minimal | Full OS overhead |
| External functions | ✅ Native support | ❌ Needs IPC |

### Why AgentFS?

Instead of traditional filesystems, I wanted an "agent filesystem" that tracks provenance:

```python
# Every operation is logged
{
    "operation": "write",
    "path": "/shared/results.json",
    "agent_id": "agent-1",
    "timestamp": "2026-02-21T10:00:00Z",
    "old_hash": "abc123",
    "new_hash": "def456",
    "reason": "processed data"
}
```

---

## The Architecture

```python
# Executor combines Monty sandbox with AgentFS filesystem
from code_execution_tool import Executor

executor = Executor()

# Execute code with filesystem access
result = executor.execute('''
    data = json_loads(fs_read('/input.json'))
    processed = process(data)
    fs_write('/output.json', json_dumps(processed))
''')

print(result.output)          # stdout
print(result.files_created)    # ['/output.json']
print(result.operations)       # audit log
```

### Monty's Limitations

Monty is intentionally restricted:
- ❌ No standard library (except sys, typing, asyncio)
- ❌ No classes (coming soon)
- ❌ No match statements (coming soon)
- ❌ No third-party imports

**Workaround:** Transform code to use external functions:

```python
# User writes:
data = json.loads(fs.read('/file.json'))

# We transform to:
data = json_loads(fs_read('/file.json'))

# And provide external functions:
external_functions = {
    'json_loads': json.loads,
    'fs_read': fs.read,
}
```

---

## Filesystem Architecture: Inside vs Outside the Sandbox

One critical design decision is where files live — inside the sandbox (secure, ephemeral) or outside (persistent, multi-agent).

### Approach 1: Files Outside Sandbox (Current Implementation)

```
┌─────────────────────────────────────────────────────────────┐
│  Python Process (host)                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  AgentFS (Python dict) ← Files stored HERE          │    │
│  │  _files = {'/data.json': b'...'}                    │    │
│  └─────────────────────────────────────────────────────┘    │
│         │                                                   │
│         │ external_functions bridge                         │
│         ▼                                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Monty (Rust sandbox)                               │    │
│  │  - Executes code                                    │    │
│  │  - Calls back to Python for fs operations           │    │
│  │  - NO internal file storage                         │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- ✅ Multi-agent coordination (files persist between executions)
- ✅ Easy to inspect and debug (access from Python)
- ⚠️ Files exposed to host process (less secure)
- ⚠️ Requires external function overhead

### Approach 2: Files Inside Sandbox (Alternative)

```python
from pydantic_monty import Monty, MemoryFile, OSAccess

# Files that live ONLY in Monty
files = [
    MemoryFile('/data/input.json', b'{"key": "value"}'),
]

os_access = OSAccess(files=files)
m = Monty('data = json_loads(open("/data/input.json").read())')
m.start(os=os_access)  # Files destroyed when Monty exits
```

**Characteristics:**
- ✅ Files never leave sandbox (more secure)
- ✅ No external function overhead
- ✅ Files auto-destroyed on completion
- ❌ No automatic multi-agent coordination
- ❌ Harder to debug (can't inspect from Python)

### The Trade-off Matrix

| Aspect | Outside Sandbox | Inside Sandbox |
|--------|-----------------|----------------|
| **Security** | Files exposed to host | Files isolated |
| **Multi-agent** | ✅ Automatic coordination | ❌ Each execution isolated |
| **Persistence** | ✅ Persistent | ❌ Ephemeral |
| **Debugging** | ✅ Easy to inspect | ❌ Hard to inspect |
| **Cleanup** | Manual | Automatic |

### Which to Choose?

**Use Outside Sandbox** when:
- Multiple agents need to share data
- You need provenance tracking across executions
- Debugging and inspection are important

**Use Inside Sandbox** when:
- Processing sensitive data that must not leak
- Each execution is independent
- Automatic cleanup is desired

For my use case, I chose **outside sandbox** to enable multi-agent workflows while mitigating security concerns through:
- Per-agent workspaces (`/workspace/agent-1/`, `/workspace/agent-2/`)
- Audit logging of all operations
- Path validation to prevent traversal
- Content scanning for secrets (future work)

---

## Key Learnings

### 1. Package Names Can Be Misleading

The GitHub repo `pydantic/monty` publishes as `monty-workspace` (test cases only), while the actual package is `pydantic-monty` on PyPI. Always verify:

```bash
pip show pydantic-monty
ls .venv/lib/python3.14/site-packages/ | grep monty
```

### 2. AST Transformation Enables External Function Injection

Monty's approach — parse Python, validate AST, execute in Rust — enables:
- Fast execution (no subprocess overhead)
- Deep security (control what AST nodes are allowed)
- External function injection (pass Python objects into Rust execution)

This transforms `json.loads()` → `json_loads()` and provides the function from the host Python process.

### 3. Agent Filesystems Need Provenance

Traditional filesystems don't track who changed what and why. For AI agents, you need:
- Audit logs of every file operation
- Per-agent workspaces for isolation
- Shared directories with locks for coordination
- Content hashing for change detection

---

## The Result

A working code execution tool with:

```bash
code-execution-tool/
├── src/code_execution_tool/
│   ├── __init__.py       # Package exports
│   ├── executor.py       # Main Executor class
│   ├── sandbox.py        # Monty wrapper
│   ├── filesystem.py     # AgentFS implementation
│   └── models.py         # Data models
├── examples/
│   └── basic_usage.py    # Usage examples
└── pyproject.toml        # uv configuration
```

### What Works

- ✅ Sandboxed Python execution via Monty
- ✅ Filesystem operations with provenance tracking
- ✅ Multi-agent coordination (file locks, watches, shared state)
- ✅ External functions (json, math, custom fs adapter)

### What Doesn't

- ❌ Classes in sandboxed code (Monty limitation)
- ❌ Standard library imports (use external functions instead)
- ❌ Third-party packages (by design)

---

## Future Work

1. **Add timeout enforcement** using Monty's `ResourceLimits`
2. **Implement file locking** for multi-agent coordination
3. **Add MCP server** integration for use with Claude Code
4. **Create CLI interface** for local development
5. **Explore inside-sandbox filesystem** using `OSAccess` for sensitive data processing
6. **Add content scanning** to detect secrets in file operations

---

## Resources

- [code-execution-tool](https://github.com/rbalachandar/code-execution-tool) - Source code
- [pydantic-monty](https://pypi.org/project/pydantic-monty/) - Monty package
- [Agent Filesystems research notes](https://github.com/rbalachandar/rbalachandar.github.io/blob/main/BLOG_IDEAS_agent_filesystem.md) - My AgentFS exploration
- [uv](https://github.com/astral-sh/uv) - Modern Python package manager

---

Building this tool taught me a lot about modern Python packaging, AI agent tooling, and the tradeoffs between security and functionality. Monty's restricted environment feels limiting at first, but it enables patterns that are actually quite elegant — explicit dependencies, external function injection, and AST-level validation.

If you're building AI agents that need to execute code, I'd recommend starting with Monty (or E2B for heavier workloads) and thinking carefully about your filesystem story. Agents that write files need better tracking than traditional filesystems provide.
