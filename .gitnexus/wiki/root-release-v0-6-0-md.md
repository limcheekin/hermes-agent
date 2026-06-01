# Root — RELEASE_v0.6.0.md

# Hermes Agent v0.6.0 Release Documentation

The v0.6.0 release (v2026.3.30) introduces significant architectural shifts, primarily focusing on multi-instance isolation (Profiles), Model Context Protocol (MCP) server capabilities, and enhanced reliability through provider fallback chains.

## Multi-Instance Architecture (Profiles)

The most fundamental change in v0.6.0 is the introduction of the **Profiles** system. This allows developers to run multiple, completely isolated Hermes instances from a single installation.

### Isolation Mechanism
Each profile maintains its own:
- **Configuration:** `config.yaml` and `.env` files.
- **Storage:** Memory, session logs, and skill databases.
- **Gateway Services:** Independent adapter configurations.
- **Token Locks:** A safety mechanism that prevents two profiles from accidentally sharing the same bot credentials (e.g., using the same Telegram token in two instances).

### Profile Management CLI
Profiles are managed via the `hermes profile` command group:
- `hermes profile create <name>`: Initializes a new isolated environment.
- `hermes -p <name>`: Executes commands within the context of a specific profile.
- `hermes profile export/import`: Facilitates sharing configurations across different machines.

The codebase now utilizes `display_hermes_home()` instead of hardcoded `~/.hermes` paths to ensure all logs and UI elements reflect the active profile's directory.

## Model Context Protocol (MCP) Integration

Hermes v0.6.0 expands its relationship with MCP in two directions: acting as a server and improving client-side tool discovery.

### MCP Server Mode
By running `hermes mcp serve`, the agent exposes its internal state to external MCP-compatible clients (like Claude Desktop or Cursor).
- **Transports:** Supports both `stdio` and `streamable_http_client`.
- **Resources:** Clients can browse active conversations, read message history, and access session attachments.

### Dynamic Tool Discovery
The agent now listens for `notifications/tools/list_changed` events from connected MCP servers. This allows Hermes to hot-reload available tools without requiring a restart of the gateway or CLI session.

## Provider & Inference Logic

### Ordered Fallback Chains
To improve reliability, the `config.yaml` now supports a `fallback_providers` list. 
- **Execution Flow:** If the primary provider returns a 429 (Rate Limit), 5xx (Server Error), or is unreachable, the agent automatically iterates through the fallback chain.
- **Rate Limit Handling:** Introduced user-friendly 429 messages with a `Retry-After` countdown.

### Provider Switching Fixes
The `hermes model` command was updated to clear stale `api_mode` settings. Previously, switching from an OpenAI-compatible provider to an Anthropic-compatible one could result in 404 errors due to cached endpoint configurations.

## Messaging Gateway Enhancements

The Gateway service received two new platform adapters and several protocol upgrades.

| Platform | Key v0.6.0 Updates |
| :--- | :--- |
| **Feishu / Lark** | New adapter supporting interactive cards, group chats, and file attachments. |
| **WeCom** | New adapter for Enterprise WeChat with callback verification. |
| **Telegram** | Added **Webhook Mode** for production deployments; added regex-based group mention gating. |
| **Slack** | **Multi-workspace OAuth** support via a centralized token file. |
| **Discord** | Visual "thinking" reactions and improved cleanup of deferred slash commands. |

## Tooling & Remote Execution

### Remote Backend Mounting
For developers using **Modal** or **Docker** as execution backends, v0.6.0 now synchronizes local state to the remote environment:
- **Skill Directories:** Local `skills.external_dirs` are mounted into the container.
- **Credentials:** Local credential files are synced using mtime+size caching to ensure remote tools have access to necessary secrets without manual configuration.

### Web & Vision Tools
- **Exa Search:** Added as a high-quality alternative to DuckDuckGo and Firecrawl.
- **Vision Guardrails:** The vision tool now strictly enforces a website-only policy and rejects non-image file types to prevent unauthorized local file disclosure.

## Security & Reliability

### Hardened Command Detection
The approval system for "dangerous" commands has been expanded. It now monitors:
- **Shell Patterns:** Improved detection of destructive shell commands.
- **Path Guards:** Explicit blocks on file tools attempting to write to sensitive system locations like `/etc/`, `/boot/`, or `docker.sock`.

### Atomic Operations
To prevent configuration corruption during crashes, `config.yaml` updates are now **atomic**. The system writes to a temporary file and performs a rename operation, ensuring the configuration remains valid even if the process is interrupted.

### Update Safety
The `hermes update` process now automatically clears `__pycache__` and utilizes lazy imports for `display_hermes_home`. This prevents `ImportError` loops caused by stale bytecode referencing deleted or moved functions.

## New Skills
Several new specialized skill sets were added to the `optional-skills` directory:
- `memento-flashcards`: Spaced repetition logic.
- `scrapling`: Advanced web scraping.
- `siyuan-note`: Integration for the SiYuan note-taking platform.
- `one-three-one-rule`: A communication framework for decision-making.