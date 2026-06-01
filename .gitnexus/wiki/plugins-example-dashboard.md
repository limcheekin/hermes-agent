# plugins — example-dashboard

# Example Dashboard Plugin

The `example-dashboard` module serves as the reference implementation for the dashboard's plugin architecture. It demonstrates how to extend the user interface via tabs and slots while providing a backend API integrated into the application's routing system.

## Plugin Manifest (`manifest.json`)

The `manifest.json` file is the entry point for the plugin system. It defines how the plugin identifies itself and where it hooks into the main application.

| Field | Description |
| :--- | :--- |
| `name` | Unique identifier for the plugin (`example`). |
| `tab` | Defines a new top-level navigation item at `/example`, positioned after the `skills` tab. |
| `slots` | Registers the plugin to render content in specific UI locations, such as `sessions:top`. |
| `entry` | The compiled JavaScript bundle for the frontend (`dist/index.js`). |
| `api` | The Python file containing the backend router (`plugin_api.py`). |

## Backend API (`plugin_api.py`)

The plugin provides a backend extension using FastAPI. The dashboard plugin system automatically mounts the `router` defined in this file.

### Routing and Namespacing
All plugin routes are namespaced to prevent collisions. The example plugin's routes are accessible under:
`/api/plugins/example/`

### Endpoints
*   **GET `/hello`**: A demonstration endpoint that returns a JSON object containing a greeting and version metadata.

```python
@router.get("/hello")
async def hello():
    return {"message": "Hello from the example plugin!", "plugin": "example", "version": "1.0.0"}
```

## UI Integration Patterns

The plugin demonstrates two primary methods of frontend extension:

1.  **Full Page (Tabs)**: By defining a `tab` in the manifest, the plugin creates a dedicated route and navigation link. This is intended for standalone features.
2.  **Component Injection (Slots)**: By listing `sessions:top` in the `slots` array, the plugin signals that it provides a component to be rendered at the top of the Sessions view.

```mermaid
graph TD
    Dashboard[Dashboard Core] -->|Loads| Manifest[manifest.json]
    Manifest -->|Registers Tab| UI[Navigation Bar]
    Manifest -->|Injects Slot| Sessions[Sessions View]
    Manifest -->|Mounts API| FastAPI[FastAPI Router]
    FastAPI -->|Serves| Hello[/api/plugins/example/hello]
```

## Development and Deployment

### Frontend Entry Point
The `entry` field points to `dist/index.js`. This file should export the React components required for the tabs and slots defined in the manifest. The dashboard's plugin loader dynamically imports this script at runtime.

### Backend Mounting
The dashboard's plugin manager looks for the `api` key in the manifest. It imports the `router` object from the specified file (`plugin_api.py`) and includes it in the main FastAPI application logic. Developers must ensure the `router` variable is an instance of `fastapi.APIRouter`.