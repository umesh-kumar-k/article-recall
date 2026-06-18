---
aliases:
  - Architects Overview
Source 2: https://www.kaggle.com/learn-guide/5-day-agents
Source 3: https://www.anthropic.com/engineering/building-effective-agents
Source 4: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
Source 5: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
Source 6: https://www.anthropic.com/engineering/managed-agents
Source 7: https://www.anthropic.com/engineering/writing-tools-for-agents
---
# Why Attention Is All You Need?


## Effective context engineering for AI agents

## Effective context engineering for AI agents

* **Key Points:**
  - Context is a critical but finite resource for AI agents.
  - In this post, we explore strategies for effectively curating and managing the context that powers them.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `AI agents`
* **Code Snippet:** None.

---

## Context engineering vs. prompt engineering

* **Key Points:**
  - At Anthropic, we view context engineering as the natural progression of prompt engineering.
  - Prompt engineering refers to methods for writing and organizing LLM instructions for optimal outcomes (see our docs for an overview and useful prompt engineering strategies).
  - Context engineering refers to the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference, including all the other information that may land there outside of the prompts.
  - In the early days of engineering with LLMs, prompting was the biggest component of AI engineering work, as the majority of use cases outside of everyday chat interactions required prompts optimized for one-shot classification or text generation tasks.
  - As the term implies, the primary focus of prompt engineering is how to write effective prompts, particularly system prompts.
  - However, as we move towards engineering more capable agents that operate over multiple turns of inference and longer time horizons, we need strategies for managing the entire context state (system instructions, tools, Model Context Protocol (MCP), external data, message history, etc).
  - An agent running in a loop generates more and more data that could be relevant for the next turn of inference, and this information must be cyclically refined.
  - Context engineering is the art and science of curating what will go into the limited context window from that constantly evolving universe of possible information.
  - In contrast to the discrete task of writing a prompt, context engineering is iterative and the curation phase happens each time we decide what to pass to the model.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Model Context Protocol (MCP)`, `system prompts`, `tools`, `external data`, `message history`
* **Code Snippet:** None.

---

## Why context engineering is important to building capable agents

* **Key Points:**
  - Despite their speed and ability to manage larger and larger volumes of data, we've observed that LLMs, like humans, lose focus or experience confusion at a certain point.
  - Studies on needle-in-a-haystack style benchmarking have uncovered the concept of context rot: as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases.
  - While some models exhibit more gentle degradation than others, this characteristic emerges across all models.
  - Context, therefore, must be treated as a finite resource with diminishing marginal returns.
  - Like humans, who have limited working memory capacity, LLMs have an "attention budget" that they draw on when parsing large volumes of context.
  - Every new token introduced depletes this budget by some amount, increasing the need to carefully curate the tokens available to the LLM.
  - This attention scarcity stems from architectural constraints of LLMs. LLMs are based on the transformer architecture, which enables every token to attend to every other token across the entire context. This results in n² pairwise relationships for n tokens.
  - As its context length increases, a model's ability to capture these pairwise relationships gets stretched thin, creating a natural tension between context size and attention focus.
  - Additionally, models develop their attention patterns from training data distributions where shorter sequences are typically more common than longer ones. This means models have less experience with, and fewer specialized parameters for, context-wide dependencies.
  - Techniques like position encoding interpolation allow models to handle longer sequences by adapting them to the originally trained smaller context, though with some degradation in token position understanding.
  - These factors create a performance gradient rather than a hard cliff: models remain highly capable at longer contexts but may show reduced precision for information retrieval and long-range reasoning compared to their performance on shorter contexts.
  - These realities mean that thoughtful context engineering is essential for building capable agents.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `transformer architecture`, `position encoding interpolation`, `context window`
* **Code Snippet:** None.

---

## The anatomy of effective context

* **Key Points:**
  - Given that LLMs are constrained by a finite attention budget, good context engineering means finding the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome.
  - System prompts should be extremely clear and use simple, direct language that presents ideas at the right altitude for the agent.
  - The right altitude is the Goldilocks zone between two common failure modes. At one extreme, we see engineers hardcoding complex, brittle logic in their prompts to elicit exact agentic behavior. This approach creates fragility and increases maintenance complexity over time. At the other extreme, engineers sometimes provide vague, high-level guidance that fails to give the LLM concrete signals for desired outputs or falsely assumes shared context. The optimal altitude strikes a balance: specific enough to guide behavior effectively, yet flexible enough to provide the model with strong heuristics to guide behavior.
  - We recommend organizing prompts into distinct sections (like <background_information>, <instructions>, ## Tool guidance, ## Output description, etc) and using techniques like XML tagging or Markdown headers to delineate these sections, although the exact formatting of prompts is likely becoming less important as models become more capable.
  - Regardless of how you decide to structure your system prompt, you should be striving for the minimal set of information that fully outlines your expected behavior. (Note that minimal does not necessarily mean short; you still need to give the agent sufficient information up front to ensure it adheres to the desired behavior.)
  - It's best to start by testing a minimal prompt with the best model available to see how it performs on your task, and then add clear instructions and examples to improve performance based on failure modes found during initial testing.
  - Tools allow agents to operate with their environment and pull in new, additional context as they work.
  - Because tools define the contract between agents and their information/action space, it's extremely important that tools promote efficiency, both by returning information that is token efficient and by encouraging efficient agent behaviors.
  - In Writing tools for AI agents – with AI agents, we discussed building tools that are well understood by LLMs and have minimal overlap in functionality.
  - Similar to the functions of a well-designed codebase, tools should be self-contained, robust to error, and extremely clear with respect to their intended use.
  - Input parameters should similarly be descriptive, unambiguous, and play to the inherent strengths of the model.
  - One of the most common failure modes we see is bloated tool sets that cover too much functionality or lead to ambiguous decision points about which tool to use.
  - If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better.
  - As we'll discuss later, curating a minimal viable set of tools for the agent can also lead to more reliable maintenance and pruning of context over long interactions.
  - Providing examples, otherwise known as few-shot prompting, is a well known best practice that we continue to strongly advise.
  - However, teams will often stuff a laundry list of edge cases into a prompt in an attempt to articulate every possible rule the LLM should follow for a particular task. We do not recommend this.
  - Instead, we recommend working to curate a set of diverse, canonical examples that effectively portray the expected behavior of the agent.
  - For an LLM, examples are the "pictures" worth a thousand words.
  - Our overall guidance across the different components of context (system prompts, tools, examples, message history, etc) is to be thoughtful and keep your context informative, yet tight.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `system prompts`, `XML tagging`, `Markdown headers`, `tools`, `few-shot prompting`
* **Code Snippet:** None.

---

## Context retrieval and agentic search

* **Key Points:**
  - In Building effective AI agents, we highlighted the differences between LLM-based workflows and agents.
  - Since we wrote that post, we've gravitated towards a simple definition for agents: LLMs autonomously using tools in a loop.
  - Working alongside our customers, we've seen the field converging on this simple paradigm.
  - As the underlying models become more capable, the level of autonomy of agents can scale: smarter models allow agents to independently navigate nuanced problem spaces and recover from errors.
  - We're now seeing a shift in how engineers think about designing context for agents.
  - Today, many AI-native applications employ some form of embedding-based pre-inference time retrieval to surface important context for the agent to reason over.
  - As the field transitions to more agentic approaches, we increasingly see teams augmenting these retrieval systems with "just in time" context strategies.
  - Rather than pre-processing all relevant data up front, agents built with the "just in time" approach maintain lightweight identifiers (file paths, stored queries, web links, etc.) and use these references to dynamically load data into context at runtime using tools.
  - Anthropic's agentic coding solution Claude Code uses this approach to perform complex data analysis over large databases.
  - The model can write targeted queries, store results, and leverage Bash commands like head and tail to analyze large volumes of data without ever loading the full data objects into context.
  - This approach mirrors human cognition: we generally don't memorize entire corpuses of information, but rather introduce external organization and indexing systems like file systems, inboxes, and bookmarks to retrieve relevant information on demand.
  - Beyond storage efficiency, the metadata of these references provides a mechanism to efficiently refine behavior, whether explicitly provided or intuitive.
  - To an agent operating in a file system, the presence of a file named test_utils.py in a tests folder implies a different purpose than a file with the same name located in src/core_logic/ Folder hierarchies, naming conventions, and timestamps all provide important signals that help both humans and agents understand how and when to utilize information.
  - Letting agents navigate and retrieve data autonomously also enables progressive disclosure—in other words, allows agents to incrementally discover relevant context through exploration.
  - Each interaction yields context that informs the next decision: file sizes suggest complexity; naming conventions hint at purpose; timestamps can be a proxy for relevance.
  - Agents can assemble understanding layer by layer, maintaining only what's necessary in working memory and leveraging note-taking strategies for additional persistence.
  - This self-managed context window keeps the agent focused on relevant subsets rather than drowning in exhaustive but potentially irrelevant information.
  - Of course, there's a trade-off: runtime exploration is slower than retrieving pre-computed data.
  - Not only that, but opinionated and thoughtful engineering is required to ensure that an LLM has the right tools and heuristics for effectively navigating its information landscape.
  - Without proper guidance, an agent can waste context by misusing tools, chasing dead-ends, or failing to identify key information.
  - In certain settings, the most effective agents might employ a hybrid strategy, retrieving some data up front for speed, and pursuing further autonomous exploration at its discretion.
  - The decision boundary for the 'right' level of autonomy depends on the task.
  - Claude Code is an agent that employs this hybrid model: CLAUDE.md files are naively dropped into context up front, while primitives like glob and grep allow it to navigate its environment and retrieve files just-in-time, effectively bypassing the issues of stale indexing and complex syntax trees.
  - The hybrid strategy might be better suited for contexts with less dynamic content, such as legal or finance work.
  - As model capabilities improve, agentic design will trend towards letting intelligent models act intelligently, with progressively less human curation.
  - Given the rapid pace of progress in the field, "do the simplest thing that works" will likely remain our best advice for teams building agents on top of Claude.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Claude Code`, `Bash`, `head`, `tail`, `glob`, `grep`, `CLAUDE.md`
* **Code Snippet:** None.

---

## Context engineering for long-horizon tasks

* **Key Points:**
  - Long-horizon tasks require agents to maintain coherence, context, and goal-directed behavior over sequences of actions where the token count exceeds the LLM's context window.
  - For tasks that span tens of minutes to multiple hours of continuous work, like large codebase migrations or comprehensive research projects, agents require specialized techniques to work around the context window size limitation.
  - Waiting for larger context windows might seem like an obvious tactic. But it's likely that for the foreseeable future, context windows of all sizes will be subject to context pollution and information relevance concerns—at least for situations where the strongest agent performance is desired.
  - To enable agents to work effectively across extended time horizons, we've developed a few techniques that address these context pollution constraints directly: compaction, structured note-taking, and multi-agent architectures.
  - Compaction is the practice of taking a conversation nearing the context window limit, summarizing its contents, and reinitiating a new context window with the summary.
  - Compaction typically serves as the first lever in context engineering to drive better long-term coherence.
  - At its core, compaction distills the contents of a context window in a high-fidelity manner, enabling the agent to continue with minimal performance degradation.
  - In Claude Code, for example, we implement this by passing the message history to the model to summarize and compress the most critical details. The model preserves architectural decisions, unresolved bugs, and implementation details while discarding redundant tool outputs or messages. The agent can then continue with this compressed context plus the five most recently accessed files. Users get continuity without worrying about context window limitations.
  - The art of compaction lies in the selection of what to keep versus what to discard, as overly aggressive compaction can result in the loss of subtle but critical context whose importance only becomes apparent later.
  - For engineers implementing compaction systems, we recommend carefully tuning your prompt on complex agent traces. Start by maximizing recall to ensure your compaction prompt captures every relevant piece of information from the trace, then iterate to improve precision by eliminating superfluous content.
  - An example of low-hanging superfluous content is clearing tool calls and results – once a tool has been called deep in the message history, why would the agent need to see the raw result again? One of the safest lightest touch forms of compaction is tool result clearing, most recently launched as a feature on the Claude Developer Platform.
  - Structured note-taking, or agentic memory, is a technique where the agent regularly writes notes persisted to memory outside of the context window. These notes get pulled back into the context window at later times.
  - This strategy provides persistent memory with minimal overhead. Like Claude Code creating a to-do list, or your custom agent maintaining a NOTES.md file, this simple pattern allows the agent to track progress across complex tasks, maintaining critical context and dependencies that would otherwise be lost across dozens of tool calls.
  - Claude playing Pokémon demonstrates how memory transforms agent capabilities in non-coding domains. The agent maintains precise tallies across thousands of game steps—tracking objectives like "for the last 1,234 steps I've been training my Pokémon in Route 1, Pikachu has gained 8 levels toward the target of 10." Without any prompting about memory structure, it develops maps of explored regions, remembers which key achievements it has unlocked, and maintains strategic notes of combat strategies that help it learn which attacks work best against different opponents.
  - After context resets, the agent reads its own notes and continues multi-hour training sequences or dungeon explorations. This coherence across summarization steps enables long-horizon strategies that would be impossible when keeping all the information in the LLM's context window alone.
  - As part of our Sonnet 4.5 launch, we released a memory tool in public beta on the Claude Developer Platform that makes it easier to store and consult information outside the context window through a file-based system. This allows agents to build up knowledge bases over time, maintain project state across sessions, and reference previous work without keeping everything in context.
  - Sub-agent architectures provide another way around context limitations. Rather than one agent attempting to maintain state across an entire project, specialized sub-agents can handle focused tasks with clean context windows. The main agent coordinates with a high-level plan while subagents perform deep technical work or use tools to find relevant information. Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens).
  - This approach achieves a clear separation of concerns—the detailed search context remains isolated within sub-agents, while the lead agent focuses on synthesizing and analyzing the results.
  - This pattern, discussed in How we built our multi-agent research system, showed a substantial improvement over single-agent systems on complex research tasks.
  - The choice between these approaches depends on task characteristics. For example: Compaction maintains conversational flow for tasks requiring extensive back-and-forth; Note-taking excels for iterative development with clear milestones; Multi-agent architectures handle complex research and analysis where parallel exploration pays dividends.
  - Even as models continue to improve, the challenge of maintaining coherence across extended interactions will remain central to building more effective agents.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Claude Code`, `Claude Developer Platform`, `Sonnet 4.5`, `memory tool`, `NOTES.md`, `sub-agent`
* **Code Snippet:** None.

---

## Conclusion

* **Key Points:**
  - Context engineering represents a fundamental shift in how we build with LLMs.
  - As models become more capable, the challenge isn't just crafting the perfect prompt—it's thoughtfully curating what information enters the model's limited attention budget at each step.
  - Whether you're implementing compaction for long-horizon tasks, designing token-efficient tools, or enabling agents to explore their environment just-in-time, the guiding principle remains the same: find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome.
  - The techniques we've outlined will continue evolving as models improve.
  - We're already seeing that smarter models require less prescriptive engineering, allowing agents to operate with more autonomy.
  - But even as capabilities scale, treating context as a precious, finite resource will remain central to building reliable, effective agents.
  - Get started with context engineering in the Claude Developer Platform today, and access helpful tips and best practices via our memory and context management cookbook.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Claude Developer Platform`, `memory`, `context management cookbook`
* **Code Snippet:** None.



---

# Effective harnesses for long-running agents

## Effective harnesses for long-running agents

* **Key Points:**
  - Agents still face challenges working across many context windows.
  - We looked to human engineers for inspiration in creating a more effective harness for long-running agents.
  - As AI agents become more capable, developers are increasingly asking them to take on complex tasks requiring work that spans hours, or even days.
  - However, getting agents to make consistent progress across multiple context windows remains an open problem.
  - The core challenge of long-running agents is that they must work in discrete sessions, and each new session begins with no memory of what came before.
  - Imagine a software project staffed by engineers working in shifts, where each new engineer arrives with no memory of what happened on the previous shift.
  - Because context windows are limited, and because most complex projects cannot be completed within a single window, agents need a way to bridge the gap between coding sessions.
  - We developed a two-fold solution to enable the Claude Agent SDK to work effectively across many context windows: an initializer agent that sets up the environment on the first run, and a coding agent that is tasked with making incremental progress in every session, while leaving clear artifacts for the next session.
* **Technical Entities (Classes/Functions/APIs):** `Claude Agent SDK`, `initializer agent`, `coding agent`, `context windows`
* **Code Snippet:** None.

---

## The long-running agent problem

* **Key Points:**
  - The Claude Agent SDK is a powerful, general-purpose agent harness adept at coding, as well as other tasks that require the model to use tools to gather context, plan, and execute.
  - It has context management capabilities such as compaction, which enables an agent to work on a task without exhausting the context window.
  - Theoretically, given this setup, it should be possible for an agent to continue to do useful work for an arbitrarily long time.
  - However, compaction isn't sufficient.
  - Out of the box, even a frontier coding model like Opus 4.5 running on the Claude Agent SDK in a loop across multiple context windows will fall short of building a production-quality web app if it's only given a high-level prompt, such as "build a clone of claude.ai."
  - Claude's failures manifested in two patterns. First, the agent tended to try to do too much at once—essentially to attempt to one-shot the app. Often, this led to the model running out of context in the middle of its implementation, leaving the next session to start with a feature half-implemented and undocumented. The agent would then have to guess at what had happened, and spend substantial time trying to get the basic app working again. This happens even with compaction, which doesn't always pass perfectly clear instructions to the next agent.
  - A second failure mode would often occur later in a project. After some features had already been built, a later agent instance would look around, see that progress had been made, and declare the job done.
  - This decomposes the problem into two parts. First, we need to set up an initial environment that lays the foundation for all the features that a given prompt requires, which sets up the agent to work step-by-step and feature-by-feature. Second, we should prompt each agent to make incremental progress towards its goal while also leaving the environment in a clean state at the end of a session.
  - By "clean state" we mean the kind of code that would be appropriate for merging to a main branch: there are no major bugs, the code is orderly and well-documented, and in general, a developer could easily begin work on a new feature without first having to clean up an unrelated mess.
  - When experimenting internally, we addressed these problems using a two-part solution:
  - Initializer agent: The very first agent session uses a specialized prompt that asks the model to set up the initial environment: an init.sh script, a claude-progress.txt file that keeps a log of what agents have done, and an initial git commit that shows what files were added.
  - Coding agent: Every subsequent session asks the model to make incremental progress, then leave structured updates.
  - The key insight here was finding a way for agents to quickly understand the state of work when starting with a fresh context window, which is accomplished with the claude-progress.txt file alongside the git history. Inspiration for these practices came from knowing what effective software engineers do every day.
* **Technical Entities (Classes/Functions/APIs):** `Claude Agent SDK`, `compaction`, `Opus 4.5`, `initializer agent`, `coding agent`, `init.sh`, `claude-progress.txt`, `git`
* **Code Snippet:** None.

---

## Environment management

* **Key Points:**
  - In the updated Claude 4 prompting guide, we shared some best practices for multi-context window workflows, including a harness structure that uses "a different prompt for the very first context window."
  - This "different prompt" requests that the initializer agent set up the environment with all the necessary context that future coding agents will need to work effectively.
* **Technical Entities (Classes/Functions/APIs):** `Claude 4`, `initializer agent`, `coding agents`
* **Code Snippet:** None.

---

## Feature list

* **Key Points:**
  - To address the problem of the agent one-shotting an app or prematurely considering the project complete, we prompted the initializer agent to write a comprehensive file of feature requirements expanding on the user's initial prompt.
  - In the claude.ai clone example, this meant over 200 features, such as "a user can open a new chat, type in a query, press enter, and see an AI response."
  - These features were all initially marked as "failing" so that later coding agents would have a clear outline of what full functionality looked like.
  - We prompt coding agents to edit this file only by changing the status of a passes field, and we use strongly-worded instructions like "It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality."
  - After some experimentation, we landed on using JSON for this, as the model is less likely to inappropriately change or overwrite JSON files compared to Markdown files.
* **Technical Entities (Classes/Functions/APIs):** `initializer agent`, `coding agents`, `JSON`, `Markdown`
* **Code Snippet:**
```json
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": [
      "Navigate to main interface",
      "Click the 'New Chat' button",
      "Verify a new conversation is created",
      "Check that chat area shows welcome state",
      "Verify conversation appears in sidebar"
    ],
    "passes": false
  }
```

---

## Incremental progress

* **Key Points:**
  - Given this initial environment scaffolding, the next iteration of the coding agent was then asked to work on only one feature at a time.
  - This incremental approach turned out to be critical to addressing the agent's tendency to do too much at once.
  - Once working incrementally, it's still essential that the model leaves the environment in a clean state after making a code change.
  - In our experiments, we found that the best way to elicit this behavior was to ask the model to commit its progress to git with descriptive commit messages and to write summaries of its progress in a progress file.
  - This allowed the model to use git to revert bad code changes and recover working states of the code base.
  - These approaches also increased efficiency, as they eliminated the need for an agent to have to guess at what had happened and spend its time trying to get the basic app working again.
* **Technical Entities (Classes/Functions/APIs):** `coding agent`, `git`
* **Code Snippet:** None.

---

## Testing

* **Key Points:**
  - One final major failure mode that we observed was Claude's tendency to mark a feature as complete without proper testing.
  - Absent explicit prompting, Claude tended to make code changes, and even do testing with unit tests or curl commands against a development server, but would fail recognize that the feature didn't work end-to-end.
  - In the case of building a web app, Claude mostly did well at verifying features end-to-end once explicitly prompted to use browser automation tools and do all testing as a human user would.
  - Providing Claude with these kinds of testing tools dramatically improved performance, as the agent was able to identify and fix bugs that weren't obvious from the code alone.
  - Some issues remain, like limitations to Claude's vision and to browser automation tools making it difficult to identify every kind of bug.
  - For example, Claude can't see browser-native alert modals through the Puppeteer MCP, and features relying on these modals tended to be buggier as a result.
* **Technical Entities (Classes/Functions/APIs):** `Claude`, `curl`, `Puppeteer MCP`
* **Code Snippet:** None.

---

## Getting up to speed

* **Key Points:**
  - With all of the above in place, every coding agent is prompted to run through a series of steps to get its bearings, some quite basic but still helpful:
  - Run pwd to see the directory you're working in. You'll only be able to edit files in this directory.
  - Read the git logs and progress files to get up to speed on what was recently worked on.
  - Read the features list file and choose the highest-priority feature that's not yet done to work on.
  - This approach saves Claude some tokens in every session since it doesn't have to figure out how to test the code.
  - It also helps to ask the initializer agent to write an init.sh script that can run the development server, and then run through a basic end-to-end test before implementing a new feature.
  - In the case of the claude.ai clone, this meant that the agent always started the local development server and used the Puppeteer MCP to start a new chat, send a message, and receive a response.
  - This ensured that Claude could quickly identify if the app had been left in a broken state, and immediately fix any existing bugs.
  - If the agent had instead started implementing a new feature, it would likely make the problem worse.
* **Technical Entities (Classes/Functions/APIs):** `coding agent`, `pwd`, `git`, `init.sh`, `Puppeteer MCP`, `Claude`
* **Code Snippet:**
```
[Assistant] I'll start by getting my bearings and understanding the current state of the project.
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Let me check the git log to see recent work.
[Tool Use] <bash - git log --oneline -20>
[Assistant] Now let me check if there's an init.sh script to restart the servers.
<Starts the development server>
[Assistant] Excellent! Now let me navigate to the application and verify that some fundamental features are still working.
<Tests basic functionality>
[Assistant] Based on my verification testing, I can see that the fundamental functionality is working well. The core chat features, theme switching, conversation loading, and error handling are all functioning correctly. Now let me review the tests.json file more comprehensively to understand what needs to be implemented next.
<Starts work on a new feature>
```

---

## Agent failure modes and solutions

* **Key Points:**
  - Problem: Claude declares victory on the entire project too early. Initializer Agent Behavior: Set up a feature list file: based on the input spec, set up a structured JSON file with a list of end-to-end feature descriptions. Coding Agent Behavior: Read the feature list file at the beginning of a session. Choose a single feature to start working on.
  - Problem: Claude leaves the environment in a state with bugs or undocumented progress. Initializer Agent Behavior: An initial git repo and progress notes file is written. Coding Agent Behavior: Start the session by reading the progress notes file and git commit logs, and run a basic test on the development server to catch any undocumented bugs. End the session by writing a git commit and progress update.
  - Problem: Claude marks features as done prematurely. Initializer Agent Behavior: Set up a feature list file. Coding Agent Behavior: Self-verify all features. Only mark features as "passing" after careful testing.
  - Problem: Claude has to spend time figuring out how to run the app. Initializer Agent Behavior: Write an init.sh script that can run the development server. Coding Agent Behavior: Start the session by reading init.sh.
* **Technical Entities (Classes/Functions/APIs):** `Claude`, `initializer agent`, `coding agent`, `JSON`, `git`, `init.sh`
* **Code Snippet:** None.

---

## Future work

* **Key Points:**
  - This research demonstrates one possible set of solutions in a long-running agent harness to enable the model to make incremental progress across many context windows.
  - However, there remain open questions.
  - Most notably, it's still unclear whether a single, general-purpose coding agent performs best across contexts, or if better performance can be achieved through a multi-agent architecture.
  - It seems reasonable that specialized agents like a testing agent, a quality assurance agent, or a code cleanup agent, could do an even better job at sub-tasks across the software development lifecycle.
  - Additionally, this demo is optimized for full-stack web app development. A future direction is to generalize these findings to other fields.
  - It's likely that some or all of these lessons can be applied to the types of long-running agentic tasks required in, for example, scientific research or financial modeling.
* **Technical Entities (Classes/Functions/APIs):** `multi-agent architecture`, `testing agent`, `quality assurance agent`, `code cleanup agent`
* **Code Snippet:** None.
