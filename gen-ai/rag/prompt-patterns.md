---
aliases:
  - Prompt Patterns
highlights: |-
  Stuff: Insert all retrieved chunks into single prompt; simple but limited by context window

  Map-Reduce: Summarize each chunk independently , then combine summaries; handles large contexts but loses cross-chunk reasoning

  Refine: Iteratively update answer as new chunks processed; better systhesis but multiple LLM calls

  Agentic: LLM decides retrieval strategy and when sufficient information obtained; most capable but least predictable
---
