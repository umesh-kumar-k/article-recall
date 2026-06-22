---
aliases:
  - Document Upload/Ingestion
Source 1: https://www.deepset.ai/blog/preprocessing-rag
Source 2: https://unstructured.io/blog/level-up-your-genai-apps-essential-data-preprocessing-for-any-rag-system
Source 3: https://www.deepset.ai/blog/customizing-rag
Source 4: https://dev.to/aiengineering/a-beginners-guide-to-document-loaders-in-langchain-36e7
Source 5: https://docs.langchain.com/oss/javascript/langchain/rag#loading-documents
---

---

[Code Examples](./code/document-upload.md) 

# A Beginner's Guide to Document Loaders in LangChain

## What Are Document Loaders?
* **Key Points:**
  - Document loaders are tools that help you bring external content into your LangChain application in a structured way. Their job is simple: take data from a source, like a PDF, website, or spreadsheet, and wrap it in a format LangChain can understand.
  - Every piece of content a loader brings in is returned as a Document object. This object has two parts: page_content, which holds the text; metadata, which includes useful details like where the data came from.
  - This structure keeps things clean and consistent, so that when you pass the document to an LLM or a vector store, everything works smoothly.
  - There's no need to manually parse or clean up your input data. Document loaders take care of the heavy lifting so you can focus on what your application is meant to do, like answer questions, summarize text, extract insights, or anything else your LLM is built for.
* **Technical Entities (Classes/Functions/APIs):** `Document`, `page_content`, `metadata`, `LLM`, `vector store`, `LangChain`

## Why Document Loaders Matter in AI Applications
* **Key Points:**
  - Document loaders may not steal the spotlight in AI applications, but they play a critical role behind the scenes.
  - Clean input leads to reliable output: If your source data is a mess, the model's responses will be too. Loaders ensure your inputs are structured and clean.
  - They bridge the gap between raw data and model-ready formats: You might have a treasure trove of info in PDFs, cloud drives, or tools like Notion. Loaders bring that into your workflow.
  - They reduce manual work: Instead of writing a custom script every time you want to read a file, loaders give you a reusable, tested solution.

## Types of Document Loaders in LangChain
* **Key Points:**
  - Loaders come in different shapes, each designed to handle a specific kind of source.
  - File-Based Loaders: Useful when your data is stored locally. They support formats like PDF, DOCX, CSV, and plain text, making it easy to pull in documents from your machine.
  - Web Loaders: These are great when your source lives online. Just point to a URL, and LangChain handles the rest, pulling content from web pages, articles, or online resources.
  - Cloud Storage Loaders: For teams working in the cloud, these loaders fetch documents from services like Google Drive, S3, or Dropbox without the need for manual downloads.
  - Third-Party Platform Loaders: LangChain also connects with tools like Notion, Slack, and GitHub, so you can stream content straight from your team's workspace or codebase.
  - Custom Loaders: If none of the built-ins fit your use case, you can define your own. LangChain makes it simple to build loaders tailored to niche or proprietary data sources.
  - Each one is built to return structured Document objects, so once your content is in, it's ready to move through your chain.
* **Technical Entities (Classes/Functions/APIs):** `PDF`, `DOCX`, `CSV`, `Google Drive`, `S3`, `Dropbox`, `Notion`, `Slack`, `GitHub`, `Custom Loaders`

## Using a Document Loader in Practice
* **Key Points:**
  - Let's put document loaders to work with a real example using LangChain.js. Say you have a PDF you'd like to load into your app; maybe a research paper, product guide, or internal policy doc.
  - Each Document contains: pageContent: the extracted text; metadata: info like page number or file path.
  - Now that your content is loaded and structured, you can: Send it to an embedding model; Use it in a retrieval-augmented generation setup; Preprocess it for chunking or summarization.
* **Technical Entities (Classes/Functions/APIs):** `PDFLoader`, `langchain/document_loaders/fs/pdf`, `embedding model`, `retrieval-augmented generation`
* **Code Snippet:**
```javascript
import { PDFLoader } from "langchain/document_loaders/fs/pdf";

const loader = new PDFLoader("example.pdf");

const documents = await loader.load();

// Let's look at the first document
console.log(documents[0].pageContent);
console.log(documents[0].metadata);
```

## Scaling Up: Working with Multiple or Large Files
* **Key Points:**
  - Once you've mastered loading a single document, the next step is scaling, it could mean loading an entire folder or managing files too large to process in one go.
  - LangChain makes this surprisingly manageable.
* **Technical Entities (Classes/Functions/APIs):** `fs`, `path`, `PDFLoader`, `RecursiveCharacterTextSplitter`, `langchain/text_splitter`
* **Code Snippet:**
```javascript
// Loading Multiple Files from a Directory
import fs from "fs";
import path from "path";
import { PDFLoader } from "langchain/document_loaders/fs/pdf";

const folderPath = "./documents";

const loadDocuments = async () => {
  const files = fs.readdirSync(folderPath);
  const allDocs = [];

  for (const file of files) {
    const fullPath = path.join(folderPath, file);
    const loader = new PDFLoader(fullPath);
    const docs = await loader.load();
    allDocs.push(...docs);
  }

  return allDocs;
};

const documents = await loadDocuments();
console.log(`Loaded ${documents.length} pages from folder.`);
```
```javascript
// Splitting Large Documents
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

const splitDocs = await splitter.splitDocuments(documents);

console.log(`Split into ${splitDocs.length} chunks`);
```

## Best Practices for Using Document Loaders
* **Key Points:**
  - Working with document loaders gets easier with experience, but a few habits can help you avoid messy surprises and scale with confidence.
  - Always validate your source content: Garbage in, garbage out. Check for missing files, corrupt formats, or content that can't be parsed before loading.
  - Choose loaders based on your data source: Don't force a PDF loader to read a .docx or scrape a webpage manually. Use the right loader, it saves time and keeps your output consistent.
  - Chunk before you embed: LLMs work best with focused context. Use text splitters to break long documents into smaller chunks before passing them to an embedding model or chain.
  - Keep metadata intact: Most loaders attach useful details like page number, file name, or URL. Hold onto that. It's gold for tracing responses or building search interfaces later.
  - Batch intelligently: If you're processing hundreds of files, batch them in groups and watch for memory spikes. A simple loop with pauses or async handling can go a long way.
  - Build for change: Assume your data source will evolve. Whether that's new formats or dynamic URLs, writing flexible loader logic from day one keeps you future-proof.
  - Document loaders may not be the flashiest part of working with LLMs, but they're the quiet engine behind any meaningful interaction with real-world data. Once you understand how to use them well, you unlock the ability to bring your knowledge, files, and systems into your AI workflows, cleanly and confidently.
  - Whether you're loading a single PDF or syncing a thousand documents from the cloud, LangChain gives you the tools to do it right. Start small. Try loading a file today. Watch how it fits into your chain. From there, scale as your project grows. The real power of AI isn't just in generating text, it's in helping you work with your own.
  
---

# The Role of Data Preprocessing in RAG 

### What is preprocessing?
* **Key Points:**
  - Preprocessing is the action of preparing your files so that your RAG system can use them to generate the best possible answers.
  - Preprocessing, indexing, and RAG itself work in tandem, and can only be evaluated and tuned together.
  - Preparing and adding data to the database is very different, almost complementary, to retrieving and further processing that data.
  - For an initial examination of the steps involved in preprocessing, we'll consider only text data.

### Step 1: Examining and extracting the data
* **Key Points:**
  - Data comes in many forms. To set up your preprocessing pipeline correctly, you need to understand it: What file formats are in your dataset? Is your data all in the same format, or are you dealing with different file types (text, powerpoint, excel, pdf, etc.?) or even data types?
  - Based on the answers to these questions, you can choose the right tools to extract the data contained in your files and unify it for further processing.

### Step 2: Cleaning the data
* **Key Points:**
  - The purpose of preprocessing is to make your data "RAG-ready" while preserving the information it contains.
  - It's useful to strip the data of format-specific characters that contain no information, such as extra whitespace, blank lines, specified substrings, regexes, page headers and footers.
  - Some of this data could also be converted into metadata to preserve important information about the document structure that will help you later.

### Step 3: Chunking the data
* **Key Points:**
  - Language processing algorithms and models have implicit preferences for the length of text snippets they can process.
  - In this step, you chunk your text data into smaller pieces so that they arrive at the optimal length.
  - There are many strategies for data chunking, largely dictated by the indexing technique we're going to use.
  - While many embedding models used to require text snippets between 200 and 300 words, we're now often looking at models that can handle larger chunks of text, such as entire PDF pages, or even entire multi-page documents in "long-context" setups.

### Step 4: Adding metadata
* **Key Points:**
  - The term "metadata" refers to labels that tell you more about your data, such as when a data point was created, who created it, and so on.
  - During the chunking process, you can also extract metadata from your original data and add it to your cleaned and chunked snippets.
  - For example, you might want to retain information about what page a snippet came from, what header it was published under, or what snippet comes immediately after it.
  - This adds context that can be very helpful in finding the right information later.

### Step 5: Indexing the data
* **Key Points:**
  - Finally, your data is ready to be indexed.
  - This crucial step involves converting the content into fixed-size vectors, storing these vectors along with the original text and associated metadata, and organizing everything into the format required by the underlying database.

### A closer look at indexing
* **Key Points:**
  - Indexing is the process of adding data to a database so that it can be easily retrieved. It is therefore complementary to retrieval.
  - You can use sparse or dense methods to index data. The most popular sparse method is BM25, a ranking algorithm. Dense methods use language models (also known as embedding models).
  - Sparse and dense methods complement each other and are often used together in a hybrid setup.
  - What's important is that you use the same method that will be used to retrieve your data later.
  - The language model you choose to embed your text chunks will determine how long your text chunks can be.
  - If you change your retriever model in the RAG pipeline itself, you'll also need to reindex all your data using the same model to embed it.
  - As for your metadata, it will be indexed along with the text data itself, so it can serve as a future filter or other context enhancer.
* **Technical Entities (Classes/Functions/APIs):** `BM25`, `embedding models`, `retriever model`, `sparse methods`, `dense methods`

### Advanced indexing pipelines
* **Key Points:**
  - In modern AI product development, customization isn't just a feature, it's a guiding principle.
  - Just as the RAG pipeline itself, your indexing pipeline will be unique to your business use case, and therefore will need to be adapted as you go through iterative development cycles.

### Named Entity Recognition (NER) for metadata extraction
* **Key Points:**
  - In your preprocessing pipeline, you can include a component that uses a small language model to identify named entities and store them as metadata along with your text.
  - These named entities can then be used as a filter at query time.
* **Technical Entities (Classes/Functions/APIs):** `Named Entity Recognition (NER)`, `small language model`

### Language classification
* **Key Points:**
  - If you have a large multilingual document collection, it's important to know what language each document is in.
  - You can add a classifier to your indexing pipeline and have it generate language labels to add as metadata to each document.

### Semantic chunking
* **Key Points:**
  - Rule-based chunking strategies run the risk of losing information.
  - Semantic chunking has gained popularity, which uses an embedding model to segment text while preserving context.

### Multimodal
* **Key Points:**
  - Text is only one type of data, and LLMs are equipped to handle all types of data, such as tables and images.
  - In business contexts, it is common to deal with documents that contain tables and charts, for example.
  - During preprocessing, you'll need to use special models to extract these types of data from a document.
  - Getting your indexing pipeline right is critical to the success of your RAG project. So you should be prepared to devote sufficient time and resources to this step – in our experience, it accounts for about 50 percent of your RAG project.

### Preprocessing and indexing in production
* **Key Points:**
  - It's relatively easy to index a few documents on one machine, but production systems often need to process millions of files quickly.
  - In production, you'll need to consider factors such as throughput (how many files can be processed in a given time) and latency (how quickly new information is available for retrieval).
  - Production indexing systems often use distributed architectures to meet these challenges.
  - They may queue indexing requests and use multiple machines to process files in parallel.
  - Technologies such as Kubernetes can be used to automatically scale the number of indexing processors based on demand.
* **Technical Entities (Classes/Functions/APIs):** `Kubernetes`

### Out-of-the-box customization and scaling of indexing pipelines with Haystack
* **Key Points:**
  - Haystack Enterprise Platform (formerly known as deepset AI Platform) excels in indexing due to its speed, flexibility and comprehensiveness.
  - It uses a parallel approach to significantly speed up the indexing process.
  - This feature is particularly beneficial for workflows that require frequent re-indexing, such as when experimenting with different embedding models.
  - The platform's modular indexing pipelines are easy to customize, allowing users to quickly add components.
  - Furthermore, Haystack offers a "set and forget" approach to data uploads, automatically managing dependencies and providing options for replacing or adding new data, streamlining the entire indexing process.
* **Technical Entities (Classes/Functions/APIs):** `Haystack Enterprise Platform`, `deepset AI Platform`

---


# Essential Data Pre-processing for Any RAG System

## Getting the Data Out of Silos: Ingestion
* **Key Points:**
  - Ingestion is the first, and often most underestimated, step in building intelligent systems with enterprise data. It's not just about collecting files or exporting data; it's about reliably accessing, contextualizing, and standardizing fragmented knowledge from a chaotic ecosystem of siloed internal platforms.
  - Ingestion is where data integrity, completeness, and usability are won or lost. If it fails, everything downstream, aka chunking, enrichment, embedding, retrieval, is compromised.
  - Enterprise data isn't neatly packaged. It's scattered across cloud storage buckets, collaboration tools, databases, SaaS applications, and more. Each source brings its own API quirks, content formats, permission models, and metadata conventions.
  - Ingesting this data at scale requires much more than one-off scripts or generic pipelines. It requires a system that can handle the diversity of source systems, preserve context, reconcile formats, and keep everything up to date efficiently.
  - Successful ingestion pipelines must meet five key requirements:
    - Connectivity: Access content from wherever it lives, across both structured and unstructured platforms.
    - Context Preservation: Capture and retain critical metadata—authorship, timestamps, permissions, and system-specific signals.
    - Normalization: Convert varied content formats into a standardized representation for downstream processing.
    - Incremental Update Support: Detect changes and only ingest what's new to keep systems fresh without ballooning costs.
    - Maintainability: Avoid brittle custom code that breaks with API changes or format drift.
  - Unstructured is designed from the ground up to meet these needs. Its production-grade connectors cover the most common enterprise data systems, including cloud storage (S3, GCS, Azure), collaboration platforms (SharePoint, Confluence, Box), business apps (Salesforce, Jira), databases, and streaming systems like Kafka.
* **Technical Entities (Classes/Functions/APIs):** `S3`, `GCS`, `Azure`, `SharePoint`, `Confluence`, `Box`, `Salesforce`, `Jira`, `Kafka`, `Unstructured`

## Wrangling Your Data: Extraction and Partitioning
* **Key Points:**
  - After ingestion, the next big step in preparing enterprise content for RAG pipelines is document partitioning and content extraction.
  - Enterprise data isn't confined to a single content type or format. It lives across PDFs, docx files, PowerPoint decks, Excel spreadsheets, HTML pages, and so on. Each of these formats encodes content differently, and extracting clean, usable text from them is a deceptively hard problem.
  - Some open-source tools offer partial coverage: one might be good at extracting content from HTML pages, another at parsing Word documents or processing spreadsheets. But stitching them together into a cohesive pipeline means you, the developer, are now responsible for standardizing their outputs—each of which likely returns text in different schemas, with different assumptions about structure and granularity.
  - Without a consistent representation, it's difficult to apply uniform downstream logic for chunking, embedding, or filtering. You end up spending more time cleaning and reconciling than building.
  - Worse still, common extraction methods tend to flatten documents into a stream of plain text, stripping away visual layout, images, and positional cues. This loss of structure degrades both chunking and retrieval accuracy, especially for dense or formatted documents like manuals, reports, or spreadsheets.
  - Unstructured solves this by applying intelligent partitioning, which breaks down diverse documents into distinct elements such as Title, NarrativeText, ListItem, Table, Image, and more; each annotated with detailed metadata like layout coordinates, page numbers, source file type, and hierarchical structure.
  - This preserves spatial and semantic context from the outset and provides a consistent, typed JSON schema across all content types. That means you can process a paragraph from a PowerPoint slide and a Jira ticket in exactly the same way without brittle custom glue code.
  - Partitioning also captures rich media artifacts that are typically lost during extraction: images are preserved as base64-encoded metadata, unlocking multimodal use cases. Tables, meanwhile, are preserved not just as flat text but in their structure converted to plain HTML, maintaining row-column relationships that are critical for accurate interpretation.
* **Technical Entities (Classes/Functions/APIs):** `Unstructured`, `Title`, `NarrativeText`, `ListItem`, `Table`, `Image`, `JSON`, `base64`, `HTML`

## Breaking It Down: Chunking Strategies
* **Key Points:**
  - Once the text is extracted, it needs to be divided into smaller segments, or "chunks." This is arguably one of the most critical preprocessing steps in RAG.
  - Chunking choices are vital for effective retrieval. Matching a specific piece of information within a smaller, focused chunk is generally more precise and efficient than matching with a massive document. Smaller chunks tend to have a better signal-to-noise ratio for retrieval.
  - There's a fundamental trade-off:
    - Smaller Chunks: Offer higher precision for retrieval, making it easier to pinpoint specific facts or matches. But they might lack sufficient context for the LLM to understand the information fully or generate a comprehensive answer.
    - Larger Chunks: Provide more context, potentially leading to better generation quality. But they can dilute the relevance signal for retrieval (making it harder to find the specific matching part) and might contain more irrelevant information (noise). They also consume more of the LLM's context window and increase processing costs.
  - Finding the right balance is key and often requires experimentation and a good understanding of underlying data and common user queries.
  - Common chunking strategies (and their limits):
    - Fixed-Size Chunking: The most basic method: divide text into equally sized blocks (e.g., 500 characters) with optional overlap. Pros: Fast & simple. Cons: Ignores document structure and meaning—often splits mid-sentence, mid-paragraph, or even mid-word. This can lead to semantic fragmentation and noisy retrieval.
    - Recursive Character-Based Chunking: This method recursively splits text using a hierarchy of separators—first by paragraphs, then newlines, then spaces—to get under the character limit. Pros: Some structure awareness; better at preserving natural breakpoints than fixed-size. Cons: Still primarily size-driven, not meaning-driven. Boundaries can be arbitrary, and important semantic groupings can still get lost.
    - Sentence-Based Chunking: Chunks are defined using sentence boundaries. Pros: Aligns well with natural language, enabling clearer reasoning for LLMs. Cons: Sentence size varies wildly, and individual sentences often lack standalone context.
  - Because Unstructured generates rich document structure during partitioning, it enables structurally-aware and semantically coherent chunking strategies that align more naturally with how humans and LLMs interpret information.
  - Unstructured Smart Chunking Strategies:
    - Basic: Combines sequential elements into chunks while preserving logical boundaries (e.g., paragraph breaks, lists).
    - By Title: Groups content under heading elements, preserving document hierarchy and section-level coherence.
    - By Page: Treats each page as a self-contained unit, ideal for scanned documents or forms where layout matters.
    - By Similarity: Leverages embeddings to group conceptually related elements.
  - These strategies reduce semantic drift, improve chunk-level relevance, and eliminate the need for brittle post-processing logic.
* **Technical Entities (Classes/Functions/APIs):** `Unstructured`

## From Text to Vectors: Embedding Generation Fundamentals
* **Key Points:**
  - The core idea enabling semantic search in most RAG systems is the conversion of text chunks into numerical representations called embeddings.
  - Embeddings are vectors (lists of numbers) in a high-dimensional space, generated by an embedding model (often a transformer-based bi-encoder model).
  - These vectors are designed such that texts with similar meanings are located closer to each other in this vector space, while dissimilar texts are farther apart. This geometric relationship allows for mathematical comparison of semantic similarity.
  - Each cleaned and chunked piece of text is fed into the chosen embedding model, which outputs a corresponding vector embedding. It's crucial that the same embedding model is used to embed the source document chunks during indexing and to embed the user query at retrieval time; otherwise, the comparison will be meaningless.
  - Here are some considerations for choosing an embedding model:
    - Model size and architecture: Larger models (e.g., OpenAI's text-embedding-3-large) generally yield higher-quality vectors but come at higher latency and cost.
    - Training data domain: Models trained on general web data might underperform on specialized enterprise domains like law, finance, or biotech. Domain-tuned models might yield better results, and in some cases you may want to consider fine-tuning an embedding model.
    - Embedding dimension: Model outputs have varying dimensionality (e.g., 256 vs. 1024 vs. 3072). Higher dimensions capture more nuance but increase storage and search costs.
  - Unstructured integrates seamlessly with major embedding providers—OpenAI, Amazon Bedrock, TogetherAI, Voyage AI, and others—so you can easily plug in the model that best fits your use case, and experiment across providers without rearchitecting your pipeline.
* **Technical Entities (Classes/Functions/APIs):** `embeddings`, `embedding model`, `transformer-based bi-encoder`, `OpenAI`, `text-embedding-3-large`, `Amazon Bedrock`, `TogetherAI`, `Voyage AI`, `Unstructured`

## Making Data Searchable: Basic Indexing with Vector Stores
* **Key Points:**
  - Once text chunks are converted into embeddings, they need to be stored in a way that allows for efficient searching based on semantic similarity. This is the role of the index, most commonly implemented using a vector database (also called a vector store).
  - A vector database enables similarity search, a process that compares a user query embedding against a collection of pre-computed document embeddings. The result is a ranked list of documents whose vector representations are most similar to the query vector, typically based on cosine similarity or Euclidean distance.
  - To handle potentially billions of vectors efficiently, vector databases don't usually compare the query vector to every single stored vector (which would be too slow). Instead, they employ Approximate Nearest Neighbor (ANN) search algorithms (like HNSW - Hierarchical Navigable Small World). ANN algorithms trade a small amount of accuracy for a massive gain in search speed, quickly finding vectors that are likely to be among the closest neighbors.
  - Many vector database options exist, both open-source and managed services. Examples include Pinecone, Weaviate, Qdrant, Milvus, Redis (with vector search capabilities), Elasticsearch/OpenSearch, etc. Unstructured connects directly to many of these, handling upload and indexing without manual effort.
* **Technical Entities (Classes/Functions/APIs):** `vector database`, `vector store`, `similarity search`, `cosine similarity`, `Euclidean distance`, `Approximate Nearest Neighbor (ANN)`, `HNSW`, `Pinecone`, `Weaviate`, `Qdrant`, `Milvus`, `Redis`, `Elasticsearch`, `OpenSearch`, `Unstructured`

## Conclusion
* **Key Points:**
  - Solid preprocessing isn't a nice-to-have. It's the bedrock of an effective RAG system. Ingestion, extraction, chunking, embedding, and indexing must all be handled with care, precision, and scalability in mind.
  - Unstructured helps you get this right from the beginning, giving your downstream applications the clean, contextualized, and actionable data they need to perform.
  - In our next post, we'll go over some advanced data preprocessing techniques like contextual chunking, entity extraction, and LLM/VLM-powered enrichments. Stay tuned.
* **Technical Entities (Classes/Functions/APIs):** `Unstructured`, `LLM`, `VLM`