---
aliases:
  - Guardrails Layer
highlights: Pre/Post procesing filters validating inputs, outputs and tool calls against security policies and business rules
Source 1: https://docs.langchain.com/oss/javascript/langchain/guardrails
Source: https://www.oreilly.com/radar/ai-agents-need-guardrails/
---
# AI Agents Need Guardrails

* **Key Points:**
  - When AI systems were just a single model behind an API, life felt simpler. You trained, deployed, and maybe fine-tuned a few hyperparameters.
  - But that world's gone. Today, AI feels less like a single engine and more like a busy city—a network of small, specialized agents constantly talking to each other, calling APIs, automating workflows, and making decisions faster than humans can even follow.
  - And here's the real challenge: The smarter and more independent these agents get, the harder it becomes to stay in control. Performance isn't what slows us down anymore. Governance is.
  - How do we make sure these agents act ethically, safely, and within policy? How do we log what happened when multiple agents collaborate? How do we trace who decided what in an AI-driven workflow that touches user data, APIs, and financial transactions?
  - That's where the idea of engineering governance into the stack comes in. Instead of treating governance as paperwork at the end of a project, we can build it into the architecture itself.
* **Technical Entities (Classes/Functions/APIs):** `API`, `APIs`
* **Code Snippet:** None.

---

## From Model Pipelines to Agent Ecosystems

* **Key Points:**
  - In the old days of machine learning, things were pretty linear. You had a clear pipeline: collect data, train the model, validate it, deploy, monitor. Each stage had its tools and dashboards, and everyone knew where to look when something broke.
  - But with AI agents, that neat pipeline turns into a web. A single customer-service agent might call a summarization agent, which then asks a retrieval agent for context, which in turn queries an internal API—all happening asynchronously, sometimes across different systems.
  - It's less like a pipeline now and more like a network of tiny brains, all thinking and talking at once. And that changes how we debug, audit, and govern. When an agent accidentally sends confidential data to the wrong API, you can't just check one log file anymore. You need to trace the whole story: which agent called which, what data moved where, and why each decision was made. In other words, you need full lineage, context, and intent tracing across the entire ecosystem.
* **Technical Entities (Classes/Functions/APIs):** `API`
* **Code Snippet:** None.

---

## Why Governance Is the Missing Layer

* **Key Points:**
  - Governance in AI isn't new. We already have frameworks like NIST's AI Risk Management Framework (AI RMF) and the EU AI Act defining principles like transparency, fairness, and accountability. The problem is these frameworks often stay at the policy level, while engineers work at the pipeline level. The two worlds rarely meet. In practice, that means teams might comply on paper but have no real mechanism for enforcement inside their systems.
  - What we really need is a bridge—a way to turn those high-level principles into something that runs alongside the code, testing and verifying behavior in real time. Governance shouldn't be another checklist or approval form; it should be a runtime layer that sits next to your AI agents—ensuring every action follows approved paths, every dataset stays where it belongs, and every decision can be traced when something goes wrong.
* **Technical Entities (Classes/Functions/APIs):** `NIST AI RMF`, `EU AI Act`
* **Code Snippet:** None.

---

## The Four Guardrails of Agent Governance

* **Key Points:**
  - Policy as code
  - Observability and auditability
  - Dynamic risk scoring
  - Regulatory mapping
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Policy as code

* **Key Points:**
  - Policies shouldn't live in forgotten PDFs or static policy docs. They should live next to your code. By using tools like the Open Policy Agent (OPA), you can turn rules into version-controlled code that's reviewable, testable, and enforceable. Think of it like writing infrastructure as code, but for ethics and compliance. You can define rules such as: Which agents can access sensitive datasets; Which API calls require human review; When a workflow needs to stop because the risk feels too high
  - This way, developers and compliance folks stop talking past each other—they work in the same repo, speaking the same language.
  - And the best part? You can spin up a Dockerized OPA instance right next to your AI agents inside your Kubernetes cluster. It just sits there quietly, watching requests, checking rules, and blocking anything risky before it hits your APIs or data stores.
  - Governance stops being some scary afterthought. It becomes just another microservice. Scalable. Observable. Testable. Like everything else that matters.
* **Technical Entities (Classes/Functions/APIs):** `Open Policy Agent (OPA)`, `Docker`, `Kubernetes`, `APIs`
* **Code Snippet:** None.

---

## Observability and auditability

* **Key Points:**
  - Agents need to be observable not just in performance terms (latency, errors) but in decision terms. When an agent chain executes, we should be able to answer: Who initiated the action? What tools were used? What data was accessed? What output was generated?
  - Modern observability stacks—Cloud Logging, OpenTelemetry, Prometheus, or Grafana Loki—can already capture structured logs and traces. What's missing is semantic context: linking actions to intent and policy.
  - Imagine extending your logs to capture not only "API called" but also "Agent FinanceBot requested API X under policy Y with risk score 0.7." That's the kind of metadata that turns telemetry into governance.
  - When your system runs in Kubernetes, sidecar containers can automatically inject this metadata into every request, creating a governance trace as natural as network telemetry.
* **Technical Entities (Classes/Functions/APIs):** `Cloud Logging`, `OpenTelemetry`, `Prometheus`, `Grafana Loki`, `API`, `Kubernetes`, `sidecar containers`
* **Code Snippet:** None.

---

## Dynamic risk scoring

* **Key Points:**
  - Governance shouldn't mean blocking everything; it should mean evaluating risk intelligently. In an agent network, different actions have different implications. A "summarize report" request is low risk. A "transfer funds" or "delete records" request is high risk.
  - By assigning dynamic risk scores to actions, you can decide in real time whether to: Allow it automatically; Require additional verification; Escalate to a human reviewer
  - You can compute risk scores using metadata such as agent role, data sensitivity, and confidence level. Cloud providers like Google Cloud Vertex AI Model Monitoring already support risk tagging and drift detection—you can extend those ideas to agent actions.
  - The point isn't to slow agents down but to make their behavior context-aware.
* **Technical Entities (Classes/Functions/APIs):** `Google Cloud Vertex AI Model Monitoring`
* **Code Snippet:** None.

---

## Regulatory mapping

* **Key Points:**
  - Frameworks like NIST AI RMF and the EU AI Act are often seen as legal mandates. In reality, they can double as engineering blueprints.
  - Governance principle - Transparency: Engineering implementation - Agent activity logs, explainability metadata
  - Governance principle - Accountability: Engineering implementation - Immutable audit trails in Cloud Logging/Chronicle
  - Governance principle - Robustness: Engineering implementation - Canary testing, rollout control in Kubernetes
  - Governance principle - Risk management: Engineering implementation - Real-time scoring, human-in-the-loop review
  - Mapping these requirements into cloud and container tools turns compliance into configuration.
* **Technical Entities (Classes/Functions/APIs):** `NIST AI RMF`, `EU AI Act`, `Cloud Logging`, `Chronicle`, `Kubernetes`
* **Code Snippet:** None.

---

## Building a Governed AI Stack

* **Key Points:**
  - Let's visualize a practical, cloud native setup—something you could deploy tomorrow.
  - [Agent Layer] ↓ [Governance Layer] → Policy Engine (OPA) → Risk Scoring Service → Audit Logger (Pub/Sub + Cloud Logging) ↓ [Tool / API Layer] → Internal APIs, Databases, External Services ↓ [Monitoring + Dashboard Layer] → Grafana, BigQuery, Looker, Chronicle
  - All of these can run on Kubernetes with Docker containers for modularity. The governance layer acts as a smart proxy—it intercepts agent calls, evaluates policy and risk, then logs and forwards the request if approved.
  - In practice: Each agent's container registers itself with the governance service. Policies live in Git, deployed as ConfigMaps or sidecar containers. Logs flow into Cloud Logging or Elastic Stack for searchable audit trails. A Chronicle or BigQuery dashboard visualizes high-risk agent activity.
  - This separation of concerns keeps things clean: Developers focus on agent logic, security teams manage policy rules, and compliance officers monitor dashboards instead of sifting through raw logs. It's governance you can actually operate—not bureaucracy you try to remember later.
* **Technical Entities (Classes/Functions/APIs):** `OPA`, `Pub/Sub`, `Cloud Logging`, `API`, `APIs`, `Grafana`, `BigQuery`, `Looker`, `Chronicle`, `Kubernetes`, `Docker`, `ConfigMaps`, `sidecar containers`, `Elastic Stack`
* **Code Snippet:** None.

---

## Lessons from the Field

* **Key Points:**
  - When I started integrating governance layers into multi-agent pipelines, I learned three things quickly:
  - It's not about more controls—it's about smarter controls. When all operations have to be manually approved, you will paralyze your agents. Focus on automating the 90% that's low risk.
  - Logging everything isn't enough. Governance requires interpretable logs. You need correlation IDs, metadata, and summaries that map events back to business rules.
  - Governance has to be part of the developer experience. If compliance feels like a gatekeeper, developers will route around it. If it feels like a built-in service, they'll use it willingly.
  - In one real-world deployment for a financial-tech environment, we used a Kubernetes admission controller to enforce policy before pods could interact with sensitive APIs. Each request was tagged with a "risk context" label that traveled through the observability stack. The result? Governance without friction. Developers barely noticed it—until the compliance audit, when everything just worked.
* **Technical Entities (Classes/Functions/APIs):** `Kubernetes admission controller`, `APIs`
* **Code Snippet:** None.

---

## Human in the Loop, by Design

* **Key Points:**
  - Despite all the automation, people should also be involved in making some decisions. A healthy governance stack knows when to ask for help. Imagine a risk-scoring service that occasionally flags "Agent Alpha has exceeded transaction threshold three times today." As an alternative to blocking, it may forward the request to a human operator via Slack or an internal dashboard. That is not a weakness but a good indication of maturity when an automated system requires a person to review it. Reliable AI does not imply eliminating people; it means knowing when to bring them back in.
* **Technical Entities (Classes/Functions/APIs):** `Slack`
* **Code Snippet:** None.

---

## Avoiding Governance Theater

* **Key Points:**
  - Every company wants to say they have AI governance. But there's a difference between governance theater—policies written but never enforced—and governance engineering—policies turned into running code.
  - Governance theater produces binders. Governance engineering produces metrics: Percentage of agent actions logged; Number of policy violations caught pre-execution; Average human review time for high-risk actions
  - When you can measure governance, you can improve it. That's how you move from pretending to protect systems to proving that you do. The future of AI isn't just about building smarter models; it's about building smarter guardrails. Governance isn't bureaucracy—it's infrastructure for trust. And just as we've made automated testing part of every CI/CD pipeline, we'll soon treat governance checks the same way: built in, versioned, and continuously improved.
  - True progress in AI doesn't come from slowing down. It comes from giving it direction, so innovation moves fast but never loses sight of what's right.
* **Technical Entities (Classes/Functions/APIs):** `CI/CD`
* **Code Snippet:** None.


---
## Guardrails help you build safe, compliant AI applications by validating and filtering content at key points in your agent's execution. They can detect sensitive information, enforce content policies, validate outputs, and prevent unsafe behaviors before they cause problems.

* **Key Points:**
  - Guardrails help you build safe, compliant AI applications by validating and filtering content at key points in your agent's execution. They can detect sensitive information, enforce content policies, validate outputs, and prevent unsafe behaviors before they cause problems.
  - Common use cases include: Preventing PII leakage; Detecting and blocking prompt injection attacks; Blocking inappropriate or harmful content; Enforcing business rules and compliance requirements; Validating output quality and accuracy
  - You can implement guardrails using middleware to intercept execution at strategic points - before the agent starts, after it completes, or around model and tool calls.
  - Guardrails can be implemented using two complementary approaches: Deterministic guardrails: Use rule-based logic like regex patterns, keyword matching, or explicit checks. Fast, predictable, and cost-effective, but may miss nuanced violations. Model-based guardrails: Use LLMs or classifiers to evaluate content with semantic understanding. Catch subtle issues that rules miss, but are slower and more expensive.
  - LangChain provides both built-in guardrails (e.g., PII detection, human-in-the-loop) and a flexible middleware system for building custom guardrails using either approach.
* **Technical Entities (Classes/Functions/APIs):** `LLMs`, `LangChain`, `middleware`
* **Code Snippet:** None.

---

## Built-in guardrails

* **Key Points:**
  - PII detection
  - Human-in-the-loop
* **Technical Entities (Classes/Functions/APIs):** `PII detection`, `Human-in-the-loop`
* **Code Snippet:** None.

---

## PII detection

* **Key Points:**
  - LangChain provides built-in middleware for detecting and handling Personally Identifiable Information (PII) in conversations. This middleware can detect common PII types like emails, credit cards, IP addresses, and more.
  - PII detection middleware is helpful for cases such as health care and financial applications with compliance requirements, customer service agents that need to sanitize logs, and generally any application handling sensitive user data.
  - The PII middleware supports multiple strategies for handling detected PII:
  - Strategy: redact - Description: Replace with [REDACTED_{PII_TYPE}] - Example: [REDACTED_EMAIL]
  - Strategy: mask - Description: Partially obscure (e.g., last 4 digits) - Example: ****-****-****-1234
  - Strategy: hash - Description: Replace with deterministic hash - Example: a8f5f167...
  - Strategy: block - Description: Raise exception when detected - Example: Error thrown
* **Technical Entities (Classes/Functions/APIs):** `piiRedactionMiddleware`, `createAgent`
* **Code Snippet:**
```typescript
import { createAgent, piiRedactionMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [customerServiceTool, emailTool],
  middleware: [
    // Redact emails in user input before sending to model
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToInput: true,
    }),
    // Mask credit cards in user input
    piiRedactionMiddleware({
      piiType: "credit_card",
      strategy: "mask",
      applyToInput: true,
    }),
    // Block API keys - raise error if detected
    piiRedactionMiddleware({
      piiType: "api_key",
      detector: /sk-[a-zA-Z0-9]{32}/,
      strategy: "block",
      applyToInput: true,
    }),
  ],
});

// When user provides PII, it will be handled according to the strategy
const result = await agent.invoke({
  messages: [{
    role: "user",
    content: "My email is john.doe@example.com and card is 5105-1051-0510-5100"
  }]
});
```

---

## Human-in-the-loop

* **Key Points:**
  - LangChain provides built-in middleware for requiring human approval before executing sensitive operations. This is one of the most effective guardrails for high-stakes decisions.
  - Human-in-the-loop middleware is helpful for cases such as financial transactions and transfers, deleting or modifying production data, sending communications to external parties, and any operation with significant business impact.
* **Technical Entities (Classes/Functions/APIs):** `humanInTheLoopMiddleware`, `createAgent`, `MemorySaver`, `Command`
* **Code Snippet:**
```typescript
import { createAgent, humanInTheLoopMiddleware } from "langchain";
import { MemorySaver, Command } from "@langchain/langgraph";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, sendEmailTool, deleteDatabaseTool],
  middleware: [
    humanInTheLoopMiddleware({
      interruptOn: {
        // Require approval for sensitive operations
        send_email: { allowAccept: true, allowEdit: true, allowRespond: true },
        delete_database: { allowAccept: true, allowEdit: true, allowRespond: true },
        // Auto-approve safe operations
        search: false,
      }
    }),
  ],
  checkpointer: new MemorySaver(),
});

// Human-in-the-loop requires a thread ID for persistence
const config = { configurable: { thread_id: "some_id" } };

// Agent will pause and wait for approval before executing sensitive tools
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Send an email to the team" }] },
  config
);

result = await agent.invoke(
  new Command({ resume: { decisions: [{ type: "approve" }] } }),
  config  // Same thread ID to resume the paused conversation
);
```

---

## Custom guardrails

* **Key Points:**
  - For more sophisticated guardrails, you can create custom middleware that runs before or after the agent executes. This gives you full control over validation logic, content filtering, and safety checks.
* **Technical Entities (Classes/Functions/APIs):** `createMiddleware`, `AIMessage`
* **Code Snippet:** None.

---

## Before agent guardrails

* **Key Points:**
  - Use "before agent" hooks to validate requests once at the start of each invocation. This is useful for session-level checks like authentication, rate limiting, or blocking inappropriate requests before any processing begins.
* **Technical Entities (Classes/Functions/APIs):** `createMiddleware`, `AIMessage`, `createAgent`
* **Code Snippet:**
```typescript
import { createMiddleware, AIMessage } from "langchain";

const contentFilterMiddleware = (bannedKeywords: string[]) => {
  const keywords = bannedKeywords.map(kw => kw.toLowerCase());

  return createMiddleware({
    name: "ContentFilterMiddleware",
    beforeAgent: {
      hook: (state) => {
        // Get the first user message
        if (!state.messages || state.messages.length === 0) {
          return;
        }

        const firstMessage = state.messages[0];
        if (firstMessage._getType() !== "human") {
          return;
        }

        const content = firstMessage.content.toString().toLowerCase();

        // Check for banned keywords
        for (const keyword of keywords) {
          if (content.includes(keyword)) {
            // Block execution before any processing
            return {
              messages: [
                new AIMessage(
                  "I cannot process requests containing inappropriate content. Please rephrase your request."
                )
              ],
              jumpTo: "end",
            };
          }
        }

        return;
      },
      canJumpTo: ['end']
    }
  });
};

// Use the custom guardrail
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, calculatorTool],
  middleware: [
    contentFilterMiddleware(["hack", "exploit", "malware"]),
  ],
});

// This request will be blocked before any processing
const result = await agent.invoke({
  messages: [{ role: "user", content: "How do I hack into a database?" }]
});
```

---

## After agent guardrails

* **Key Points:**
  - Use "after agent" hooks to validate final outputs once before returning to the user. This is useful for model-based safety checks, quality validation, or final compliance scans on the complete agent response.
* **Technical Entities (Classes/Functions/APIs):** `createMiddleware`, `AIMessage`, `initChatModel`, `createAgent`
* **Code Snippet:**
```typescript
import { createMiddleware, AIMessage, initChatModel } from "langchain";

const safetyGuardrailMiddleware = () => {
  const safetyModel = initChatModel("gpt-5.4-mini");

  return createMiddleware({
    name: "SafetyGuardrailMiddleware",
    afterAgent: {
      hook: async (state) => {
        // Get the final AI response
        if (!state.messages || state.messages.length === 0) {
          return;
        }

        const lastMessage = state.messages[state.messages.length - 1];
        if (lastMessage._getType() !== "ai") {
          return;
        }

        // Use a model to evaluate safety
        const safetyPrompt = `Evaluate if this response is safe and appropriate.
        Respond with only 'SAFE' or 'UNSAFE'.

        Response: ${lastMessage.content.toString()}`;

        const result = await safetyModel.invoke([
          { role: "user", content: safetyPrompt }
        ]);

        if (result.content.toString().includes("UNSAFE")) {
          return {
            messages: [
              new AIMessage(
                "I cannot provide that response. Please rephrase your request."
              )
            ],
            jumpTo: "end",
          };
        }

        return;
      },
      canJumpTo: ['end']
    }
  });
};

// Use the safety guardrail
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, calculatorTool],
  middleware: [safetyGuardrailMiddleware()],
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "How do I make explosives?" }]
});
```

---

## Combine multiple guardrails

* **Key Points:**
  - You can stack multiple guardrails by adding them to the middleware array. They execute in order, allowing you to build layered protection.
* **Technical Entities (Classes/Functions/APIs):** `createAgent`, `piiRedactionMiddleware`, `humanInTheLoopMiddleware`
* **Code Snippet:**
```typescript
import { createAgent, piiRedactionMiddleware, humanInTheLoopMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, sendEmailTool],
  middleware: [
    // Layer 1: Deterministic input filter (before agent)
    contentFilterMiddleware(["hack", "exploit"]),

    // Layer 2: PII protection (before and after model)
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToInput: true,
    }),
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToOutput: true,
    }),

    // Layer 3: Human approval for sensitive tools
    humanInTheLoopMiddleware({
      interruptOn: {
        send_email: { allowAccept: true, allowEdit: true, allowRespond: true },
      }
    }),

    // Layer 4: Model-based safety check (after agent)
    safetyGuardrailMiddleware(),
  ],
});
```

---

## Additional resources

* **Key Points:**
  - Middleware documentation - Complete guide to custom middleware
  - Middleware API reference - Complete guide to custom middleware
  - Human-in-the-loop - Add human review for sensitive operations
  - Testing agents - Strategies for testing safety mechanisms
* **Technical Entities (Classes/Functions/APIs):** `Middleware API`, `Human-in-the-loop`, `Testing agents`
* **Code Snippet:** None.