## FEATURE:

Develop a robust Pydantic AI multi-agent system. The primary agent will be a **Research Agent** that utilizes the Brave Search API to perform research queries. This Research Agent will have the capability to delegate email drafting tasks to a **sub-agent, the Email Draft Agent**, which will integrate with the Gmail API to create email drafts. The entire system should be accessible via a command-line interface (CLI) that provides real-time streaming responses and visibility into the tools being used.

The system must support multiple LLM providers and handle API authentication securely through environment variables.

## EXAMPLES:

To ensure best practices and consistent architecture, refer to the following examples in the `examples/` folder. Do not copy them directly, but use them as inspiration for structuring the new code:

- `examples/cli.py`: This file demonstrates the desired CLI structure, including how to handle streaming responses and display tool visibility. Use its pattern for the main `cli.py` file of this project.
- `examples/agent/`: This directory contains patterns for creating Pydantic AI agents.
    - `examples/agent/agent.py`: Provides the core structure for agent creation, tool registration, and managing agent dependencies. Follow this pattern for both the `research_agent.py` and `email_agent.py`.
    - `examples/agent/providers.py`: Illustrates how to configure multiple LLM providers. Adopt this pattern for `agents/providers.py` to support various LLMs.
    - Read through all other files in this directory to understand best practices for agent development within this framework.

## DOCUMENTATION:

The following documentation and references are crucial for successful implementation:

- **Pydantic AI Documentation**:
    - [https://ai.pydantic.dev/agents/](https://ai.pydantic.dev/agents/) (Core agent creation patterns)
    - [https://ai.pydantic.dev/multi-agent-applications/](https://ai.pydantic.dev/multi-agent-applications/) (Multi-agent system patterns, particularly agent-as-tool)
- **Gmail API Documentation**:
    - [https://developers.google.com/gmail/api/guides/sending](https://developers.google.com/gmail/api/guides/sending) (Details on Gmail API authentication and draft creation)
    - [https://github.com/googleworkspace/python-samples/blob/main/gmail/snippet/send%20mail/create_draft.py](https://github.com/googleworkspace/python-samples/blob/main/gmail/snippet/send%20mail/create_draft.py) (Official Gmail draft creation example to follow)
- **Brave Search API Documentation**:
    - [https://api-dashboard.search.brave.com/app/documentation](https://api-dashboard.search.brave.com/app/documentation) (Brave Search API REST endpoints for integration)

## OTHER CONSIDERATIONS:

Keep the following critical points in mind during development:

-   **Asynchronous Operations**: Pydantic AI strictly requires asynchronous functions throughout. Avoid synchronous functions in an asynchronous context.
-   **Gmail API Authentication**: The Gmail API necessitates an OAuth2 flow on the first run. Ensure `credentials.json` is properly handled and `token.json` is stored securely (e.g., in a `credentials/` directory).
-   **Brave API Rate Limits**: Be mindful of Brave API rate limits (e.g., 2000 requests/month on the free tier) to prevent service interruptions.
-   **Agent-as-Tool Pattern**: When using the agent-as-tool pattern, it is CRITICAL to pass `ctx.usage` for accurate token tracking.
-   **Email Encoding**: Gmail drafts require base64 encoding with proper MIME formatting for the email content.
-   **Imports**: Always use absolute imports for cleaner and more maintainable code.
-   **Secure Credential Storage**: Store all sensitive credentials in `.env` files and **never** commit them to version control.
-   **Environment Variables**: Utilize `python-dotenv` and `load_env()` for managing environment variables.
-   **Documentation**: Create a comprehensive `README.md` that includes setup, installation, usage instructions, API key configuration, and an architecture diagram.
-   **Project Structure**: Adhere to the desired codebase tree outlined in the `EXAMPLE_multi_agent_prp.md` for organizing files and modules.
-   **Testing**: Implement comprehensive Pytest unit tests for all new features, covering happy paths, edge cases, and failure cases, aiming for 80%+ coverage. Mock external API calls where necessary.
-   **File Size**: No single file should exceed 500 lines of code. Refactor into smaller modules if a file approaches this limit.
-   **Python Style**: Follow PEP8, use type hints, and format code with `black`. Write Google-style docstrings for all functions.
