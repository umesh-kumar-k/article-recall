---
aliases:
  - Vector Search
Source 1: https://docs.langchain.com/oss/javascript/integrations/vectorstores#openai
Source 2: https://docs.langchain.com/oss/javascript/langchain/knowledge-base#3-vector-stores
Source 3: https://docs.langchain.com/oss/python/langgraph/stores#semantic-search
---
# Langchain Vector Search

## 3. Vector stores
* **Key Points:**
  - LangChain VectorStore objects contain methods for adding text and Document objects to the store, and querying them using various similarity metrics. They are often initialized with embedding models, which determine how text data is translated to numeric vectors.
  - LangChain includes a suite of integrations with different vector store technologies. Some vector stores are hosted by a provider and require specific credentials to use; some run in separate infrastructure that can be run locally or via a third-party; others can run in-memory for lightweight workloads.
* **Technical Entities (Classes/Functions/APIs):** `Memory`, `MongoDB`, `Pinecone`, `Qdrant`, `Redis`, `@langchain/classic`, `MemoryVectorStore`
* **Code Snippet:**
```typescript
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";

const vectorStore = new MemoryVectorStore(embeddings);
```

## Seeding the vector store
* **Key Points:**
  - Let's seed the store with content from a PDF. Here is a sample PDF — a 10-k filing for Nike from 2023. We'll read the PDF directly with a small helper and split it into smaller chunks before indexing.
  - A page may be too coarse a representation for retrieval and downstream question-answering. Further splitting helps ensure that the meanings of relevant portions of the document are not "washed out" by surrounding text. We use RecursiveCharacterTextSplitter, which recursively splits a document using common separators like new lines until each chunk is the appropriate size. This is the recommended text splitter for generic text use cases.
  - We can now index the chunks into the vector store.
  - Note that most vector store implementations will allow you to connect to an existing vector store— e.g., by providing a client, index name, or other information. See the documentation for a specific integration for more detail.
  - Once we've instantiated a VectorStore that contains documents, we can query it. VectorStore includes methods for querying: Synchronously and asynchronously; By string query and by vector; With and without returning similarity scores; By similarity and maximum marginal relevance (to balance similarity with query to diversity in retrieved results). The methods will generally include a list of Document objects in their outputs.
* **Technical Entities (Classes/Functions/APIs):** `readFileSync`, `Document`, `PDFParse`, `RecursiveCharacterTextSplitter`, `addDocuments`, `similaritySearch`, `similaritySearchWithScore`, `similaritySearchVectorWithScore`, `@langchain/core/documents`, `pdf-parse`, `@langchain/textsplitters`
* **Code Snippet:**
```typescript
// Loading PDF pages
import { readFileSync } from "node:fs";
import { Document } from "@langchain/core/documents";
import { PDFParse } from "pdf-parse";

// Below is a minimal helper for demonstration purposes.
async function loadPdfPages(filePath: string): Promise<Document[]> {
  const parser = new PDFParse({
    data: new Uint8Array(readFileSync(filePath)),
  });
  try {
    const { pages } = await parser.getText();
    return pages.map(
      (page) =>
        new Document({
          pageContent: page.text,
          metadata: { source: filePath, page: page.num - 1 },
        })
    );
  } finally {
    await parser.destroy();
  }
}

const filePath = "../../data/nke-10k-2023.pdf";
const docs = await loadPdfPages(filePath);
console.log(docs.length);
// 107

// Splitting documents
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

const allSplits = await textSplitter.splitDocuments(docs);

console.log(allSplits.length);
// 516

// Indexing chunks
await vectorStore.addDocuments(allSplits);
```

## Usage
* **Key Points:**
  - Embeddings typically represent text as a "dense" vector such that texts with similar meanings are geometrically close. This lets us retrieve relevant information just by passing in a question, without knowledge of any specific key-terms used in the document.
* **Technical Entities (Classes/Functions/APIs):** `similaritySearch`, `similaritySearchWithScore`, `similaritySearchVectorWithScore`, `embedQuery`
* **Code Snippet:**
```typescript
// Return documents based on similarity to a string query
const results1 = await vectorStore.similaritySearch(
  "When was Nike incorporated?"
);

console.log(results1[0]);
// Document {
//     pageContent: 'direct to consumer operations sell products...',
//     metadata: {'page': 4, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 3125}
// }

// Return scores
const results2 = await vectorStore.similaritySearchWithScore(
  "What was Nike's revenue in 2023?"
);

console.log(results2[0]);
// Score: 0.23699893057346344
// Document {
//     pageContent: 'Table of Contents...',
//     metadata: {'page': 35, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}
// }

// Return documents based on similarity to an embedded query
const embedding = await embeddings.embedQuery(
  "How were Nike's margins impacted in 2023?"
);

const results3 = await vectorStore.similaritySearchVectorWithScore(
  embedding,
  1
);

console.log(results3[0]);
// Document {
//     pageContent: 'FISCAL 2023 COMPARED TO FISCAL 2022...',
//     metadata: {
//         'page': 36,
//         'source': '../example_data/nke-10k-2023.pdf',
//         'start_index': 0
//     }
// }
```


---

# Vector store integrations

## Overview
* **Key Points:**
  - A vector store stores embedded data and performs similarity search.

## Interface
* **Key Points:**
  - LangChain provides a unified interface for vector stores, allowing you to:
    - addDocuments - Add documents to the store.
    - delete - Remove stored documents by ID.
    - similaritySearch - Query for semantically similar documents.
  - This abstraction lets you switch between different implementations without altering your application logic.

## Initialization
* **Key Points:**
  - Most vectorstores in LangChain accept an embedding model as an argument when initializing the vector store.
* **Technical Entities (Classes/Functions/APIs):** `OpenAIEmbeddings`, `MemoryVectorStore`, `@langchain/openai`, `@langchain/classic/vectorstores/memory`
* **Code Snippet:**
```typescript
import { OpenAIEmbeddings } from "@langchain/openai";
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";

const embeddings = new OpenAIEmbeddings({
  model: "text-embedding-3-small",
});
const vectorStore = new MemoryVectorStore(embeddings);
```

## Adding documents
* **Key Points:**
  - You can add documents to the vector store by using the addDocuments function.
* **Technical Entities (Classes/Functions/APIs):** `Document`, `addDocuments`, `@langchain/core/documents`
* **Code Snippet:**
```typescript
import { Document } from "@langchain/core/documents";
const document = new Document({
  pageContent: "Hello world",
});
await vectorStore.addDocuments([document]);
```

## Deleting documents
* **Key Points:**
  - You can delete documents from the vector store by using the delete function.
* **Technical Entities (Classes/Functions/APIs):** `delete`
* **Code Snippet:**
```typescript
await vectorStore.delete({
  filter: {
    pageContent: "Hello world",
  },
});
```

## Similarity search
* **Key Points:**
  - Issue a semantic query using similaritySearch, which returns the closest embedded documents.
  - Many vector stores support parameters like: k — number of results to return; filter — conditional filtering based on metadata
* **Technical Entities (Classes/Functions/APIs):** `similaritySearch`
* **Code Snippet:**
```typescript
const results = await vectorStore.similaritySearch("Hello world", 10);
```

## Similarity metrics & indexing
* **Key Points:**
  - Embedding similarity may be computed using: Cosine similarity; Euclidean distance; Dot product.
  - Efficient search often employs indexing methods such as HNSW (Hierarchical Navigable Small World), though specifics depend on the vector store.

## Metadata filtering
* **Key Points:**
  - Filtering by metadata (e.g., source, date) can refine search results.
* **Technical Entities (Classes/Functions/APIs):** `similaritySearch`
* **Code Snippet:**
```typescript
vectorStore.similaritySearch("query", 2, { source: "tweets" });
```


---

# Semantic search
* **Key Points:**
  - Beyond simple retrieval, the store also supports semantic search, allowing you to find memories based on meaning rather than exact matches. To enable this, configure the store with an embedding model.
  - Now when searching, you can use natural language queries to find relevant memories.
  - You can control which parts of your memories get embedded by configuring the fields parameter or by specifying the index parameter when storing memories.
* **Technical Entities (Classes/Functions/APIs):** `InMemoryStore`, `init_embeddings`, `store.search`, `store.put`, `index`
* **Code Snippet:**
```python
from langchain.embeddings import init_embeddings

store = InMemoryStore(
    index={
        "embed": init_embeddings("openai:text-embedding-3-small"),  # Embedding provider
        "dims": 1536,                              # Embedding dimensions
        "fields": ["food_preference", "$"]              # Fields to embed
    }
)

# Find memories about food preferences
memories = store.search(
    namespace_for_memory,
    query="What does the user like to eat?",
    limit=3  # Return top 3 matches
)

# Store with specific fields to embed
store.put(
    namespace_for_memory,
    str(uuid.uuid4()),
    {
        "food_preference": "I love Italian cuisine",
        "context": "Discussing dinner plans"
    },
    index=["food_preference"]  # Only embed "food_preferences" field
)

# Store without embedding (still retrievable, but not searchable)
store.put(
    namespace_for_memory,
    str(uuid.uuid4()),
    {"system_info": "Last updated: 2024-01-01"},
    index=False
)
```

