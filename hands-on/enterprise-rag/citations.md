---
aliases:
  - Citations
Source 1: https://www.tensorlake.ai/blog/rag-citations
Source 2: https://dev.to/tensorlake/make-rag-provable-page-bbox-citations-for-all-extracted-data-4ipc
Source 3: https://docs.cohere.com/docs/rag-citations
---
[Code Reference](./code/document-citation.md) 

# Citation-Aware RAG: How to add Fine Grained Citations in Retrieval and Response Synthesis

## Introduction
* **Key Points:**
  - Citations are table stakes for agentic applications. Without them, users can't trust the output. That's why products like Perplexity and Google AI Search always return sources alongside answers. Even dev tools like Cursor cite line numbers and file names when suggesting code changes.
  - If you're already building RAG applications with custom pipelines or agentic frameworks like LangChain, adding citations doesn't require rebuilding your system. The techniques below work as an enhancement layer on top of your existing retrieval setup.
  - At its core, generating citations in RAG is about preserving source information at index time. While for simple use cases the citation is a hyperlink or file reference, for long documents with hundreds of pages or thousands of tokens, just a file reference isn't enough.
  - Precise citations, like linking claims to exact paragraphs, table cells, and figures, separate professional agentic applications from chatbot demos. Users don't just get answers; they get evidence.

## How to Build Citation-Ready Document Chunks
* **Key Points:**
  - Most RAG tutorials stop at converting documents into Markdown, chunking, and indexing. This works fine for demos but fails in production. The moment you need to verify sources, audit responses, or meet compliance requirements, basic chunking leaves you empty-handed. To support citation-aware RAG, every chunk needs to carry:
    - File name
    - Page number
    - Spatial metadata (bounding boxes of each line, figure, or table)
  - The challenge is balancing fidelity with noise. If you add all spatial metadata directly into the chunk text, the content becomes polluted. But if you merge multiple lines into a larger chunk without citation anchors, you lose the ability to cite precise lines.
  - The solution: insert lightweight citation anchors into the text and store fragment-specific spatial metadata separately as chunk metadata. This keeps the text clean while preserving fine-grained citation targets.

## Prerequisites: Document Parsing with Bounding Boxes and Vector Storage
* **Key Points:**
  - Before building citation-aware retrieval, two capabilities are essential:
    - OCR with Spatial Metadata: Converting PDFs or images to plain Markdown with models like Gemini Pro or OpenAI's vision models makes fine-grained citations impossible. While these VLMs excel at text extraction, they don't return the grounding information needed for citations: bounding boxes, element coordinates, or spatial relationships between document components. Bounding box coordinates for each document element are required to link responses back to source locations. Extracting text, tables, and images with bounding boxes is the spatial information that anchors citations to the exact location in the source.
    - Metadata-Aware Storage: Vector databases like Pinecone, Qdrant, Weaviate, and PgVector support storing bounding boxes, page numbers, or paragraph IDs (any relevant and contextual metadata) alongside chunks, making them available during retrieval and accessible for your RAG application's end user.
  - Most existing RAG setups already store some metadata (file names, chunk IDs). Citation support extends this pattern by adding bounding box coordinates, typically adding ~10-15% to your storage overhead but enabling full source traceability.
* **Technical Entities (Classes/Functions/APIs):** `Gemini Pro`, `OpenAI's vision models`, `Pinecone`, `Qdrant`, `Weaviate`, `PgVector`

## Document Parsing: From Layout to Citable Chunks
* **Key Points:**
  - Tensorlake's Document AI makes building citation-aware retrieval easy. The Document AI API parses a page into elements with: Content, Page number, Bounding boxes.
  - The document layout returned from the parse endpoint contains all the information you need to create context-aware RAG:
    - Pages: The document layout is a list of pages. Each page contains a page number, dimensions, a page classification reason (optional), and a list of page fragments. The page numbers and dimensions can be referenced by each of the page fragments for each page object.
    - Page Fragments: Each page fragment contains a fragment type (e.g. section_header, table, text), content, bounding box coordinates, and reading order. Using the fragment type can help contextualize fragments when chunking.
  - When you're ready to create chunks, you can iterate through page fragment objects and create appropriately sized chunks by combining them. As you create the chunks, you can create contextualized metadata to help during retrieval.
  - This approach adds minimal overhead to the chunk text while still letting the retriever and LLM map answers back to exact locations in the source. It's all you need in the preprocessing stage to enable citation-aware RAG.
  - This chunking strategy works with standard RAG frameworks. If you're using LangChain's RecursiveCharacterTextSplitter, for example, you can adapt these techniques by modifying how you construct your Document objects to include the citation metadata.
  - The retrieval side stays familiar. Whether you're using similarity search, hybrid retrieval, or reranking. Your existing query engines don't need changes. The citation magic happens in two places: how you structure chunks (above) and how you prompt the LLM (below).
* **Technical Entities (Classes/Functions/APIs):** `Tensorlake Document AI`, `LangChain's RecursiveCharacterTextSplitter`

## Returning Citations with LLM Responses
* **Key Points:**
  - Once your chunks carry anchors, retrieval doesn't really change. You can use the same dense, hybrid, or reranker setup you already have. The real magic happens during response generation.
  - If you're building the full RAG application, there are two small additions you might want to consider:
    - Hide the anchors in prose, while keeping them in output. Upon retrieval, the LLM will see the chunks with inline markers like <c>2.1</c>, but you can instruct it: "don't print section anchors in your sentences — just return them as citation IDs." This will ensure the results are leveraging the anchors, but your user experience is cleaner.
    - Turn IDs into clickable evidence. When the model outputs the citation IDs, you can look them up in your metadata store. Because of how you stored the data, each ID maps back to a page and bounding box. From there you can generate a deep link or highlight that takes the user straight to the source.
  - This prompting pattern works across different LLMs (OpenAI, Anthropic, local models) and can be adapted for different frameworks. In LangChain, for example, you'd modify your prompt template.
  - And that's it! Most of the heavy lifting for citation generation happens in preprocessing: embedding spatial information into chunks and storing it as metadata in the vector store. On the retrieval side, it's just a matter of resolving citation IDs back to their coordinates and rendering them in the UI or turning them into links to the source.

## Beyond RAG: Citations in Structured Data Extraction
* **Key Points:**
  - The citation techniques we've covered work perfectly for conversational RAG applications. But what if you need structured data extraction with citations? Think financial reports where you extract specific metrics, or legal documents where you pull contract terms; scenarios where you need both the extracted values and proof of where they came from.
  - Tensorlake's Structured Extraction API solves this with a single parameter. Unlike building citation pipelines manually (which requires coordinating parsing, chunking, retrieval, and response generation), this API handles the entire citation chain in a single call, similar to how Anthropic's Claude or OpenAI's structured outputs work, but with source linking.
  - This lets you trace every extracted field back to its source location in the document. Every number in your database can link back to its exact source location in the original document.
  - The same citation principles apply: you get page numbers, bounding boxes, and verifiable evidence, but without building the citation infrastructure yourself.
* **Technical Entities (Classes/Functions/APIs):** `Tensorlake DocumentAI`, `StructuredExtractionOptions`, `FinancialMetrics`, `provide_citations=True`, `parse_and_wait`
* **Code Snippet:**
```python
from tensorlake.documentai import DocumentAI, StructuredExtractionOptions
from pydantic import BaseModel, Field
 
class FinancialMetrics(BaseModel):
    revenue: float = Field(description="Annual revenue")
    net_income: float = Field(description="Net income")
    eps: float = Field(description="Earnings per share")
 
doc_ai = DocumentAI()
 
structured_extraction_options = [
    StructuredExtractionOptions(
        schema_name="FinancialMetrics",
        json_schema=FinancialMetrics,
        provide_citations=True  # <-- Citations for every field
    )
]
 
result = doc_ai.parse_and_wait(
    file="path/to/10k-filing.pdf",
    structured_extraction_options=structured_extraction_options
)
```