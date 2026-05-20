# AGENTS.md

## Project
Personal Memory Agent

GitHub Repository:
- https://github.com/leehyeoksu/personal-memory-agent

## Goal
Build a personal AI memory agent that stores goals, todos, diaries, reflections, interest links/images, chat logs, memory chunks, and state scores.

This project is not just a diary app or a todo app.
It is a personal memory system that connects:
1. goals / todos
2. diaries / reflections
3. interest materials such as links, images, screenshots, and notes

The agent should later answer questions based on the user's past context.

## Stack
- Backend: FastAPI
- Main DB: MongoDB
- Vector DB: ChromaDB first, Qdrant later
- LLM: Mock LLM first, then Hugging Face Transformers or llama.cpp/GGUF with Qwen/Gemma
- Frontend: React or Next.js later
- Development Environment: WSL Ubuntu
- Package Environment: Python venv

## Agent Roles
- Gemini: research agent
- Claude: planner and reviewer
- Codex: implementation agent

## Python Environment Rules
- This project may start as an empty folder.
- If no virtual environment exists, create one at `.venv`.
- Use Python venv, not global pip installation.
- Install backend dependencies inside `.venv`.
- Add dependency list to `requirements.txt`.
- Document activation and run commands in README.
- For WSL/Linux, use:
  - `python3 -m venv .venv`
  - `source .venv/bin/activate`
  - `pip install -r requirements.txt`
  - `uvicorn app.main:app --reload`

## MVP Direction
Build the smallest useful backend first.

MVP should include:
- FastAPI project structure
- goal APIs
- todo APIs
- diary/reflection APIs
- interest item APIs for links/images/notes
- mock memory search
- mock LLM response
- mock state score calculation
- MongoDB connection structure
- in-memory fallback when MongoDB is unavailable
- README run instructions

Do not implement yet:
- real local LLM connection
- real embedding model
- real ChromaDB/Qdrant connection
- image analysis/OCR
- link crawling/body extraction
- complex frontend
- authentication
- calendar sync
- external web search

## Model Strategy
- Do not hardcode a specific LLM in the first MVP.
- Start with MockLLMProvider.
- Build an LLMProvider interface before connecting real models.
- Do not require Ollama. Use Hugging Face Transformers or llama.cpp/GGUF as the first local runtime candidates.
- First local candidates:
  - qwen3:4b
  - qwen3:8b
  - gemma3:4b
  - gemma3:12b
- Later candidates:
  - qwen3:14b
  - Gemma 4 series
  - llama.cpp / GGUF models
- Evaluate models by:
  - Korean diary understanding
  - memory chunk usage
  - JSON score schema stability
  - response latency
  - hallucination control
  - local execution feasibility

## Data Design Direction
Core collections or in-memory equivalents:

- goals
  - id
  - title
  - category
  - priority
  - deadline
  - success_criteria
  - risk
  - created_at
  - updated_at

- todos
  - id
  - title
  - related_goal_id
  - priority
  - due_date
  - status
  - created_at
  - completed_at

- daily_logs
  - id
  - date
  - content
  - mood
  - energy
  - tags
  - related_goal_ids
  - related_todo_ids
  - created_at

- interest_items
  - id
  - item_type: link, image, note
  - title
  - description
  - url
  - file_path
  - source
  - tags
  - related_goal_ids
  - related_log_ids
  - created_at

- memory_chunks
  - id
  - source_type: goal, todo, daily_log, interest_item, chat_message
  - source_id
  - content
  - summary
  - tags
  - embedding_status
  - created_at

- state_evaluations
  - id
  - date
  - goal_alignment
  - task_completion
  - avoidance_pattern
  - overload_risk
  - interest_consistency
  - net_state_score
  - reason
  - created_at

## Rules for Codex
- MVP first.
- Do not over-engineer.
- Keep modules small.
- Do not hardcode API keys, tokens, passwords, or private credentials.
- Prefer mock LLM until memory retrieval pipeline works.
- Do not add unnecessary frameworks.
- Do not connect real external services unless explicitly asked.
- After code changes, run basic syntax/import checks if possible.
- Summarize changed files.
- Explain any command that failed and why.

## Git Rules
- This project should be managed with Git.
- Use the remote repository:
  - https://github.com/leehyeoksu/personal-memory-agent
- Before making changes, check:
  - `git status`
- After making changes, check:
  - `git status`
  - `git diff`
- Do not commit broken code if basic checks fail.
- Do not commit secrets, `.env`, API keys, tokens, or local credentials.
- Keep `.venv/`, `__pycache__/`, `.pytest_cache/`, and local environment files out of Git.
- If `.gitignore` is missing, create one.
- Use clear commit messages.
- Prefer small commits grouped by feature.

## Git Commit / Push Policy
- Codex may prepare changes and suggest a commit message.
- Codex may create a commit only when explicitly asked.
- Codex must not push to GitHub unless the user explicitly asks to push.
- Before pushing, Codex should show:
  - changed files
  - commit message
  - basic check result
  - target branch
- Default branch should be `main`.
- If remote origin is missing, configure it with:
  - `git remote add origin https://github.com/leehyeoksu/personal-memory-agent.git`
- If branch is not main, use:
  - `git branch -M main`
- Push command:
  - `git push -u origin main`

## Done Means
- API route exists.
- README run instructions exist.
- Basic check command is documented.
- Changed files are summarized.
- Git status is clean or remaining uncommitted changes are explained.
