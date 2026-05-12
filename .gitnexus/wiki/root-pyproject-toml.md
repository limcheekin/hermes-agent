# Root — pyproject.toml

# Project Configuration (pyproject.toml)

The `pyproject.toml` file serves as the central manifest for the `hermes-agent` ecosystem. It defines the build system, core and optional dependencies, CLI entry points, and configuration for development tools like `pytest` and `ty`.

## Build System and Metadata

The project uses `setuptools` as the build backend. 

*   **Package Name:** `hermes-agent`
*   **Version:** `0.12.0`
*   **Python Requirement:** `>=3.11`
*   **License:** MIT

## Core Dependencies

The core dependencies are pinned to known-good version ranges to ensure stability and limit the supply chain attack surface. They are categorized into functional groups:

| Category | Packages |
| :--- | :--- |
| **LLM Clients** | `openai`, `anthropic` |
| **Data & Logic** | `pydantic`, `jinja2`, `pyyaml`, `tenacity` |
| **CLI & UI** | `rich`, `prompt_toolkit`, `fire` |
| **Networking** | `httpx[socks]`, `requests` |
| **Built-in Features** | `croniter` (Scheduling), `edge-tts` (Speech), `PyJWT` (Auth) |
| **Search & Tools** | `exa-py`, `firecrawl-py`, `parallel-web`, `fal-client` |

## Optional Dependencies (Extras)

`hermes-agent` uses a modular dependency strategy. Most integrations and heavy features are optional to keep the base installation lean.

### Integration Extras
*   **Cloud/Compute:** `modal`, `daytona`, `vercel`.
*   **Messaging:** `messaging` (Telegram, Discord, Slack), `slack`, `matrix`, `dingtalk`, `feishu`.
*   **AI/LLM Providers:** `mistral`, `bedrock`.
*   **Protocols:** `mcp` (Model Context Protocol), `acp` (Agent Client Protocol).

### Feature Extras
*   **`voice`:** Includes `faster-whisper` and `sounddevice` for local STT.
*   **`web`:** Includes `fastapi` and `uvicorn` for the local dashboard.
*   **`google`:** Includes API clients for Workspace (Gmail, Calendar, Drive).
*   **`rl`:** Reinforcement Learning tools including `atroposlib` and `tinker`.
*   **`termux`:** A curated bundle for Android/Termux environments that avoids non-Android wheels.

### Development Extras
*   **`dev`:** Includes `pytest`, `pytest-asyncio`, `debugpy`, and `ruff`.

## CLI Entry Points

The configuration defines three primary command-line interfaces:

```toml
[project.scripts]
hermes = "hermes_cli.main:main"
hermes-agent = "run_agent:main"
hermes-acp = "acp_adapter.entry:main"
```

1.  **`hermes`**: The main interactive CLI and management tool.
2.  **`hermes-agent`**: Direct entry point to run an agent instance.
3.  **`hermes-acp`**: Entry point for the Agent Client Protocol adapter.

## Package Structure

The project uses a hybrid layout of top-level modules and packages, explicitly defined in the `[tool.setuptools]` section.

### Included Packages
The following directories are treated as packages:
*   `agent`, `tools`, `hermes_cli`, `gateway`, `tui_gateway`, `cron`, `acp_adapter`, `plugins`.

### Top-Level Modules
Several utility and core modules reside directly in the root for simplified imports:
*   `run_agent.py`, `model_tools.py`, `toolsets.py`, `batch_runner.py`, `cli.py`, `hermes_constants.py`, `hermes_logging.py`, etc.

## Development Configuration

### Testing (Pytest)
*   **Path:** Tests are located in the `/tests` directory.
*   **Markers:** Defines an `integration` marker for tests requiring external API keys or services.
*   **Execution:** Defaults to skipping integration tests and using `pytest-xdist` (`-n auto`) for parallel execution.

### Type Checking (Ty)
The project uses `ty` for environment and type configuration, targeting Python 3.13. It is configured with several overrides to ignore specific linting/typing errors (`unresolved-import`, `invalid-method-override`) that may occur due to the dynamic nature of agent tool loading.

### Linting (Ruff)
The current configuration excludes all files from `ruff` analysis (`exclude = ["*"]`), suggesting linting is handled via other workflows or is currently disabled at the project root level.