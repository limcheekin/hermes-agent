# Root — package.json

# Root Configuration: package.json

The `package.json` file at the root of the Hermes Agent project defines the Node.js environment required for the agent's browser automation capabilities. While the primary agent logic is executed via Python (`run_agent.py`), the browser-based toolsets rely on a Node.js stack to interface with web environments.

## Purpose

This module manages the JavaScript dependencies necessary for the agent to perform web browsing, DOM manipulation, and automated interaction. It ensures that the underlying browser drivers and automation frameworks are versioned and installed correctly alongside the Python components.

## Key Dependencies

The project utilizes two primary packages for browser interaction:

*   **`@askjo/camofox-browser` (^1.5.2)**: Provides the core browser engine capabilities, likely focused on anti-detection or specialized browsing patterns required for AI agents.
*   **`agent-browser` (^0.26.0)**: A high-level wrapper or interface designed to bridge the gap between AI agents and browser automation frameworks.

## System Requirements

The project enforces a specific runtime environment to ensure compatibility with modern JavaScript features used in the browser toolsets:

*   **Node.js Engine**: `>=20.0.0`
*   **Dependency Overrides**: The project explicitly pins `lodash` to version `4.18.1` via the `overrides` field to ensure consistent behavior and address potential security vulnerabilities in transitive dependencies.

## Integration Flow

The Node.js environment acts as a subsystem for the main Python agent. The relationship between the package configuration and the execution flow is illustrated below:

```mermaid
graph TD
    A[run_agent.py] -- Invokes --> B[Python Toolsets]
    B -- Interfaces with --> C[Node.js Environment]
    C -- Loads --> D[@askjo/camofox-browser]
    C -- Loads --> E[agent-browser]
    subgraph "package.json Management"
    D
    E
    end
```

## Scripts and Lifecycle

### postinstall
The `postinstall` script serves as a bridge for developers setting up the environment. Once `npm install` is executed, it provides immediate feedback:
```bash
✅ Browser tools ready. Run: python run_agent.py --help
```
This indicates that the JavaScript-based browser drivers are successfully staged and the user should return to the Python entry point to operate the agent.

## Development Notes

- **Private Package**: The module is marked `"private": true`, preventing accidental publication to the npm registry.
- **Repository**: Linked to [NousResearch/Hermes-Agent](https://github.com/NousResearch/Hermes-Agent), which serves as the central hub for both the Python logic and these Node.js configurations.
- **Contribution**: When adding new browser-based tools, developers should check if additional Node.js packages are required and update the `dependencies` section accordingly, ensuring they are compatible with Node 20+.