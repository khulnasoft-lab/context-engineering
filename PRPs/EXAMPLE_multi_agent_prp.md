name: "Multi-Agent System: Research Agent with Email Draft Sub-Agent"
description: |
  ## Purpose
  Build a production-grade Pydantic AI multi-agent system, featuring a Research Agent (using Brave Search API) and a subordinate Email Draft Agent (using Gmail API). Demonstrate the “agent-as-tool” pattern with robust context, validation, and integration best practices.

  ## Core Principles
  1. **Context is King:** All needed docs, examples, and caveats must be surfaced.
  2. **Validation Loops:** Provide executable tests/lints for iterative, self-correcting development.
  3. **Information Dense:** Use keywords and codebase patterns for maximal relevance.
  4. **Progressive Success:** Start simple, validate, enhance in stages.
  5. **Project Rules:** Follow all rules in `CLAUDE.md`.

---

# Table of Contents
1. [Goal](#goal)
2. [Why](#why)
3. [What](#what)
4. [Success Criteria](#success-criteria)
5. [All Needed Context](#all-needed-context)
6. [Known Gotchas & Library Quirks](#known-gotchas--library-quirks)
7. [Implementation Blueprint](#implementation-blueprint)
8. [Integration Points](#integration-points)
9. [Validation Loop](#validation-loop)
10. [Final Validation Checklist](#final-validation-checklist)
11. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
12. [Confidence Score](#confidence-score)

---

## Goal

Build a CLI-based system where:
- Users submit research queries.
- The Research Agent utilizes Brave Search API for research.
- The Research Agent can invoke an Email Draft Agent (via Gmail API) to draft emails.
- Responses stream back to the user with tool/action visibility.

---

## Why

- **Business value:** Automates research-to-email workflows, reducing manual effort.
- **Integration:** Demonstrates advanced Pydantic AI multi-agent patterns and robust API integrations.
- **Problems solved:** Fast, context-rich communication for research-heavy teams.

---

## What

- User interacts via CLI.
- Research Agent performs web search (Brave API).
- Research Agent can delegate to Email Agent for Gmail draft creation.
- Data models ensure type safety and validation.
- All interactions, tool calls, and errors are visible and logged.

### Specific Deliverables

- [ ] CLI interface for user queries.
- [ ] Research Agent (Brave Search).
- [ ] Email Agent (Gmail Drafting).
- [ ] External API integrations (Brave + Gmail).
- [ ] Typed data models (Pydantic).
- [ ] Comprehensive tests (unit, integration).
- [ ] Complete updated documentation.

---

## Success Criteria

- [ ] Research Agent returns Brave API results.
- [ ] Email Agent drafts Gmail emails with proper OAuth.
- [ ] CLI streams responses and logs tool usage.
- [ ] All tests/lints/type checks pass.
- [ ] Error cases (e.g., auth/rate limit) are handled and logged.
- [ ] No sensitive credentials are committed.

---

## All Needed Context

### Documentation & References

```yaml
- url: https://ai.pydantic.dev/agents/
  why: Agent creation, tool registration patterns.
- url: https://ai.pydantic.dev/multi-agent-applications/
  why: Multi-agent patterns, agent-as-tool, passing ctx.usage.
- url: https://developers.google.com/gmail/api/guides/sending
  why: Gmail API authentication and draft creation.
- url: https://api-dashboard.search.brave.com/app/documentation
  why: Brave Search API endpoints and authentication.
- file: examples/cli.py
  why: CLI streaming and tool visibility pattern.
- file: examples/agent/agent.py
  why: Agent setup, dependencies, tool registration, error handling.
- file: examples/agent/providers.py
  why: LLM provider configuration.
```

### Current Codebase Tree

```bash
.
├── src/
│   └── ...
├── examples/
│   └── ...
├── tests/
│   └── ...
├── PRPs/
│   └── EXAMPLE_multi_agent_prp.md
├── INITIAL.md
├── CLAUDE.md
├── requirements.txt
├── .env.example
└── README.md
```

### Desired Codebase Tree

```bash
.
├── agents/
│   ├── research_agent.py
│   ├── email_agent.py
│   ├── providers.py
│   └── models.py
├── tools/
│   ├── brave_search.py
│   └── gmail_tool.py
├── config/
│   └── settings.py
├── tests/
│   ├── test_research_agent.py
│   ├── test_email_agent.py
│   ├── test_brave_search.py
│   ├── test_gmail_tool.py
│   └── test_cli.py
├── cli.py
├── .env.example
├── requirements.txt
├── README.md
└── credentials/.gitkeep
```

---

## Known Gotchas & Library Quirks

> **Important:** Review and address these before/during implementation.

- **Async throughout:** Pydantic AI requires async for all agent/tool functions.
- **Gmail OAuth2:** Requires user browser auth on first run; store tokens securely in `credentials/`.
- **Brave API limits:** Free tier is 2k req/month; handle 401/429 errors and implement retries.
- **ctx.usage:** Always pass when invoking agents as tools for accurate token/cost tracking.
- **Absolute imports:** Use them for maintainability.
- **No sensitive creds in VCS:** Use `.env`; never commit credentials.
- **No >500 LOC files:** Refactor if needed.

---

## Implementation Blueprint

### Data Models (`agents/models.py`)

```python
from pydantic import BaseModel, Field
from typing import List, Optional

class ResearchQuery(BaseModel):
    query: str = Field(..., description="Research topic")
    max_results: int = Field(10, ge=1, le=50)
    include_summary: bool = Field(True)

class BraveSearchResult(BaseModel):
    title: str
    url: str
    description: str
    score: float = Field(0, ge=0, le=1)

class EmailDraft(BaseModel):
    to: List[str]
    subject: str
    body: str
    cc: Optional[List[str]] = None
    bcc: Optional[List[str]] = None

class ResearchEmailRequest(BaseModel):
    research_query: str
    email_context: str
    recipient_email: str
    email_subject: Optional[str] = None
```

### Task List (in Order)

```yaml
- Setup config/settings.py and .env.example (env loading, validate keys)
- Implement tools/brave_search.py (async, handle errors, return model)
- Implement tools/gmail_tool.py (OAuth2, MIME, draft creation)
- Implement agents/email_agent.py (agent, tool registration)
- Implement agents/research_agent.py (multi-agent, tool registration, ctx.usage)
- Implement cli.py (streaming, context, colorized output)
- Implement tests/* (mock APIs, happy path and edge cases)
- Write README.md (setup, usage, architecture diagram)
```

### Example Task Pseudocode

```python
# tools/brave_search.py
async def search_brave(query: str, count: int = 10) -> List[BraveSearchResult]:
    # Load API key from settings
    # Async GET to Brave API
    # Raise on 401/429, handle timeouts
    # Return parsed BraveSearchResult list
    ...

# agents/research_agent.py
@research_agent.tool
async def draft_email_from_research(ctx, req: ResearchEmailRequest) -> str:
    # Compose email context, call EmailAgent as tool, pass ctx.usage
    # Return success/failure string
    ...
```

---

## Integration Points

- **Env:** Set all keys in `.env`, never hardcode.
- **Gmail OAuth:** Browser flow, token saved in `credentials/`.
- **Requirements:** `pydantic-ai`, `httpx`, `google-api-python-client`, `python-dotenv`, `typer`, `pytest`, etc.

---

## Validation Loop

1. **Syntax/Types:**  
   `ruff check . --fix`  
   `mypy .`

2. **Unit Tests:**  
   `pytest tests/ --cov=agents --cov=tools --cov-report=term-missing`

3. **Integration Test:**  
   Run CLI, perform real search+draft, verify draft in Gmail.

---

## Final Validation Checklist

- [ ] All tests/lints/types pass
- [ ] Gmail OAuth works first run
- [ ] Brave API results returned
- [ ] Research agent calls Email Agent as tool
- [ ] CLI logs tool usage, errors handled
- [ ] No secrets in git
- [ ] Docs updated and clear

---

## Anti-Patterns to Avoid

- ❌ Hardcoded credentials  
- ❌ Synchronous code in async agents  
- ❌ Skipping Gmail OAuth flow  
- ❌ Ignoring rate limits  
- ❌ Not passing `ctx.usage` in multi-agent calls  
- ❌ Large files (>500 LOC)  
- ❌ Generic exception handling  
- ❌ Skipping/faking tests to “make green”  
- ❌ Violating CLAUDE.md rules

---

## Confidence Score: 9/10

- Clear patterns, examples, docs, and API references
- Strong anti-patterns, robust validation, and test guidance
- Only minor uncertainty around browser OAuth UX on first run
