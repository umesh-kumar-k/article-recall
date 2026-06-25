---
aliases:
  - Retry
Source 1: https://docs.langchain.com/oss/python/langchain/middleware/built-in#tool-retry
Source 2: https://docs.langchain.com/oss/python/langchain/middleware/built-in#model-retry
---
## Tool retry

Automatically retry failed tool calls with configurable exponential backoff. Tool retry is useful for the following:

- Handling transient failures in external API calls.
- Improving reliability of network-dependent tools.
- Building resilient agents that gracefully handle temporary errors.

**API reference:** [`ToolRetryMiddleware`](https://reference.langchain.com/python/langchain/agents/middleware/tool_retry/ToolRetryMiddleware)

```
from langchain.agents import create_agent
from langchain.agents.middleware import ToolRetryMiddleware


agent = create_agent(
    model="gpt-5.5",
    tools=[search_tool, database_tool, api_tool],
    middleware=[
        ToolRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,
            initial_delay=1.0,
            max_delay=60.0,
            jitter=True,
            tools=["api_tool"],
            retry_on=(ConnectionError, TimeoutError),
            on_failure="continue",
        ),
    ],
)

```


## Model retry

Automatically retry failed model calls with configurable exponential backoff. Model retry is useful for the following:

- Handling transient failures in model API calls.
- Improving reliability of network-dependent model requests.
- Building resilient agents that gracefully handle temporary model errors.

**API reference:** [`ModelRetryMiddleware`](https://reference.langchain.com/python/langchain/agents/middleware/model_retry/ModelRetryMiddleware)

```

from langchain.agents import create_agent
from langchain.agents.middleware import ModelRetryMiddleware


# Basic usage with default settings (2 retries, exponential backoff)
agent = create_agent(
    model="gpt-5.5",
    tools=[search_tool],
    middleware=[ModelRetryMiddleware()],
)

# Custom exception filtering
class TimeoutError(Exception):
    """Custom exception for timeout errors."""
    pass

class ConnectionError(Exception):
    """Custom exception for connection errors."""
    pass

# Retry specific exceptions only
retry = ModelRetryMiddleware(
    max_retries=4,
    retry_on=(TimeoutError, ConnectionError),
    backoff_factor=1.5,
)


def should_retry(error: Exception) -> bool:
    # Only retry on rate limit errors
    if isinstance(error, TimeoutError):
        return True
    # Or check for specific HTTP status codes
    if hasattr(error, "status_code"):
        return error.status_code in (429, 503)
    return False

retry_with_filter = ModelRetryMiddleware(
    max_retries=3,
    retry_on=should_retry,
)

# Return error message instead of raising
retry_continue = ModelRetryMiddleware(
    max_retries=4,
    on_failure="continue",  # Return AIMessage with error instead of raising
)

# Custom error message formatting
def format_error(error: Exception) -> str:
    return f"Model call failed: {error}. Please try again later."

retry_with_formatter = ModelRetryMiddleware(
    max_retries=4,
    on_failure=format_error,
)

# Constant backoff (no exponential growth)
constant_backoff = ModelRetryMiddleware(
    max_retries=5,
    backoff_factor=0.0,  # No exponential growth
    initial_delay=2.0,  # Always wait 2 seconds
)

# Raise exception on failure
strict_retry = ModelRetryMiddleware(
    max_retries=2,
    on_failure="error",  # Re-raise exception instead of returning message
)

```


