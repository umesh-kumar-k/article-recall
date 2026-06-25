---
aliases:
  - Multi Agent skills workflow
Source 1: https://docs.langchain.com/oss/javascript/langchain/multi-agent/skills
---
## Skills
* **Key Points:**
  - In the **skills** architecture, specialized capabilities are packaged as invocable "skills" that augment an agent's behavior.
  - Skills are primarily prompt-driven specializations that an agent can invoke on-demand.
  - This pattern is conceptually identical to [Agent Skills](https://agentskills.io/) and [llms.txt](https://llmstxt.org/) (introduced by Jeremy Howard), which uses tool calling for progressive disclosure of documentation.
  - The skills pattern applies progressive disclosure to specialized prompts and domain knowledge rather than just documentation pages.

## Key characteristics
* **Key Points:**
  - Prompt-driven specialization: Skills are primarily defined by specialized prompts
  - Progressive disclosure: Skills become available based on context or user needs
  - Team distribution: Different teams can develop and maintain skills independently
  - Lightweight composition: Skills are simpler than full sub-agents
  - Reference awareness: Skills can reference scripts, templates, and other resources

## When to use
* **Key Points:**
  - Use the skills pattern when you want a single agent with many possible specializations, you don't need to enforce specific constraints between skills, or different teams need to develop capabilities independently.
  - Common examples include coding assistants (skills for different languages or tasks), knowledge bases (skills for different domains), and creative assistants (skills for different formats).

## Basic implementation
* **Key Points:**
  - Use the skills pattern when you want a single agent with many possible specializations, you don't need to enforce specific constraints between skills, or different teams need to develop capabilities independently.
* **Technical Entities (Classes/Functions/APIs):** `tool`, `createAgent`, `z`
* **Code Snippet:**
```typescript
import { tool, createAgent } from "langchain";
import * as z from "zod";

const loadSkill = tool(
  async ({ skillName }) => {
    // Load skill content from file/database
    return "";
  },
  {
    name: "load_skill",
    description: `Load a specialized skill.

Available skills:
- write_sql: SQL query writing expert
- review_legal_doc: Legal document reviewer

Returns the skill's prompt and context.`,
    schema: z.object({
      skillName: z
        .string()
        .describe("Name of skill to load")
    })
  }
);

const agent = createAgent({
  model: "gpt-5.5",
  tools: [loadSkill],
  systemPrompt: (
    "You are a helpful assistant. " +
    "You have access to two skills: " +
    "write_sql and review_legal_doc. " +
    "Use load_skill to access them."
  ),
});
```

## Extending the pattern
* **Key Points:**
  - When writing custom implementations, you can extend the basic skills pattern in several ways:
  - **Dynamic tool registration**: Combine progressive disclosure with state management to register new tools as skills load.
  - **Hierarchical skills**: Skills can define other skills in a tree structure, creating nested specializations.
  - **Reference awareness**: While each skill only has one prompt, this prompt can reference the location of other assets and provide information on when the agent should use those assets.
  - When those assets become relevant, the agent will know that those files exist and read them into memory as needed to complete tasks.
  - This also follows the progressive disclosure pattern and limits the information in the context window.