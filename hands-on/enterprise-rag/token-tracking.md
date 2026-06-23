---
aliases:
  - Token Usage Tracking
Source 1: https://www.braintrust.dev/articles/how-to-track-llm-token-usage-2026
Source 2: https://docs.langchain.com/langsmith/cost-tracking
---
# Cost tracking

* **Key Points:**
  - Building agents at scale introduces non-trivial, usage-based costs that can be difficult to track.
  - LangSmith automatically records LLM token usage and costs for major providers, and also allows you to submit custom cost data for any additional components.
  - This gives you a single, unified view of costs across your entire application, which makes it easy to monitor, understand, and debug your spend.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith`

## View costs in the LangSmith UI
* **Key Points:**
  - In the LangSmith UI, you can explore usage and spend three ways: as a breakdown within individual traces, as aggregated metrics in project stats, and in dashboards.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith UI`, `trace tree`, `project stats`, `dashboards`, `session_id`, `thread_id`, `conversation_id`

### Token and cost breakdowns
* **Key Points:**
  - The UI separates token usage and costs into three categories:
    - Input: Tokens in the prompt sent to the model. Subtypes include: cache reads, text tokens, image tokens, etc.
    - Output: Tokens generated in the response from the model. Subtypes include: reasoning tokens, text tokens, image tokens, etc.
    - Other: Costs from tool calls, retrieval steps, or any custom runs.
* **Technical Entities (Classes/Functions/APIs):** `trace tree`, `project stats`, `dashboards`

#### In the trace tree
* **Key Points:**
  - The trace tree shows the most detailed view of token usage and cost (for a single trace).
  - It displays the total usage for the entire trace, aggregated values for each parent run and token and cost breakdowns for each child run.
  - Open any run inside a tracing project to view its trace tree.
  - When tracking costs across threads, ensure that all child runs include the thread metadata (`session_id`, `thread_id`, or `conversation_id`).
  - Without thread metadata on child runs, token counts and costs from those runs won't be included in thread-level aggregations.
* **Technical Entities (Classes/Functions/APIs):** `trace tree`, `session_id`, `thread_id`, `conversation_id`

#### In project stats
* **Key Points:**
  - The project stats panel shows the total token usage and cost for all traces in a project.
* **Technical Entities (Classes/Functions/APIs):** `project stats`

#### In dashboards
* **Key Points:**
  - Dashboards help you explore cost and token usage trends over time.
  - The prebuilt dashboard for a tracing project shows total costs and a cost breakdown by input and output tokens.
  - You may also configure custom cost tracking charts in custom dashboards.
* **Technical Entities (Classes/Functions/APIs):** `dashboards`, `custom dashboards`

## Cost tracking
* **Key Points:**
  - You can track costs in two ways:
    - Automatically: derived from token counts and model prices for LLM calls.
    - Manually: specified directly on any run, including non-LLM types.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `@traceable`, `LangSmith wrappers`, `usage_metadata`, `ls_provider`, `ls_model_name`

### LLM calls: Automatically track costs based on token counts
* **Key Points:**
  - To compute cost automatically from token usage, you need to provide token counts, the model and provider, and the model price.
  - Skip this section if you are calling LLMs with LangChain, using `@traceable` with OpenAI or Anthropic (or an OpenAI-compatible model), or using a LangSmith wrapper for OpenAI or Anthropic.
* **Technical Entities (Classes/Functions/APIs):** `usage_metadata`, `traceable`, `get_current_run_tree`, `ls_provider`, `ls_model_name`
* **Code Snippet:**
```python
from langsmith import traceable, get_current_run_tree

inputs = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "I'd like to book a table for two."},
]

@traceable(
    run_type="llm",
    metadata={"ls_provider": "my_provider", "ls_model_name": "my_model"}
)
def chat_model(messages: list):
    # Imagine this is the real model output format your application expects
    assistant_message = {
        "role": "assistant",
        "content": "Sure, what time would you like to book the table for?"
    }

    # Token usage you compute or receive from the provider
    token_usage = {
        "input_tokens": 27,
        "output_tokens": 13,
        "total_tokens": 40,
        "input_token_details": {"cache_read": 10}
    }

    # Attach token usage to the LangSmith run
    run = get_current_run_tree()
    run.set(usage_metadata=token_usage)

    return assistant_message

chat_model(inputs)
```
```python
from langsmith import traceable

inputs = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "I'd like to book a table for two."},
]
output = {
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "Sure, what time would you like to book the table for?"
            }
        }
    ],
    "usage_metadata": {
        "input_tokens": 27,
        "output_tokens": 13,
        "total_tokens": 40,
        "input_token_details": {"cache_read": 10}
    },
}

@traceable(
    run_type="llm",
    metadata={"ls_provider": "my_provider", "ls_model_name": "my_model"}
)
def chat_model(messages: list):
    return output

chat_model(inputs)
```

### Usage Metadata Schema and Cost Calculation
* **Key Points:**
  - The following fields in the usage_metadata dict are recognized by LangSmith.
* **Technical Entities (Classes/Functions/APIs):** `usage_metadata`, `input_tokens`, `output_tokens`, `total_tokens`, `input_token_details`, `output_token_details`, `input_cost`, `output_cost`, `total_cost`, `input_cost_details`, `output_cost_details`
* **Code Snippet:**
```python
# Notice that LangSmith computes the cache_read cost and then for any
# remaining input_tokens, the default input price is applied.
input_cost = 5 * 1e-6 + (20 - 5) * 2e-6  # 3.5e-5
output_cost = 10 * 3e-6  # 3e-5
total_cost = input_cost + output_cost  # 6.5e-5
```
* **Key Points:**
  - The cost for a run is computed greedily from most-to-least specific token type.
  - LangSmith does not reflect updates to the model pricing map in the costs for traces already logged.
  - Backfilling model pricing changes is not supported.
* **Technical Entities (Classes/Functions/APIs):** `Model Name`, `Input Price`, `Input Price Breakdown`, `Output Price`, `Output Price Breakdown`, `Model Activation Date`, `Match Pattern`, `Provider`

### LLM calls: Send costs directly
* **Key Points:**
  - Gemini 2.5 Pro Preview and Gemini 2.5 Pro use a stepwise cost function, which LangSmith supports by default.
  - For any other model with non-linear pricing, calculate costs client-side and send them as usage_metadata as shown in the following code
* **Technical Entities (Classes/Functions/APIs):** `Gemini 2.5 Pro Preview`, `Gemini 2.5 Pro`, `usage_metadata`, `traceable`, `get_current_run_tree`, `input_cost`, `input_cost_details`, `output_cost`
* **Code Snippet:**
```python
from langsmith import traceable, get_current_run_tree

inputs = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "I'd like to book a table for two."},
]

@traceable(
    run_type="llm",
    metadata={"ls_provider": "my_provider", "ls_model_name": "my_model"}
)
def chat_model(messages: list):
    llm_output = {
        "choices": [
            {
                "message": {
                    "role": "assistant",
                    "content": "Sure, what time would you like to book the table for?"
                }
            }
        ],
        "usage_metadata": {
            # Specify cost (in dollars) for the inputs and outputs
            "input_cost": 1.1e-6,
            "input_cost_details": {"cache_read": 2.3e-7},
            "output_cost": 5.0e-6,
        },
    }
    run = get_current_run_tree()
    run.set(usage_metadata=llm_output["usage_metadata"])
    return llm_output["choices"][0]["message"]

chat_model(inputs)
```

### Other runs: Send costs
* **Key Points:**
  - You can also send cost information for any non-LLM runs, such as tool calls.
* **Technical Entities (Classes/Functions/APIs):** `total_cost`, `usage_metadata`, `traceable`, `get_current_run_tree`
* **Code Snippet:**
```python
from langsmith import traceable, get_current_run_tree

# Example tool: get_weather
@traceable(run_type="tool", name="get_weather")
def get_weather(city: str):
    # Your tool logic goes here
    result = {
        "temperature_f": 68,
        "condition": "sunny",
        "city": city,
    }

    # Cost for this tool call (computed however you like)
    tool_cost = 0.0015

    # Attach usage metadata to the LangSmith run
    run = get_current_run_tree()
    run.set(usage_metadata={"total_cost": tool_cost})

    # Return only the actual tool result (no usage info)
    return result

tool_response = get_weather("San Francisco")
```
```python
from langsmith import traceable

# Example tool: get_weather
@traceable(run_type="tool", name="get_weather")
def get_weather(city: str):
    # Your tool logic goes here
    result = {
        "temperature_f": 68,
        "condition": "sunny",
        "city": city,
    }

    # Attach tool call costs here
    return {
        **result,
        "usage_metadata": {
            "total_cost": 0.0015,   # <-- cost for this tool call
        },
    }

tool_response = get_weather("San Francisco")
```