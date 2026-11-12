# MYRAA — AI Desktop Companion

<p align="center">
  <strong>A private, voice-enabled AI companion for Windows with a 3D character, screen awareness, and desktop interaction.</strong>
</p>

<p align="center">
  <img alt="Version" src="https://img.shields.io/badge/version-1.5.2-7c3aed">
  <img alt="Platform" src="https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-2563eb">
  <img alt="Frontend" src="https://img.shields.io/badge/frontend-React%2019%20%2B%20Vite-61dafb">
  <img alt="Backend" src="https://img.shields.io/badge/backend-Node.js%20%2B%20TypeScript-3178c6">
  <img alt="AI" src="https://img.shields.io/badge/AI-Google%20Gemini-4285f4">
</p>

MYRAA is a Windows desktop AI companion built around natural voice interaction and a real-time 3D character. It combines Google's Gemini models with a local application backend and a Python desktop-control service, allowing the assistant to communicate, understand permitted screen content, work with desktop applications, and assist with everyday computer tasks.

This repository contains the **MYRAA 1.5.2 source code**. It is intended for developers who want to inspect, run, test, or build the application.

> **Important:** MYRAA is designed primarily for Windows 10/11. Some desktop-control, native integration, and packaging features depend on Windows-specific APIs and should not be assumed to work on macOS or Linux.

---

## Contents

- [What MYRAA does](#what-myraa-does)
- [Core capabilities](#core-capabilities)
- [Technology stack](#technology-stack)
- [Architecture](#architecture)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Running MYRAA](#running-myraa)
- [Building the Windows application](#building-the-windows-application)
- [Testing](#testing)
- [Privacy and security](#privacy-and-security)
- [Known limitations and notes](#known-limitations-and-notes)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License and third-party assets](#license-and-third-party-assets)

---

## What MYRAA does

MYRAA is more than a text chatbot in a desktop window. The application is structured as a local desktop assistant with several connected parts:

1. A React interface for conversations, settings, and character presentation.
2. A Three.js-based 3D character and animation system.
3. A Node.js/TypeScript backend that manages Gemini communication and application services.
4. An Electron desktop shell for Windows integration.
5. A Python FastAPI agent for interacting with the local Windows desktop.
6. Supporting systems for memory, permissions, task planning, screen understanding, and API integrations.

The application is designed so that AI requests are routed through the local backend rather than exposing the Gemini API key directly to the browser interface.

## Core capabilities

The repository contains implementation for the following areas. Actual availability can depend on platform support, permissions, configuration, and whether the required local services are installed.

### AI conversation and voice

- Gemini-powered conversational experience.
- Real-time voice interaction through the Gemini Live integration.
- Microphone capture and audio playback.
- Voice activity detection and interruption handling.
- Text and speech turn coordination.
- Configurable model routing for different capabilities.

### 3D character experience

- Real-time 3D character rendering using Three.js.
- Character animation, procedural idle behaviour, gaze, expressions, and lip-sync support.
- Character configuration and graphics-quality controls.
- Character import and character-studio tooling for supported MMD/PMX assets.
- Included character-state videos for idle, thinking, and talking presentation.

**Character model notice:** A redistributable 3D character model is not included in this source archive. Import a model that you have permission to use through the character-import flow or the character tools in the application. Do not add third-party models to a public repository unless their licence explicitly permits redistribution.

### Screen awareness and vision

- Screen-sharing and screen-vision pipeline code.
- Screen capture and frame-processing support.
- Gemini vision integration for interpreting permitted visual context.
- Screen-awareness controls and permission gates.

Screen access is sensitive. It should only be enabled when the user intentionally grants access. Stop screen sharing when it is no longer needed.

### Windows desktop assistance

The Python desktop agent provides a local tool layer for supported Windows actions, including:

- Opening and closing applications.
- Opening websites and performing web searches.
- Reading, creating, renaming, moving, and managing files in supported locations.
- Keyboard, mouse, and window interaction.
- Clipboard operations.
- Screenshots and screen text extraction/OCR.
- Basic system, CPU, memory, and GPU information where supported.
- Coding-related file creation and script execution.

Some operations require additional Windows permissions or optional software. The agent is intended to run locally and communicate with the Node backend over loopback.

### Memory and cognition

The codebase includes systems for:

- Conversation and structured memory.
- Memory retrieval, correction, and persistence.
- Event-driven context and situation tracking.
- Attention, relevance, and initiative decisions.
- Goal planning, progress tracking, and cancellation.
- Skill/workflow records and outcome tracking.
- Model routing, bounded retries, and task limits.

Autonomous or proactive behaviours are controlled by configuration and permission gates. They should not be interpreted as unlimited or unrestricted background access.

### API Hub

MYRAA includes an API Hub that can import and validate a public API catalogue, search provider capabilities, check provider status, and call a limited set of verified declarative adapters.

The presence of an API in a catalogue does **not** mean it is automatically trusted or callable. The adapter system applies restrictions to destinations, methods, timeouts, redirects, credentials, and response handling.

---

## Technology stack

| Area | Technologies |
|---|---|
| User interface | React 19, TypeScript, Vite |
| Styling and UI motion | Tailwind CSS 4, Motion |
| 3D rendering | Three.js, MMD parser |
| Desktop application | Electron |
| Backend | Node.js, TypeScript, Express, WebSocket |
| AI integration | Google GenAI SDK, Gemini Live and model APIs |
| Desktop automation | Python, FastAPI, Uvicorn |
| Windows integration | pywin32, pywinauto, PyAutoGUI and related Windows libraries |
| Testing and tooling | TypeScript compiler, Node test runner, Python unittest, tsx |
| Packaging | electron-builder, esbuild, PyInstaller |

Dependency versions and scripts are defined in `package.json`, `package-lock.json`, and `desktop_agent/requirements.txt`.

---

## Architecture

At a high level, the application is organised like this:

```text
                     ┌─────────────────────────────┐
                     │       Electron Shell        │
                     │  Windows desktop lifecycle  │
                     └──────────────┬──────────────┘
                                    │
                     ┌──────────────▼──────────────┐
                     │       React Renderer        │
                     │ Chat • Voice • 3D Character │
                     │ Settings • Screen Sharing   │
                     └──────────────┬──────────────┘
                                    │
                         HTTP / WebSocket / IPC
                                    │
                     ┌──────────────▼──────────────┐
                     │   Node.js / TypeScript      │
                     │ Express • Gemini • Runtime  │
                     │ Memory • Cognition • Safety │
                     └───────┬─────────────┬───────┘
                             │             │
                   Gemini API│             │Local HTTP
                             │             │
                ┌────────────▼───┐   ┌─────▼────────────────┐
                │ Google Gemini  │   │ Python Desktop Agent │
                │ Live / Models  │   │ FastAPI / Windows    │
                └────────────────┘   └──────────┬───────────┘
                                               │
                                    Windows Apps / Files /
                                    Input / Screen / System
```

### Important backend boundaries

- The renderer does not need direct access to the Gemini API key.
- Gemini requests are handled by the local Node backend.
- The Python desktop agent is a separate local service used for Windows-specific tools.
- Electron communicates with its backend through local process integration.
- User data and runtime state are stored separately from the packaged application files.

---

## Repository structure

The project is split into feature-focused directories:

| Path | Responsibility |
|---|---|
| `src/` | React application, UI, 3D character system, character animation, and interface components |
| `src/character/` | Character rendering, animation, behaviour, expressions, lip-sync, and character tools |
| `electron/` | Electron main process, preload bridge, desktop companion, and Windows integration |
| `server.ts` | Main Express/WebSocket backend and service orchestration |
| `server_memory.ts` | Conversation-memory integration |
| `server_paths.ts` | Runtime data paths and API-key resolution |
| `server_screenVision.ts` | Screen-vision processing pipeline |
| `runtime/` | MYRAA runtime and character-library services |
| `desktop_agent/` | Python FastAPI desktop-control service and Windows tools |
| `agent/` | Task execution, planning, action catalogues, and agent safety logic |
| `cognition/` | Event-driven cognition, attention, goals, planning, and structured memory |
| `models/` | Model registry, routing, provider integrations, retry, and usage handling |
| `voice/` | Gemini Live voice-agent tool definitions and voice integration |
| `permissions/` | Permission and action-confirmation logic |
| `secrets/` | Protected secret storage |
| `memory/` | Supporting memory utilities |
| `presence/` | Conversation and user-presence state |
| `settings/` | Application settings |
| `api_hub/` | API catalogue, provider registry, health checks, and verified adapters |
| `character_import/` | Character archive and PMX import utilities |
| `assets/` | Character-state videos and other application assets |
| `native/` | Native Windows integration |
| `shared/` | Shared utilities and safety helpers |
| `docs/` | Architecture and API Hub documentation |
| `tests/` | Application-level tests |
| `scripts/` | Development and build scripts |
| `tools/` | Developer and character-import tools |
| `build/` | Application icons and build resources |

---

## Requirements

Install the following before running the project.

### Required

- **Windows 10 or Windows 11 (64-bit)** for the intended desktop experience.
- **Node.js 20 or newer**.
- **npm**, included with Node.js.
- **Python 3.11+** for the Python desktop agent and Python tests.
- A **Google Gemini API key** from [Google AI Studio](https://aistudio.google.com/).

### Optional

- **Tesseract OCR** for additional OCR functionality. Windows OCR support is also used where available.
- **PyInstaller** when building the bundled Python desktop agent.
- A compatible MMD/PMX character model that you are licensed to use.

Check your installed versions:

```powershell
node --version
npm --version
python --version
```

---

## Getting started

### 1. Download the source

Clone the repository:

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Or download the repository ZIP from GitHub and extract it.

### 2. Install Node dependencies

From the project root:

```bash
npm install
```

### 3. Install Python desktop-agent dependencies

Use Python 3.11 or newer:

```powershell
python -m pip install -r desktop_agent/requirements.txt
```

If the `python` command points to a different interpreter, use the full path to your Python 3.11+ executable.

### 4. Configure Gemini

MYRAA requires your own Gemini API key for AI features.

Get a key from:

[Google AI Studio](https://aistudio.google.com/)

When prompted by the application, enter your own key. The Windows desktop application stores the key using Windows DPAPI-protected storage.

For local development, you may alternatively copy the example environment file:

```powershell
Copy-Item .env.example .env
```

Then edit `.env` and set the required values. Never commit a real API key, token, or secret to GitHub.

### 5. Start the development server

```bash
npm run dev
```

Open the local address printed in the terminal. The Vite configuration uses port `3000` when the server's development setup selects that default.

For the Electron desktop experience, use:

```bash
npm run app
```

This builds the application and launches the Electron shell. Windows-specific desktop features are intended to be used from the desktop application.

---

## Configuration

The `.env.example` file documents the available environment variables. Copy it to `.env` only for local development and adjust settings as needed.

### Gemini and model routing

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini API key for local development |
| `MYRAA_FAST_MODEL` | Model route for fast tasks |
| `MYRAA_REASONING_MODEL` | Model route for reasoning tasks |
| `MYRAA_CODE_MODEL` | Model route for coding tasks |
| `MYRAA_VISION_MODEL` | Model route for vision tasks |
| `MYRAA_SPEECH_MODEL` | Model route for speech/Live tasks |
| `MYRAA_EMBEDDING_MODEL` | Model route for embedding tasks |

Use model identifiers supported by your Google AI account and the installed SDK. The example values are defaults, not a guarantee that every model is available to every account.

### Cognition and autonomy

The example environment file also includes feature switches for cognition, initiative, proactive speech, skill learning, reflection, and desktop awareness.

Examples:

```env
MYRAA_ENABLE_COGNITION=true
MYRAA_ENABLE_INITIATIVE=true
MYRAA_ENABLE_PROACTIVE_SPEECH=true
MYRAA_ENABLE_DESKTOP_AWARENESS=true
MYRAA_ENABLE_SCREEN_AWARENESS=false
PAUSE_AUTONOMY=false
```

Review `.env.example` for the full list of feature flags, permission switches, model routes, and execution limits.

### Runtime data

The backend supports a writable data directory through `MYRAA_DATA_DIR`. Packaged Electron builds use a per-user data location so settings, logs, and other runtime data do not need to be written into the installation directory.

Do not upload personal runtime data, memory files, logs, or local secret stores to a public repository.

---

## Running MYRAA

### Development mode

```bash
npm run dev
```

Runs the TypeScript backend/development application.

### Electron desktop mode

```bash
npm run app
```

Builds the application bundle and launches Electron.

### Production server build

```bash
npm run build
npm start
```

The build script creates the Vite renderer build and bundles the Node backend into `dist/server.cjs`.

---

## Building the Windows application

The repository includes electron-builder configurations for Windows installers.

### 1. Prepare the Python agent

Install PyInstaller:

```powershell
python -m pip install pyinstaller
```

Build the agent:

```powershell
pyinstaller desktop_agent.spec --distpath agent_dist --workpath agent_build
```

### 2. Build the public distribution

```powershell
npm run dist:public
```

The public build uses `electron-builder.public.yml` and outputs Windows installer/portable artifacts under `release-public/`.

The public configuration intentionally excludes `assets/characters/`. Users must import their own properly licensed character model.

### 3. Build the owner/personal distribution

```powershell
npm run dist
```

This uses `electron-builder.yml` and expects the character assets configured for that build to be present. Do not use this configuration to distribute models unless their licences permit it.

Builds target Windows x64 NSIS installer and portable formats.

---

## Testing

Run the TypeScript type check:

```bash
npm run lint
```

Run the main test suite:

```bash
npm test
```

The repository also defines focused test commands:

```bash
npm run test:agent
npm run test:characters
npm run test:cognition
npm run test:screen-vision
npm run test:real-browser
npm run test:api-hub
npm run test:python
```

Some tests require Windows-specific dependencies or a suitable Windows environment. A passing test suite on one platform does not guarantee that every desktop integration works on another platform.

---

## Privacy and security

MYRAA can interact with sensitive parts of a computer. Review permissions and understand the behaviour of any tool before enabling it.

- **Use your own API key.** Never publish it in source code, screenshots, issue reports, or commits.
- **Keep credentials local.** The application is designed to keep Gemini calls behind the local backend and to store the Windows desktop key using DPAPI-protected storage.
- **Grant screen access intentionally.** Screen capture can expose private messages, documents, account information, and other visible content.
- **Review desktop actions.** File operations, keyboard/mouse input, application control, and script execution can affect the local machine.
- **Confirm sensitive actions.** The codebase includes confirmation gates for higher-risk actions and separate confirmation handling for power operations.
- **Treat external content as untrusted.** Text from websites, documents, and screens should not be treated as trusted instructions.
- **Keep autonomy bounded.** Review the permission switches and execution limits in `.env.example` before enabling proactive or automated behaviour.
- **Do not expose local services publicly.** The desktop agent and local backend are intended for local use; do not forward their ports to the public internet.

MYRAA is a development project, not a substitute for reviewing actions before they affect important files, accounts, or system settings.

---

## Known limitations and notes

- The intended desktop platform is Windows 10/11. Native desktop functionality is platform-dependent.
- A Gemini API key is not included. Each user must supply their own.
- A redistributable 3D character model is not included in this source archive. Import a model whose licence allows your intended use.
- OCR quality depends on the installed OCR engine, language data, screen content, and Windows support.
- Gemini model availability, API quotas, and pricing are controlled by Google and may change.
- Some desktop tools require optional dependencies, Windows permissions, or additional system configuration.
- The source archive contains implementation and tests, but that alone is not a guarantee that every feature is production-ready on every machine.

---

## Troubleshooting

### `npm install` fails

- Confirm that Node.js 20+ and npm are installed.
- Run the command from the directory containing `package.json`.
- If the dependency tree is corrupted, remove `node_modules` and run `npm install` again.

### Python packages fail to install

- Confirm that Python 3.11+ is selected.
- Upgrade pip:

```powershell
python -m pip install --upgrade pip
```

- Retry:

```powershell
python -m pip install -r desktop_agent/requirements.txt
```

### MYRAA cannot connect to Gemini

- Check that your API key is valid.
- Confirm that the selected model is available to your account.
- Check network connectivity and API quota.
- Do not paste your API key into public logs or issue reports.

### Desktop tools are unavailable

- Run MYRAA on Windows.
- Confirm the Python desktop-agent dependencies are installed.
- Check that the local agent can start and that no security software is blocking local loopback communication.
- Review the terminal output and application logs for the specific failure.

### Character does not appear

- Import a compatible MMD/PMX model that you have permission to use.
- Check the character import report and graphics-quality settings.
- Try a lower graphics-quality preset if rendering performance is limited.

---

## Contributing

Contributions, bug reports, and suggestions are welcome.

Before submitting a change:

1. Explain the issue or improvement clearly.
2. Keep changes focused and consistent with the existing architecture.
3. Avoid committing API keys, personal data, local memory files, logs, build output, or unlicensed character assets.
4. Run the relevant type checks and tests.
5. Include reproduction steps for bug reports.

For larger changes, open an issue or discussion first so the proposed design can be reviewed.

---

## License and third-party assets

No explicit open-source licence is declared in the supplied source archive. Unless a licence is added by the project owner, do not assume that the source code is released under MIT, Apache-2.0, GPL, or another open-source licence. All rights remain with their respective owners.

Third-party libraries remain subject to their own licences. Character models, animations, audio, and other media may have separate usage and redistribution restrictions.

If you intend to publish this repository publicly, add a licence only after deciding what rights you want to grant and confirming that all included assets may be redistributed.

---

<p align="center">
  <strong>MYRAA</strong><br>
  A more personal way to interact with your PC.
</p>
