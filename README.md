# GPT-5 Local MCP — Multimodal Chat Platform for Developers

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Fryzz27/gpt5mcp/releases)

![AI Concept](https://images.unsplash.com/photo-1531297484001-80022131f5a1?ixlib=rb-4.0.3&q=80&w=1200&auto=format&fit=crop&sat=-20)

GPT-5 Local MCP (gpt5mcp) builds a local multimodal chat platform that runs GPT-5 style models on your own hardware. The project targets developers who need an offline assistant, a testbed for model integration, or a local API for research. The README below explains features, install steps, usage, configuration, examples, and maintenance.

- Repository: gpt5mcp  
- Project: GPT-5 Local MCP  
- Releases: https://github.com/Fryzz27/gpt5mcp/releases (download the release file and execute it)

Table of contents
- Features
- Architecture
- Quick start (download & run)
- Install (detailed)
- Configuration
- Run modes
- API examples
- Client example (node + python)
- Model management
- Troubleshooting
- Security notes
- Contributing
- License
- Acknowledgements

Features
- Local inference for GPT-5-style multimodal models (text + images + audio).
- Modular connector system for swapping models, tokenizers, and backends.
- Minimal local API compatible with common SDKs.
- Worker-based pipeline for batching and concurrency.
- Persistent context store with simple vector index.
- Native file upload and processing hooks.
- Authentication plugin points for local deployment.
- CLI and web UI for testing and development.

Architecture
- Core: minimal runtime that routes requests between API, workers, and model backends.
- Workers: stateless processes that load models and serve inference.
- Store: local vector index and context DB (lightweight, file-based).
- API: REST + WebSocket endpoints for chat, files, and model control.
- UI: small React app for local testing (optional).
- Plugins: tokenizer, preprocessors, postprocessors.

Quick start (download & run)
1. Visit the Releases page and download the release asset:
   https://github.com/Fryzz27/gpt5mcp/releases
2. Download the release file (for example gpt5mcp-v1.0.0-linux.tar.gz or gpt5mcp-v1.0.0-windows.exe) and execute it.
3. Run the bundled CLI to start the server:
   - Linux / macOS:
     - tar xzf gpt5mcp-v1.0.0-linux.tar.gz
     - ./gpt5mcp start
   - Windows:
     - Double-click gpt5mcp-v1.0.0-windows.exe or run from PowerShell

If the release link does not work, check the Releases section on GitHub.

Install (detailed)

Prerequisites
- Hardware: GPU recommended for large model inference. CPU-only runs are supported for small models.
- OS: Linux, macOS, Windows (WSL recommended for Windows).
- Disk: 10+ GB free for basic models; 50+ GB for larger model sets.
- Software: Docker optional, Python 3.10+ for local development, Node 16+ for the UI.

From Releases (recommended for non-dev use)
- Download the appropriate release asset for your OS from:
  https://github.com/Fryzz27/gpt5mcp/releases
- Extract and run the bundled startup script or executable.
- The installer will detect GPU drivers and configure runtime options.

From source (developer)
- Clone the repo:
  git clone https://github.com/Fryzz27/gpt5mcp.git
  cd gpt5mcp
- Create a Python venv:
  python -m venv .venv
  source .venv/bin/activate
- Install dependencies:
  pip install -r requirements.txt
- Build the UI:
  cd ui
  npm install
  npm run build
- Start the server:
  python -m gpt5mcp.server --config config/local.yaml

Configuration
- config/local.yaml contains runtime options:
  - port: API port (default 8080)
  - model_dir: path to local models
  - db_path: path to context DB
  - workers: number of worker processes
  - auth: simple token auth or disable for local dev
- Environment variables:
  - GPT5MCP_PORT
  - GPT5MCP_MODEL_DIR
  - GPT5MCP_WORKERS
- Example config snippet:
  api:
    host: 0.0.0.0
    port: 8080
  model:
    default: local-gpt5-small
    model_dir: ./models
  workers:
    count: 2

Run modes
- Dev mode: hot reload, debug logging, local models only.
  python -m gpt5mcp.server --dev
- Production mode: optimized, process manager, TLS support.
  ./gpt5mcp start --prod
- Docker mode:
  docker build -t gpt5mcp:latest .
  docker run -p 8080:8080 -v $(pwd)/models:/app/models gpt5mcp:latest

API (HTTP)
- Base: GET /health
- Chat: POST /v1/chat
  - body: { "model": "local-gpt5-small", "messages": [{ "role": "user", "content": "Hello" }], "context_id": "session-123" }
  - returns: { "id": "...", "choices": [...] }
- Stream: WebSocket /ws/chat for streaming tokens
- Files: POST /v1/files (multipart) to upload images or audio
- Model control: GET /v1/models, POST /v1/models/load, POST /v1/models/unload

Example HTTP request (curl)
- Text chat:
  curl -X POST http://localhost:8080/v1/chat \
    -H "Content-Type: application/json" \
    -d '{"model":"local-gpt5-small","messages":[{"role":"user","content":"Describe a sunrise."}], "context_id":"demo"}'

Client example (Node.js)
- Install:
  npm install axios ws
- Simple script:
  const axios = require('axios');
  async function chat() {
    const resp = await axios.post('http://localhost:8080/v1/chat', {
      model: 'local-gpt5-small',
      messages: [{ role: 'user', content: 'Show code to reverse a string in Python.' }]
    });
    console.log(resp.data);
  }
  chat();

Client example (Python)
- Install:
  pip install requests
- Simple script:
  import requests
  resp = requests.post('http://localhost:8080/v1/chat', json={
    "model": "local-gpt5-small",
    "messages": [{"role": "user", "content": "List steps for unit testing a function."}]
  })
  print(resp.json())

Model management
- Models live in model_dir. Each model is a folder with config.json and weights.
- Use the model API to load/unload at runtime:
  POST /v1/models/load { "name": "local-gpt5-small", "path": "./models/local-gpt5-small" }
- Supported backends:
  - native PyTorch
  - ONNX Runtime
  - TensorRT (optional)
- Tokenizers:
  - Supports Hugging Face tokenizers and internal BPE.
  - You can swap tokenizer plugin in config.

Persistence & context
- The platform stores chat context per session id.
- Vector index uses FAISS or a light fallback index.
- Context length: configurable via config/local.yaml (default 2048 tokens).
- You can flush context per session via DELETE /v1/context/{id}.

Security notes
- The release includes a basic token auth option. For local, you can disable auth.
- For production, enable TLS and a secure auth layer.
- Avoid exposing the server to public networks without proper controls.

Logging & metrics
- Logs go to stdout and rotate in logs/.
- Metrics endpoint: /metrics (Prometheus format)
- Tracing: optional OpenTelemetry hooks.

Troubleshooting
- Server fails to start:
  - Check port conflict and logs/logs.log.
  - Verify model paths in config.
- Model fails to load:
  - Confirm model files exist and match backend.
  - Check GPU driver compatibility.
- Web UI shows blank:
  - Build UI (cd ui && npm run build) and confirm static files served.
- If the release link is not available, check the Releases section on GitHub:
  https://github.com/Fryzz27/gpt5mcp/releases

Performance tuning
- Use GPU backend for large models.
- Increase workers for higher concurrency.
- Batch inference where possible.
- Tune context length to balance memory and coherency.

Contributing
- Code style: follow the linters in .github/workflows.
- Branching:
  - main for stable
  - develop for active changes
  - feature/* for features
- PR checklist:
  - Tests pass
  - Documentation updated
  - Lint checks pass
- How to run tests:
  pytest tests

Roadmap (planned)
- Plugin marketplace for model adapters.
- Native quantized inference for larger models.
- More sample agents and templates.
- Better Windows support and installer.

Assets & badges
[![Releases](https://img.shields.io/badge/Release-Download-green?logo=github)](https://github.com/Fryzz27/gpt5mcp/releases)

Files and folders
- /server — server runtime and API
- /models — model bundles and configs
- /workers — worker processes and loaders
- /ui — React app for local testing
- /scripts — utility scripts for packaging
- README.md — this file

License
- MIT License — see LICENSE file.

Acknowledgements
- Model loader inspired by community projects.
- UI components use open-source React libraries.
- Image assets by Unsplash contributors.