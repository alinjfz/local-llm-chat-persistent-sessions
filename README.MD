# Ollama Conversational Interface: Self-Hosted AI Chat Application with Persistent Multi-Session Management

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-2.x-000000?style=flat&logo=flask&logoColor=white)
![React](https://img.shields.io/badge/React-18.3-61DAFB?style=flat&logo=react&logoColor=black)
![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-7952B3?style=flat&logo=bootstrap&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-Llama%203.1%3A8b-FF6B35?style=flat)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

A fully self-hosted AI chat application that runs a large language model locally on your machine with zero API costs and complete data privacy. Built with a Flask REST backend and a React frontend, it supports multiple independent conversations with automatic persistence — all data stays on your machine.

---

## What This Project Does

This application lets you have rich, multi-turn conversations with the **Llama 3.1:8b** model running locally via [Ollama](https://ollama.com). Every conversation is automatically saved to disk and available across sessions through a collapsible sidebar. You can start a new chat, switch between past conversations, and delete ones you no longer need — all without touching an API key or sending data to any external service.

---

## Features

- **Local LLM Inference** — Runs Llama 3.1:8b entirely on your machine via Ollama. No API key required. No data leaves your device.
- **Multi-Session Conversation Management** — Maintain multiple independent chat threads simultaneously. Switch between them from the sidebar without losing context.
- **Automatic Persistence** — Every message is saved to disk immediately after it is sent. Closing and reopening the app restores all conversations exactly as you left them.
- **Collapsible Sidebar** — Toggle the conversation history panel open or closed. Each session is listed with a delete button for cleanup.
- **Rich Text Rendering** — AI responses render markdown formatting including code blocks, inline code, bold text, and headings — making technical answers readable.
- **Progressive Web App (PWA)** — Includes a web app manifest so the interface can be installed as a desktop or mobile app directly from the browser.
- **Auto-Dismissing Error Alerts** — Network and server errors surface as non-blocking banners that clear automatically after 3 seconds.

---

## Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    Browser  (localhost:3000)                   │
│                                                                │
│  ┌─────────────┐  ┌───────────┐  ┌──────────┐  ┌────────────┐  │
│  │   App.js    │  │ ChatBox   │  │ Sidebar  │  │ ErrorAlert │  │
│  │ (State &    │  │ (Messages │  │ (Session │  │ (Banners)  │  │
│  │ Handlers)   │  │ & Input)  │  │  List)   │  │            │  │
│  └──────┬──────┘  └───────────┘  └──────────┘  └────────────┘  │
│         │                                                      │
│         │  apiService.js (Fetch API)                           │
└─────────┼──────────────────────────────────────────────────────┘
          │ HTTP
          ▼
┌─────────────────────────────────┐
│     Flask REST API (port 5000)  │
│                                 │
│  POST /api/chat                 │
│  GET  /api/conversations        │
│  POST /api/conversations        │
│  DELETE /api/conversations?id=  │
│                                 │
│  conversations.json  (storage)  │
└──────────────┬──────────────────┘
               │ HTTP POST
               ▼
┌──────────────────────────────────────┐
│   Ollama  (localhost:11434)          │
│   Llama 3.1:8b — local inference     │
└──────────────────────────────────────┘
```

### Data Flow

1. User types a message and submits.
2. React sends `POST /api/chat` to Flask.
3. Flask forwards the prompt to Ollama's local REST endpoint.
4. Ollama runs inference with Llama 3.1:8b and returns the response.
5. Flask passes the response back to React.
6. React appends the exchange to the active conversation, renders it, then immediately fires `POST /api/conversations` to persist the updated session.

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend Framework | React 18.3 | Component-based UI and state management |
| UI Library | Bootstrap 5.3 + React-Bootstrap | Responsive layout and icons |
| Backend Framework | Flask 2.x (Python) | REST API and file I/O |
| CORS Handling | flask-cors | Cross-origin requests from the React dev server |
| LLM Runtime | Ollama | Local inference engine |
| LLM Model | Llama 3.1:8b | Conversational language model |
| Persistence | JSON file (conversations.json) | Session storage without a database dependency |
| HTTP Client (Frontend) | Fetch API (native) | API calls from React |
| HTTP Client (Backend) | requests (Python) | Calls to Ollama |

---

## Prerequisites

- **Python 3.10+** with `pip`
- **Node.js 18+** with `npm`
- **Ollama** installed — [download here](https://ollama.com/download)
- **Llama 3.1:8b** model pulled locally

Pull the model before running the app:

```bash
ollama pull llama3.1:8b
```

---

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/alinjfz/local-llm-chat-persistent-sessions.git
cd local-llm-chat-persistent-sessions
```

### 2. Set up the Python backend

```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Install the React frontend dependencies

```bash
cd ../frontend
npm install
```

### 4. Start everything

From the project root, the launcher script starts Ollama, the Flask server, and the React dev server together:

```bash
cd ..
python run.py
```

Or start each service manually in separate terminals:

```bash
# Terminal 1 — LLM service
ollama serve

# Terminal 2 — Flask backend
cd backend && source venv/bin/activate && python server.py

# Terminal 3 — React frontend
cd frontend && npm start
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## API Reference

All endpoints are prefixed with `/api`.

| Method | Endpoint | Description | Request Body |
|---|---|---|---|
| `POST` | `/api/chat` | Send a message, receive an LLM response | `{ "message": "string" }` |
| `GET` | `/api/conversations` | Retrieve all saved conversations | — |
| `POST` | `/api/conversations` | Save or update a conversation | `{ "id": number, "conversation": [...] }` |
| `DELETE` | `/api/conversations?id={id}` | Delete a conversation by ID | — |

### Conversation Schema

```json
{
  "id": 1723120865256,
  "conversation": [
    {
      "message": "What is a transformer model?",
      "response": "A transformer is a deep learning architecture..."
    }
  ]
}
```

---

## Project Structure

```
local-llm-chat-persistent-sessions/
├── run.py                          # Launcher: starts Ollama + Flask + React
├── README.md
├── LICENSE
│
├── backend/                       # Flask REST API
│   ├── server.py                   # Entry point — creates and runs the Flask app
│   ├── requirements.txt
│   └── app/
│       ├── __init__.py             # App factory, blueprint registration, CORS config
│       ├── routes.py               # All API endpoints and Ollama integration
│       ├── models.py               # Reserved for future database models
│       └── static/
│           └── conversations.json  # Persistent conversation store
│
├── frontend/                      # React frontend
│   ├── package.json
│   ├── public/
│   │   ├── index.html
│   │   └── manifest.json           # PWA manifest
│   └── src/
│       ├── App.js                  # Root component — state, handlers, layout
│       ├── App.css                 # Layout styles (sidebar + chat section)
│       ├── services/
│       │   └── apiService.js       # All HTTP calls to the Flask backend
│       └── components/
│           ├── ChatBox.js          # Message list, markdown renderer, input field
│           ├── ChatBox.css         # Message bubble styles and animations
│           ├── Sidebar.js          # Collapsible conversation history panel
│           └── ErrorAlert.js       # Auto-dismissing error notification
│
└── test/
    └── main.py                     # Standalone CLI script for testing Ollama directly
```

---

## Configuration

The application runs on fixed localhost ports by default:

| Service | Default Port | Configured In |
|---|---|---|
| React dev server | 3000 | `frontend/package.json` |
| Flask API | 5000 | `backend/server.py` |
| Ollama | 11434 | `backend/app/routes.py` |

To swap to a different model, update the `"model"` field and `EXTERNAL_API_URL` in [backend/app/routes.py](backend/app/routes.py):

```python
EXTERNAL_API_URL = "http://0.0.0.0:11434/api/chat"

data = {
    "model": "llama3.2:3b",   # any model you have pulled via ollama pull
    ...
}
```

---

## Troubleshooting

**Ollama not responding**
```bash
# Verify Ollama is running
curl http://localhost:11434/api/tags

# Start it manually if not
ollama serve
```

**CORS errors in the browser console**
Make sure Flask is running on port 5000. The CORS config in `app/__init__.py` allows `localhost:3000` and `localhost:3001` only.

**"Failed to load conversations" on first launch**
`conversations.json` is created automatically after the first message is saved. This error on a fresh install resolves itself once you send your first message.

**502 / connection refused errors**
Ollama must be running before Flask is started. Start services in this order: Ollama → Flask → React.

---

## Roadmap

This is **version 1** — a clean, working foundation focused on local LLM chat and session persistence. The next version expands it into a full memory-augmented AI assistant with:

- **Episodic memory via PostgreSQL** — Structured storage of conversation summaries accessible across all sessions
- **Semantic memory via ChromaDB** — Facts and user preferences extracted from conversations, retrieved by vector similarity search
- **Memory injection pipeline** — Relevant past memories automatically injected into each new prompt for contextual continuity
- **Memory management UI** — Browse, search, and delete stored memories per session
- **Importance scoring and memory pruning** — Automatic cleanup of low-value memories to keep context relevant
- **FastAPI backend migration** — Async endpoints and automatic Swagger documentation
- **Docker Compose** — One-command full-stack deployment

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
