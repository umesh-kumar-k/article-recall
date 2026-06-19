1. Core LLM Concepts

	1. Transformer Architecture
		- [Attention Mechanism:](attention-mechanism.md)  Core operation where every token attends to all others;determines how context is weighted - understanding this explains model strenghts/limits
		- Context Window: Maximum tokens(input + output) a model can process in one call; directly drives architecture decision on chunking, memory & cost
		- Tokenization: Text is split into sub-word tokens (BPE, SentencePiece); token count != word count - affects pricing , latency & prompt design.
		- Temperature & Sampling: Controls output randomness; low temperature deterministic tasks (code,extraction), high for creative tasks.
		- Top-P/Top-K Sampling: Nuclues sampling strategies that constrin token selection space; tune for quality vs diversity trade-off
		- Hallucination: Model generates plausible but factually incorrect output; a structural risk, not a bug - architecture must compensate via grounding and verification
		- Emerging Capabilities: Larger models exhibit skills not explicity trained for (reasoning,code gen); capability thresholds vary by model size & training date.
		- Multimodal Models: Models accepting text + image/audio/video(GPT-4o, Gemini); expanding input modalities changes integration architecture significantly.
	
	2. Model Types & Selection
		- Frontier Models (Closed-Source): GPT-4o, Claude 3.5, Gemini 2.5 - highest capability, API -only , vendor lock-in-risk, per-token pricing at scale becomes expensive.
		- Open-Weight Models: Llama 3.x, Mistral, Qwen, Gemma - self-hostable , no data-egress concerns, ideal for regulated industries and cost optimization at volume
		- Small Language Models(SLMs): Phi-3/4, Llama 3B/8B - designed to run on device or constrained infra; suitable for narrow, well-defined tasks.
		- Embedding Models: text-embedding-3-large(OpenAI), Cohere Embed, BGE - purpose-built for semantic similarity;separate from generative models, must be versioned independently.
		- Reasoning Models: o1,o3, DeepSeek-R1 - use chain-of--thought internally before responding; higher latency/cost,justified for complex multi-step problems;
		- Model Benchmarks: MMLU, HumanEval, HEML, MT-Bench - use to compare models objectively; but benchmark != production performance, always run task-specific evals.

3. Prompt Engineering
		1. System Prompt: Defines model persona, constraints , output format -acts as the "contract" between the application & the LLM;version-controlled like code.
		2. Few-Shot prompting: Prioviding input-output examples inside the prompt to guide model behaviour without retraining; most cost-effective capability shaping technique.
		3. Chain-of-Thoght(CoT): Instructing model to reason step by step before answering improves accuracy on multi step tasks, increases token usage & latency.
		4. Structured Output/JSON Mode: Forcing model to respond in a strict schema(JSON,XML) using constrained decoding or output parsers; critical for downstream system integration.
		5. Prompt Injection: Attack vector where malicious user input overrides system instructions; a security architecture concern not just a prompt concern.
		6. Prompt Versioning & Management: Treat prompts a first class artifacts - store in version control (LangSmith, PromptLayer) test regression on prompt changes.
		7. Meta-Prompting: Using an LLM to generate or optimize prompts for another LLM;emerging pattern for automated prompt tuning in production pipeline

4. Fine-Tuning & Model Adaptation
	1. When to fine tune: Only when propmt engineering + RAG cannot achieve required behaviour, style or domain accuracy; not a first resort
	2. Supervised Fine-Tuning(SFT): Training on curated  input-output pairs to adapt model to specific tasks/tone; requires quality data curation, which is the hardest part.
	3. LoRA/QLoRA: Parameter-efficient fine tuning techniques that add small adapter metrics, reducing GPU memory 4-8x; standard approach for adapting open-weight models.
	4. RLHF(Reinforcement Learning from Human Feedback): Alignment technique used during base model training; not typically done by product teams but important to understand its role in model behaviour.
	5. RLAIF: Replaces human labelers with another AI model for preference data generation; more scalable, used increasingly in enterprise fine-tuning pipeliness
	6. DPO(Direct Preference Optimization): Simpler alternative to RLHF for alignment; trains directly on preference pairs without a reward model- lower infrastructure complexity
	7. Instruction Tuning: Fine-tuning on diverse instruction- following datasets to improve general task compliance;often done on top of base open-weight models.
	8. Fine Tuning vs RAG vs Prompting; Architectural decision matrix - prompting for behaviour shaping, RAG for knowledge injection, fine-tuning for style/task specialization.
	9. Catastrophic Forgetting: Fine-tuning can degrade model's general capabilities; mitigated by mixing general data with task-specific data.
	10. Model Merging: Combining weights from multiple fine-tuned models(TIES, DARE merge); emerging technique to gain multi-task capability without retraining

5. LLM Evaluation & Observability
	1. Evals(LLM Evaluation): Systematic testing of LLM outputs against quality criteria - the engineering equivalent of unit/integration tests; non-negotiable in production systems.
	2. LLM-as-Judge: Using a frontier model to score another model's output on dimensions like accuracy, toxicity, relevance; scalable but requires calibration against human baselines
	3. Reference based vs Reference free Evals: Comparing to ground -truth answers vs assessing quality without a reference; the latter required when no golden dataset exists.
	4. RAGAS/TruLens: Eval frameworks specifically for LLM pipelines; measure faithfulness, answer relevance, context precision without manual labeling.
	5. Tracing & Observability: Capturing full LLM call chains(inputs,outputs,latency,token counts,tool calls) using LangSmith, Langfuse or Branitrust for debugging & cost attribution.
	6. Semantic Drift Detection: Monitoring model output distribution over time to catch silent degradation when model versions change under the same API.
	7. Guardrails/Output Validation: Runtime checks on LLM outputs for safety schema compliance, toxicity (Guardrails AI, NeMo Guardrails)l placed both pre & post LLM call
	8. Red-Teaming: Adversarial testing of LLM applications for jailbreaks, propmt injections ,PII leakage and harmful outputs before production deployment.
	9. Human-in-the-Loop feedback: Capturing thumbs-up/down or correction signals from production users to build a continuous improvement dataset.

6. LLM Application Architecture Patterns
	1. LLM Gateway/Proxy: Centralized Layer(LiteLLM, KongAI , Azure APIM) for routing, rate-limiting, key management, model fallback & cost tracking across all LLM calls.
	2. Semantic Caching: Caching LLM responses based on semantic similarity of queries (not exact match) using vector lookup; reduce latency & API cost significantly.
	3. Orchestration Layer: Coordinates multi-step LLM workflows - LangChain,LlamaIndex or custom code; avoid over relying on framework abstractions in complex systems.
	4. Event Driven LLM pipelines: Async processing of LLM tasks via message queues (Kafka,SQL) decoupling inference from request lifecycle for long-running tasks.
	5. Streaming Responses: Token by Token streaming via SSE or WebSockets; critical for perceived UX performance in chat interfaces.
	6. Fan-Out/Parallel inference: Running multiple LLM calls concurrently and aggregating results; used for multi-perspective analysis or ensemble outputs.
	7. Fallback & Circuit Breaker: Automatic model fallback(GPT-4o -> GPT-4o-mini) on rate limit or error;must account for output quality degradation downstream
	8. Context Management: Explicity managing what goes into each LLM call's context window - trimming, summarizing or prioritizing by relevance to stay withing limits
	9. Memory Architectures: In context(prompt), external (vector store),episodic(session summaries), entity(knowledge graph) - each has different scope and persistence characteristics.

7. LLMOps - Operating LLMs in Production
	1. LLMOps vs MLOps: MLOps focuses on model training pipelines; LLMOps focuses on prompt management, inference infrastructure , eval pipelines and output monitoring.
	2. Model versioning: LLM API provides silently update models; pin model versions in production(gpt-4o-2024-08-06) to prevent behaviour drift.
	3. Inference Optomization: vLLM,TGI(Text Generation Inference), TensorRT-LLM - production serving frameworks with continuous batching, KV cache management, quanitization.
	4. Quantization: Reducing model weight precision(FP 16 -> INT8/INT4) to lower GPU memory footprint; trade-off between model size & output quality.
	5. KV CacheL Reusing computed attention state for common propmt prefixes; architectural feature of inference servers that reduces latency on repeated system propmts;
	6. Model Serving Platforms: Triton Inference Server, BentoML, Ray Serve, Modal - manage model lifecycle, auto-scaling, and multi-model serving in production.
	7. Cost Attribution: Token usage per user/feature/team; essential for enterprise biling, quota management & identifying runaway pipelines.
	8. Prompt/Response Logging Pipeline: PII srubbling, audit trails, and compliance logging for regulated industries; must be designed before go-live.
	9. A/B Testing LLM Changes: Routing a % of traffic to a new model/prompt version; requires eval frmework to detect quality changes statistically.

8. Security & Governance
	1. Propmt Injection Attacks: User input manipulates system instructions; mitigated by input sanitization, privilege separation and output validation - not fully solvable by prompting alone.
	2. Data privacy/PII Leakage: LLMs can memorize and regurgitate training data or user inputs; requires PII detection, data masking, and careful context design.
	3. AI Acceptable Use Policies: Enterprise governance layer defining permitted use cases, data classifications allowed in LLM context, and human oversight requirements.
	4. Model Access Control: Gating which user/teams/applications can access which models and with what context privileges - critical in multi - tenant enterprise platforms.
	5. Supply Chain Risj (Open-Weight Models): Downloading fine-tuned open models from HuggingFace introduces model poisoning risk;scan and validate weights before deployment.
	6. Output Filtering: Blocking toxix, harmful or policy violating content at the infrastructure layer regardless of what the model produces.
	7. Audit Loggin: Immutable logs of LLM inputs/outputs for compliance e-discovery and incident investigation - design the schema before building the system.
	8. EU AI Act/Regulatory Compliance: Classifies AI systems by risk level; high-risk systems(HR , credit, hiring) require transparency, human oversight & documentation obligations.

9. Vector Databases & Embeddings (Beyond RAG)
	1. Embedding Use Cases Beyond Search: Classification, clustering, anomoly detection, duplicate detection, recommendation - embeddings are a general purpose semantic representation layer.
	2. Vector DB Options: Pinecone(managed, simple), Weaviate(multi-model, graph features),Qdrant(high-performance,self hostable),pgvector(Postgres extension, reduces stack complexity)
	3. Indexing Algorithms: HNSW(high recall, high memory), IVF (lower memory, approximate), Flat(exact, slow at scale) - choose based on dataset size & recall requirements.
	4. Embedding Versiononing: Changing the embedding model invalidates all stored vectors; requires full re-indexing - treat as a schema  migration event.
	5. Hybrid Storage Architecture: Pairing vector DB with a relational/document store - vectors for semantic lookup, structured DB for metadata filtering, joins and ACID transactions.
	6. Multi-Tenancy in Vector DBs: Namespace or collection per tenant vs metadata filtering - namespace isolation is safer for compliance , filtering is more cost -efficient

10. Structured Data & LLM Integration
	1. Text to SQL: LLM translates natural language to SQL queries against relational databases; requires schema context injection and validation before execution
	2. Function Calling/Tool Use: LLMs output structured calls to predefined functions/APIs instead free text; backbone of agent & workflow automation patterns.
	3. Structured Extraction: Using LLMs to extract typed entities from unstructured documents(contracts, emails, PDFs ); paired with schema validation(Pydantic,Zod)
	4. Semantic Layer: Mapping business terminology to data model concepts enabling LLMs to query enterprise data without exposing raw schema - emerging pattern in enterprise BI
	5. LLM + Knowledge Graph: Pairing LLMs with graph databases (Neo4j) for multi-hop reasoning over interconnected entities; better than vectors for relational fact retrieval.

11. Agentic Patterns without full agents (Workflows)
	1. Prompt Chaining : Sequential LLM calls where ouput of one is input to next; simpler and more predictable than full agent loops - prefer for well defined workflows.
	2. Routing Pattern: A classifier LLM routes user intent to the appropriate specialized propmt/model/pipeline - reduces context window bloat and improves accuracy.
	3. Map-Reduce over LLM: Process large documents in chunks (map), then synthesize results(reduce); common for summarization , analysis of long-term content
	4. Parallelization: Running independently LLM subtatsks concurrently then merging results; improves latency for decomposable workflows
	5. Human-in-the Loop Checkpoints: Pausing automated LLM workflows for human approval before consequential actions(sending emails, making DB writes)

12. Multi-Model architecture
	1. Vision-Language Models(VLMs): GPT-4o , LLaVA, Gemini - process image + text together; useful for document understanding, UI analysis, diagram interpretation.
	2. Document Intelligence: Combing OCR, layout parsing(Azure Document Intelligence,AWS Textract) and LLMs to extract structured data from complex documents.
	3. Speech to Text + LLM: Whisper + LLM pitpiline for voice-driven interfaces; latency is a key architectural concern in real time use cases.
	4. Image Generation Integration: DALL-E3 Stable Diffusion, Flux - treat as external services in architecture; propmt management and safety filtering apply
	5. Multimodal Embeddings; CLIP, ImageBind- embed images and text in a shared semantic space enabling cross-modal search & comparison.