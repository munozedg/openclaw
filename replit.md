# SensAI Projects

## Overview
SensAI Projects is an AI-powered Progressive Web Application (PWA) designed for comprehensive project and task management. It features a conversational AI interface utilizing multiple AI models to interact with project data. The application offers various HTML5 canvas-rendered visualizations, including Kanban, Timeline, Network, Charts, and Calendar, to enhance project management efficiency, boost productivity, and support informed decision-making. The project's vision is to revolutionize project management through advanced AI capabilities and intuitive user experiences, offering features like multi-user sprint optimization with semantic skill matching and AI-powered work journals.

## Developer Resources
For comprehensive onboarding, see **`DEVELOPER_GUIDE.md`** which includes architecture diagrams, setup instructions, database schemas, and contribution guidelines.

## User Preferences
Preferred communication style: Simple, everyday language.
**NEVER use the word "Redmine"** in user-facing UI or messages. Use "SensAI API", "project system", or just refer to projects/tasks directly. This is a core branding requirement.
Version management: **Always update the version number after each change** in these 3 files:
1. `package.json` - "version" field
2. `public/manifest.json` - "version" field
3. `public/service-worker.js` - CACHE_NAME constant (e.g., 'sensai-projects-v2.4.27') **AND** APP_VERSION constant

This ensures the PWA updates correctly in production.

## System Architecture

### Frontend
The frontend is a vanilla JavaScript Single Page Application (SPA) built as a PWA, utilizing Service Workers and a Web App Manifest for offline capabilities. It employs a `SensaiApp` class for managing core functionalities. The UI/UX is chat-first, responsive, and features HTML5 canvas visualizations that can operate as full-screen overlays or in side-by-side/stacked layouts. The design system is a Shadcn-inspired dark theme, using HSL-based CSS variables and Inter typography, optimized for performance with `requestAnimationFrame`-batched rendering and Service Worker asset caching. The layout is mobile-first with a collapsible sidebar and fixed input bar, supporting interactive action widgets.

### Backend
The backend is an Express.js application supporting session middleware for OAuth authentication and integration with multiple AI providers. It implements an AI agent architecture with function calling, a stateless request-response model, and Server-Sent Events (SSE) for streaming AI responses. Server-side caching and Gzip compression are also employed. A separate FastAPI server provides additional services like calendar scheduling, metrics, and a GraphQL API.

### Multi-Agent Architecture
The system utilizes a **Plan-and-Execute** pattern, orchestrated by LangGraph, involving a **Supervisor Agent** for plan generation, an **Executor Agent** for executing plan steps via tool calls, and a **Validator Agent** for fact-checking and confidence scoring. A Proactive Task Automation System uses `pg-boss` and LangGraph for background task processing.

### Key Architectural Decisions and Features
-   **Authentication**: Firebase OAuth for identity and one-time Redmine API key verification, supporting multi-tenancy and multi-account profiles.
-   **Multi-Lingual Support**: Full bilingual support (English/Spanish) for UI and AI responses.
-   **AI Interaction**: Web Speech API for voice input/TTS, hybrid semantic memory (`pgvector`), modular agent skills, AI task enhancement, contextual "Follow-Up Suggestions," and LLM-based entity extraction. Includes a `closeCollectiveSprint` tool for team-wide sprint optimization using multi-user OR-Tools with hybrid skill matching.
-   **Visualizations**: Multiple HTML5 canvas-rendered modes including Kanban, Timeline, Network, Charts, Calendar, Eisenhower Matrix, Pareto Analysis, and Statistics View.
-   **Calendar View**: Monthly/weekly toggles, color-coded events, ICS subscription, and sprint scheduling with OR-Tools CP-SAT Solver. Real-time Google Calendar sync is available.
-   **PWA Update System**: Automatic and seamless updates via Service Worker.
-   **Security**: User IDs use SHA-256 hashes, AES-256-CBC encryption for stored API keys.
-   **Context Management**: Automatic message truncation for LLM context limits and user context caching in PostgreSQL.
-   **Metrics System**: LangGraph tools for project and portfolio analytics.
-   **Entity Resolution**: Centralized `entity-resolver.js` with fuzzy matching, integrated with GraphQL.
-   **Priority Scoring System**: Custom Redmine fields and a FastAPI `ScoringService` calculate a composite "Priority Score."
-   **GraphQL API**: `/graphql` endpoint with Apollo Server (Node.js) and Strawberry GraphQL (FastAPI) providing types and mutations for project data.
-   **Shared Service Layer**: `IssueService`, `ProjectService`, `UserService` in FastAPI for consistent data transfer.
-   **LLM Cost Optimization**: Smart prompt routing, token budgets, context compression, response caching, lazy tool loading, and token tracking.
-   **Agent Skills Architecture**: Standardized skill definitions, `SkillLoader`, `SkillRouter` for intelligent selection, and role-based filtering.
-   **Reporting**: Interactive Weekly Report, quick prompts, and Executive Portfolio Overview.
-   **Work Journals**: Database table `project_journals` with sections (next_actions, recent_decisions, blockers, key_files, notes) and a CRUD API for project context recovery.
-   **Mentions System**: User-to-user and user-to-AI messaging with real-time notifications and queueing for offline users.
-   **Strategic Project Management Database Schema**: Includes tables for organizational structure (departments, strategic_objectives, project_objectives), enhanced project/task fields, task management (assignments, dependencies, status history, comments), milestones, risk management, a comprehensive skills system, user capacity/work patterns, stakeholder management, and document management with embeddings.

## External Dependencies

### AI Services
-   **OpenAI GPT Models**: GPT-4o, GPT-4o Mini, GPT-4 Turbo, GPT-3.5 Turbo
-   **Google Gemini Models**: Gemini 2.0 Flash, Gemini 1.5 Pro, Gemini 1.5 Flash
-   **Anthropic Claude Models**: Claude Sonnet 4.5, Claude Opus 4.5, Claude Haiku 4.5
-   **Mistral AI Models**: Mistral Large, Mistral Medium, Mistral Small, Codestral
-   **Local LLMs**: Llama 3.2, Llama 3.1, Mistral, Code Llama, Phi-3 (via Ollama or LM Studio)

### Project Management Integration
-   **SensAI Projects API**: `https://api.sensai.plus`
-   **Redmine 6**: Self-hosted instance for core project data.
-   **FastAPI Server**: Provides calendar scheduling, metrics, enhancement endpoints, and GraphQL API (deployed on Digital Ocean).
-   **FastMCP Server**: MCP (Model Context Protocol) server exposing tools for AI assistant integration.
-   **Academic Literature Search**: PubMed (NCBI E-utilities) and Google Scholar (via scholarly library).

### Databases
-   **PostgreSQL (Neon)**: The primary persistent data store for all project data, user accounts, session storage, tenant management, Redmine profiles, semantic memories, mentioned entities, user context caches, `pg-boss` job queue, calendar_chunks, sprint_logs, activities, time_entries, task_dependencies, project_milestones, task_history, task_analysis, and the strategic project management schema tables.
-   **ChromaDB (FastAPI)**: Used for vector embeddings for semantic search, persisted locally on the FastAPI server.

## Recent Changes

### v2.5.21 (January 2026)
- **Hyper-Personal AI Knowledge System**: Major feature for personalized AI assistance:
  - New PostgreSQL tables with pgvector: `user_knowledge`, `project_goals`, `task_outcomes`, `skill_profiles`, `team_insights`, `conversation_context`
  - `KnowledgeEmbedder` service generates and stores OpenAI embeddings (1536 dimensions)
  - `TaskOutcomeCapture` automatically logs task completions with skill extraction and performance metrics
  - `PersonalContextRetriever` fetches relevant knowledge during AI conversations using vector similarity
  - `TeamInsightEngine` analyzes skill distribution, finds best assignees, tracks performance trends
  - Goal alignment scoring measures how well tasks align with project objectives
  - REST API endpoints at `/knowledge/*` for all knowledge operations
  - Integrated into chat handlers to inject personal context into AI responses
  - FastAPI server includes all new knowledge models and services

### v2.5.20 (January 2026)
- **LLM-Based Skill Router**: Upgraded dispatcher to use semantic LLM routing:
  - `SkillsRegistry` reads SKILL.md metadata and builds routing prompts
  - `LLMRouter` sends task + skill descriptions to fast LLM (GPT-4o-mini/Gemini Flash)
  - Supports OpenAI, Gemini, and Anthropic models for routing
  - Automatic fallback to keyword matching when LLM unavailable
  - Async `identify_skills_async()` for non-blocking routing

### v2.5.19 (January 2026)
- **Skills Engine for FastAPI**: Added modular agent system with decoupled skills:
  - Created `.sensai/skills/` directory with SKILL.md template format
  - Implemented `SkillLoader` class using python-frontmatter for skill parsing
  - Implemented `Dispatcher` with keyword/pattern matching for task-to-skill mapping
  - Added LangGraph integration with `hydrate_agent` node for dynamic skill injection
  - Sample skills: project-management and code-analysis
  - Test suite in `fastapi_server/tests/test_skills.py`

### v2.5.18 (January 2026)
- **Multi-Profile Support**: Advanced users can now activate multiple profiles simultaneously:
  - Added `is_primary` flag to distinguish the main account (linked to registration email)
  - Primary profile cannot be deactivated and cannot be deleted
  - Toggle endpoint `/api/redmine-profiles/:id/toggle` for multi-select activation
  - Frontend checkboxes sync with server state (is_active)
  - Data from all active profiles is combined in AI context and visualizations
  - Basic users: 1 profile, Advanced users: up to 3 profiles
  - "Primary" badge displayed on the main profile

### v2.5.17 (January 2026)
- **Single Logo Layout**: Removed duplicate logo from sidebar, keeping only the topbar brand for a cleaner interface.
- **Wider Desktop Margins**: Increased chat padding on large screens (6rem at 1200px+, 12rem at 1600px+, 16rem at 1920px+).

### v2.5.16 (January 2026)
- **Centered Chat Layout**: Modernized chat interface with centered messages (max-width 800px) and responsive side margins on wide screens, similar to ChatGPT and Claude.

### v2.5.15 (January 2026)
- **Offline TTS with Sherpa-ONNX**: Added free, unlimited offline text-to-speech capability:
  - Integrated sherpa-onnx native Node.js addon with Piper VITS models
  - English (vits-piper-en_US-lessac-medium) and Spanish (vits-piper-es_ES-davefx-medium) voices
  - 22050Hz WAV output with adjustable speech speed
  - UI toggle in Settings → Voice Configuration → TTS Premium → Provider: "Sherpa (Offline)"
  - Added Licenses & Acknowledgments section in Settings for open-source attribution
  - Zero API costs, works completely offline, privacy-focused operation
  - Models stored in `./models/tts/` (~100MB total)

### v2.5.13 (January 2026)
- **Single-Account Hierarchical Projects**: Major architecture pivot from multi-profile to single-account with hierarchical projects:
  - Replaced multi-profile system with single Redmine account per user
  - Added subscription tier system (Free=1, Pro=3, Enterprise=5 root projects)
  - Created `/api/root-projects` and `/api/project-hierarchy` endpoints
  - Added `project_hierarchy` PostgreSQL table for fast hierarchy caching
  - Replaced profile selector UI with project hierarchy selector
  - Foundation for future subscription billing tiers

### v2.5.12 (January 2026)
- **Complete Demo Removal**: Removed ALL remaining demo mode references. The app now treats all profiles equally without any special demo handling.