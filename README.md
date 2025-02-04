# Pipecat Lead Qualifier

Pipecat Lead Qualifier is a modular voice assistant application that uses a FastAPI server to orchestrate bot workflows and a Next.js client for user interactions.

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
  - [Server](#server)
  - [Client](#client)
- [Setup and Installation](#setup-and-installation)
- [Running the Application](#running-the-application)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Project Overview

The project qualifies leads by guiding users through a series of conversational steps. It comprises two main components:

1. **Server:** Built with FastAPI, this component manages bot workflows and handles integrations (e.g., Daily rooms, transcription, TTS, and OpenAI services).
2. **Client:** Developed with Next.js and TypeScript, this component serves as the front-facing widget.

## Architecture

### Server

#### Directory Structure
```
server/
├── __init__.py            # Package initialization and version info
├── main.py                # FastAPI server entry point
├── Dockerfile             # Container configuration
├── requirements.txt       # Python dependencies
├── bots/                  # Bot implementations
│   ├── __init__.py
│   ├── base_bot.py        # Shared bot framework
│   ├── flow.py            # Flow-based bot implementation
│   └── simple.py          # Simple bot implementation
├── config/                # Configuration management
│   ├── __init__.py
│   └── settings.py        # Environment and app settings
├── prompts/               # LLM system prompts
│   ├── __init__.py
│   └── prompts.py         # Prompt templates and helpers
├── services/              # External API integrations
│   ├── __init__.py
│   └── calcom_api.py      # Cal.com API client
└── utils/                 # Common utilities
    ├── __init__.py
    └── run_helpers.py     # Bot lifecycle helpers
```

#### Key Components

- **`main.py`**  
  The FastAPI server entry point that handles:
  - Room creation and management
  - Bot process lifecycle
  - HTTP endpoints for browser and RTVI access
  - Connection credential management

- **`bots/`**  
  Contains bot implementations with:
  - `base_bot.py`: Shared framework and service initialization
  - `flow.py`: Sophisticated flow-based conversation logic
  - `simple.py`: Basic single-prompt implementation

- **`config/`**  
  Manages application configuration:
  - Environment variable validation
  - Type-safe settings classes
  - Default value handling

- **`services/`**  
  External API integrations:
  - Cal.com API for appointment scheduling
  - Future integrations can be added here

- **`utils/`**  
  Common utilities and helpers:
  - Bot lifecycle management
  - Shared helper functions

### Client

#### Directory Structure
- **Directory:** `client/`

#### Overview
- Developed with Next.js using TypeScript with strict type checking.
- Follows Next.js conventions: routes under `/app` (or `/pages`), shared components in `/components`, and styles in `/styles` or via CSS Modules.
- Managed with pnpm for dependency handling.

## Setup and Installation

### Environment Setup

Create a `.env` file with the required environment variables:
```bash
cat << 'EOF' > .env
DAILY_API_KEY=your_daily_api_key
DEEPGRAM_API_KEY=your_deepgram_api_key
OPENAI_API_KEY=your_openai_api_key

# Optional variables (override defaults as needed)
DAILY_API_URL=https://api.daily.co/v1
TTS_PROVIDER=cartesia              # Optional: indicates a preference, but CARTESIA_API_KEY is what drives the Cartesia integration.
DEEPGRAM_VOICE=aura-athena-en
CARTESIA_API_KEY=your_cartesia_api_key  # Provide only if you want to use Cartesia TTS.
CARTESIA_VOICE=your_cartesia_voice_identifier
OPENAI_MODEL=gpt-4o
OPENAI_TEMPERATURE=0.2
BOT_TYPE=flow                      # or "simple"
EOF
```

Ensure that the `.env` file is excluded from version control:
```bash
grep -qxF ".env" .gitignore || echo ".env" >> .gitignore
```

### Advanced Configuration Options

In addition to the required environment variables (`DAILY_API_KEY`, `DEEPGRAM_API_KEY`, `CARTESIA_API_KEY`, and `OPENAI_API_KEY`), you can customize the behavior of the application by setting the following optional environment variables in your `.env` file:

- **DAILY_API_URL**  
  - **Default:** `https://api.daily.co/v1`  
  - **Usage:** Sets the base URL for the Daily API. This URL is used when initializing the Daily transport in the bot.
  
- **TTS_PROVIDER**  
  - **Default:** `deepgram`  
  - **Usage:** Determines which Text-to-Speech service your bot will use.  
    - If set to `cartesia`, the application uses the `CartesiaTTSService` and relies on the **CARTESIA_API_KEY** and **CARTESIA_VOICE** values.  
    - Otherwise, it uses `DeepgramTTSService` along with **DEEPGRAM_API_KEY** and **DEEPGRAM_VOICE**.

- **DEEPGRAM_VOICE**  
  - **Default:** `aura-athena-en`  
  - **Usage:** Specifies the voice identifier for the Deepgram TTS service when **TTS_PROVIDER** is not set to `cartesia`.

- **CARTESIA_VOICE**  
  - **Default:** `79a125e8-cd45-4c13-8a67-188112f4dd22`  
  - **Usage:** Specifies the voice identifier for the Cartesia TTS service. This is used only if **TTS_PROVIDER** is set to `cartesia`.

- **OPENAI_MODEL**  
  - **Default:** `gpt-4o`  
  - **Usage:** Determines which OpenAI model to use when initializing the LLM service in the bot.

- **OPENAI_TEMPERATURE**  
  - **Default:** `0.2`  
  - **Usage:** Controls the temperature (randomness) of the OpenAI language model. The value is passed as part of the `InputParams` to the LLM service.

- **BOT_TYPE**  
  - **Default:** `simple`  
  - **Valid Values:** `simple` or `flow`  
  - **Usage:** Specifies which bot variant to launch. This setting is checked at startup and used to determine the corresponding bot implementation within the `server/bots/` package.

These optional variables are processed by the `AppConfig` class in `config/settings.py`. In the `bots/base_bot.py` module, the configuration is used as follows:

- **STT & TTS Initialization:**  
  The bot initializes Deepgram's STT service using **DEEPGRAM_API_KEY**. Depending on **TTS_PROVIDER**, it either initializes the Deepgram TTS service (using **DEEPGRAM_VOICE**) or the Cartesia TTS service (using **CARTESIA_API_KEY** and **CARTESIA_VOICE**).

- **LLM Setup:**  
  The bot sets up the OpenAI LLM service with **OPENAI_API_KEY**, **OPENAI_MODEL**, and additional parameters (such as **OPENAI_TEMPERATURE** bundled into the `InputParams`). This is used to drive conversation logic.

- **Bot Behavior:**  
  The **BOT_TYPE** variable determines whether a "simple" or "flow" bot implementation is executed; this affects the orchestration and logic sequence within the bot's processing pipeline.

By adjusting these variables in your `.env` file, you can fine-tune service integrations and bot behavior without modifying the application code.

### Server Setup

Navigate to the `server` directory, set up a virtual environment, and install the dependencies:
```bash
cd server
python -m venv venv
# Activate the virtual environment:
# On Linux/macOS:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate
pip install -r requirements.txt
pip install -e "../external/pipecat[daily,openai,deepgram,cartesia,silero]"
pip install -e "../external/pipecat-flows"
```

### Client Setup

Navigate to the `client` directory and install dependencies using pnpm:
```bash
cd ../client
pnpm install
```

## Running the Application

### Server

#### Local Development
Run the server from the `server` directory:
```bash
python -m main --bot-type flow  # Use "simple" instead of "flow" for the simple bot variant
```

#### Docker Container
Build and run the server in a Docker container:
```bash
docker build -t pipecat-server:latest -f server/Dockerfile .
docker run -p 7860:7860 pipecat-server:latest
```

### Client

#### Development Mode
Run the Next.js client in development mode:
```bash
pnpm dev
```

#### Production Build
To build and start the production version of the client:
```bash
pnpm build
pnpm start
```

## Testing

- **Server:**  
  Follow the testing guidelines provided in the codebase. Tests should use the Arrange-Act-Assert pattern to verify API endpoints and bot functionalities.
  
- **Client:**  
  Execute frontend tests using your preferred test runner (e.g., Jest).

## Contributing

Please adhere to the project conventions:
- Write clear, modular functions with single purposes.
- Follow SOLID principles.
- For client-side development, adhere to Next.js and TypeScript guidelines.
- For server-side development, follow FastAPI and Pipecat conventions.

## License

[MIT License](LICENSE.md)

## Contact

For any questions or issues, please contact [john@askjohngeorge.com](mailto:john@askjohngeorge.com).