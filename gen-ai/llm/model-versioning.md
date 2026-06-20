---
aliases:
  - Model Versioning
Source 1: https://tianpan.co/blog/2026-05-05-ai-model-apis-invisible-software-dependencies
Source 2: https://genesisclawbot.github.io/llm-drift/blog/llm-version-pinning.html
---

## Why LLM Version Pinning Doesn't Protect You — And What Does

## The Appeal of Version Pinning
* **Key Points:**
  - The logic is sound on paper. Software engineers version-pin dependencies all the time. numpy==1.26.4 gives you exactly what you tested against. Why should LLM model versions be different?
  - The difference is the contract. When you pin numpy==1.26.4, the package maintainers are contractually bound by semantic versioning: patch releases don't break APIs. LLM providers have a fundamentally different contract:
  - "We may modify, update, or discontinue the Services or Models at any time. We may update the underlying models from time to time to improve safety, quality, and performance." — OpenAI Terms of Service (paraphrased), 2026
  - That clause exists for good reasons — safety patches, jailbreak mitigations, performance improvements. But it means your "pinned" version can change behaviour without the version string changing.
* **Technical Entities (Classes/Functions/APIs):** `numpy==1.26.4`

## The Evidence: Pinned Versions That Changed
* **Key Points:**
  - ⚠ Documented Cases of "Frozen" Versions Drifting
  - gpt-4o-2024-08-06 — In January 2025, multiple developers on r/LLMDevs reported unexpected behaviour changes. The model began adding explanatory context to responses that had previously been terse and structured. The version string didn't change.
  - claude-3-5-sonnet-20241022 — Anthropic's minor updates to Constitution AI alignment affected instruction-following on a subset of prompts. Not announced in release notes.
  - gpt-5.2 Instant (Feb 10, 2026) — Released as "GPT-5.2 Instant improves response style." JSON extraction prompts with "return ONLY" instructions began including preamble text in some cases.
  - Each of these cases had the same shape: the developer believed they were insulated, discovered the breakage through a user complaint or a failed production job, and spent hours debugging what turned out to be upstream model behaviour.
* **Technical Entities (Classes/Functions/APIs):** `gpt-4o-2024-08-06`, `claude-3-5-sonnet-20241022`, `gpt-5.2 Instant`

## Why This Happens
* **Key Points:**
  - LLM providers update "pinned" models for several reasons they rarely announce:
  - Safety and alignment patches — When a new jailbreak or harmful output pattern is discovered, it gets patched across all active model versions, not just the latest. This is the right call for safety; it's a breaking change for your pipeline.
  - Infrastructure changes — The same model weights running on different hardware or inference software (quantisation, batching strategy, temperature implementation) can produce subtly different outputs.
  - RLHF updates — Reinforcement learning from human feedback is an ongoing process. Fine-tuning on new preference data affects all model endpoints that share the underlying weights.
  - System prompt changes — Providers occasionally update system-level instructions that run before your prompt. These aren't disclosed.

## What Version Pinning Actually Gives You
* **Key Points:**
  - Version pinning protects you from the predictable. It doesn't protect you from the silent.
* **Technical Entities (Classes/Functions/APIs):** N/A

## The Right Mental Model: Treat LLMs Like External APIs with No SLA
* **Key Points:**
  - The closest analogy isn't a versioned library. It's a third-party REST API that can change response format without notice, has no changelog, and whose SLA only covers availability — not behaviour.
  - How do engineers handle APIs like that? With contract tests. You write tests that assert the API returns the format you expect, and you run them continuously. When the format changes, your test fails before your production code fails.
  - Prompt regression testing is the same discipline applied to LLMs.

## A Practical Prompt Regression Testing Setup
* **Key Points:**
  - Step 1: Define your acceptance criteria explicitly
  - Step 2: Establish a baseline against today's behaviour
  - Step 3: Run checks on a schedule, not on demand
  - Step 4: Score, don't binary-pass/fail
* **Code Snippet:**
```yaml
# Instead of: "returns a summary"
# Define: is_valid_json, has_keys:title,summary,sentiment, max_length:500

validators:
  - is_valid_json
  - has_keys: [title, summary, sentiment]
  - sentiment_is_one_of: [positive, negative, neutral]
  - max_length: 500
```

## The Operational Benefit: Time to Detection
* **Key Points:**
  - CI/CD tests are necessary but not sufficient. They catch regressions you introduce. They don't catch regressions the LLM provider introduces between your deployments.

## Combining Both: Version Pinning + Continuous Monitoring
* **Key Points:**
  - The right answer isn't to abandon version pinning — it's to layer it with continuous monitoring. Use the pinned version to reduce surface area (fewer unexpected capability changes), and use automated testing to catch everything that slips through.

---


# AI Model APIs Are Software Dependencies You Can't See, Pin, or Track

## What Providers Actually Change (and Don't Tell You)
* **Key Points:**
  - Deprecations are the visible part. OpenAI publishes a deprecation schedule that follows predictable patterns: preview models get 3–6 months of notice, production models get longer windows, and the Assistants API got an unusually generous 17-month runway before its 2026 shutdown. Azure enforces 90–120 days of formal notice for GA model retirements. These timelines are workable if you catch them.
  - Behavioral changes are the invisible part. These are what actually break production.
  - The sycophancy incident was one example. Another happened when OpenAI updated the default routing for gpt-4o in October 2024 and developers immediately noticed the model "often fails to call function calls when it should." No announcement, no changelog entry, no version bump — just a sudden increase in structured-output failures that teams had to discover through production monitoring.
  - Users also documented gpt-4o beginning to repeat items in parsed outputs or skip them entirely compared to behavior from weeks prior.
  - Anthropic's recent history shows the same pattern. When a new Claude version showed well-documented regressions in multi-file editing and agentic task performance, the behavioral shifts arrived without any changelog entry. The engineering community tracked it through forum posts and incident reports; Anthropic published nothing official. Teams relying on that model for coding workflows discovered the regression the same way: users complained, engineers scrambled, no one knew what changed.
  - This is the gap that matters. Providers are disciplined about announcing deprecation dates. They're not disciplined — and arguably can't be — about announcing behavioral drift.
* **Technical Entities (Classes/Functions/APIs):** `gpt-4o`, `Assistants API`, `Claude`

## Why Semver Doesn't Apply Here
* **Key Points:**
  - Semantic versioning works because breaking changes can be defined contractually. If you call calculateTax(amount, rate) and the function signature changes, you have a breaking change. The contract is precise.
  - Foundation model output contracts don't work this way. The model's "API" is the HTTP endpoint and the request schema. What the model returns for a given prompt is not contractually specified — it's statistically characterized. Whether an output is "correct" depends on user context, task type, and quality standards that vary by application. A change that improves math performance while degrading creative writing is a regression for one team and an improvement for another.
  - This isn't a failure of provider discipline. It's a category mismatch. SemVer assumes you can enumerate breaking changes. For a system with billions of possible inputs and outputs that are evaluated subjectively, you can't enumerate them. The providers know this, which is why they've settled on date-stamped snapshots as a versioning proxy rather than semantic version contracts.
  - The more significant problem is when providers retreat even from snapshot guarantees. When a major provider dropped version pinning — forcing users onto aliases that always route to the latest underlying snapshot — it removed the last tool engineers had for maintaining output stability between their own deployments. You can lock your code. You can't lock the model.
* **Technical Entities (Classes/Functions/APIs):** `calculateTax(amount, rate)`

## The SBOM Blind Spot
* **Key Points:**
  - Traditional Software Bills of Materials enumerate your dependencies with enough detail to assess vulnerabilities and audit your supply chain. They list packages, versions, licenses, and transitive dependencies. SBOM tooling integrates with package managers, registries, and CI pipelines.
  - Foundation model APIs don't appear in any of this. There's no npm package for gpt-4o. No pip version specifier for claude-3-5-sonnet. No lockfile entry for gemini-2.5-pro. Your SBOM has a complete picture of every library you import and no picture of the model API that processes a significant fraction of your application's logic.
  - An emerging category called AI-BOMs (AI Bill of Materials) is trying to address this. The CycloneDX specification now includes ML/AI component support. SPDX 3.0 introduced structured AI/ML component fields. These standards can represent model versions, dataset provenance, training pipeline details, and runtime dependencies in a machine-readable format.
  - The practical problem is that no major model provider publishes an AI-BOM. You can't fetch one from OpenAI's API. Anthropic doesn't ship one with Claude. The standards exist; the data doesn't. Building your own AI-BOM means manually maintaining a document that lists the model endpoints your application calls, the snapshot versions or aliases in use, and the evaluation benchmarks you're using to characterize expected behavior. That's useful, but it's entirely disconnected from the automated SBOM tooling that makes traditional dependency management scalable.
* **Technical Entities (Classes/Functions/APIs):** `gpt-4o`, `claude-3-5-sonnet`, `gemini-2.5-pro`, `CycloneDX`, `SPDX 3.0`

## The Dependency Discipline That Actually Works
* **Key Points:**
  - Since you can't pin a model to a specific behavior and can't rely on changelogs for behavioral changes, the only viable strategy is behavioral verification: defining what your application expects the model to do and continuously checking whether it still does it.
  - Behavioral fingerprinting as a pinning proxy. The functional equivalent of a lockfile for model behavior is a golden evaluation dataset: a representative set of inputs with characterized expected outputs. These aren't unit tests with exact string matches — they're rubric-based evaluations that define what "good" looks like for your task. When a model changes, your golden dataset catches it before users do. The dataset becomes your version contract.
  - The practical discipline here is maintaining a structured evaluation suite that runs against every deployment. Platforms like Braintrust support GitHub Actions integration that runs evals on every PR and blocks merges when quality scores drop below threshold. This is the closest available analog to dependency pinning: you're not locking the model version, but you're enforcing that any model change must still pass your quality bar.
  - Build your own changelogs. Since providers don't reliably document behavioral changes, engineering teams that care about stability monitor them directly. This means:
    - Subscribing to official deprecation pages, not just following product announcements
    - Running a small percentage (commonly 5%) of production requests through an asynchronous evaluation pipeline, with quality scores tracked over time
    - Monitoring deterministic output properties — format compliance, constraint adherence, refusal rates — as leading indicators that something changed before quality metrics fully degrade
    - Watching community forums and developer communities for the informal signal that a model behavior changed before the official signal
  - The community-aggregated changelogs that have appeared (third-party sites tracking provider updates across vendors) exist precisely because official sources are fragmented. Incorporating these into your incident response runbooks — as you would for any critical upstream dependency — is currently the state of the practice.
  - SDLC integration for model deprecation notices. Model deprecations are announced with lead time, but they're announced in documentation pages and community boards, not in the systems engineers monitor. The engineering discipline is treating model aliases and endpoint versions as tracked configuration items, with calendar-based alerting for known deprecation dates. When a model you depend on enters its deprecation window, that should generate a tracking item in the same system where you manage library upgrades.
  - This sounds obvious, but most teams don't do it. They find out about deprecations when the deprecated model returns 404 or routes to an unexpected replacement, rather than months earlier when they could have planned the migration.
  - Offline evaluation before production migration. When a provider does release a new model version or announces an alias rotation, treat it as you would a major dependency upgrade: run your golden evaluation dataset against the new version in a staging environment before pointing production traffic at it. This is the model migration discipline that catches the regressions that changelogs won't mention — the behavioral differences that only show up in your specific use case.
* **Technical Entities (Classes/Functions/APIs):** `Braintrust`, `GitHub Actions`

## The Asymmetry That Makes This Hard
* **Key Points:**
  - There's a structural asymmetry between how fast providers can change models and how fast teams can adapt. Providers update models continuously, often without clear communication. Teams discover behavioral regressions through user reports, then spend days diagnosing a root cause that ultimately traces to a model update they had no visibility into.
  - Traditional software dependencies have solved this asymmetry with infrastructure: package registries, lockfiles, dependency bots, security advisory feeds, SBOMs. None of this infrastructure exists for foundation model dependencies. The responsibility falls entirely on engineering teams to build the monitoring, evaluation, and alerting that would elsewhere be handled by tooling.
  - That's not a temporary gap. Model output is non-deterministic, task-specific, and subjectively evaluated — properties that make it structurally incompatible with the supply chain infrastructure built for deterministic, contractually-specified software components. The tooling that works for this category (evaluation frameworks, behavioral monitoring, regression testing) is different in kind, not just different in implementation.
  - The teams that handle model dependency risk well are the ones that have made this infrastructure a first-class engineering investment: maintained evaluation suites with coverage across their actual use cases, monitoring that surfaces behavioral changes before users do, and operational discipline for model migrations that treats a version bump like the system-level change it is. The teams that struggle are the ones that discovered, too late, that the dependency they weren't tracking had changed.