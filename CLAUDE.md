# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mistral Vibe is an open-source CLI coding assistant powered by Mistral's models. It provides a conversational interface with tools for file manipulation, code searching, version control, and command execution.

## Common Commands

```bash
# Install dependencies
uv sync --all-extras

# Run the CLI
uv run vibe

# Run tests
uv run pytest                           # All tests (parallel by default)
uv run pytest tests/test_agent_tool_call.py  # Single test file
uv run pytest -v                        # Verbose output

# Linting and formatting
uv run ruff check .                     # Check for lint issues
uv run ruff check --fix .               # Auto-fix lint issues
uv run ruff format .                    # Format code
uv run ruff format --check .            # Check formatting only

# Type checking
uv run pyright

# Run all pre-commit hooks
uv run pre-commit run --all-files

# Bump version
uv run scripts/bump_version.py patch    # 1.0.0 -> 1.0.1
uv run scripts/bump_version.py minor    # 1.0.0 -> 1.1.0
uv run scripts/bump_version.py major    # 1.0.0 -> 2.0.0
```

## Architecture

### Directory Structure

- `vibe/` - Main package
  - `cli/` - CLI interface using Textual TUI framework
    - `textual_ui/` - Textual app, widgets, handlers, and renderers
    - `autocompletion/` - Path and slash command completion
  - `core/` - Core business logic
    - `agent.py` - Main Agent class orchestrating LLM interactions and tool execution
    - `tools/` - Tool system (base classes, manager, MCP integration, builtins)
    - `llm/` - LLM backend abstraction (Mistral, generic providers)
    - `middleware.py` - Request middleware (auto-compact, context warnings, limits)
    - `config.py` - Configuration loading from `config.toml`
  - `acp/` - Agent Client Protocol mode for IDE integration
  - `setup/` - Onboarding and setup screens

### Key Components

**Agent** (`vibe/core/agent.py`): Central orchestrator that manages conversation flow, tool execution, and LLM interactions. Uses middleware pipeline for auto-compacting, turn limits, and price limits.

**ToolManager** (`vibe/core/tools/manager.py`): Discovers and instantiates tools from builtin directory, custom user tools (`~/.vibe/tools/`), and MCP servers.

**BaseTool** (`vibe/core/tools/base.py`): Abstract base class for all tools. Tools define name, description, parameters, and permission levels (always/ask/never).

**LLM Backends** (`vibe/core/llm/backend/`): Factory pattern supporting Mistral and generic OpenAI-compatible providers.

### Entry Points

- `vibe` -> `vibe/cli/entrypoint.py:main()` - Interactive CLI mode
- `vibe-acp` -> `vibe/acp/entrypoint.py:main()` - Agent Client Protocol server mode

### Configuration

Config files are loaded from `./.vibe/config.toml` (project-local) or `~/.vibe/config.toml` (global). Custom agents can be defined in `~/.vibe/agents/`. Custom system prompts go in `~/.vibe/prompts/`.

## Code Style

### Fundamentals
- Python 3.12+ required
- Use `uv` for all Python commands (never bare `python` or `pip`)
- Line length: 88 characters
- All files must have `from __future__ import annotations`
- Google-style docstrings

### Modern Python Patterns
- Modern type hints: `list`, `dict`, `|` for unions (not `Optional`, `Union`, `List`, `Dict`)
- Use `match-case` over `if/elif/else` chains for pattern matching
- Use walrus operator (`:=`) when it simplifies assignment + conditional
- Prefer early returns and guard clauses ("never nester" style)
- Use `pathlib.Path` over `os.path`
- Use f-strings, comprehensions, and context managers
- Use `StrEnum` with `auto()` for string enums; UPPERCASE members

### Pydantic
- Prefer Pydantic v2's native validation over ad-hoc parsing
- Use `model_validate`, `field_validator`, `from_attributes`, field aliases
- For discriminated unions, use sibling classes with shared mixins (not subclass field narrowing)

### Code Quality
- Write declarative, minimalist code - express intent, avoid boilerplate
- No inline `# type: ignore` or `# noqa` - fix types at the source using generics, Protocols, `isinstance` guards, or `typing.cast`
- Only document exceptions explicitly raised in the function (not every theoretical exception)
