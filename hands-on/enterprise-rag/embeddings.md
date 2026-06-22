---
aliases:
  - Embeddings
Source 1: https://medium.com/bright-ai/choosing-the-right-embedding-for-rag-in-generative-ai-applications-8cf5b36472e1
Source 2: https://www.sbert.net/docs/quickstart.html
Source 3: https://ragwalla.com/blog/choosing-between-cosine-similarity-dot-product-and-euclidean-distance-for-rag-applications
---
[Code Examples](./code/document-embeddings.md) 
# Choosing the Right Embedding Model for RAG in Generative AI

## Embedding Models are the key to effective Retrieval Augmented Generation (RAG)
* **Key Points:**
  - Embedding models create fixed-length vector representations of text, focusing on semantic meaning for tasks like similarity comparison.
  - LLMs (Large Language Models) are generative AI models that can understand and produce general language tasks and has more flexibility of input/output formats

## A. Embedding Models

### 1. Static Embeddings
* **Key Points:**
  - Static embeddings generate fixed vector representations for each word in the vocabulary, regardless of the context or order in which the word appears. While contextual embeddings, produces different vectors for the same word based on its context within a sentence.
  - With Word2Vec, GloVE, Doc2Vec (Dense vector based) and TF-IDF (keyword /Sparse vector based), the vectors for "access" and "account" in both the query and the log will be similar, returning relevant results based on cosine similarity
  - Limitations:
    - Polysemy Issue: Words with multiple meanings (e.g., "bank") have the same vector regardless of context [river bank, financial bank]
    - Context Insensitivity once embeddings are generated: Cannot differentiate between "access denied" due to various reasons (e.g., incorrect password, account lockout).
* **Technical Entities (Classes/Functions/APIs):** `Word2Vec`, `GloVE`, `Doc2Vec`, `TF-IDF`

### 2. Contextual Embeddings
* **Key Points:**
  - BERT, RoBERTa, SBERT, ColBERT, MPNet
  - Bidirectional: Captures context from both directions within a sentence, leading to a deep understanding of the entire sentence.
  - Focused Context: Primarily designed for understanding the context within relatively short spans of text (e.g., sentences or paragraphs).
  - BERT, RoBERTa , all-MiniLM-L6-v2 or SBERT (Masked language Model), Paraphrase-MPNet-Base-v2 (Permutated Language Model) embeddings capture the context and understand that "can't access my account" is related to "access denied" and "cannot login" because they all involve issues with account access. Good choice for retrieval step
  - ColBERT (Contextualized Late Interaction over BERT) is a retrieval model that uses BM25 for initial document retrieval and then applies BERT-based contextual embeddings for detailed re-ranking, optimizing both efficiency and contextual relevance in information retrieval tasks.
  - Limitations:
    - Context Limitation: Masked and Permuted Language Model is good at understanding context within a given text span (like a sentence or paragraph), but it doesn't have the capacity to generate text or handle tasks beyond understanding and retrieving relevant documents.
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `RoBERTa`, `SBERT`, `ColBERT`, `MPNet`, `all-MiniLM-L6-v2`, `Paraphrase-MPNet-Base-v2`, `BM25`

### 3. GPT-Based Embeddings
* **Key Points:**
  - Unidirectional: Captures context from the left side only, building understanding sequentially as it generates text.
  - Broad Context: Can maintain coherence over longer text sequences, making them effective for generating extended passages of text.
  - OpenAI's text-embedding-3 -large; google-gecko-text-embedding; amazon-titan; GTR-T5 is Google's open-source embedding model for semantic search using the T5 LLM as a base; E5 (v1 and v2) is the newest embedding model from Microsoft.
  - Generative-based embeddings: Good for the generation step of RAG. They recognize that "cannot login after password reset" and "login failed after updating security settings" are related to "can't access my account." They can also generate relevant responses based on deeper understanding and broader context.
  - Limitations:
    - Generative models like GPT can be more resource-intensive than purely contextual models like BERT.
* **Technical Entities (Classes/Functions/APIs):** `OpenAI text-embedding-3-large`, `google-gecko-text-embedding`, `amazon-titan`, `GTR-T5`, `E5 (v1 and v2)`, `T5`

## B. Large Language Models (LLMs)
* **Key Points:**
  - Combine the retrieved information (embedding) for response generation by LLM models: Open AI GPT 4o; Google Gemini Pro; Anthropic Claude3.5 Sonner
* **Technical Entities (Classes/Functions/APIs):** `Open AI GPT 4o`, `Google Gemini Pro`, `Anthropic Claude3.5 Sonner`

## Metrics for choosing Embeddings
* **Key Points:**
  - MTEB retrieval score (Huggingface Massive Text Embedding Benchmark): ex: Google gecko > Open AI text embedding 3 large > miniLM (Sbert); GTR-T5 (google's open source) is good MTEB retrieval score but slow
  - Latency (Speed in response): Latency is often measured at the 95th percentile(p95, not on a log scale). OpenAI API p95 responses took almost a minute from GCP and almost 600 ms from AWS. ex: all-miniLM (Sbert) < Google gecko < Open AI text embedding 3 large; all-miniLM (Sbert) being a small model is faster, it is also default embedding for vector database like chroma, which makes their deployment easy if it works for your usecase
* **Technical Entities (Classes/Functions/APIs):** `MTEB retrieval score (Huggingface Massive Text Embedding Benchmark)`, `95th percentile (p95)`, `chroma`


---

# Choosing Between Cosine Similarity, Dot Product, and Euclidean Distance for RAG Applications

## Understanding Vector Similarity
* **Key Points:**
  - In RAG workflows, we store document or text embeddings as vectors in a database. When we want to retrieve the most relevant items from our stored vectors, we need a way to measure how close (or similar) these vectors are to each other. Three common approaches are: Cosine Similarity, Dot Product, Euclidean Distance.

## 1. Cosine Similarity
* **Key Points:**
  - Cosine similarity looks at the angle between two vectors rather than their overall magnitude. Imagine you have two arrows on a dartboard. The difference in their directions (angles) tells you how "similar" they are, irrespective of how long each arrow is.
  - When to Use Cosine Similarity:
    - Comparing Text Documents: Often used in NLP tasks because two documents can be similar in topic (direction of the arrow) even if one is much longer than the other (length of the arrow).
    - Ignoring Magnitude Differences: If you only care about the direction or overall pattern, cosine similarity is a strong choice.
  - Cosine Similarity Example: Suppose you have a short marketing blurb and a lengthy press release on the same subject. Their word counts differ wildly, but they're talking about the same topic. Cosine similarity focuses on the overlapping keywords and themes rather than total word count, making it perfect for identifying that both are indeed about, say, "new product launches."

## 2. Dot Product
* **Key Points:**
  - Dot product measures a combination of direction and magnitude. If you multiply the lengths of two vectors and the cosine of the angle between them, you get the dot product. This means if your vectors are large and align closely, you get a higher dot product value.
  - When to Use Dot Product:
    - Weighted Matches: If the magnitude (length of the embedding) is significant—maybe you really do care that one document is larger or has higher "importance" in some feature dimension.
    - Neural Network Outputs: Often used in final layers or attention mechanisms where magnitude of embeddings is deliberately scaled or normalized in certain ways.
  - Dot Product Example: Picture a scenario where each vector encodes not just the direction (topic) but also how frequently that topic is discussed in the content. A bigger vector might mean the topic is covered more in-depth. If both vectors are huge and point in roughly the same direction, the dot product will be very large—great for capturing that strong alignment in both topic and quantity.

## 3. Euclidean Distance
* **Key Points:**
  - Euclidean distance is the straight-line distance between two points in a multi-dimensional space—just like a ruler in geometry class. If you imagine each vector as a point on a graph, Euclidean tells you exactly how far one point is from the other.
  - When to Use Euclidean Distance:
    - Spatial/Geometric Concepts: When your data naturally fits a scenario where direct distance matters. Think of a GPS coordinate system, or an image pixel comparison.
    - Clustering: Many clustering algorithms (like k-means) rely on Euclidean distance to group similar data points together.
  - Euclidean Distance Example: Consider you're clustering customer preferences in a 2D space: one axis is "preferred price range," and the other is "interest in premium features." If two customers fall close to each other in this coordinate system, Euclidean distance will confirm they're likely to have similar preferences.

## Making Your Choice
* **Key Points:**
  - Cosine Similarity: Great for text or any scenario where direction matters more than magnitude. "We care about how similar the themes or angles are, ignoring length."
  - Dot Product: Useful when magnitude is also crucial. "Two large vectors that point in the same direction should rank higher than two small ones."
  - Euclidean Distance: Ideal for geometric-style problems or clustering. "We want to know how far apart two points are, literally, in multidimensional space."

## Summary
* **Key Points:**
  - Choosing the right similarity measure depends on your use case. If you're dealing with text embeddings and want to isolate direction over magnitude, go with Cosine Similarity. When overall vector size or weighting matters, Dot Product shines. And if your model benefits from real geometric distances—like clustering points in space—Euclidean Distance is the way to go.

---


# SBert Transformers

## Sentence Transformer
* **Key Points:**
  - Characteristics of Sentence Transformer (a.k.a bi-encoder) models:
    - Calculates a fixed-size vector representation (embedding) given texts, images, audio, or video.
    - Embedding calculation is often efficient, embedding similarity calculation is very fast.
    - Applicable for a wide range of tasks, such as semantic textual similarity, semantic search, clustering, classification, paraphrase mining, and more.
    - Often used as a first step in a two-step retrieval process, where a Cross-Encoder (a.k.a. reranker) model is used to re-rank the top-k results from the bi-encoder.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `SentenceTransformer.encode`, `SentenceTransformer.encode_query`, `SentenceTransformer.encode_document`, `SentenceTransformer.similarity`, `SentenceTransformer.similarity_pairwise`
* **Code Snippet:**
```python
from sentence_transformers import SentenceTransformer

# 1. Load a pretrained Sentence Transformer model
model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

# The sentences to encode
sentences = [
    "The weather is lovely today.",
    "It's so sunny outside!",
    "He drove to the stadium.",
]

# 2. Calculate embeddings by calling model.encode()
embeddings = model.encode(sentences)
print(embeddings.shape)
# [3, 384]

# 3. Calculate the embedding similarities
similarities = model.similarity(embeddings, embeddings)
print(similarities)
# tensor([[1.0000, 0.6660, 0.1046],
#         [0.6660, 1.0000, 0.1411],
#         [0.1046, 0.1411, 1.0000]])
```

## Multimodal
* **Key Points:**
  - With SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2") we pick which Sentence Transformer model we load. In this example, we load sentence-transformers/all-MiniLM-L6-v2, which is a MiniLM model finetuned on a large dataset of over 1 billion training pairs.
  - Using SentenceTransformer.similarity(), we compute the similarity between all pairs of sentences. As expected, the similarity between semantically related inputs is higher than between unrelated ones.
  - Multimodal models like Qwen/Qwen3-VL-Embedding-2B can also encode images, audio, or video into the same embedding space.
  - Finetuning Sentence Transformer models is easy and requires only a few lines of code. For more information, see the Training Overview section.
  - Read Sentence Transformer > Usage > Speeding up Inference for tips on how to speed up inference of models by up to 2x-3x.

## Cross Encoder
* **Key Points:**
  - Characteristics of Cross Encoder (a.k.a reranker) models:
    - Calculates a similarity score given pairs of inputs (typically text, but also images or other modalities).
    - Generally provides superior performance compared to a Sentence Transformer (a.k.a. bi-encoder) model.
    - Often slower than a Sentence Transformer model, as it requires computation for each pair rather than each text.
    - Due to the previous 2 characteristics, Cross Encoders are often used to re-rank the top-k results from a Sentence Transformer model.
* **Technical Entities (Classes/Functions/APIs):** `CrossEncoder`, `CrossEncoder.rank`, `CrossEncoder.predict`
* **Code Snippet:**
```python
from sentence_transformers import CrossEncoder

# 1. Load a pretrained CrossEncoder model
model = CrossEncoder("cross-encoder/stsb-distilroberta-base")

# We want to compute the similarity between the query sentence...
query = "A man is eating pasta."

# ... and all sentences in the corpus
corpus = [
    "A man is eating food.",
    "A man is eating a piece of bread.",
    "The girl is carrying a baby.",
    "A man is riding a horse.",
    "A woman is playing violin.",
    "Two men pushed carts through the woods.",
    "A man is riding a white horse on an enclosed ground.",
    "A monkey is playing drums.",
    "A cheetah is running behind its prey.",
]

# 2. We rank all sentences in the corpus for the query
ranks = model.rank(query, corpus)

# Print the scores
print("Query: ", query)
for rank in ranks:
    print(f"{rank['score']:.2f}\t{corpus[rank['corpus_id']]}")
"""
Query:  A man is eating pasta.
0.67    A man is eating food.
0.34    A man is eating a piece of bread.
0.08    A man is riding a horse.
0.07    A man is riding a white horse on an enclosed ground.
0.01    The girl is carrying a baby.
0.01    Two men pushed carts through the woods.
0.01    A monkey is playing drums.
0.01    A woman is playing violin.
0.01    A cheetah is running behind its prey.
"""

# 3. Alternatively, you can also manually compute the score between two sentences
import numpy as np

sentence_combinations = [[query, sentence] for sentence in corpus]
scores = model.predict(sentence_combinations)

# Sort the scores in decreasing order to get the corpus indices
ranked_indices = np.argsort(scores)[::-1]
print("Scores:", scores)
print("Indices:", ranked_indices)
"""
Scores: [0.6732372, 0.34102544, 0.00542465, 0.07569341, 0.00525378, 0.00536814, 0.06676237, 0.00534825, 0.00516717]
Indices: [0 1 3 6 2 5 7 4 8]
"""
```

## Multimodal (Cross Encoder)
* **Key Points:**
  - With CrossEncoder("cross-encoder/stsb-distilroberta-base") we pick which CrossEncoder model we load. CrossEncoder models can also work with multimodal inputs: Qwen/Qwen3-VL-Reranker-2B can rank images and text by relevance to a query.
  - Finetuning CrossEncoder models is easy and requires only a few lines of code. For more information, see the Training Overview section.
  - Read CrossEncoder > Usage > Speeding up Inference for tips on how to speed up inference of models by up to 2x-3x.

## Sparse Encoder
* **Key Points:**
  - Characteristics of Sparse Encoder models:
    - Calculates sparse vector representations where most dimensions are zero.
    - Provides efficiency benefits for large-scale retrieval systems due to the sparse nature of embeddings.
    - Often more interpretable than dense embeddings, with non-zero dimensions corresponding to specific tokens.
    - Complementary to dense embeddings, enabling hybrid search systems that combine the strengths of both approaches.
* **Technical Entities (Classes/Functions/APIs):** `SparseEncoder`, `SparseEncoder.encode`, `SparseEncoder.similarity`, `SparseEncoder.sparsity`
* **Code Snippet:**
```python
from sentence_transformers import SparseEncoder

# 1. Load a pretrained SparseEncoder model
model = SparseEncoder("naver/splade-cocondenser-ensembledistil")

# The sentences to encode
sentences = [
    "The weather is lovely today.",
    "It's so sunny outside!",
    "He drove to the stadium.",
]

# 2. Calculate sparse embeddings by calling model.encode()
embeddings = model.encode(sentences)
print(embeddings.shape)
# [3, 30522] - sparse representation with vocabulary size dimensions

# 3. Calculate the embedding similarities (using dot product by default)
similarities = model.similarity(embeddings, embeddings)
print(similarities)
# tensor([[   35.629,     9.154,     0.098],
#         [    9.154,    27.478,     0.019],
#         [    0.098,     0.019,    29.553]])

# 4. Check sparsity statistics
stats = SparseEncoder.sparsity(embeddings)
print(f"Sparsity: {stats['sparsity_ratio']:.2%}")  # Typically >99% zeros
print(f"Avg non-zero dimensions per embedding: {stats['active_dims']:.2f}")
```

## Sparse Encoder Details
* **Key Points:**
  - With SparseEncoder("naver/splade-cocondenser-ensembledistil") we load a pretrained SPLADE model that generates sparse embeddings. SPLADE (SParse Lexical AnD Expansion) models use MLM prediction mechanisms to create sparse representations that are particularly effective for information retrieval tasks.
  - Finetuning Sparse Encoder models is easy and requires only a few lines of code. For more information, see the Training Overview section.
* **Technical Entities (Classes/Functions/APIs):** `SPLADE`