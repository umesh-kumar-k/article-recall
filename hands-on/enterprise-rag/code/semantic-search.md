---
aliases:
  - Semantic Search
Source 1: https://docs.langchain.com/oss/python/langchain/knowledge-base
---
# Build a semantic typescript/javascript search engine with LangChain

* **Key Points:**
  - This tutorial will familiarize you with LangChain's embedding and vector store abstractions.
  - These abstractions are designed to support retrieval of data— from (vector) databases and other sources — for integration with LLM workflows.
  - They are important for applications that fetch data to be reasoned over as part of model inference, as in the case of retrieval-augmented generation, or RAG.
  - Here we will build a search engine over a PDF document.
  - This will allow us to retrieve passages in the PDF that are similar to an input query.
  - The guide also includes a minimal RAG implementation on top of the search engine.

## Concepts
* **Key Points:**
  - This guide focuses on retrieval of text data.
  - We will cover the following concepts: Documents; Text splitters; Embeddings; Vector stores and retrievers.

## Setup
### Installation
* **Key Points:**
  - This guide reads a PDF using the pdf-parse package
* **Technical Entities (Classes/Functions/APIs):** `pdf-parse`

### LangSmith
* **Key Points:**
  - Many of the applications you build with LangChain will contain multiple steps with multiple invocations of LLM calls.
  - As these applications get more and more complex, it becomes crucial to be able to inspect what exactly is going on inside your chain or agent.
  - The best way to do this is with LangSmith.
* **Technical Entities (Classes/Functions/APIs):** `LangSmith`

## 1. Documents
* **Key Points:**
  - LangChain implements a Document abstraction, which is intended to represent a unit of text and associated metadata.
  - It has three attributes: pageContent: a string representing the content; metadata: a dict containing arbitrary metadata; id: (optional) a string identifier for the document.
  - The metadata attribute can capture information about the source of the document, its relationship to other documents, and other information.
  - Note that an individual Document object often represents a chunk of a larger document.
* **Technical Entities (Classes/Functions/APIs):** `Document`, `pageContent`, `metadata`, `id`, `@langchain/core/documents`
* **Code Snippet:**
```typescript
import { Document } from "@langchain/core/documents";

const documents = [
  new Document({
    pageContent:
      "Dogs are great companions, known for their loyalty and friendliness.",
    metadata: { source: "mammal-pets-doc" },
  }),
  new Document({
    pageContent: "Cats are independent pets that often enjoy their own space.",
    metadata: { source: "mammal-pets-doc" },
  }),
];
```

## 2. Embeddings
* **Key Points:**
  - Vector search is a common way to store and search over unstructured data (such as unstructured text).
  - The idea is to store numeric vectors that are associated with the text.
  - Given a query, we can embed it as a vector of the same dimension and use vector similarity metrics (such as cosine similarity) to identify related text.
  - LangChain supports embeddings from dozens of providers.
  - These models specify how text should be converted into a numeric vector.
* **Technical Entities (Classes/Functions/APIs):** `OpenAIEmbeddings`, `embedQuery()`, `@langchain/openai`
* **Code Snippet:**
```typescript
npm i @langchain/openai
```
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
```

## 3. Vector stores
* **Key Points:**
  - LangChain VectorStore objects contain methods for adding text and Document objects to the store, and querying them using various similarity metrics.
  - They are often initialized with embedding models, which determine how text data is translated to numeric vectors.
  - LangChain includes a suite of integrations with different vector store technologies.
* **Technical Entities (Classes/Functions/APIs):** `VectorStore`, `MemoryVectorStore`, `@langchain/classic/vectorstores/memory`
* **Code Snippet:**
```typescript
npm i @langchain/classic
```
```typescript
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";

const vectorStore = new MemoryVectorStore(embeddings);
```

### Seeding the vector store
* **Key Points:**
  - Let's seed the store with content from a PDF.
  - A page may be too coarse a representation for retrieval and downstream question-answering.
  - Further splitting helps ensure that the meanings of relevant portions of the document are not "washed out" by surrounding text.
  - We use RecursiveCharacterTextSplitter, which recursively splits a document using common separators like new lines until each chunk is the appropriate size.
  - This is the recommended text splitter for generic text use cases.
  - Note that most vector store implementations will allow you to connect to an existing vector store— e.g., by providing a client, index name, or other information.
  - Once we've instantiated a VectorStore that contains documents, we can query it.
  - VectorStore includes methods for querying: Synchronously and asynchronously; By string query and by vector; With and without returning similarity scores; By similarity and maximum marginal relevance (to balance similarity with query to diversity in retrieved results).
  - The methods will generally include a list of Document objects in their outputs.
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter`, `splitDocuments()`, `addDocuments()`, `similaritySearch()`, `similaritySearchWithScore()`, `similaritySearchVectorWithScore()`, `@langchain/textsplitters`
* **Code Snippet:**
```typescript
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
```
```typescript
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

const allSplits = await textSplitter.splitDocuments(docs);

console.log(allSplits.length);
```
```typescript
await vectorStore.addDocuments(allSplits);
```
```typescript
const results1 = await vectorStore.similaritySearch(
  "When was Nike incorporated?"
);

console.log(results1[0]);
```
```typescript
const results2 = await vectorStore.similaritySearchWithScore(
  "What was Nike's revenue in 2023?"
);

console.log(results2[0]);
```
```typescript
const embedding = await embeddings.embedQuery(
  "How were Nike's margins impacted in 2023?"
);

const results3 = await vectorStore.similaritySearchVectorWithScore(
  embedding,
  1
);

console.log(results3[0]);
```

## 4. Retrievers
* **Key Points:**
  - LangChain VectorStore objects do not subclass Runnable.
  - LangChain Retrievers are Runnables, so they implement a standard set of methods (e.g., synchronous and asynchronous invoke and batch operations).
  - Although we can construct retrievers from vector stores, retrievers can interface with non-vector store sources of data, as well (such as external APIs).
  - Vectorstores implement an as_retriever method that will generate a Retriever, specifically a VectorStoreRetriever.
  - These retrievers include specific search_type and search_kwargs attributes that identify what methods of the underlying vector store to call, and how to parameterize them.
  - Retrievers can easily be incorporated into more complex applications, such as retrieval-augmented generation (RAG) applications that combine a given question with retrieved context into a prompt for a LLM.
* **Technical Entities (Classes/Functions/APIs):** `Retrievers`, `Runnable`, `asRetriever()`, `VectorStoreRetriever`, `search_type`, `search_kwargs`, `mmr`, `batch()`
* **Code Snippet:**
```typescript
const retriever = vectorStore.asRetriever({
  searchType: "mmr",
  searchKwargs: {
    fetchK: 1,
  },
});

await retriever.batch([
  "When was Nike incorporated?",
  "What was Nike's revenue in 2023?",
]);
```

## Next steps
* **Key Points:**
  - You've now seen how to build a semantic search engine over a PDF document.
  - For more on embeddings: Overview; Available integrations
  - For more on vector stores: Overview; Available integrations
  - For more on RAG, see: Build a Retrieval Augmented Generation (RAG) App