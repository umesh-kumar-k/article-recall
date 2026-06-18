---
aliases:
  - Embedding Models
Source 1: https://medium.com/bright-ai/choosing-the-right-embedding-for-rag-in-generative-ai-applications-8cf5b36472e1
Source 2: https://www.mongodb.com/company/blog/technical/how-choose-best-embedding-model-for-your-llm-application
Source 3: https://milvus.io/ai-quick-reference/what-are-dense-and-sparse-embeddings
Source 4: https://mlokhandwalas.medium.com/dense-and-sparse-embeddings-a-comprehensive-overview-c5f6473ee9d0
tags:
  - embedding
  - embedding-models
  - rag
highlights: |-
  Transform text/data into dense vector representations 768/4096 dimensions , enabling semantic similarity search; choice impacts retrieval quality significantly


  OpenAI text-embedding-3: 1536 or 3073 dimensions; high quality; API dependency and cost

  Cohere Embed v3: Multilingual,optimized for RAG, compression-aware; competetive quality

  Azure OpenAI Embeddings: OpenAI models with enterprise SLA and data residency guarantees

  BGE(BAAI): Open-source SOTA models (bge-large-en-v1.5); self-hostable on GPUs

  E5(Microsoft): Instruction-turned embeddings; string zero shot cross domain performance

  Voyage AI: Specialized domain embeddings(code,finance, legal);higher accuracy than general-purpose

  Sentence Transformers: Ecosystem for fine-tuning custom domain embeddings; requires ML expertise
---
# Dense and Sparse Embeddings: A Comprehensive Overview

## Understanding Embeddings
* **Key Points:**
  - "Embeddings are vectors that represent data in a continuous space."
  - "In NLP, embeddings typically represent words, phrases, or sentences, allowing them to be processed mathematically."
  - "The key idea is to convert complex, high-dimensional data into lower-dimensional vectors while preserving meaningful relationships."
  - "In word embeddings, words that are semantically similar (like 'king' and 'queen') should have vector representations that are close to each other in the embedding space."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Introduction to Sparse Embeddings
* **Key Points:**
  - "Sparse embeddings are a type of embedding where the majority of values in the vector are zero."
  - "These embeddings are generally high-dimensional, with most dimensions inactive or zero."
  - "Sparse embeddings were among the earliest forms of word representation in NLP, prominently seen in techniques like one-hot encoding and bag-of-words models."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Key Characteristics of Sparse Embeddings
* **Key Points:**
  - "High Dimensionality: Sparse embeddings often involve vectors with thousands or even millions of dimensions."
  - "Locality: Sparse embeddings tend to represent each word independently. There is little overlap or shared information between the embeddings for different words."
  - "Interpretability: Sparse embeddings are more interpretable since each dimension usually represents a specific feature (e.g., the presence or absence of a specific word)."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Techniques Involving Sparse Embeddings
* **Key Points:**
  - "One-Hot Encoding: A simple form of sparse embedding where each word in the vocabulary is represented as a vector of zeros with a single '1' in the position corresponding to the word."
  - "Bag-of-Words (BoW): A technique where each document is represented as a sparse vector, with each dimension corresponding to the count of a particular word in the document."
  - "TF-IDF (Term Frequency-Inverse Document Frequency): A technique that enhances BoW by assigning weights to words based on their frequency in a document relative to their frequency across all documents."
* **Technical Entities (Classes/Functions/APIs):** `One-Hot Encoding`, `Bag-of-Words (BoW)`, `TF-IDF (Term Frequency-Inverse Document Frequency)`
* **Code Snippet:** None

---

## Advantages of Sparse Embeddings
* **Key Points:**
  - "Simplicity and Interpretability: Sparse embeddings are straightforward to implement and understand."
  - "Exact Representation: Sparse embeddings capture the exact presence or absence of features (e.g., words) without approximations."
  - "Good for High-Dimensional Data: When dealing with large vocabularies, sparse embeddings can be more memory-efficient because most of the values are zero and can be stored using sparse matrix representations."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Limitations of Sparse Embeddings
* **Key Points:**
  - "High Dimensionality: The large number of dimensions can be computationally expensive and lead to inefficiencies in storage and processing."
  - "Lack of Semantic Information: Sparse embeddings do not inherently capture semantic relationships between words."
  - "Poor Generalization: Sparse embeddings do not generalize well to unseen words or documents, making them less effective in tasks that require semantic understanding."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Introduction to Dense Embeddings
* **Key Points:**
  - "Dense embeddings, in contrast, represent data as lower-dimensional vectors where every value is non-zero (or near-zero)."
  - "These embeddings are compact and encode more complex relationships between data points."
  - "Dense embeddings became popular with the advent of techniques like word2vec, GloVe, and, more recently, transformer-based models like BERT and GPT."
* **Technical Entities (Classes/Functions/APIs):** `word2vec`, `GloVe`, `BERT`, `GPT`
* **Code Snippet:** None

---

## Key Characteristics of Dense Embeddings
* **Key Points:**
  - "Low Dimensionality: Dense embeddings are typically much lower in dimensionality compared to sparse embeddings, often ranging from 50 to 1,000 dimensions."
  - "Distributed Representations: Unlike sparse embeddings, where each dimension represents a distinct feature, dense embeddings distribute information across all dimensions."
  - "Semantic Relationships: Dense embeddings capture semantic relationships between data points."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Techniques Involving Dense Embeddings
* **Key Points:**
  - "word2vec: A popular method introduced by Google, word2vec learns dense embeddings by predicting the context of words in a corpus (Skip-gram) or by predicting a word given its context (CBOW)."
  - "GloVe (Global Vectors for Word Representation): GloVe captures global word co-occurrence statistics and produces dense embeddings that reflect the relationships between words."
  - "BERT (Bidirectional Encoder Representations from Transformers): BERT uses transformer networks to generate dense contextual embeddings, where the meaning of a word depends on its surrounding context."
* **Technical Entities (Classes/Functions/APIs):** `word2vec`, `Skip-gram`, `CBOW`, `GloVe (Global Vectors for Word Representation)`, `BERT (Bidirectional Encoder Representations from Transformers)`
* **Code Snippet:** None

---

## Advantages of Dense Embeddings
* **Key Points:**
  - "Compact Representations: Dense embeddings require fewer dimensions, making them more memory-efficient and faster to process than sparse embeddings."
  - "Semantic Richness: Dense embeddings capture nuanced relationships between words, such as analogies (e.g., 'king' is to 'queen' as 'man' is to 'woman')."
  - "Generalization: Dense embeddings can generalize better to new data, as they capture underlying patterns and relationships rather than just surface-level features."
  - "Adaptability to Complex Models: Dense embeddings are well-suited for deep learning models, enabling more sophisticated applications like sentiment analysis, machine translation, and text summarization."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Limitations of Dense Embeddings
* **Key Points:**
  - "Lack of Interpretability: Dense embeddings are less interpretable than sparse embeddings. It is difficult to pinpoint what each dimension represents, making them more of a 'black box.'"
  - "Training Complexity: Generating dense embeddings often requires complex training procedures and large datasets."
  - "Overfitting Risk: Dense embeddings, particularly in overparameterized models, can be prone to overfitting if not properly regularized."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Key Differences Between Sparse and Dense Embedding
* **Key Points:**
  - (No text provided; image reference present)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Use Cases and Applications
* **Key Points:**
  - "Sparse Embeddings: Document Classification: Sparse embeddings like TF-IDF are often used in traditional document classification tasks, especially when interpretability is important."
  - "Sparse Embeddings: Search and Information Retrieval: Sparse embeddings are widely used in search engines, where the focus is on exact keyword matching and relevance scoring."
  - "Sparse Embeddings: Text Mining and Feature Extraction: In exploratory text mining, sparse embeddings can be useful for feature extraction and identifying the most important terms in a corpus."
  - "Dense Embeddings: Sentiment Analysis: Dense embeddings are crucial in sentiment analysis, enabling models to understand nuanced expressions of sentiment that sparse methods miss."
  - "Dense Embeddings: Machine Translation: Dense embeddings form the backbone of modern machine translation systems, allowing them to capture complex relationships between languages."
  - "Dense Embeddings: Question Answering and Chatbots: Dense embeddings are used in conversational AI to understand context and generate coherent responses."
  - "Dense Embeddings: Recommendation Systems: Dense embeddings are applied in recommendation systems to capture user preferences and item features in a unified space."
* **Technical Entities (Classes/Functions/APIs):** `TF-IDF`
* **Code Snippet:** None

---

## Performance Trade-offs
* **Key Points:**
  - "Storage and Computational Requirements: Sparse embeddings can be more storage-efficient when dealing with extremely large vocabularies due to their high dimensionality but sparse nature."
  - "Dense embeddings, while requiring more complex models to generate, offer better computational efficiency in many scenarios due to their lower dimensionality."
  - "Interpretability vs. Expressiveness: Sparse embeddings provide clear, interpretable representations, making them useful in applications where transparency is key."
  - "Dense embeddings, however, excel in capturing complex, abstract relationships that sparse embeddings miss."
  - "Model Generalization: Dense embeddings typically generalize better across tasks and domains."
  - "Sparse embeddings, while effective in well-defined domains, struggle with tasks requiring broad generalization."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Future Trends and Developments
* **Key Points:**
  - "Hybrid Models: Combining the strengths of sparse and dense embeddings is an area of active research."
  - "Contextual Embeddings: Advances like BERT and GPT continue to push the boundaries of dense embeddings, enabling even deeper understanding of language."
  - "Multimodal Embeddings: There is increasing interest in embeddings that can incorporate multiple modalities, such as text, images, and audio, within a unified embedding space."
  - "Efficient Training and Inference: As dense embeddings become more prevalent, there is a growing focus on optimizing the training and inference processes to make these methods more accessible and less resource-intensive."
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `GPT`
* **Code Snippet:** None

---

## Conclusion
* **Key Points:**
  - "Both sparse and dense embeddings play crucial roles in the landscape of NLP and machine learning."
  - "Sparse embeddings, with their high dimensionality and interpretability, remain relevant in traditional applications like information retrieval and feature extraction."
  - "Dense embeddings, on the other hand, offer powerful tools for capturing rich semantic relationships, making them indispensable in modern NLP tasks such as translation, sentiment analysis, and conversational AI."
  - "The choice between sparse and dense embeddings depends on the specific requirements of the task, including factors like interpretability, dimensionality, and the need for semantic understanding."
  - "As research continues, the line between these two approaches is blurring, with hybrid models and context-aware embeddings becoming increasingly prominent."
  - "Regardless of the approach, embeddings will remain at the heart of how machines understand and process language."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None


---


# Choosing the Right Embedding Model for RAG in Generative AI

## Recap
* **Key Points:**
  - "Embedding models create fixed-length vector representations of text, focusing on semantic meaning for tasks like similarity comparison."
  - "LLMs (Large Language Models) are generative AI models that can understand and produce general language tasks and has more flexibility of input/output formats"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Use Case Scenario
* **Key Points:**
  - "A user reaches out to a customer support chatbot with an issue related to their account. The issue is logged in the company's data warehouse. The Gen AI chatbot needs to retrieve relevant information from the data warehouse to diagnose and resolve the issue."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## A. Embedding Models
### 1. Static Embeddings
* **Key Points:**
  - "Static embeddings generate fixed vector representations for each word in the vocabulary, regardless of the context or order in which the word appears."
  - "While contextual embeddings, produces different vectors for the same word based on its context within a sentence."
  - "Polysemy Issue: Words with multiple meanings (e.g., 'bank') have the same vector regardless of context [river bank, financial bank]"
  - "Context Insensitivity once embeddings are generated: Cannot differentiate between 'access denied' due to various reasons (e.g., incorrect password, account lockout)."
* **Technical Entities (Classes/Functions/APIs):** `Word2Vec`, `GloVE`, `Doc2Vec`, `TF-IDF`
* **Code Snippet:** None

---

### 2. Contextual Embeddings
* **Key Points:**
  - "BERT, RoBERTa, SBERT, ColBERT, MPNet"
  - "Bidirectional: Captures context from both directions within a sentence, leading to a deep understanding of the entire sentence."
  - "Focused Context: Primarily designed for understanding the context within relatively short spans of text (e.g., sentences or paragraphs)."
  - "BERT, RoBERTa , all-MiniLM-L6-v2 or SBERT (Masked language Model), Paraphrase-MPNet-Base-v2 (Permutated Language Model) embeddings capture the context and understand that 'can't access my account' is related to 'access denied' and 'cannot login' because they all involve issues with account access. Good choice for retrieval step"
  - "ColBERT (Contextualized Late Interaction over BERT) is a retrieval model that uses BM25 for initial document retrieval and then applies BERT-based contextual embeddings for detailed re-ranking, optimizing both efficiency and contextual relevance in information retrieval tasks."
  - "Context Limitation: Masked and Permuted Language Model is good at understanding context within a given text span (like a sentence or paragraph), but it doesn't have the capacity to generate text or handle tasks beyond understanding and retrieving relevant documents."
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `RoBERTa`, `SBERT`, `ColBERT`, `MPNet`, `all-MiniLM-L6-v2`, `Paraphrase-MPNet-Base-v2`, `BM25`
* **Code Snippet:** None

---

### 3. GPT-Based Embeddings
* **Key Points:**
  - "Unidirectional: Captures context from the left side only, building understanding sequentially as it generates text."
  - "Broad Context: Can maintain coherence over longer text sequences, making them effective for generating extended passages of text."
  - "OpenAI's text-embedding-3 -large"
  - "google-gecko-text-embedding"
  - "amazon-titan"
  - "GTR-T5 is Google's open-source embedding model for semantic search using the T5 LLM as a base"
  - "E5 (v1 and v2) is the newest embedding model from Microsoft."
  - "Generative-based embeddings: Good for the generation step of RAG. They recognize that 'cannot login after password reset' and 'login failed after updating security settings' are related to 'can't access my account.' They can also generate relevant responses based on deeper understanding and broader context."
  - "Generative models like GPT can be more resource-intensive than purely contextual models like BERT."
* **Technical Entities (Classes/Functions/APIs):** `text-embedding-3-large` (OpenAI), `google-gecko-text-embedding`, `amazon-titan`, `GTR-T5` (Google), `E5 (v1 and v2)` (Microsoft), `T5`
* **Code Snippet:** None

---

## B. Large Language Models (LLMs)
* **Key Points:**
  - "Combine the retrieved information (embedding) for response generation by LLM models"
  - "Open AI GPT 4o"
  - "Google Gemini Pro"
  - "Anthropic Claude3.5 Sonner"
* **Technical Entities (Classes/Functions/APIs):** `GPT 4o` (OpenAI), `Gemini Pro` (Google), `Claude3.5 Sonnet` (Anthropic)
* **Code Snippet:** None

---

## Overall Summary
* **Key Points:**
  - (No text provided; image reference present)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Metrics for choosing Embeddings
### MTEB retrieval score (Huggingface Massive Text Embedding Benchmark)
* **Key Points:**
  - "ex: Google gecko > Open AI text embedding 3 large > miniLM (Sbert)"
  - "GTR-T5 (google's open source) is good MTEB retrieval score but slow"
* **Technical Entities (Classes/Functions/APIs):** `MTEB (Massive Text Embedding Benchmark)`, `Google gecko`, `Open AI text embedding 3 large`, `miniLM (Sbert)`, `GTR-T5`
* **Code Snippet:** None

---

### 2. Latency (Speed in response)
* **Key Points:**
  - "Latency is often measured at the 95th percentile(p95, not on a log scale)."
  - "OpenAI API p95 responses took almost a minute from GCP and almost 600 ms from AWS."
  - "ex: all-miniLM (Sbert) < Google gecko < Open AI text embedding 3 large"
  - "all-miniLM (Sbert) being a small model is faster, it is also default embedding for vector database like chroma, which makes their deployment easy if it works for your usecase"
* **Technical Entities (Classes/Functions/APIs):** `all-miniLM (Sbert)`, `Google gecko`, `Open AI text embedding 3 large`, `chroma`
* **Code Snippet:** None