# Grok-Local Hermes Suite — Interface Contract

## 1. Local Server & Static Hosting (`gateway/platforms/api_server.py`)
- Bind strictly to `127.0.0.1:8642` when `HERMES_OFFLINE_MODE=1`.
- Serve compiled frontend static files from `web_ui/dist/` at `/`.
- Auth header: `Authorization: Bearer <API_SERVER_KEY>`.

## 2. REST & SSE Endpoints
- `POST /v1/chat/completions` (SSE stream):
  - Standard OpenAI delta chunks PLUS structured custom SSE events:
    - `event: reasoning_delta` -> `{"session_id": str, "text": str}`
    - `event: tool_step` -> `{"id": str, "tool": str, "status": "running|done|error", "summary": str}`
    - `event: approval_required` -> `{"approval_id": str, "tool": str, "args": dict, "reason": str}`
- `GET /api/sessions` & `GET /api/sessions/{id}` -> SQLite + FTS5 query from `SessionDB`.
- `GET /api/toolsets` & `POST /api/sessions/{id}/mode` -> toggle `auto | code | gui | research | chat_only`.
- `POST /api/approvals/{approval_id}` -> body `{"decision": "approve|always_allow|edit|discard", "edited_args": dict|null}`.
- `GET /api/library` -> lists `~/.hermes/pages/` (pinned, tags, git revision) + workspace artifacts.
- `POST /api/pages/{slug}` -> saves user edits, computes `git diff`, queues diff into next turn reflection.
- `POST /api/recorder/start` & `POST /api/recorder/stop` -> `{"mode": "browser|terminal", "skill_name": str}`.
- `WS /ws/voice` -> binary PCM16 16kHz audio in/out (Silero VAD -> Faster-Whisper -> AIAgent -> Kokoro TTS).

## 3. Plugin Tools (`plugins/grok_local_suite/`)
- `vault_use(vault_id: str, mode: str = "playwright_fill|subprocess_env")`
- `page_manage(action: str, slug: str, content: str = "", title: str = "")`
- `handoff_to_bot(profile: str, task: str, files: list[str] = [])`
- `gui_observe(scope: str = "active_window", force_vision: bool = False)`
- `gui_click(element_id: str = None, x: int = None, y: int = None, button: str = "left", clicks: int = 1)`
- `gui_type(text: str, element_id: str = None, press_enter: bool = False)`
- `gui_hotkey(keys: list[str])`
- `gui_scroll(direction: str = "down", amount: int = 5, element_id: str = None)`
