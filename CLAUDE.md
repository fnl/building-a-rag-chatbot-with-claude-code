# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start the application:**
```bash
./run.sh
```

**Manual startup:**
```bash
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Install dependencies:**
```bash
uv sync
```

**Environment setup:**
Create `.env` file with:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture

This is a RAG (Retrieval-Augmented Generation) system for course materials. The system processes structured course documents and enables semantic search with AI-powered responses.

### Core Data Flow
`DocumentProcessor` → `VectorStore` (ChromaDB) → `SearchTools` → `AIGenerator` (Claude) → Response

### Key Components

**RAGSystem** (`rag_system.py`) - Main orchestrator that coordinates all components

**DocumentProcessor** (`document_processor.py`) - Handles structured course document parsing:
- Expects format: Course metadata (title/link/instructor) + numbered lessons
- Intelligent sentence-based chunking with configurable overlap
- Adds contextual prefixes: "Course [title] Lesson [X] content: [chunk]"

**VectorStore** (`vector_store.py`) - ChromaDB wrapper:
- Separates course metadata from content chunks
- Maintains Course → Lesson → Chunk relationships
- Uses `all-MiniLM-L6-v2` embeddings

**SearchTools** (`search_tools.py`) - Extensible tool system with `ToolManager` and `CourseSearchTool`

**AIGenerator** (`ai_generator.py`) - Claude API integration with conversation context

**SessionManager** (`session_manager.py`) - Tracks conversation history per session

### Configuration
All settings centralized in `config.py`:
- Chunk size: 800 characters
- Chunk overlap: 100 characters  
- Max search results: 5
- Conversation history: 2 messages
- Claude model: `claude-sonnet-4-20250514`

### API Endpoints
- `POST /api/query` - Main RAG query endpoint
- `GET /api/courses` - Course statistics
- Static frontend served at root

### Document Format
Course documents must follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [optional]
[lesson content...]

Lesson 1: [title]
[content...]
```

### Data Models
- `Course` - title, instructor, lessons, course_link
- `Lesson` - lesson_number, title, lesson_link
- `CourseChunk` - content, course_title, lesson_number, chunk_index

The system auto-loads documents from `/docs` on startup and persists vector data in `./chroma_db`.