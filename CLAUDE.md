# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Retrieval-Augmented Generation (RAG) system for querying course materials. It uses ChromaDB for vector storage, Anthropic's Claude for AI generation, and provides a web interface for interaction.

## Essential Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Dependencies Management
```bash
# Install/update dependencies (uses uv package manager)
uv sync

# Add new dependency
uv add package_name
```

### Environment Setup
Create `.env` file in project root:
```
ANTHROPIC_API_KEY=your_api_key_here
```

## Architecture & Key Concepts

### Data Flow Pipeline
The system processes user queries through a sophisticated pipeline:

1. **Frontend** (`frontend/script.js`) → POST to `/api/query`
2. **API Layer** (`backend/app.py`) → FastAPI endpoint handling
3. **RAG Orchestration** (`backend/rag_system.py`) → Coordinates all components
4. **Tool-Based Search** (`backend/search_tools.py`) → AI decides when to search
5. **Vector Search** (`backend/vector_store.py`) → ChromaDB semantic search
6. **AI Generation** (`backend/ai_generator.py`) → Claude synthesizes response

### Document Processing Strategy
- **Chunking**: 800 chars with 100 char overlap, sentence-boundary aware (`backend/document_processor.py`)
- **Embeddings**: Local `all-MiniLM-L6-v2` model (384-dim vectors)
- **Storage**: ChromaDB with two collections: `course_catalog` and `course_content`
- **Context Enrichment**: Each chunk includes course title and lesson number

### Tool Architecture
The system uses Anthropic's tool-calling feature where Claude decides whether to search:
- `CourseSearchTool` implements the `Tool` interface
- `ToolManager` orchestrates tool execution
- Sources are tracked separately for attribution

### Session Management
- Conversation history maintained per session
- Configurable history length (default: 2 exchanges)
- Session IDs generated server-side

### Key Configuration (`backend/config.py`)
- `CHUNK_SIZE`: 800 (characters per chunk)
- `CHUNK_OVERLAP`: 100 (overlap between chunks)
- `MAX_RESULTS`: 5 (search results returned)
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514

## Important Implementation Details

### Vector Store Initialization
On startup, `app.py:88-98` loads documents from `docs/` folder. The system checks for existing courses to avoid re-processing.

### Search Resolution
`vector_store.py:79-83` performs fuzzy course name matching, allowing partial matches like "MCP" to find "Model Context Protocol" courses.

### AI System Prompt
Located in `ai_generator.py:8-30`, instructs Claude to:
- Use search tool only for course-specific questions
- One search maximum per query
- Provide concise, educational responses
- No meta-commentary about search process

### Frontend Markdown Support
Uses `marked.js` for rendering AI responses with proper formatting.

## Common Development Tasks

### Adding New Course Documents
1. Place `.txt` files in `docs/` folder
2. Format: First 3 lines must be Course Title, Course Link, Course Instructor
3. Lessons marked with pattern: `Lesson X: Title`
4. Restart server to auto-load

### Modifying Chunk Size
Update `backend/config.py`:
- `CHUNK_SIZE` and `CHUNK_OVERLAP`
- Clear ChromaDB: `rm -rf backend/chroma_db/`
- Restart to rebuild embeddings

### Debugging Search Results
- Check `backend/search_tools.py:88-114` for result formatting
- Sources tracked in `CourseSearchTool.last_sources`
- Search limit configurable via `MAX_RESULTS`

## Database Locations
- ChromaDB: `backend/chroma_db/`
- Course documents: `docs/`
- Frontend served from: `frontend/`

## API Endpoints
- `POST /api/query` - Process user queries
- `GET /api/courses` - Get course statistics
- `/` - Serve frontend (static files)
- `/docs` - FastAPI automatic documentation

## Testing Approach
Currently no test suite implemented. When adding tests:
- Use pytest for backend testing
- Mock Anthropic API calls in tests
- Test vector search with sample embeddings