# Root — RELEASE_v0.4.0.md

# Release Module: v0.4.0 (v2026.3.23)

The `RELEASE_v0.4.0.md` module serves as the architectural manifest for the "Platform Expansion" release of the Hermes Agent. This version marks a transition from a CLI-centric tool to a multi-interface platform, introducing a standardized API server, expanded messaging gateway capabilities, and a sophisticated context management system.

## Core Architectural Shifts

### 1. OpenAI-Compatible API Server
The agent now exposes a local server implementing the `/v1/chat/completions` specification. This allows Hermes to act as a drop-in replacement for OpenAI in existing developer workflows.
- **Persistence:** Uses a SQLite-backed `ResponseStore` to ensure response data survives server restarts.
- **Job Management:** Introduces the `/api/jobs` REST endpoint for programmatic control over the agent's cron system.
- **Security:** Implements CORS origin protection and input whitelisting to harden the server against unauthorized browser-based access.

### 2. Gateway & Messaging Expansion
The Gateway layer has been refactored to support a broader range of communication protocols.
- **New Adapters:** Signal, DingTalk, SMS (Twilio), Mattermost, Matrix, and Webhooks.
- **Resiliency:** Implements exponential backoff for failed platform connections, ensuring the gateway remains stable during network interruptions.
- **Vision Support:** Extended to Telegram and Discord DMs, allowing the agent to process inline images and attachments directly within messaging threads.

### 3. Context & Memory Management
Version 0.4.0 introduces significant optimizations to how the agent handles long-running conversations and external data.

```mermaid
graph TD
    A[User Input] --> B{Context Handler}
    B --> C[@ References]
    B --> D[Prompt Caching]
    B --> E[Context Compression]
    C --> F[File/URL Injection]
    D --> G[Anthropic Cache Turns]
    E --> H[Structured Summaries]
    H --> I[SQLite SessionDB]
```

- **@ Context References:** Developers can now use `@file` and `@url` syntax. The system performs tab-completion in the CLI and injects the referenced content directly into the prompt context.
- **Prompt Caching:** The `AIAgent` instances are now cached per session. This preserves Anthropic prompt caches across turns, significantly reducing latency and token costs for large contexts.
- **Structured Compression:** Replaces naive truncation with iterative, structured summaries. It includes "token-budget tail protection" to ensure the most recent messages remain intact while older history is summarized.

## Provider & Inference Layer

The provider interface has been expanded to support new authentication patterns and discovery mechanisms:
- **GitHub Copilot:** Implements a full OAuth 2.1 PKCE flow for token validation and routing.
- **Dynamic Context Detection:** The agent now queries local endpoints (like llama.cpp via `/v1/props`) to determine actual context window sizes rather than relying on hardcoded defaults.
- **Eager Fallback:** If a primary provider returns a rate-limit error, the agent immediately attempts the request with a configured backup model.

## Tooling & MCP (Model Context Protocol)

The tool system has been decoupled to allow for better management of external capabilities.
- **`hermes mcp` CLI:** A new management suite for installing and configuring MCP servers.
- **Standalone Toolsets:** MCP servers can now be exposed as independent toolsets, allowing for granular permissioning.
- **OAuth 2.1 Integration:** Supports PKCE-based authentication for tools requiring user-level authorization (e.g., Google Calendar or GitHub).

## Security Hardening

This release includes a comprehensive security pass targeting common agentic vulnerabilities:
- **SSRF Protection:** Implemented for `vision_tools` and `web_tools` to prevent the agent from being used to probe internal networks.
- **Shell Injection:** Hardened `_expand_path` and `terminal_tool` to prevent malicious path suffixes and command injection.
- **PII Redaction:** A new redacting formatter for logs and outputs to prevent sensitive environment variables or keys from being leaked to the LLM or logs.
- **Pre-execution Scanning:** The `terminal_tool` now includes a pattern scanner to block known malicious code patterns before they reach the shell.

## Reliability & State Persistence

- **SessionDB Locking:** Added thread locks to `SessionDB` methods to prevent database corruption during concurrent writes in multi-platform gateway environments.
- **Cron Recovery:** The scheduler now scales the "missed-job grace window" based on frequency and can recover one-shot jobs that were missed during downtime.
- **Streaming Stability:** Streaming is now the default mode. The agent loop includes logic to handle tool progress displays and reasoning blocks without breaking the stream concatenation.