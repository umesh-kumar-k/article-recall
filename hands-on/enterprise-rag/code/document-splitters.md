---
aliases:
  - Document Splitters
Source 3: https://docs.langchain.com/oss/javascript/integrations/splitters
Source 2: https://docs.langchain.com/oss/javascript/langchain/rag#splitting-documents
Source 1: https://docs.langchain.com/oss/python/integrations/splitters
---
# Langchain splitters

## Text splitter integrations
* **Key Points:**
  - Text splitters break large docs into smaller chunks that will be retrievable individually and fit within model context window limit.
  - There are several strategies for splitting documents, each with its own advantages.
  - For most use cases, start with the RecursiveCharacterTextSplitter. It provides a solid balance between keeping context intact and managing chunk size. This default strategy works well out of the box, and you should only consider adjusting it if you need to fine-tune performance for your specific application.
* **Technical Entities (Classes/Functions/APIs):** `@langchain/textsplitters`, `@langchain/core`, `RecursiveCharacterTextSplitter`

## Text structure-based
* **Key Points:**
  - Text is naturally organized into hierarchical units such as paragraphs, sentences, and words. We can leverage this inherent structure to inform our splitting strategy, creating split that maintain natural language flow, maintain semantic coherence within split, and adapts to varying levels of text granularity. LangChain's RecursiveCharacterTextSplitter implements this concept.
  - The RecursiveCharacterTextSplitter attempts to keep larger units (e.g., paragraphs) intact. If a unit exceeds the chunk size, it moves to the next level (e.g., sentences). This process continues down to the word level if necessary.
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter`, `splitText`
* **Code Snippet:**
```typescript
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const splitter = new RecursiveCharacterTextSplitter({ chunkSize: 100, chunkOverlap: 0 })
const texts = splitter.splitText(document)
```

## Length-based
* **Key Points:**
  - An intuitive strategy is to split documents based on their length. This simple yet effective approach ensures that each chunk doesn't exceed a specified size limit. Key benefits of length-based splitting: Straightforward implementation; Consistent chunk sizes; Easily adaptable to different model requirements.
  - Types of length-based splitting: Token-based: Splits text based on the number of tokens, which is useful when working with language models; Character-based: Splits text based on the number of characters, which can be more consistent across different types of text.
* **Technical Entities (Classes/Functions/APIs):** `TokenTextSplitter`, `CharacterTextSplitter`
* **Code Snippet:**
```typescript
import { TokenTextSplitter } from "@langchain/textsplitters";

const splitter = new TokenTextSplitter({ encodingName: "cl100k_base", chunkSize: 100, chunkOverlap: 0 })
const texts = splitter.splitText(document)
```

## Document structure-based
* **Key Points:**
  - Some documents have an inherent structure, such as HTML, Markdown, or JSON files. In these cases, it's beneficial to split the document based on its structure, as it often naturally groups semantically related text. Key benefits of structure-based splitting: Preserves the logical organization of the document; Maintains context within each chunk; Can be more effective for downstream tasks like retrieval or summarization.

## Splitting documents
* **Key Points:**
  - Our loaded document is over 42k characters which is too long to fit into the context window of many models. Even for those models that could fit the full post in their context window, models can struggle to find information in very long inputs.
  - To handle this we'll split the Document into chunks for embedding and vector storage. This should help us retrieve only the most relevant parts of the blog post at run time.
  - As in the semantic search tutorial, we use a RecursiveCharacterTextSplitter, which will recursively split the document using common separators like new lines until each chunk is the appropriate size. This is the recommended text splitter for generic text use cases.
* **Technical Entities (Classes/Functions/APIs):** `RecursiveCharacterTextSplitter`, `splitDocuments`
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