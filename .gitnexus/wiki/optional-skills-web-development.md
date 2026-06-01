# optional-skills — web-development

# Web Development Skills: Page Agent

The `web-development` module provides tools for embedding AI-native UX patterns directly into client-side applications. Its primary focus is **page-agent**, an in-page GUI agent that allows end-users to interact with a website using natural language.

Unlike server-side automation tools (e.g., Playwright or Browserbase) that operate on a site from the outside, `page-agent` lives within the Document Object Model (DOM) of the host application. It interprets the DOM as text and executes actions like clicking, typing, and navigating based on LLM instructions.

## Architecture Overview

`page-agent` operates as a bridge between a natural language interface and the browser's DOM API. It does not use screenshots or multi-modal vision; instead, it relies on a text-based representation of the accessibility tree and DOM structure.

```mermaid
graph TD
    User[User Input] --> UI[page-agent UI Panel]
    UI --> Core[@page-agent/core]
    Core --> LLM[LLM Endpoint /v1/chat]
    LLM --> Core
    Core --> Controller[@page-agent/page-controller]
    Controller --> DOM[Browser DOM]
    DOM -.-> Core
```

## Integration Patterns

### 1. Production Integration (npm)
For production SaaS or admin tools, install the package and initialize the agent with a custom LLM proxy.

```bash
npm install page-agent
```

**Implementation:**
```javascript
import { PageAgent } from 'page-agent';

const agent = new PageAgent({
    model: 'gpt-4o-mini',
    // Point to your backend proxy to protect API keys
    baseURL: 'https://api.your-domain.com/v1/ai-proxy', 
    apiKey: 'SESSION_TOKEN', 
    language: 'en-US',
});

// Programmatic execution
await agent.execute('Find the user named "Alice" and click Edit');

// Or display the built-in GUI
agent.panel.show();
```

### 2. Rapid Prototyping (CDN)
For evaluation or internal demos, the agent can be injected via a single script tag. This uses a pre-bundled IIFE (Immediately Invoked Function Expression).

```html
<script 
  src="https://cdn.jsdelivr.net/npm/page-agent@1.8.0/dist/iife/page-agent.demo.js" 
  crossorigin="true">
</script>
```

## Core Configuration Reference

The `PageAgent` constructor accepts a configuration object with the following key fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `model` | `string` | The LLM model name (e.g., `qwen3.5-plus`, `gpt-4o`). |
| `baseURL` | `string` | OpenAI-compatible endpoint (Ollama, OpenRouter, DashScope). |
| `apiKey` | `string` | Authentication token for the LLM provider. |
| `language` | `string` | UI and interaction language (default: `en-US`). |
| `allowList` | `string[]` | Optional list of allowed domains or selectors for security. |

## Monorepo Structure

When contributing to the module or building from source, the project is organized as an npm workspace monorepo:

*   **`packages/page-agent`**: The main entry point including the UI panel.
*   **`packages/core`**: The orchestration logic that handles the loop between LLM responses and action execution.
*   **`packages/page-controller`**: Low-level DOM operations, element highlighting, and visual feedback.
*   **`packages/llms`**: Standardized client for communicating with OpenAI-compatible APIs.
*   **`packages/ui`**: React-based components for the floating agent panel and i18n support.
*   **`packages/extension`**: A WXT-based browser extension implementation of the agent.

## Security & Production Readiness

### API Key Exposure
The `apiKey` passed to the `PageAgent` constructor is visible in the client-side JavaScript bundle. **Never hardcode production LLM keys.**
*   **Recommended:** Create a backend endpoint that forwards requests to the LLM provider and injects the real API key server-side. Point the `baseURL` to this endpoint.

### Content Security Policy (CSP)
Strict CSP headers may block the agent from:
1.  Loading from a CDN (requires `script-src`).
2.  Connecting to the LLM endpoint (requires `connect-src`).
3.  Executing inline scripts.
If your site has a strict CSP, self-host the `page-agent` assets and proxy the LLM traffic through your own domain.

### Data Masking
The agent reads the DOM to understand the page state. Use the provided hooks in the `core` package to mask sensitive user data (PII) before the DOM tree is sent to the LLM.

## Development Workflow

To modify the agent or test local changes:

1.  **Install Dependencies:** Requires Node 22.13+ or 24+.
    ```bash
    npm ci
    ```
2.  **Environment Setup:** Create a `.env` file in the root with `LLM_BASE_URL` and `LLM_API_KEY`.
3.  **Local Demo:**
    ```bash
    npm run dev:demo
    ```
    This serves the IIFE bundle at `http://localhost:5174/page-agent.demo.js`. You can inject this into any site via the browser console for testing.