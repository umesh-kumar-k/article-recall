---
aliases:
  - Reference Based vs Reference Free Evals
Source 1: https://www.evidentlyai.com/llm-guide/llm-evaluation-metrics
---
# LLM evaluation metrics and methods, explained simply


* **Key Points:**
  - "LLM evaluations are quality checks that help make sure your AI product does what it's supposed to, whether that's writing code or handling support tickets."
  - "They are different from benchmarking: you're testing your system on your tasks, not just comparing models. You need these checks throughout the AI product lifecycle, from testing to live monitoring."
  - "This guide focuses on automated LLM evaluations."
* **Technical Entities (Classes/Functions/APIs):** `Evidently library`

## TL;DR
* **Key Points:**
  - "LLM evaluation methods fall into two main types: reference-based and reference-free."
  - "Reference-based LLM evaluation methods compare responses to known ground truth answers using exact matching, word overlap, embedding similarity, or LLMs as judges. For classification and ranking (common in RAG), there are task-specific quality metrics."
  - "Reference-free LLM evaluation methods assess outputs through proxy metrics and custom criteria using regular expressions, text statistics, programmatic validation, custom LLM judges, and ML-based scoring."
  - "LLM as a judge is one of the most popular methods. It prompts an LLM to score outputs based on custom criteria and can handle conversation-level evaluations."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `LLM as a judge`

## How LLM evals work
* **Key Points:**
  - "LLM evals help assess if the system is doing well in terms of capabilities and safety."
  - "Since LLMs are non-deterministic and have unique failure patterns, this is an ongoing process. For example, you may run evals every time you change a prompt."
  - "Manually reviewing LLM outputs is ideal but not scalable, so you need automation."
  - "Automated LLM evaluation workflows have two parts. First, you need data: this could be synthetic examples, curated test cases, or real logs from your LLM app. Second, you need a scoring method. This might return a pass/fail, a label, or a numerical score that tells you if the output is sound. In the end you always get some metric, but there are many ways to arrive at it."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Do you have reference examples?
* **Key Points:**
  - "To pick the right method and metric, you first need to identify the LLM evaluation scenario. They fall into two main buckets: Reference-based: you compare outputs to predefined answers. Reference-free: you evaluate the outputs directly."
  - "Reference-based LLM evaluations compare outputs to known answers, often referred to as 'ground truth.' Once you create this test dataset, your job is to measure how closely the new LLM outputs match these known responses. Techniques range from an exact match to semantic comparisons."
  - "Reference-free LLM evaluations don't require predefined answers. Instead, they assess outputs based on proxy metrics or specific qualities like tone, structure, or safety."
  - "Some LLM evaluation methods, like LLM judges, work well in both cases. However, many metrics are limited to reference-based scenarios, making this distinction useful."
* **Technical Entities (Classes/Functions/APIs):** `LLM judges`

### Is there a single right answer?
* **Key Points:**
  - "Even if you have reference examples, an important question is whether each input has a single correct answer. If it does, you can use straightforward, deterministic checks. It's easy to compare if you get what you expect."
  - "When LLMs handle predictive tasks, you can also rely on established machine learning metrics."
  - "However, you don't always have a single 'perfect' answer. In translation, content generation, or summarization, there are multiple valid outputs and exact matching won't work. Instead, you need comparison methods like semantic similarity that can handle variations."
  - "A similar thing applies to reference-free LLM evals. Sometimes, you can use objective checks like running generated code to see if it works or verifying JSON keys. But in most cases, you'd deal with open-ended LLM evaluations. So you need to find ways to quantify subjective qualities or come up with proxy metrics."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Dataset-level evaluations
* **Key Points:**
  - "Some metrics naturally work on entire datasets. For example, classification metrics like precision or F1-score aggregate the results across all predictions and give you a single quality measure."
  - "In contrast, LLM-specific evaluation methods typically assess each response individually. For example, you might use LLM judges or calculate semantic similarity for each output. But there's no built-in mechanism to combine these scores into a single performance metric. It's up to you how to aggregate results across your test set or live responses."
  - "To avoid drowning in metrics, it helps to define clear test conditions."
* **Technical Entities (Classes/Functions/APIs):** `LLM judges`

## LLM evaluation methods
* **Key Points:**
  - "Finally, let's move on to LLM evaluation methods! We'll break them down by whether or not you need ground truth data and go through methods one by one."
* **Technical Entities (Classes/Functions/APIs):** `Evidently Python library`

### Reference-based evals
* **Key Points:**
  - "Reference-based LLM evals are great for experiments. When you're testing different prompts, models, or configurations, you need a way to track progress. Otherwise, you're just blindly trying ideas. Running repeated checks on your test set helps you see if you're getting more things right over time."
  - "In LLM regression testing, the same approach helps you confirm that updates don't break what's already working or introduce new bugs."
  - "The evaluation process itself is simple: Pass your test inputs through the system. Generate new outputs. Compare them to the referenced answers."
  - "But here's the catch: your LLM evaluation is only as good as the test dataset. You've got to put in the work — label some data, create synthetic examples, or pull from production logs. The dataset needs to be diverse and kept up to date as you find new user scenarios or issues."
* **Technical Entities (Classes/Functions/APIs):** None specified

#### Classification metrics
* **Key Points:**
  - "TL;DR: These LLM evaluation metrics help quantify performance across a dataset of examples for tasks like binary and multi-class classification."
  - "Classification is about predicting a discrete label for each input. This often appears as a component in larger workflows, but LLMs can also handle them directly."
  - "An intuitive metric here is accuracy, which tells you how many classifications are correct. However, it's not always the best measure."
  - "You might also need per-class metrics. For instance, in content moderation, your system might perfectly detect 'offensive language' but miss a lot of 'spam.' Tracking only overall accuracy could hide this imbalance when you have multiple categories."
* **Technical Entities (Classes/Functions/APIs):** `Accuracy`, `Precision`, `Recall`, `F1-Score`, `Per-class metrics`

#### Ranking metrics
* **Key Points:**
  - "TL;DR: These metrics measure performance for tasks like retrieval (including RAG) and recommendations by evaluating how well systems rank relevant results."
  - "When we talk about ranking tasks, we usually mean either search or recommendations, both of which are important in LLM applications."
  - "Rank-agnostic metrics focus on whether relevant items are retrieved, regardless of their order. (Did the system find the items it should find?) Rank-aware metrics consider item order, rewarding systems that display relevant results near the top."
  - "Evaluations often target top-K results (e.g., top-5 or top-10), since systems usually retrieve many items but display or use only a few. For example, Hit Rate@5 measures how often at least one relevant result shows up in the top 5."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Precision @k`, `Recall @k`, `nDCG@K` (Normalized Discounted Cumulative Gain), `Hit Rate @K`, `MRR@K` (Mean Reciprocal Rank)

#### Deterministic matching
* **Key Points:**
  - "TL;DR: Whenever you can write code to verify outputs against correct responses."
  - "In these cases, you can perform deterministic matching to check outputs programmatically, similar to software unit tests. But with LLMs, getting one answer right doesn't mean others will be correct, so you need a variety of test inputs to ensure good coverage."
  - "If the output is structured as JSON, like { 'job role': 'AI engineer', 'min_experience_yrs': '3' }, you can also match the JSON key-value pairs."
  - "In stress-testing, you might compare all outputs to a single list of expected words or phrases. For instance, if you want a chatbot to avoid mentioning competitors, you could create challenging prompts and test if the responses contain the expected refusal words."
* **Technical Entities (Classes/Functions/APIs):** `Exact Match`, `Fuzzy Match`, `Word or Item Match`, `JSON match`, `Unit test pass rate`

#### Overlap-based metrics
* **Key Points:**
  - "TL;DR: Comparing the responses using word, item or character overlap."
  - "Until now, we mostly looked at constrained tasks where there is a single right outcome. But many LLM applications generate free-form texts. In these cases, you may have an example response but won't expect to match it precisely."
  - "To address this, the machine learning research community came up with overlap-based metrics. They reflect how many shared symbols, words, or word sequences there are between the reference and the generated response."
  - "It's common to use averages to summarize the scores across multiple examples, but variations are possible, like recomputing BLEU or ROUGE for the entire example dataset."
  - "But here is the rub: while overlap-based metrics have been foundational in NLP research, they often don't correlate well with human judgements, and aren't suitable for very open-ended tasks. Modern alternatives, such as embedding- or LLM-based evaluations, offer more context-aware assessments."
* **Technical Entities (Classes/Functions/APIs):** `BLEU`, `ROUGE-n`, `ROUGE-l`, `METEOR`, `Levenstein distance`

#### Semantic similarity
* **Key Points:**
  - "TL;DR: Using pre-trained embedding models for semantic matching."
  - "Both exact match and overlap-based metrics compare words and elements of responses, but they don't consider their meaning. But that's what we often care about!"
  - "Semantic similarity methods help compare meaning instead of words. They use pre-trained models like BERT, many of which are open-source. These models turn text into vectors that capture the context and relationships between words. By comparing the distance between these vectors, you can get a sense of how similar the texts really are."
  - "The limitation of these LLM evaluation methods is that they depend entirely on the embedding model you use. It may not capture the nuances of meaning well, and false matches are possible. Focusing only on the token- or sentence-level comparisons can also sometimes miss the broader context of the word passage."
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `BERTScore`, `MoverScore`, `COMET`

#### LLM as a judge
* **Key Points:**
  - "TL;DR: Prompting an LLM to compare responses or choose the best one."
  - "While making a semantic match ('Do these two responses mean the same thing?') is often the goal, embedding-based similarity isn't always the most precise option. Embeddings can capture general ideas but miss important details."
  - "To get better results, you can use an LLM as a judge for similarity matching."
  - "The process is close to how you use LLMs in your product — except here, the task is a narrow classification. For instance, you could pass a reference and new response to LLM and ask: 'Does the RESPONSE convey the same meaning as the REFERENCE? Yes or no'."
  - "This method is very adaptable. You can specify what to prioritize in matching — such as precise terminology, omissions, or style consistency. You can also ask the LLM to explain its reasoning. This makes the results easier to parse and improves the quality of the evals."
  - "Another way to use LLMs is through pairwise comparison, where you give the LLM the two responses and ask it to determine which is better. This is where the nickname 'LLM as a Judge' comes from (see Zheng et al, 2023)."
  - "Both methods might require tuning to match your preferences: an LLM judge requires its own evaluation! Pairwise comparisons, in particular, can also be biased — LLMs may favor longer or more polished responses or outputs similar to their own."
* **Technical Entities (Classes/Functions/APIs):** `LLM as a judge`, `Similarly matching`, `Pairwise comparison`

### Reference-free evals
* **Key Points:**
  - "If matching two answers is not an option, use reference-free evaluations. This applies to: Production LLM monitoring: live, ongoing checks and guardrails. Multi-turn tasks: chatbots or agents where it's hard to design a reference. Nuanced criteria: assessing tone, style, or other subjective qualities. High-volume evals: testing criteria like response safety or format adherence on large synthetic datasets."
  - "In these cases, you can measure custom qualities or use proxy metrics — quantifiable properties that say something useful about the output, starting with simple text length."
  - "It's worth noting that while we often refer to checks like 'average helpfulness' as LLM quality metrics, they don't have a strict mathematical definition like NDCG or precision. You tailor their implementation to your use case, and that's where most of the work goes."
* **Technical Entities (Classes/Functions/APIs):** None specified

#### Regular expressions
* **Key Points:**
  - "TL;DR. Tracking frequency of words and patterns, like topical or risky keywords."
  - "Regular expressions are a simple way to check for specific keywords, phrases, or structures in text. While they might seem basic they can also be surprisingly useful."
  - "Regex helps with structure checks, too. If your output needs specific elements (e.g., headers or disclaimers in the generated report), string matching can confirm that all sections are present."
  - "Once you define your patterns, regex can efficiently scan large datasets. It's fast, reliable, and doesn't require external API calls. Plus, it adds a welcome touch of certainty in an otherwise unpredictable field — if the pattern exists, regex will detect it."
  - "That said, regex has its limitations. It's strict by nature, meaning it won't catch typos or variations unless you explicitly define them. You'll need to carefully design and maintain your pattern definitions to cover the full range of possible outputs."
* **Technical Entities (Classes/Functions/APIs):** `Regular expressions`

#### Text stats
* **Key Points:**
  - "TL;DR. Tracking statistics like word or sentence count."
  - "You can often tie quality to measurable text features. For instance, text length matters for tasks like generating social media posts, help articles, or summaries — outputs should be concise but still meaningful."
  - "Checks like word and sentence count are quick, cheap, and easy to implement, making them useful at every stage. In LLM regression testing: are all outputs within expected limits? In LLM monitoring: what's the average, minimum, or maximum length? Individual outliers can often signal problems, like longer prompts containing jailbreaks."
  - "You can also explore metrics like readability scores. These provide a rough estimate of the education level someone would need to understand the text."
* **Technical Entities (Classes/Functions/APIs):** `Word, sentence, or symbol count`, `Non-letter character count`, `Language detection`, `Stopword ratio`, `Readability scores (like Flesch-Kincaid)`, `Named entity count`

#### Deterministic validation
* **Key Points:**
  - "TL;DR. Validating format and structure for tasks like code or JSON generation."
  - "There are other programmatic checks beyond regex and text stats. If your LLM generates structured content like SQL, code, JSON, or interacts with APIs and databases, you can often write code to verify correctness at least partially."
  - "These checks focus on format, structure, or functionality."
* **Technical Entities (Classes/Functions/APIs):** `Format adherence`, `Field completeness`, `Code execution success`, `Data quality validation`

#### Semantic similarity
* **Key Points:**
  - "TL;DR. Checking if responses align with inputs, context, or known patterns."
  - "Semantic similarity isn't just for reference matching. It's also useful when you don't have a ground truth response."
  - "Input-output similarity. For example, if you're turning bullet points into a full email, you can measure how closely it matches the original points. This helps check whether it stays true to the source."
  - "Response-context similarity. In RAG tasks, you can compare the generated answers to the retrieved context. High similarity would reflect that the model is properly using the information. Low similarity could suggest hallucination — when the model fabricates unsupported details."
  - "Pattern similarity. For example, you can compare a model's response to a set of denial templates. Even if the wording varies, high similarity can indicate that the response is a refusal."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `Response relevance`, `Response groundedness`, `Denial detection`

#### LLM-as-judge
* **Key Points:**
  - "TL;DR: Prompting LLMs to evaluate outputs against custom criteria."
  - "We already looked at reference-based LLM judges, but you can do the same for any custom property that you can't define through rules. Just explain your criteria in natural language, put them in a prompt, and let the LLM judge the output and return a score or a label."
  - "Such LLM judges help scale human labeling efforts and are one of the most universal evaluation methods."
  - "Direct evaluation. You can judge any standalone text qualities. For example, check if a chatbot response has the right tone or if user queries are appropriate."
  - "Context-based evaluation. You can also assess outputs alongside inputs or retrieved data. In this case, you pass two pieces of text to the evaluator."
  - "Conversation-level evaluation. You can also assess entire user sessions by asking the LLM to review transcripts. This is particularly useful for AI agents and chatbots."
  - "Several more complex approaches build on this idea. For example: SelfCheckGPT: Generates multiple responses to the same prompt and compares them for consistency, flagging hallucinations if answers diverge. (Manakul et al., 2023); G-EVAL: Combines CoT with form-based evaluations, where the model first generates step-by-step instructions tailored to specific criteria. (Liu et al., 2023); FineSurE: Decomposes complex evaluations (e.g., faithfulness, completeness) into discrete criteria and uses fact-by-fact checking. (Song et al., 2024); Juries of models: Asks several different LLMs at once. (Verga, et al., 2024)"
  - "Navigating these nicknames can be confusing: terms like G-Eval or GPTScore may sound like specific metrics, but they're actually prompting techniques for LLM evaluations. There are also fine-tuned evaluation models like Prometheus (Li et al., 2024)."
  - "Studies also show that complex techniques don't always yield better results. For example, simply asking the LLM to think step by step can outperform G-Eval.(Chiang et al, 2023). It's always best to test different approaches against your own assessments!"
  - "While LLM judges are powerful, their quality depends entirely on the prompt and underlying LLM. Just like system prompts, the evaluation prompts need to be refined and tested. Also, be cautious with domains like medicine or finance — generic LLMs may not provide reliable evaluations for specialized topics."
* **Technical Entities (Classes/Functions/APIs):** `LLM judges`, `SelfCheckGPT`, `G-EVAL`, `FineSurE`, `Juries of models`, `Prometheus`, `Helpfulness`, `Groundedness / Faithfulness`, `Politeness / Tone`, `Toxicity / Bias`, `Relevance`, `Safety`

#### Model-based scoring
* **Key Points:**
  - "TL;DR: Using pre-trained ML models to score your texts."
  - "To use machine learning for LLM evaluations, you don't always need a large, general-purpose LLM. Smaller models can often perform just as well for specific tasks."
  - "These models are trained on labeled data to predict or detect specific features."
  - "Many of these qualities apply broadly across use cases. Thanks to open-source communities, you can access publicly available pre-trained models. You can run them locally without incurring API costs or sending sensitive data externally."
  - "Additionally, if you have labeled data (or collected enough assessments from your LLM judges), you can train your own specialized models. By fine-tuning a pre-trained model with your examples, you can create a highly tailored evaluator that is cheap and fast to run."
  - "Of course, there's a catch: you need to know what each ML model does and test it against your criteria. The quality will vary if your data is very different from what the model was trained on."
* **Technical Entities (Classes/Functions/APIs):** `Topic classification`, `PII detection`, `Toxicity`, `Sentiment`, `Alignment (NLI model)`

## Takeaways
* **Key Points:**
  - "There are tons of LLM evaluation methods and metrics out there, but you won't need all of them for every app. Focus on what matters most based on your use case and the errors you're running into."
  - "First, always look at your data. Nothing builds intuition better than looking at real-world examples. This helps you spot patterns, shape your quality criteria, and understand failures. Bring your whole team in early — especially domain experts — and curate test data."
  - "Next, define what 'quality' means for your app. Consider the basics: response structure, length, and language. What are your positive indicators, like tone or helpfulness? What risks do you want to avoid, like toxic responses, denials, or irrelevant content?"
  - "Finally, choose your LLM evaluation methods wisely. Don't just go for a complex, obscure LLM metric you can't explain or match to your labels. Start by making your own qualitative assessments. Then, think back to metrics and decide how to implement them."
  - "When in doubt, start with LLM judges. They're widely used in practice and offer a flexible alternative to human review, especially for open-ended or session-level evaluations. Plus, they're easy to get started with — just write a custom prompt and refine it as needed."
* **Technical Entities (Classes/Functions/APIs):** `LLM judges`