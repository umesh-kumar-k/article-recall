---
aliases:
  - Document Upload
Source 1: https://docs.langchain.com/oss/javascript/langchain/rag#loading-documents
Source 3: https://docs.langchain.com/oss/javascript/langchain/knowledge-base
Source 2: https://docs.langchain.com/oss/javascript/integrations/document_loaders
---
# Indexing > Document Upload

## Indexing
* **Key Points:**
  - Indexing commonly works as follows:
    - Load: First we need to load our data into Document objects.
    - Split: Text splitters break large Documents into smaller chunks. This is useful both for indexing data and passing it into a model, as large chunks are harder to search over and won't fit in a model's finite context window.
    - Store: We need somewhere to store and index our splits, so that they can be searched over later. This is often done using a VectorStore and Embeddings model.
* **Technical Entities (Classes/Functions/APIs):** `Document`, `VectorStore`, `Embeddings`

## Loading documents
* **Key Points:**
  - We need to first load the blog post contents into a list of Document objects.
* **Technical Entities (Classes/Functions/APIs):** `cheerio`, `Document`, `@langchain/core/documents`, `loadWebPage`
* **Code Snippet:**
```typescript
import * as cheerio from "cheerio";
import { Document } from "@langchain/core/documents";

// Below is a minimal helper for demonstration purposes.
async function loadWebPage(
  url: string,
  selector: string = "body",
): Promise<Document[]> {
  const response = await fetch(url);
  const html = await response.text();
  const $ = cheerio.load(html);
  return [
    new Document({
      pageContent: $(selector).text(),
      metadata: { source: url },
    }),
  ];
}

const docs = await loadWebPage(
  "https://lilianweng.github.io/posts/2023-06-23-agent/",
  "p",
);

console.assert(docs.length === 1);
console.log(`Total characters: ${docs[0].pageContent.length}`);
```

## Splitting documents
* **Key Points:**
  - Our loaded document is over 42k characters which is too long to fit into the context window of many models. Even for those models that could fit the full post in their context window, models can struggle to find information in very long inputs.
  - To handle this we'll split the Document into chunks for embedding and vector storage. This should help us retrieve only the most relevant parts of the blog post at run time.
  - As in the semantic search tutorial, we use a RecursiveCharacterTextSplitter, which will recursively split the document using common separators like new lines until each chunk is the appropriate size. This is the recommended text splitter for generic text use cases.
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter`, `@langchain/textsplitters`
* **Code Snippet:**
```typescript
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});
const allSplits = await splitter.splitDocuments(docs);
console.log(`Split blog post into ${allSplits.length} sub-documents.`);
```

## Documents
* **Key Points:**
  - LangChain implements a Document abstraction, which is intended to represent a unit of text and associated metadata. It has three attributes:
    - pageContent: a string representing the content;
    - metadata: a dict containing arbitrary metadata;
    - id: (optional) a string identifier for the document.
  - The metadata attribute can capture information about the source of the document, its relationship to other documents, and other information. Note that an individual Document object often represents a chunk of a larger document.
  - We can generate sample documents when desired.
* **Technical Entities (Classes/Functions/APIs):** `Document`, `@langchain/core/documents`
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