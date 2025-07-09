name: "Feature Name: Brief and Descriptive"
description: |
  ## Purpose
  This template is optimized for AI agents to implement features with sufficient context and self-validation capabilities, aiming to achieve working code through iterative refinement. It ensures comprehensive guidance for developing new functionalities within the existing codebase.

  ## Core Principles
  1.  **Context is King**: Include ALL necessary documentation, examples, and caveats.
  2.  **Validation Loops**: Provide executable tests/lints the AI can run and fix.
  3.  **Information Dense**: Use keywords and patterns from the codebase.
  4.  **Progressive Success**: Start simple, validate, then enhance.
  5.  **Global rules**: Be sure to follow all rules in `CLAUDE.md`.

---

## Goal
[What needs to be built - be specific about the end state and desired functionality. Focus on the 'what' and 'how' this feature will enable users or other systems.]

## Why
-   **Business value**: [Explain the direct business value or user impact this feature provides.]
-   **Integration**: [Describe how this feature integrates with existing systems or demonstrates advanced patterns.]
-   **Problems solved**: [Identify the specific problems this feature solves and for whom.]

## What
[Describe the user-visible behavior and core technical requirements. Be explicit about interactions, data flows, and expected outcomes.]

### Specific Deliverables:
-   [ ] A clear, tangible component or functionality (e.g., "A CLI application for user interaction").
-   [ ] An agent with specific capabilities (e.g., "A Research Agent capable of performing web searches").
-   [ ] Integration with external APIs (e.g., "Integration with Gmail API for email drafting").
-   [ ] Data models and validation (e.g., "Pydantic models for request/response validation").
-   [ ] Comprehensive testing (e.g., "Unit and integration tests for all new components").
-   [ ] Updated documentation (e.g., "A clear `README.md` with setup and usage instructions").

### Success Criteria
-   [ ] [Specific measurable outcome 1 (e.g., "Research Agent successfully searches via Brave API").]
-   [ ] [Specific measurable outcome 2 (e.g., "Email Agent creates Gmail drafts with proper authentication").]
-   [ ] [Specific measurable outcome 3 (e.g., "CLI provides streaming responses with tool visibility").]
-   [ ] [Specific measurable outcome 4 (e.g., "All tests pass and code meets quality standards").]
-   [ ] [Specific measurable outcome 5 (e.g., "Error cases are handled gracefully and logged appropriately").]

## All Needed Context

### Documentation & References (list all context needed to implement the feature)
```yaml
# MUST READ - Include these in your context window
- url: [Official API docs URL]
  why: [Specific sections/methods you'll need, e.g., "Core agent creation patterns" or "Authentication flow for X API"]
  
- file: [path/to/example.py]
  why: [Pattern to follow for agent creation, tool registration, dependencies, CLI structure, etc. Highlight specific functions or classes to mimic.]
  
- doc: [Library documentation URL] 
  section: [Specific section about common pitfalls, best practices, or advanced features of the library.]
  critical: [Key insight that prevents common errors or unlocks a crucial pattern.]

- docfile: [PRPs/ai_docs/file.md]
  why: [Reference to internal documentation, e.g., "docs that the user has pasted into the project for context."]
```

### Current Codebase tree (run `tree` in the root of the project to get an overview of the codebase)

```bash
.
├── src/                      # Main source code directory
│   ├── existing_module.py    # Example existing file
│   └── utils/
│       └── helpers.py        # Common utility functions
├── examples/                 # Reference examples (e.g., from `Context Engineering Template`)
│   ├── agent/
│   │   ├── agent.py
│   │   ├── providers.py
│   │   └── ...
│   └── cli.py
├── tests/                    # Unit and integration tests
│   ├── test_existing.py
│   └── ...
├── PRPs/                     # Product Requirements Prompts
│   └── templates/
│       └── prp_base.md
├── INITIAL.md                # Initial feature request
├── CLAUDE.md                 # Project-specific rules and guidelines
├── requirements.txt          # Python dependencies
├── .env.example              # Example environment variables
└── README.md                 # Project documentation
```

### Desired Codebase tree with files to be added and responsibility of file

```bash
.
├── agents/                       # Contains Pydantic AI agents
│   ├── __init__.py               # Package initialization
│   ├── research_agent.py         # Primary agent responsible for research
│   ├── email_agent.py            # Sub-agent responsible for email drafting
│   ├── providers.py              # LLM provider configuration and management
│   └── models.py                 # Pydantic models for agent-specific data validation
├── tools/                        # External API integrations as tools
│   ├── __init__.py               # Package initialization
│   ├── brave_search.py           # Brave Search API wrapper
│   └── gmail_tool.py             # Gmail API wrapper for draft creation
├── config/                       # Configuration files
│   ├── __init__.py               # Package initialization
│   └── settings.py               # Environment variable loading and settings
├── tests/                        # Tests for new features, mirroring main structure
│   ├── __init__.py               # Package initialization
│   ├── test_research_agent.py    # Tests for the Research Agent
│   ├── test_email_agent.py       # Tests for the Email Draft Agent
│   ├── test_brave_search.py      # Tests for the Brave Search tool
│   ├── test_gmail_tool.py        # Tests for the Gmail tool
│   └── test_cli.py               # Tests for the command-line interface
├── cli.py                        # Main command-line interface entry point
├── .env.example                  # Template for environment variables
├── requirements.txt              # Updated Python dependencies
├── README.md                     # Comprehensive project documentation
└── credentials/.gitkeep          # Placeholder for securely stored Gmail credentials
```

### Known Gotchas & Library Quirks

```python
# CRITICAL: Pydantic AI requires async throughout - avoid synchronous functions in an asynchronous context.
# CRITICAL: Gmail API requires an OAuth2 flow on the first run; ensure `credentials.json` is handled and `token.json` is securely stored (e.g., in a `credentials/` directory).
# CRITICAL: Brave API has rate limits (e.g., 2000 requests/month on the free tier); implement proper error handling and retry mechanisms.
# CRITICAL: When using the agent-as-tool pattern, it is CRITICAL to pass `ctx.usage` for accurate token tracking and cost management.
# CRITICAL: Gmail drafts require base64 encoding with proper MIME formatting for the email content.
# CRITICAL: Always use absolute imports for cleaner and more maintainable code (e.g., `from agents.research_agent import ResearchAgent`).
# CRITICAL: Store all sensitive credentials in `.env` files and **never** commit them to version control.
# CRITICAL: Utilize `python-dotenv` and `load_env()` for managing environment variables.
# CRITICAL: No single file should exceed 500 lines of code. Refactor into smaller modules if a file approaches this limit.
```

## Implementation Blueprint

### Data models and structure

Create the core data models, ensuring type safety and consistency.

```python
# agents/models.py - Core data structures for agents and tools
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class ResearchQuery(BaseModel):
    query: str = Field(..., description="Research topic to investigate")
    max_results: int = Field(10, ge=1, le=50, description="Maximum number of search results to retrieve")
    include_summary: bool = Field(True, description="Whether to include a summarized overview of results")

class BraveSearchResult(BaseModel):
    title: str = Field(..., description="Title of the search result")
    url: str = Field(..., description="URL of the search result")
    description: str = Field(..., description="Snippet or description of the search result")
    score: float = Field(0.0, ge=0.0, le=1.0, description="Relevance score of the result")

class EmailDraft(BaseModel):
    to: List[str] = Field(..., min_items=1, description="List of recipient email addresses")
    subject: str = Field(..., min_length=1, description="Subject of the email draft")
    body: str = Field(..., min_length=1, description="Body content of the email draft")
    cc: Optional[List[str]] = Field(None, description="Optional list of CC email addresses")
    bcc: Optional[List[str]] = Field(None, description="Optional list of BCC email addresses")

class ResearchEmailRequest(BaseModel):
    research_query: str = Field(..., description="The query used for research")
    email_context: str = Field(..., description="Contextual information for drafting the email, typically derived from research results")
    recipient_email: str = Field(..., description="The primary recipient's email address")
    email_subject: Optional[str] = Field(None, description="Optional subject for the email")
```

### List of tasks to be completed in the order they should be completed

```yaml
Task 1: Setup Configuration and Environment
  CREATE config/settings.py:
    - PATTERN: Use `pydantic-settings` (or `os.getenv` if `pydantic-settings` is not established) for configuration.
    - Load environment variables with sensible defaults.
    - Validate that required API keys are present upon initialization.
  CREATE .env.example:
    - Include all required environment variables with clear descriptions and placeholder values.
    - Follow the pattern from `examples/README.md` for environmental variable setup.

Task 2: Implement Brave Search Tool
  CREATE tools/brave_search.py:
    - PATTERN: Implement asynchronous functions as shown in `examples/agent/tools.py`.
    - Use `httpx` (already in `requirements.txt`) for asynchronous HTTP requests to the Brave Search API.
    - Handle API rate limits, network errors, and non-200 HTTP responses gracefully, potentially with retries.
    - Return a list of `BraveSearchResult` models.

Task 3: Implement Gmail Tool
  CREATE tools/gmail_tool.py:
    - PATTERN: Follow the OAuth2 flow as described in the official Gmail quickstart guide.
    - Store the `token.json` securely in the `credentials/` directory.
    - Implement a function to create an email draft, ensuring proper MIME encoding and base64 formatting.
    - Handle authentication refresh automatically.

Task 4: Create Email Draft Agent
  CREATE agents/email_agent.py:
    - PATTERN: Structure the agent following `examples/agent/agent.py`, using `pydantic_ai.Agent`.
    - Use the `deps_type` pattern for dependency injection if the agent requires specific external resources.
    - Register the `gmail_tool`'s draft creation function as an `@agent.tool`.
    - The agent should accept parameters to create an `EmailDraft` model and return a confirmation.

Task 5: Create Research Agent
  CREATE agents/research_agent.py:
    - PATTERN: Implement the multi-agent pattern as described in the Pydantic AI documentation ([https://ai.pydantic.dev/multi-agent-applications/](https://ai.pydantic.dev/multi-agent-applications/)).
    - Register the `brave_search` tool for performing web queries.
    - Register the `email_agent.run()` method as a tool, enabling the Research Agent to delegate email drafting.
    - Ensure `RunContext` is properly used for dependency injection and to pass `ctx.usage` for token tracking when delegating tasks.

Task 6: Implement CLI Interface
  CREATE cli.py:
    - PATTERN: Follow the streaming response and tool visibility pattern from `examples/cli.py`.
    - Implement a clean command-line interface using `typer` or `argparse` (if not already specified in `requirements.txt`).
    - Handle asynchronous operations correctly using `asyncio.run()`.
    - Manage conversation context for multi-turn interactions.
    - Provide clear, color-coded output indicating agent actions and tool usage.

Task 7: Add Comprehensive Tests
  CREATE tests/test_research_agent.py:
    - PATTERN: Mirror the testing structure found in `examples/tests/`.
    - Implement unit tests for the Research Agent, covering search functionality and email delegation.
    - Mock external API calls (Brave Search, Gmail) to ensure tests are isolated and fast.
    - Aim for comprehensive coverage, including happy paths, edge cases, and error scenarios.
  CREATE tests/test_email_agent.py:
    - Implement unit tests for the Email Draft Agent, focusing on draft creation and authentication.
  CREATE tests/test_brave_search.py:
    - Implement unit tests for the Brave Search tool, covering API calls and response parsing.
  CREATE tests/test_gmail_tool.py:
    - Implement unit tests for the Gmail tool, focusing on OAuth flow and draft mechanics.
  CREATE tests/test_cli.py:
    - Implement tests for the CLI's user interaction and output.

Task 8: Create Documentation
  CREATE README.md:
    - PATTERN: Follow the structure and content level of `examples/README.md`.
    - Include detailed setup instructions (dependencies, environment variables).
    - Provide clear steps for API key configuration for Brave Search and Gmail.
    - Describe usage examples of the CLI.
    - Include an architecture diagram of the multi-agent system.
    - Detail the project structure.
```

### Per task pseudocode as needed added to each task

*(See your original for detailed pseudocode; keep this for implementation guidance.)*

### Integration Points

```yaml
ENVIRONMENT:
  - add to: .env
  - vars: |
      # LLM Configuration
      LLM_PROVIDER=openai
      LLM_API_KEY=sk-...
      LLM_MODEL=gpt-4
      # Brave Search API Key
      BRAVE_API_KEY=BSA...
      # Gmail API Credentials (path to credentials.json)
      GMAIL_CREDENTIALS_PATH=./credentials/credentials.json

CONFIG:
  - Gmail OAuth: The first run of the Gmail tool will typically open a browser for user authorization.
  - Token storage: The OAuth token will be automatically stored in `./credentials/token.json` after successful authorization. This file should be `.gitignore`d.

DEPENDENCIES:
  - Update `requirements.txt` with:
    - `pydantic-ai`
    - `httpx`
    - `google-api-python-client`
    - `google-auth-httplib2`
    - `google-auth-oauthlib`
    - `python-dotenv`
    - `typer`
    - `rich`
    - `pytest`
    - `pytest-cov`
    - `ruff`
    - `mypy`
```

## Validation Loop

*(Syntax, tests, integration, and checklist as in your template—keep for validation and delivery gates.)*

## Anti-Patterns to Avoid

  - ❌ **Don't hardcode API keys or sensitive credentials**; always use environment variables and secure loading mechanisms.
  - ❌ **Don't use synchronous functions in an asynchronous agent context**; Pydantic AI requires asynchronous operations throughout.
  - ❌ **Don't skip the OAuth flow setup for Gmail**; it's a necessary security step for API access.
  - ❌ **Don't ignore API rate limits** (e.g., Brave Search); implement proper retry mechanisms or back-offs.
  - ❌ **Don't forget to pass `ctx.usage`** when calling a sub-agent as a tool; this is critical for accurate token tracking and cost management in multi-agent systems.
  - ❌ **Don't commit `credentials.json` or `token.json` files** to version control.
  - ❌ **Don't create new patterns** when existing, established ones (from `examples/` or Pydantic AI docs) work perfectly.
  - ❌ **Don't skip validation** because "it should work"; always include robust input validation and error handling.
  - ❌ **Don't ignore failing tests**; debug, fix the root cause, and re-run until all tests pass.
  - ❌ **Don't catch all exceptions generically**; be specific in exception handling to allow for targeted debugging and error recovery.
  - ❌ **Don't write files longer than 500 lines of code**; refactor into smaller, focused modules.

## Confidence Score: 9/10

High confidence due to:

  - Very clear and detailed examples to follow within the provided codebase.
  - Well-documented external APIs (Brave Search, Gmail).
  - Established and well-supported patterns for multi-agent systems in Pydantic AI.
  - Comprehensive validation gates and executable tests provided within the PRP.
  - Explicit anti-patterns to avoid, directly addressing common pitfalls.

Minor uncertainty on the exact nuances of the Gmail OAuth first-time setup UX, but the official documentation provides clear guidance which mitigates this.
