---
aliases:
  - Self Reflection
highlights: Agent evaluates its own outputs for quality, correctness and goal alignment; triggers retry or alternative strategies on failures
Source 1: https://huggingface.co/blog/Kseniase/reflection
---
# How Do Agents Learn from Their Own Mistakes? The Role of Reflection in AI

## How Do Agents Learn from Their Own Mistakes? The Role of Reflection in AI

* **Key Points:**
  - Last week, we explored how reasoning and planning work together to make AI more effective – allowing models not just to think, but to structure their thoughts into goal-driven actions. Yet, even with strong reasoning and planning, AI still lacks something essential: the ability to learn from its own decisions.
  - That's where Reflection comes in. More than just thinking ahead, agentic AI needs to analyze past actions, recognize mistakes, and refine its strategies – just like humans do (sometimes). Without reflection, AI risks repeating errors instead of improving.
  - Andrew Ng sees Reflection as a key design pattern for agentic AI, enabling models to critique and refine their own outputs for better decision-making.
  - Today, we dive into Reflection as a core building block of agentic AI, exploring how frameworks like Reflexion and ReAct (and others) enable self-assessment and iterative learning. As AI moves toward greater autonomy, self-reflection is emerging as a critical capability – closing the loop between action and continuous learning.
* **Technical Entities (Classes/Functions/APIs):** `Reflection`, `Reflexion`, `ReAct`
* **Code Snippet:** None.

---

## Philosophical Roots

* **Key Points:**
  - The ability to reflect – to analyze one's own thoughts and actions – has long been recognized as fundamental to intelligence. Socrates championed the practice of questioning one's own beliefs, arguing that only through introspection can we separate sound reasoning from flawed assumptions.
  - Ancient Eastern philosophy echoed this idea, with Confucius placing reflection above both imitation and experience as the noblest path to wisdom. Throughout history, reflection has been seen as the mechanism that sharpens judgment, refines decision-making, and fuels personal and intellectual growth.
  - From the Stoics, who practiced nightly self-examination, to Descartes, who probed the very nature of thought itself, reflection has been a core theme in philosophy. Thinkers like Aristotle and Kant drew distinctions between contemplation and action, recognizing that purposeful deliberation is key to meaningful decision-making.
  - In more recent times, John Dewey described reflective thought as the careful and persistent evaluation of beliefs in light of evidence, enabling individuals to act with foresight rather than impulse.
  - Donald Schön later expanded on this by distinguishing between reflection-in-action – adjusting and adapting in real time – and reflection-on-action, where individuals analyze past decisions to improve future ones. This idea reinforced the notion that expert decision-making isn't just about planning but also about dynamically evaluating and refining actions as they unfold.
  - These concepts have profoundly influenced fields from cognitive science to education, shaping our understanding of how learning, reasoning, and action must work together to drive true intelligence. Today, these ideas are finding their way into AI.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Understanding Reflection in AI

* **Key Points:**
  - In the context of AI agents, reflection refers to an agent's ability to think about its own actions and results in order to self-correct and improve. It is essentially the AI analog of human introspection or "System 2" deliberative thinking. Instead of merely reacting instinctively (like a fast, heuristic System 1 response), a reflective AI will pause to analyze what it has done, identify errors or suboptimal steps, and adjust its strategy. (the concepts of System 1 and System 2 thinking were proposed by Daniel Kahneman in his book "Thinking, Fast and Slow"). This mechanism allows an AI agent to learn from its experiences without new external training data, by internally evaluating feedback. Through self-reflection, an agent can correct previous mistakes and generate improved solutions on the next attempt, embodying a form of self-improvement loop.
  - AI pioneer Andrew Ng sees Reflection as a core component of agentic AI, alongside Planning, Tool Use, and Multi-agent Collaboration – the building blocks that could define the next wave of AI progress. Rather than just generating answers, reflective AI models critique and refine their own outputs, identifying flaws, making improvements, and iterating until they reach a stronger result.
  - This self-review loop – generate → critique → improve – has already shown major performance gains across tasks like coding, writing, and question-answering. Ng emphasizes that Reflection enables AI to self-correct without always needing external feedback, making systems more reliable and autonomous.
  - Reflection in AI does not stand alone – it interplays with other core components of an agentic system. As we wrote before, an autonomous agent typically has several building blocks working in concert. First is Profiling, where the agent is given a role or objective that frames its behavior (defining its identity, goals, and constraints). Next comes Knowledge, which provides the agent's base information or access to facts (such as a knowledge base or pretrained model of the world). The agent also maintains Memory – storing context from past interactions or steps (both short-term, like the current conversation or trajectory, and long-term, like accumulated experience). With knowledge and memory, the agent engages in Reasoning and Planning, breaking down tasks, making inferences, and formulating an action plan. It then takes Actions to execute the plan (e.g. calling tools, producing outputs, or moving in an environment). Finally comes Reflection: the agent evaluates the outcomes of those actions against the goals, learning from any mistakes or unexpected results. This reflective step "evaluates outcomes to influence future reasoning and behavior," closing the feedback loop in the agent's workflow.
  - Crucially, reflection allows the agent to adjust itself dynamically. In a well-designed agentic system, this means the operational flow is cyclic: profiling defines the role, knowledge provides context, memory and reasoning guide an action, the agent acts, and then reflection assesses how that action went. Any lessons learned (e.g. an action failed or a reasoning path led nowhere) are fed back into the agent's memory or planning module to inform the next cycle. Over time, this leads to better performance as the agent accumulates reflective insights. This idea of continuous self-improvement via reflection has strong theoretical appeal – it's essentially a form of on-the-fly adaptation. Notably, it does not require retraining the model's weights each time; instead, the learning happens at the knowledge and planning level through natural language or symbolic feedback. Researchers have even likened this to a kind of "verbal reinforcement learning" for language agents, where the model reinforces good behaviors through linguistic feedback rather than gradient updates.
  - In summary, reflection in AI provides a mechanism for an agent to meta-reason about its own reasoning, enabling a higher level of autonomy.
* **Technical Entities (Classes/Functions/APIs):** `System 1`, `System 2`, `Profiling`, `Knowledge`, `Memory`, `Reasoning and Planning`, `Actions`, `Reflection`
* **Code Snippet:** None.

---

## Implementation Details: Reflexion and ReAct Frameworks

* **Key Points:**
  - To make these ideas concrete, several AI research works have implemented reflection mechanisms for AI agents. Two notable frameworks are Reflexion and ReAct, which integrate reflection and reasoning into large language model (LLM) agents in different ways. Below, we explore how each works and how they impact an agent's decision-making.
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Reflexion`, `ReAct`
* **Code Snippet:** None.

---

## Reflexion: Learning via Self-Feedback

* **Key Points:**
  - The Reflexion framework (Shinn et al., 2023) was explicitly designed to embed a reflection loop into LLM-based agents. Instead of fine-tuning the model with reinforcement learning, Reflexion keeps the model frozen and uses text-based feedback as a form of reinforcement.
  - In a typical Reflexion agent, the LLM acts as an Actor that attempts a task (for example, writing code to solve a problem or taking steps in a game environment). The outcome of that attempt (including any feedback from the environment, like errors or success signals) is then fed into a Self-Reflection prompt, where the model is asked to critique its recent attempt and suggest improvements. The key idea is that the agent generates a textual "reflection" on what went wrong or could be done better. This reflection is stored in the agent's memory and provided as additional context for the next try. In effect, the agent carries along a growing log of lessons learned (in plain language) that biases its next action toward better outcomes.
  - By iterating this process, the Reflexion agent learns from trial and error rapidly within a single session. For example, if the agent's first attempt at a task fails, the reflection step might note: "I got stuck in a loop, perhaps try a different strategy or tool next time." The next run, conditioned on this advice, is more successful. Notably, Reflexion can incorporate various types of feedback signals: a numeric reward, an error message, or a human hint can all be converted into the reflection prompt to guide the agent. This flexibility makes it a general approach to on-the-fly improvement. In experiments, Reflexion achieved impressive gains. On a coding benchmark (HumanEval), a Reflexion-augmented GPT-4 agent reached 91% success, compared to 80% by GPT-4 without reflection – effectively surpassing the prior state-of-the-art on that task. Likewise, on decision-making tasks in simulated environments, Reflexion agents dramatically outperformed their non-reflective counterparts, solving 130 out of 134 challenges in the AlfWorld environment when using self-evaluation feedback. These results show that giving AI agents a structured way to "think about what they did" and remember those insights can yield state-of-the-art performance in complex tasks.
* **Technical Entities (Classes/Functions/APIs):** `Reflexion`, `LLM`, `Actor`, `Self-Reflection`, `GPT-4`, `HumanEval`, `AlfWorld`
* **Code Snippet:** None.

---

## ReAct: Interleaving Reasoning and Acting

* **Key Points:**
  - While Reflexion focuses on learning from trial outcomes, ReAct (Yao et al., 2023) is a framework that tightly couples reasoning steps with action steps in a single loop. The name "ReAct" reflects "Reason + Act": the agent is prompted to alternate between thinking (Chain-of-Thought style reasoning) and acting (executing an API call or environment action). Traditionally, an AI either reasons entirely in its hidden state (as with chain-of-thought prompting) or is forced to act step-by-step without much internal reflection. ReAct instead lets the model generate explicit reasoning traces and take actions in an interleaved manner.
  - The Thought entries are the model's natural language reflections or calculations, and the Actions are commands from a predefined toolkit (search, calculator, etc.) or moves in an environment. By designing the prompt this way, ReAct achieves a tight integration of reflection and action at each step of a task. The reasoning traces help the model maintain coherence and avoid going off-track, while the actions let it query external resources or change the environment based on those thoughts.
  - This synergy proved powerful. For instance, on knowledge-intensive question answering tasks like HotpotQA, ReAct agents were able to avoid hallucinations by using the reasoning steps to decide when to call a Wikipedia search tool for evidence. The result was more factual, reliable answers. On interactive decision-making benchmarks (like a virtual home environment ALFWorld and a web shopping task WebShop), ReAct significantly outperformed agents that used only actions or only chain-of-thought, improving success rates by +34% and +10% respectively over prior methods. Moreover, because ReAct forces the model to spell out its thought process, the resulting solution trajectories were more interpretable to humans (we can read the agent's chain of thought) and easier to trust. ReAct thus exemplifies how built-in reflection (reasoning) can guide actions in real time, leading to smarter and more transparent agent behavior. Today, the ReAct prompting pattern (Thought→Action→Observation loops) has been adopted in many LLM agent implementations, often serving as the backbone for tools like AutoGPT and LangChain's agents.
* **Technical Entities (Classes/Functions/APIs):** `ReAct`, `Chain-of-Thought`, `API`, `HotpotQA`, `ALFWorld`, `WebShop`, `AutoGPT`, `LangChain`
* **Code Snippet:**
```vbnet
Module ReflectionAgent
    Sub Main()
        ' Thought 1: The agent reasons about what to do next
        Dim thought1 As String = "The agent reasons about what to do next"
        Console.WriteLine("Thought 1: " & thought1)

        ' Action 1: The agent acts (e.g., “Search(query)”)
        Dim action1 As String = "Search(query)"
        Console.WriteLine("Action 1: " & action1)

        ' Observation 1: The agent sees result of the action
        Dim observation1 As String = "The agent sees result of the action"
        Console.WriteLine("Observation 1: " & observation1)

        ' Thought 2: Agent reflects on the observation
        Dim thought2 As String = "Agent reflects on the observation"
        Console.WriteLine("Thought 2: " & thought2)

        ' Action 2: Next action
        Dim action2 As String = "Next action"
        Console.WriteLine("Action 2: " & action2)

        ' ... (and so on)
    End Sub
End Module
```

---

## Impact on Decision-Making

* **Key Points:**
  - Both Reflexion and ReAct demonstrate that adding reflection can greatly enhance an agent's decision-making. Reflexion shows the value of post-hoc reflection – after completing a trial, reflect and try again – to gradually converge on a correct solution. ReAct shows the value of online reflection – intermixing thinking steps within the action sequence – to make better decisions on the fly. These are not mutually exclusive; in fact, Reflexion's original paper uses ReAct as the base Actor policy within its framework. An agent can reason during each step (ReAct style) and also reflect after a full attempt (Reflexion style). The common theme is that explicit reflective reasoning reduces errors and improves performance across a variety of tasks. By integrating reflection, agents can handle more complex problems with less hallucination and more robustness than purely reactive agents.
* **Technical Entities (Classes/Functions/APIs):** `Reflexion`, `ReAct`, `Actor`
* **Code Snippet:** None.

---

## Other Reflection-Based Approaches and Innovations

* **Key Points:**
  - Beyond Reflexion and ReAct, AI research is developing new ways to integrate reflection for continuous self-improvement.
  - Self-Refine enables AI to iteratively critique and improve its own outputs, acting as both writer and editor. By refining responses through multiple feedback loops, models achieve better accuracy and coherence – all without additional training. Even top-tier models like GPT-4 have shown significant quality gains when prompted to reflect.
  - Chain-of-Hindsight (CoH) shifts reflection into training rather than inference, allowing models to learn from their past mistakes by seeing where previous outputs went wrong. This method builds a model that internalizes reflective reasoning, reducing the likelihood of repeating errors.
  - Tree-of-Thoughts and Search-based Reflection enhance decision-making by having AI explore multiple reasoning paths, evaluate different approaches, and select the best one. Instead of committing to a single line of reasoning, the model dynamically backtracks and corrects itself, mimicking human problem-solving strategies.
  - Multi-Agent Reflection brings collaboration into the equation, where one AI generates, another critiques, and together they refine outputs. This approach has been successful in coding, writing, and strategic decision-making, proving that peer review isn't just for humans – AI agents can also benefit from externalized reflection.
  - Constitutional AI, introduced by Anthropic, takes reflection a step further by making models assess their responses against ethical and factual principles. Instead of relying solely on human feedback, AI evaluates whether its outputs align with predefined guidelines, reducing harmful or biased responses.
  - Each of these approaches builds on the same principle: AI must not just reason, but also reflect on its reasoning. Whether through self-evaluation, training-based learning, or multi-agent collaboration, reflection is becoming a cornerstone of AI adaptability, pushing systems toward greater autonomy, accuracy, and reliability.
* **Technical Entities (Classes/Functions/APIs):** `Self-Refine`, `GPT-4`, `Chain-of-Hindsight (CoH)`, `Tree-of-Thoughts`, `Multi-Agent Reflection`, `Constitutional AI`, `Anthropic`
* **Code Snippet:** None.

---

## Future Directions and Emerging Trends

* **Key Points:**
  - Reflection in AI is evolving rapidly, with researchers pushing toward more autonomous, reliable, and adaptive systems. Here's where things are headed:
  - Long-Term Memory & Lifelong Learning – Right now, AI reflection is short-lived, often resetting between sessions. Future AI will retain past reflections, building persistent memory to refine decision-making over time.
  - Scaling Reflection with More Context & Multimodal Inputs – As models handle longer context windows and integrate multimodal data (text, images, sensor inputs), reflection will expand beyond language and become more dynamic and adaptive. Today, AI mostly critiques text-based outputs, but future systems could analyze video of failed actions, evaluate physical-world changes, or refine visual outputs. Imagine a robot watching its own failed grasp attempt and adjusting, or an AI correcting an image where it mistakenly drew six fingers instead of five. Multimodal reflection enables AI to self-correct across different data types, making it essential for embodied systems and real-world applications, where text alone isn't enough to capture errors or guide improvement
  - Higher-Quality Self-Reflection – Not all reflections are useful. AI needs better self-assessment mechanisms, possibly using multiple models (one generating, another evaluating) or external tools for validation. Constitutional AI is also advancing, where models reflect against ethical and factual standards to refine responses before outputting them.
  - Efficiency & Smarter Triggers – Reflection comes at a computational cost, but future AI will trigger reflection only when needed – when uncertain, when detecting a potential error, or when refining a complex answer. This will balance quality and speed, making reflection an on-demand capability rather than a default step.
  - Integration with Learning & Adaptation – Over time, reflection and training will merge. AI might log its reflections as training data, using them to improve performance permanently, not just in-session. Some researchers – like with Gödel Agent – even explore AI that self-modifies its reasoning processes, nudging toward meta-learning and self-improving code.
  - Human-AI Collaboration via Reflection – AI that reflects can explain its reasoning, improving transparency and trust. Imagine an AI assistant that not only gives an answer but also tells you why it's confident – or what it's unsure about. This is already taking shape in medical AI and decision-support systems, where reflective AI can highlight alternative possibilities and guide human experts.
  - In short, AI reflection is moving toward persistence, scalability, and deeper integration with reasoning and learning. The more AI can think about its thinking, the more adaptable and intelligent it becomes – pushing us closer to AI that not only executes tasks, but actively learns and improves from every decision it makes.
* **Technical Entities (Classes/Functions/APIs):** `Gödel Agent`, `Constitutional AI`
* **Code Snippet:** None.

---

## Concluding Thoughts

* **Key Points:**
  - Reflection is broadly applicable. Whether the domain is code, games, knowledge work, or creative writing, the ability for an AI to look back and learn improves performance. It's not just about squeezing out a better score on a benchmark – it translates to more reliable and effective AI in real-world deployments. An AI customer service agent that reflects might handle unusual queries better by recalling similar past cases from memory. An AI tutor that reflects on a student's progress could adapt its teaching strategy more intelligently. We're beginning to see real-world systems incorporate these ideas. As tools and libraries (like LangChain and AutoGen) make it easier to add reflection loops, we can expect to see many more practical applications taking advantage of this strategy.
  - In the next episode, we continue our Agentic series and explore Action and Tools. Reflection and Action are two sides of the same coin in agentic AI. Action provides experience; reflection ensures learning. Every action taken feeds data into reflection, which then refines future decisions. Without reflection, AI risks repeating mistakes; without action, reflection has nothing to analyze. What tools are there for that? Stay tuned.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `AutoGen`
* **Code Snippet:** None.