---
aliases:
Source 1: https://docs.langchain.com/oss/javascript/integrations/embeddings#openai
Source 2: https://docs.langchain.com/oss/javascript/langchain/rag#storing-documents
Source 3: https://docs.langchain.com/oss/javascript/langchain/knowledge-base#2-embeddings
---

## Missing Code Snippet 1: OpenAI Embeddings Setup

This should have appeared under the **"Embeddings"** section after the provider list:

```typescript
import { OpenAIEmbeddings } from "@langchain/openai";

const embeddings = new OpenAIEmbeddings({
  model: "text-embedding-3-large"
});
const vector1 = await embeddings.embedQuery(documents[0].pageContent);
const vector2 = await embeddings.embedQuery(documents[1].pageContent);

assert vector1.length === vector2.length;
console.log(`Generated vectors of length ${vector1.length}\n`);
console.log(vector1.slice(0, 10));
// Generated vectors of length 1536
// [-0.008586574345827103, -0.03341241180896759, -0.008936782367527485, ...]
```

---

## Missing Code Snippet 2: Storing Documents with VectorStore

This should have appeared under the **"Storing documents"** section:

```typescript
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";
import { OpenAIEmbeddings } from "@langchain/openai";

const vectorStore = new MemoryVectorStore(
  new OpenAIEmbeddings({ model: "ollama:devstral-2" }),
);
await vectorStore.addDocuments(allSplits);
```

---

## Updated Complete Extraction with All Code Snippets

Here is the corrected full extraction with all code snippets included:

---

# Langchain Embeddings

## Embeddings
* **Key Points:**
  - Vector search is a common way to store and search over unstructured data (such as unstructured text). The idea is to store numeric vectors that are associated with the text. Given a query, we can embed it as a vector of the same dimension and use vector similarity metrics (such as cosine similarity) to identify related text.
  - LangChain supports embeddings from dozens of providers. These models specify how text should be converted into a numeric vector.
* **Technical Entities (Classes/Functions/APIs):** `OpenAI`, `Azure`, `AWS`, `VertexAI`, `MistralAI`, `Cohere`, `@langchain/openai`, `OpenAIEmbeddings`
* **Code Snippet:**
```typescript
import { OpenAIEmbeddings } from "@langchain/openai";

const embeddings = new OpenAIEmbeddings({
  model: "text-embedding-3-large"
});
const vector1 = await embeddings.embedQuery(documents[0].pageContent);
const vector2 = await embeddings.embedQuery(documents[1].pageContent);

assert vector1.length === vector2.length;
console.log(`Generated vectors of length ${vector1.length}\n`);
console.log(vector1.slice(0, 10));
// Generated vectors of length 1536
// [-0.008586574345827103, -0.03341241180896759, -0.008936782367527485, ...]
```

## Storing documents
* **Key Points:**
  - Now we need to index our 66 text chunks so that we can search over them at runtime. Following the semantic search tutorial, our approach is to embed the contents of each document split and insert these embeddings into a vector store. Given an input query, we can then use vector search to retrieve relevant documents.
  - We can embed and store all of our document splits in a single command using the vector store and embeddings model selected at the start of the tutorial.
* **Technical Entities (Classes/Functions/APIs):** `MemoryVectorStore`, `OpenAIEmbeddings`, `addDocuments`, `@langchain/classic/vectorstores/memory`, `@langchain/openai`
* **Code Snippet:**
```typescript
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";
import { OpenAIEmbeddings } from "@langchain/openai";

const vectorStore = new MemoryVectorStore(
  new OpenAIEmbeddings({ model: "ollama:devstral-2" }),
);
await vectorStore.addDocuments(allSplits);
```

## Go deeper
* **Key Points:**
  - Embeddings: Wrapper around a text embedding model, used for converting text to embeddings.
  - Integrations: 30+ integrations to choose from.
  - Interface: API reference for the base interface.
  - VectorStore: Wrapper around a vector database, used for storing and querying embeddings.
  - Integrations: 40+ integrations to choose from.
  - Interface: API reference for the base interface.
  - This completes the Indexing portion of the pipeline. At this point we have a query-able vector store containing the chunked contents of our blog post. Given a user question, we should ideally be able to return the snippets of the blog post that answer the question.

## Embedding model integrations
* **Key Points:**
  - Embedding models transform raw text—such as a sentence, paragraph, or tweet—into a fixed-length vector of numbers that captures its semantic meaning. These vectors allow machines to compare and search text based on meaning rather than exact words.
  - In practice, this means that texts with similar ideas are placed close together in the vector space. For example, instead of matching only the phrase "machine learning", embeddings can surface documents that discuss related concepts even when different wording is used.

### How it works
* **Key Points:**
  - Vectorization — The model encodes each input string as a high-dimensional vector.
  - Similarity scoring — Vectors are compared using mathematical metrics to measure how closely related the underlying texts are.

### Similarity metrics
* **Key Points:**
  - Several metrics are commonly used to compare embeddings:
    - Cosine similarity — measures the angle between two vectors.
    - Euclidean distance — measures the straight-line distance between points.
    - Dot product — measures how much one vector projects onto another.

### Interface
* **Key Points:**
  - LangChain provides a standard interface for text embedding models (e.g., OpenAI, Cohere, Hugging Face) via the Embeddings interface.
  - Two main methods are available:
    - embedDocuments(documents: string[]) → number[][]: Embeds a list of documents.
    - embedQuery(text: string) → number[]: Embeds a single query.
  - The interface allows queries and documents to be embedded with different strategies, though most providers handle them the same way in practice.

### Caching
* **Key Points:**
  - Embeddings can be stored or temporarily cached to avoid needing to recompute them.
  - Caching embeddings can be done using a CacheBackedEmbeddings. This wrapper stores embeddings in a key-value store, where the text is hashed and the hash is used as the key in the cache.
  - The main supported way to initialize a CacheBackedEmbeddings is fromBytesStore. It takes the following parameters:
    - underlyingEmbeddings: The embedder to use for embedding.
    - documentEmbeddingStore: Any BaseStore for caching document embeddings.
    - options.namespace: (optional, defaults to "") The namespace to use for the document cache. Helps avoid collisions (e.g., set it to the embedding model name).
  - In production, you would typically use a more robust persistent store, such as a database or cloud storage. Please see stores integrations for options.
* **Technical Entities (Classes/Functions/APIs):** `CacheBackedEmbeddings`, `BaseStore`, `fromBytesStore`, `@langchain/classic/embeddings/cache_backed`, `InMemoryStore`, `@langchain/core/stores`
* **Code Snippet:**
```typescript
import { CacheBackedEmbeddings } from "@langchain/classic/embeddings/cache_backed";
import { InMemoryStore } from "@langchain/core/stores";

const underlyingEmbeddings = new OpenAIEmbeddings();

const inMemoryStore = new InMemoryStore();

const cacheBackedEmbeddings = CacheBackedEmbeddings.fromBytesStore(
  underlyingEmbeddings,
  inMemoryStore,
  {
    namespace: underlyingEmbeddings.model,
  }
);

// Example: caching a query embedding
const tic = Date.now();
const queryEmbedding = cacheBackedEmbeddings.embedQuery("Hello, world!");
console.log(`First call took: ${Date.now() - tic}ms`);

// Example: caching a document embedding
const tic = Date.now();
const documentEmbedding = cacheBackedEmbeddings.embedDocuments(["Hello, world!"]);
console.log(`Cached creation time: ${Date.now() - tic}ms`);
```

---

I will ensure future extractions include **all code snippets** from the article. Let me know if you need any other sections re-extracted with the full code!