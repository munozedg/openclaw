# SensAI Projects - Developer Onboarding Guide

> Version 2.5.21 | Last Updated: January 2026

Welcome to SensAI Projects! This guide will help you understand the architecture, set up your development environment, and start contributing.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Tech Stack](#tech-stack)
4. [Repository Structure](#repository-structure)
5. [Development Setup](#development-setup)
6. [Multi-Agent Architecture](#multi-agent-architecture)
7. [Hyper-Personal Knowledge System](#hyper-personal-knowledge-system)
8. [Database Schemas](#database-schemas)
9. [Key Patterns & Conventions](#key-patterns--conventions)
10. [API Reference](#api-reference)
11. [Contribution Guidelines](#contribution-guidelines)

---

## Project Overview

SensAI Projects is an AI-powered Progressive Web Application (PWA) for project and task management. It features:

- **Conversational AI Interface** - Chat with AI using multiple models (GPT-4, Gemini, Claude)
- **Multi-Agent Architecture** - Supervisor/Executor/Validator pattern with LangGraph
- **HTML5 Canvas Visualizations** - Kanban, Timeline, Network, Calendar views
- **Hyper-Personal AI** - Learns your work patterns, skills, and project goals
- **Offline Capabilities** - PWA with Service Worker caching
- **Multi-Profile Support** - Connect multiple project accounts per user

### Critical Branding Rule

**NEVER use "Redmine" in user-facing UI or messages.** Use "SensAI API", "project system", or refer to projects/tasks directly.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (Browser)                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                        Vanilla JS SPA (PWA)                              │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │ │
│  │  │  Chat UI │  │  Canvas  │  │ Settings │  │  Voice   │  │  Service  │  │ │
│  │  │          │  │  Views   │  │  Panel   │  │  I/O     │  │  Worker   │  │ │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └───────────┘  │ │
│  └───────┼─────────────┼────────────┼─────────────┼────────────────────────┘ │
└──────────┼─────────────┼────────────┼─────────────┼──────────────────────────┘
           │             │            │             │
           ▼             ▼            ▼             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Node.js Backend (Replit)                             │
│                              Express.js :5000                                │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                 │
│  │   AI Agents    │  │   GraphQL API  │  │   Auth/OAuth   │                 │
│  │  (LangGraph)   │  │  (Apollo)      │  │  (Firebase)    │                 │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                 │
│          │                   │                   │                          │
│  ┌───────┴───────────────────┴───────────────────┴───────┐                  │
│  │              Shared Services & Tools                   │                  │
│  │  • Entity Resolver  • Token Tracker  • TTS (Sherpa)   │                  │
│  │  • Conversation Memory  • pg-boss Job Queue           │                  │
│  └───────────────────────────┬───────────────────────────┘                  │
└──────────────────────────────┼──────────────────────────────────────────────┘
                               │
           ┌───────────────────┼───────────────────┐
           ▼                   ▼                   ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  PostgreSQL      │  │  FastAPI Server  │  │  External APIs   │
│  (Neon)          │  │  (Digital Ocean) │  │                  │
│  ┌────────────┐  │  │  :8000           │  │  • OpenAI        │
│  │ pgvector   │  │  │  ┌────────────┐  │  │  • Gemini        │
│  │ embeddings │  │  │  │ Knowledge  │  │  │  • Claude        │
│  └────────────┘  │  │  │ System     │  │  │  • Mistral       │
│  ┌────────────┐  │  │  └────────────┘  │  │  • SensAI API    │
│  │ User Data  │  │  │  ┌────────────┐  │  └──────────────────┘
│  │ Sessions   │  │  │  │ OR-Tools   │  │
│  │ Profiles   │  │  │  │ Scheduler  │  │
│  └────────────┘  │  │  └────────────┘  │
└──────────────────┘  └──────────────────┘
```

---

## Tech Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| Vanilla JavaScript | SPA without framework overhead |
| HTML5 Canvas | High-performance visualizations |
| Web Speech API | Voice input and TTS |
| Service Worker | Offline support, caching |
| Firebase SDK | OAuth authentication |

### Node.js Backend (Replit)
| Technology | Purpose |
|------------|---------|
| Express.js | HTTP server, API routes |
| Apollo Server | GraphQL API |
| LangGraph | Multi-agent orchestration |
| pg-boss | Background job queue |
| Sherpa-ONNX | Offline text-to-speech |

### FastAPI Server (Digital Ocean)
| Technology | Purpose |
|------------|---------|
| FastAPI | Async Python API |
| SQLAlchemy | ORM with async support |
| pgvector | Vector similarity search |
| OR-Tools | Sprint scheduling optimization |
| Strawberry | GraphQL (Python) |

### Databases
| Database | Purpose |
|----------|---------|
| PostgreSQL (Neon) | Primary data store |
| pgvector extension | Embedding storage & similarity search |
| ChromaDB | Alternative vector store (FastAPI) |

### AI Models
| Provider | Models |
|----------|--------|
| OpenAI | GPT-4o, GPT-4o-mini, GPT-4 Turbo |
| Google | Gemini 2.0 Flash, Gemini 1.5 Pro |
| Anthropic | Claude Sonnet 4.5, Claude Opus 4.5 |
| Mistral | Mistral Large, Codestral |
| Local | Llama 3.2, Phi-3 (via Ollama) |

---

## Repository Structure

```
sensai-projects/
├── server.js                 # Main Express.js application (~6000 lines)
├── package.json              # Node.js dependencies
├── public/                   # Frontend assets (served statically)
│   ├── app.js               # Main SPA application (~400KB)
│   ├── styles.css           # Global styles (~140KB)
│   ├── index.html           # Entry point
│   ├── service-worker.js    # PWA offline support
│   ├── manifest.json        # PWA manifest
│   └── components/          # Reusable UI components
│
├── fastapi_server/           # Python backend (separate deployment)
│   ├── main.py              # FastAPI entry point
│   ├── app/
│   │   ├── models.py        # Pydantic models
│   │   ├── knowledge_models.py  # Knowledge system tables
│   │   ├── database.py      # SQLAlchemy setup
│   │   ├── services/        # Business logic
│   │   │   ├── knowledge_embedder.py
│   │   │   ├── personal_context_retriever.py
│   │   │   ├── task_outcome_capture.py
│   │   │   ├── team_insight_engine.py
│   │   │   ├── issue_service.py
│   │   │   ├── project_service.py
│   │   │   └── user_service.py
│   │   ├── routers/
│   │   │   └── knowledge.py  # Knowledge API endpoints
│   │   └── graphql/         # Strawberry GraphQL
│   └── skills/              # Agent skill definitions
│
├── graphql/                  # Node.js GraphQL layer
│   ├── schema.js            # Type definitions
│   ├── resolvers/           # Query/mutation resolvers
│   └── dataloaders/         # N+1 query optimization
│
├── skills/                   # AI agent skills (Node.js)
│   ├── en/                  # English skill prompts
│   └── es/                  # Spanish skill prompts
│
├── prompts/                  # System prompts for AI agents
├── auth/                     # Firebase authentication
├── models/                   # Sherpa-ONNX TTS models (~100MB)
│   └── tts/
│       ├── vits-piper-en_US-lessac-medium/
│       └── vits-piper-es_ES-davefx-medium/
│
├── scripts/                  # Utility scripts
├── test/                     # Test files
├── replit.md                 # Project state & preferences
└── DEVELOPER_GUIDE.md        # This file
```

### Key Files

| File | Purpose |
|------|---------|
| `server.js` | Main backend - AI agents, API routes, WebSocket, SSE |
| `public/app.js` | Entire frontend SPA |
| `public/styles.css` | All CSS (Shadcn-inspired dark theme) |
| `fastapi_server/main.py` | FastAPI entry point |
| `replit.md` | Project documentation (keep updated!) |

---

## Development Setup

### Prerequisites

- Node.js 20+
- Python 3.11+ (for FastAPI development)
- PostgreSQL access (provided by Replit/Neon)
- API keys for AI providers

### Environment Variables

#### Required Secrets (set via Replit Secrets panel)

```bash
# AI Providers
OPENAI_API_KEY=sk-...
GOOGLE_AI_API_KEY=...
ANTHROPIC_API_KEY=sk-ant-...

# Database
DATABASE_URL=postgresql://...

# Firebase Auth
FIREBASE_API_KEY=...
FIREBASE_PROJECT_ID=...
FIREBASE_SERVICE_ACCOUNT={"type":"service_account",...}

# SensAI Project API (admin access for multi-user operations)
SENSAI_ADMIN_API_KEY=...
```

#### Environment Variables (non-secret)

```bash
SENSAI_API_URL=https://api.sensai.plus      # SensAI Projects API base URL
FASTAPI_URL=https://api.sensai.plus         # FastAPI server (same as above, aliased)
CALENDAR_RECALCULATE_ENABLED=true
NODE_ENV=development
```

### Running Locally on Replit

1. **Start the Server**
   The workflow `Start Server` runs:
   ```bash
   LD_LIBRARY_PATH=$PWD/node_modules/sherpa-onnx-linux-x64:$LD_LIBRARY_PATH node server.js
   ```

2. **Access the App**
   - Open the Webview pane or use the provided URL
   - The app runs on port 5000

### Running FastAPI Server (Local Development)

```bash
cd fastapi_server
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

---

## Multi-Agent Architecture

SensAI uses a **Plan-and-Execute** pattern with three specialized agents:

```
┌──────────────────────────────────────────────────────────────┐
│                     LangGraph Orchestrator                    │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │  SUPERVISOR  │───▶│   EXECUTOR   │───▶│  VALIDATOR   │   │
│  │              │    │              │    │              │   │
│  │ • Analyzes   │    │ • Runs tools │    │ • Fact-check │   │
│  │   user query │    │ • API calls  │    │ • Confidence │   │
│  │ • Creates    │    │ • Data fetch │    │   scoring    │   │
│  │   plan steps │    │              │    │ • Corrections│   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│        │                    │                    │           │
│        ▼                    ▼                    ▼           │
│  claude-sonnet-4.5    gpt-4o-mini         gemini-2.5-flash  │
└──────────────────────────────────────────────────────────────┘
```

### Agent Roles

| Agent | Model | Responsibility |
|-------|-------|----------------|
| **Supervisor** | Claude Sonnet 4.5 | Analyzes queries, creates execution plans |
| **Executor** | GPT-4o-mini | Executes plan steps via function calling |
| **Validator** | Gemini 2.5 Flash | Fact-checks responses, assigns confidence |

### Skills System

Skills are modular capabilities injected into agents. Located in:

| Location | Purpose |
|----------|---------|
| `fastapi_server/skills/` | FastAPI agent skills (9 specialized workflows) |
| `.sensai/skills/` | SKILL.md templates with frontmatter metadata |
| `skills/en/` | Node.js English prompt skills |
| `skills/es/` | Node.js Spanish prompt skills |

**FastAPI Skills** (`fastapi_server/skills/`):
- `c_level-portfolio-review/` - Executive portfolio analysis
- `data-analysis/` - Data processing and insights
- `ic-focus-today/` - Individual contributor daily focus
- `literature-review/` - Academic literature search
- `manager-daily-replan/` - Manager planning workflows
- `portfolio-analysis/` - Project portfolio metrics
- `progress-report/` - Status report generation
- `scientific-review/` - Research paper analysis
- `scientific-writing/` - Academic writing assistance

**SKILL.md Format** (`.sensai/skills/`):
```yaml
---
id: project-management
name: Project Management
description: Create, update, query projects and tasks
triggers:
  - create project
  - list tasks
  - update issue
tools:
  - createProject
  - listIssues
  - updateIssue
---

## Instructions
When the user asks about projects or tasks...
```

**Skill Router Flow:**
1. User message → LLM Router (GPT-4o-mini or Gemini Flash)
2. Router selects relevant skills from registry
3. Selected skills injected into Executor context
4. Tools become available for function calling

---

## Hyper-Personal Knowledge System

The Knowledge System enables the AI to learn and remember user-specific context.

### Architecture

```
User Actions                    Knowledge Services
    │                                │
    ├── Complete Task ──────────▶ TaskOutcomeCapture
    │                              • Extract skills used
    │                              • Log success/failure
    │                              • Generate embeddings
    │
    ├── Set Project Goal ───────▶ KnowledgeEmbedder
    │                              • Embed goal text
    │                              • Store with metadata
    │
    ├── Chat with AI ───────────▶ PersonalContextRetriever
    │                              • Vector similarity search
    │                              • Fetch relevant history
    │                              • Inject into prompt
    │
    └── View Recommendations ───▶ TeamInsightEngine
                                   • Skill distribution analysis
                                   • Best assignee suggestions
                                   • Performance trends
```

### Database Tables

All tables use **pgvector** with 1536-dimensional OpenAI embeddings:

| Table | Purpose |
|-------|---------|
| `user_knowledge` | Personal facts, preferences, learned patterns |
| `project_goals` | Strategic objectives with embeddings |
| `task_outcomes` | Task completion history with skills used |
| `skill_profiles` | User skill levels based on outcomes |
| `team_insights` | Team-level patterns and recommendations |
| `conversation_context` | Recent conversation summaries |

### API Endpoints

```
POST /knowledge/context          # Get personalized context for AI
POST /knowledge/embed            # Store new knowledge
POST /knowledge/goal             # Set project goal
POST /knowledge/task-outcome     # Log task completion
GET  /knowledge/team-insights    # Get team recommendations
GET  /knowledge/goal-alignment   # Score task-goal alignment
```

---

## Database Schemas

**Canonical source:** `deployment/schemas/database_schema.sql` (779 lines)

The schema below is a summary. **Always refer to the canonical SQL file for authoritative definitions.**

Key tables:

### Core User Tables

```sql
users (
    id TEXT PRIMARY KEY,
    email TEXT,
    full_name VARCHAR(255),
    sensai_api_key TEXT,
    sensai_user_id TEXT,
    user_logins TEXT[],
    user_ids INTEGER[],
    user_profiles JSONB
)

tenants (id SERIAL PRIMARY KEY, name, domain, settings JSONB, is_active)

redmine_profiles (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) REFERENCES users(id),
    profile_name, redmine_url, redmine_login,
    redmine_user_id INTEGER,
    api_key_encrypted TEXT,
    is_active BOOLEAN DEFAULT false,
    timezone VARCHAR(255) DEFAULT 'America/Bogota'
)

cached_redmine_users (
    id SERIAL PRIMARY KEY,
    redmine_id INTEGER UNIQUE,
    login, firstname, lastname, email,
    embedding vector(1536),  -- pgvector for semantic search
    department_id, manager_id, fte_capacity
)
```

### Project Management Tables

```sql
projects (
    id SERIAL PRIMARY KEY,
    redmine_id INTEGER UNIQUE,
    identifier, name, description,
    parent_id INTEGER,
    embedding vector(1536),
    portfolio_priority TEXT,
    strategic_objective_id INTEGER,
    health_status TEXT DEFAULT 'green'
)

project_journals (id, project_id, section, content, updated_at)
```

### Knowledge System (pgvector - 1536 dimensions)

```sql
user_knowledge (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    knowledge_type VARCHAR(50),  -- 'fact', 'preference', 'pattern', 'lesson'
    title VARCHAR(255),
    content TEXT,
    embedding vector(1536),
    extra_data JSONB,  -- Note: 'extra_data' not 'metadata' (SQLAlchemy reserved)
    tenant_id INTEGER
)

project_goals (
    id SERIAL PRIMARY KEY,
    project_id INTEGER NOT NULL,
    goal_text TEXT,
    priority VARCHAR(20),
    target_date DATE,
    embedding vector(1536)
)

task_outcomes (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    task_id INTEGER,
    project_id INTEGER,
    outcome VARCHAR(20),  -- 'success', 'partial', 'failure', 'cancelled'
    duration_hours FLOAT,
    skills_used TEXT[],
    lessons_learned TEXT,
    embedding vector(1536)
)

skill_profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    skill_name VARCHAR(100),
    proficiency_score FLOAT,  -- 0.0 to 1.0
    task_count INTEGER,
    avg_completion_time FLOAT,
    embedding vector(1536)
)

team_insights (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER,
    project_id INTEGER,
    insight_type VARCHAR(50),
    title VARCHAR(255),
    description TEXT,
    confidence_score FLOAT,
    embedding vector(1536)
)

conversation_context (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    session_id VARCHAR(100),
    summary TEXT,
    key_topics TEXT[],
    embedding vector(1536)
)
```

### Column Naming Convention

**Important:** SQLAlchemy reserves `metadata` as a class attribute. Our tables use `extra_data` for JSON metadata columns.

---

## Key Patterns & Conventions

### Code Style

1. **No comments in code** unless explicitly requested
2. **Vanilla JS** - No React, Vue, or framework dependencies
3. **Async/await** - All I/O operations are async
4. **Error handling** - Wrap external calls in try/catch
5. **HSL colors** - Use CSS variables for theming

### CSS Variables (Dark Theme)

```css
--background: 240 10% 3.9%;
--foreground: 0 0% 98%;
--primary: 221 83% 53%;
--secondary: 240 3.7% 15.9%;
--accent: 240 3.7% 15.9%;
--destructive: 0 62% 30%;
--muted: 240 3.7% 15.9%;
```

### Version Bumping

**Always update version in 3 files after changes:**

1. `package.json` → `"version": "2.5.21"`
2. `public/manifest.json` → `"version": "2.5.21"`
3. `public/service-worker.js` → `CACHE_NAME = 'sensai-projects-v2.5.21'` and `APP_VERSION = '2.5.21'`

### API Response Format

```javascript
// Success
{ success: true, data: {...} }

// Error
{ success: false, error: "Error message" }
```

### AI Context Limits

- Max context: 120,000 tokens
- Auto-truncation of message history (last 10 messages)
- Semantic memory uses vector similarity (top 5 results)

---

## API Reference

### REST Endpoints (Node.js)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/chat` | POST | Standard AI chat |
| `/api/chat/stream` | POST | SSE streaming chat |
| `/api/projects` | GET | List projects |
| `/api/issues` | GET/POST | Task CRUD |
| `/api/redmine-profiles` | GET/POST | Profile management |
| `/api/tts` | POST | Text-to-speech |
| `/graphql` | POST | GraphQL queries |

### GraphQL Queries

```graphql
query GetProjects {
  projects {
    id
    name
    status
    issues { id subject priority }
  }
}

mutation CreateIssue($input: IssueInput!) {
  createIssue(input: $input) {
    id
    subject
  }
}
```

### FastAPI Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/schedule` | POST | OR-Tools sprint scheduling |
| `/metrics/portfolio` | GET | Portfolio analytics |
| `/enhance/task` | POST | AI task enhancement |
| `/knowledge/*` | Various | Knowledge system |
| `/graphql` | POST | Strawberry GraphQL |

---

## Contribution Guidelines

### Getting Started

1. Read this guide completely
2. Set up your development environment
3. Review `replit.md` for current state
4. Check the issue tracker for tasks

### Making Changes

1. **Branch naming:** `feature/description` or `fix/description`
2. **Keep changes focused** - One feature/fix per PR
3. **Update version numbers** after each change
4. **Update `replit.md`** with significant changes

### Testing

```bash
# Run Node.js tests
npm test

# Run FastAPI tests
cd fastapi_server
pytest tests/
```

### Pull Request Checklist

- [ ] Version numbers updated (3 files)
- [ ] `replit.md` updated if needed
- [ ] No "Redmine" in user-facing text
- [ ] No hardcoded API keys or secrets
- [ ] Error handling for external calls
- [ ] Mobile-responsive if UI change

### Communication

- Keep code comments minimal
- Use clear variable/function names
- Document complex logic in `replit.md`

---

## Quick Reference

### Common Tasks

**Add a new AI tool:**
1. Define tool in `server.js` under `tools` array
2. Add handler in switch statement
3. Document in skill files if needed

**Add a new visualization:**
1. Create render function in `public/app.js`
2. Add canvas setup in view switcher
3. Register in view menu

**Add a new knowledge type:**
1. Update `knowledge_models.py` enum
2. Add embedding method in `knowledge_embedder.py`
3. Add retrieval in `personal_context_retriever.py`

### Debugging Tips

- Check browser console for frontend errors
- Check workflow logs for backend errors
- Use `/health` endpoint to verify server status
- Enable `DEBUG=true` for verbose logging

---

## Need Help?

- Review `replit.md` for project context
- Check existing code patterns before adding new ones
- Consult the skills files for AI behavior examples

Welcome to the team!
