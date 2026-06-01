# Root — RELEASE_v0.7.0.md

# Release Documentation: Hermes Agent v0.7.0 (v2026.4.3)

The v0.7.0 release, titled "The Resilience Release," focuses on architectural extensibility, security hardening, and improved reliability for production gateway deployments. Key architectural shifts include the transition to a pluggable memory provider system and the introduction of thread-safe credential pooling.

## 🏗️ Core Architecture Updates

### Pluggable Memory Provider Interface
Memory management has been refactored into an extensible plugin system. Developers can now implement custom memory backends by inheriting from a provider Abstract Base Class (ABC) and registering them via the Hermes plugin system.

*   **Reference Implementation:** The Honcho integration has been updated to serve as the reference plugin, featuring profile-scoped host and peer resolution.
*   **Session Persistence:** API server sessions now persist to a shared `SessionDB`, and token usage is tracked across non-CLI sessions.
*   **Search Improvements:** FTS5 queries now correctly handle dotted terms (e.g., filenames or namespaces) by quoting terms during session searches.

### Same-Provider Credential Pools
To handle rate limits and key exhaustion, Hermes now supports multiple API keys for a single provider.
*   **Rotation Strategy:** Uses a `least_used` strategy to distribute load.
*   **Failover:** Automatic rotation is triggered by `401 Unauthorized` errors.
*   **Configuration:** Managed via the `credential_pool` key in `config.yaml` or the setup wizard.
*   **Smart Routing:** Pool state is preserved during provider fallback events, preventing eager fallbacks on `429 Too Many Requests` until the pool is exhausted.

### Agent Loop & Context Management
*   **Thinking Block Persistence:** Anthropic `<thinking>` signatures are now preserved across tool-use turns to maintain model reasoning continuity.
*   **Compression Death Spiral Protection:** Logic added to detect and halt loops where API disconnects trigger repeated, failing context compression attempts.
*   **Deterministic Call IDs:** Replaced random UUIDs with deterministic `call_id` fallbacks to improve prompt cache hit rates.

## 🔧 Tooling & Browser System

### Camofox Anti-Detection Backend
A new stealth-focused browser backend using Camoufox is available.
*   **Stealth:** Designed to bypass anti-bot detections during web navigation.
*   **Debugging:** Supports persistent sessions with VNC URL discovery for visual inspection of the agent's browser state.
*   **Security:** SSRF checks can be bypassed for local backends (Camofox/Headless Chromium) via `browser.allow_private_urls`.

### File Operations & Diffs
*   **Inline Diff Previews:** The `write_file` and `patch_file` operations now generate inline diffs within the tool activity feed.
*   **Stale File Detection:** Tools now warn if a file has been modified by an external process since the agent last read it, preventing accidental overwrites.

### ACP (Agent Control Protocol) & MCP
*   **Client-Provided MCP Servers:** Editor integrations (VS Code, Zed, JetBrains) can now register their own MCP servers. Hermes dynamically picks these up as additional tools available to the agent.
*   **Stability:** Improved MCP server reload timeouts and event loop handling for HTTP-based MCP servers.

## 📱 Gateway & Messaging Platforms

The Gateway has undergone a significant hardening pass to resolve race conditions in high-concurrency environments.

### Approval Routing
A major fix addresses the "stuck session" issue where approval commands were swallowed.
*   **Guard Logic:** `/approve` and `/deny` commands are now routed through a `running-agent` guard.
*   **Thread Blocking:** When an agent is waiting for user approval, the thread blocks similarly to the CLI, ensuring the tool result is correctly injected once approved.

### Platform Specifics
*   **Discord:** Introduced a button-based approval UI and configurable message reactions.
*   **Telegram:** Implemented skill-aware slash commands. Installed skills are dynamically registered as Telegram commands (subject to the 100-command API limit).
*   **Slack:** Added `slack.reply_in_thread` configuration to support threaded conversation flows.

## 🔒 Security Hardening

### Exfiltration Blocking
Hermes now implements active scanning of browser URLs and LLM responses to prevent secret exfiltration.
*   **Pattern Matching:** Scans for high-entropy strings and known secret formats (e.g., GitHub tokens) even when encoded in Base64 or URL formats.
*   **Sandbox Redaction:** The `execute_code` tool now redacts detected secrets from its standard output.

### Path & Directory Protection
*   **Credential Guarding:** Expanded protections to block the agent from reading or writing to sensitive directories: `.docker`, `.azure`, and `.config/gh`.
*   **Zip-Slip Protection:** Validation of tar/zip member paths during profile imports to prevent path traversal attacks.

## 💻 CLI & User Experience

### New Slash Commands
| Command | Function |
| :--- | :--- |
| `/yolo` | Toggles dangerous command approvals for the current session. |
| `/btw` | Allows ephemeral side-questions that are excluded from the main conversation context. |
| `/profile` | Displays active profile configuration without exiting the chat. |

### TUI Improvements
*   **Layout:** The TUI is now pinned to the bottom of the terminal on startup to eliminate whitespace gaps.
*   **Session Management:** `/history` and `/resume` now provide a direct list of recent sessions for faster context restoration.
*   **Input Handling:** Dragged file paths are now detected as file references rather than being misinterpreted as slash commands.

## 🚀 Update & Deployment
*   **Fork Sync:** `hermes update` now detects if the installation is a fork and offers to sync with the upstream repository.
*   **Systemd:** The `/update` command now uses `setsid` instead of `systemd-run` to bypass common permission issues in containerized environments.
*   **LXC Support:** The gateway service can now be configured to run as root for compatibility with specific LXC and container environments.