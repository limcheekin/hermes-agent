# Root — setup-hermes.sh

# setup-hermes.sh

The `setup-hermes.sh` script is the primary entry point for initializing a development or production environment for the Hermes Agent. It automates environment detection, dependency resolution, and system integration, ensuring the agent is ready for use with minimal manual intervention.

## Overview

The script follows a dual-path execution logic based on the host environment:
1.  **Desktop/Server Path:** Leverages [uv](https://github.com/astral-sh/uv) for high-performance Python management, virtual environment creation, and lockfile-verified dependency installation.
2.  **Android/Termux Path:** Utilizes the Python standard library `venv` and `pip` to accommodate the specific constraints and pre-compiled package requirements of the Termux environment.

## Execution Flow

```mermaid
graph TD
    A[Start] --> B{Is Termux?}
    B -- Yes --> C[Use stdlib venv + pip]
    B -- No --> D[Locate/Install uv]
    C --> E[Python 3.11+ Check]
    D --> F[uv Python 3.11 Provisioning]
    E --> G[Create venv]
    F --> G
    G --> H{Install Deps}
    H -- Termux --> I[pip install .[termux]]
    H -- Desktop --> J[uv sync --all-extras]
    I --> K[Sync Skills & Symlink CLI]
    J --> K
    K --> L[Run Setup Wizard]
```

## Key Components

### 1. Platform Detection
The script identifies the environment using the `is_termux` function, which checks for the existence of the `TERMUX_VERSION` environment variable or the standard Termux `$PREFIX` path. This determines whether the script uses `uv` or falls back to standard Python tooling.

### 2. Python Environment Management
*   **Desktop:** The script targets **Python 3.11**. It uses `uv python find` to locate an existing installation or `uv python install` to provision it automatically.
*   **Termux:** It verifies that the system `python` is at least version 3.11. If not, it directs the user to run `pkg install python`.

### 3. Dependency Installation
The script installs the package in editable mode (`-e .`), allowing developers to modify the source code without reinstalling.

*   **Extras:** 
    *   On Desktop, it attempts to install the `[all]` extra set.
    *   On Termux, it attempts the `[termux]` extra set, which includes platform-specific dependencies.
*   **Lockfile Verification:** On Desktop, if a `uv.lock` file is present, the script performs a `uv sync --locked` to ensure a reproducible environment. If the lockfile is stale or the sync fails, it falls back to a standard `uv pip install`.
*   **Constraints:** In Termux, it utilizes `constraints-termux.txt` to ensure compatibility with pre-built binaries available in the Termux repositories.

### 4. Submodule Integration
The script checks for the presence of the `tinker-atropos` directory (the Reinforcement Learning training backend). If found and the environment is not Termux, it installs the submodule in editable mode.

### 5. System Integration
To provide a seamless CLI experience, the script performs the following:
*   **CLI Symlinking:** It symlinks the `hermes` executable from the virtual environment to a user-facing bin directory (`~/.local/bin` on Desktop or `$PREFIX/bin` on Termux).
*   **PATH Modification:** On Desktop, it detects the active shell (`bash` or `zsh`) and appends `~/.local/bin` to the `PATH` in the corresponding configuration file (`.zshrc`, `.bashrc`, or `.bash_profile`) if it is not already present.
*   **Environment Files:** It initializes a `.env` file from `.env.example` if one does not exist.

### 6. Skill Seeding
The script ensures that bundled skills are available to the agent by running `tools/skills_sync.py`. This script synchronizes the skills from the repository into the user's Hermes home directory (defaulting to `~/.hermes/skills/`).

## Usage

Run the script from the root of the repository:

```bash
chmod +x setup-hermes.sh
./setup-hermes.sh
```

### Post-Installation
Upon successful completion, the script offers to run the **Setup Wizard** (`hermes setup`). This interactive CLI tool configures essential API keys and preferences stored in the `.env` file.

## Optional Dependencies
The script checks for `ripgrep` (`rg`), which is used by the agent for high-performance file searching. If missing, it provides an interactive prompt to install it via the system package manager (`apt`, `dnf`, `brew`, `pkg`) or `cargo`.