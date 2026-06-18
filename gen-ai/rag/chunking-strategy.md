---
aliases:
  - Chunking Strategy
Source 1: https://towardsdatascience.com/breaking-it-down-chunking-techniques-for-better-rag-3fd288bf25a0/
Source 2: https://towardsdatascience.com/rag-101-chunking-strategies-fdc6f6c2aaec/
tags:
  - rag
  - chunking
high: Document segmentation approach (fixed size,semantic, recursive) that balances context completeness vs retrieval precision; typically 256 -1024 tokens
---
#  Chunking Techniques for Better RAG

## Preamble : RAG and the need for a Knowledge Base
* **Key Points:**
  - "Retrieval Augmented Generation, or RAG, has emerged to be one of the most popular techniques in the applied generative AI world."
  - "Despite Large Language Models demonstrating unprecedented ability to generate text, their responses are not always correct."
  - "Upon more careful observation, you may notice that LLM responses are plagued with sub-optimal information and inherent memory limitations."
  - "RAG addresses these limitations of LLMs by providing them with information external to these models. Thereby, resulting in LLM responses that are more reliable and trustworthy."
  - "The technique of retrieving relevant information from an external source, augmenting that information as an input to the LLM, thereby enabling the LLM to generate an accurate response is called Retrieval Augmented Generation"
  - "To create a RAG system, it is important that access to this external information is provided programatically to the LLM."
  - "To enable this access a persistent knowledge base becomes an integral part of the RAG system."
  - "This knowledge base acts as the non-parametric memory of the system where information can be searched and fetched from to provide to the LLM."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval Augmented Generation)`, `LLMs (Large Language Models)`, `ChatGPT`
* **Code Snippet:** None

---

## The creation of the non-parametric memory is the critical pre-requisite for any RAG system (Source: Image by Author)
* **Key Points:**
  - "The creation of knowledge base can be summarised in four steps – Data Loading: Connect to the external source and parse the information from source documents."
  - "Data Splitting: Break down long pieces of text into smaller manageable sizes or 'chunks'. This process is called 'chunking'. Chunking is crucial to the efficient creation of knowledge base for RAG systems. This will be the focus of this article."
  - "Data Conversion: Converting the chunks into a format that is suited for retrieval. Embeddings is a popular format to store unstructured information for retrieval."
  - "Data Storage: Store the information in persistent storage for real time retrieval in RAG systems. A special type of databases have become popular for the storage of embeddings. These are called vector databases."
  - "The four steps mentioned above form the indexing pipeline of a RAG system."
  - "Indexing pipeline is an 'offline' process that creates the knowledge base to be used during the real-time interaction with the RAG system."
  - "The real-time interaction is facilitated by the generation pipeline and together these two form the core of a basic RAG system."
* **Technical Entities (Classes/Functions/APIs):** `vector databases`
* **Code Snippet:** None

---

## Understanding Chunking: What is it and Why are we talking about it?
* **Key Points:**
  - "In cognitive psychology, chunking is defined as process by which individual pieces of information are bound together into a meaningful whole. (https://psycnet.apa.org/record/2003-09163-002) and a chunk is a familiar collection of elementary units."
  - "The idea is that chunking is an essential technique through which human beings perceive the world and commit to memory."
  - "The simplest example is how we remember long sequences of digits like phone numbers, credit card numbers, dates or even OTPs. We don't remember the entire sequences but in our minds, we break them down into chunks."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## What is Chunking in RAG?
* **Key Points:**
  - "The role of chunking in RAG and the underlying idea is somewhat similar to what it is in real life."
  - "Once you've extracted and parsed text from the source, instead of committing it all to memory as a single element, you break it down into smaller chunks."
  - "Breaking down long pieces of text into manageable sizes is called Chunking"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## The Benefits of Chunking
* **Key Points:**
  - "There are two main benefits of chunking – It leads to better retrieval of information. If a chunk represents a single idea (or fact) it can be retrieved with more confidence that if there are multiple ideas (or facts) within the same chunk."
  - "It leads to better generation. The retrieved chunk has information that is focussed on the user query and does not have any other text that may confuse the LLM. Therefore, the generation is more accurate and coherent."
  - "Context Window of LLMs: LLMs, due to the inherent nature of the technology, have a limit on the number of tokens (loosely, words) they can work with at a time."
  - "This includes both the number of tokens in the prompt (or the input) and the number of tokens in the completion (or the output)."
  - "The limit on the total number of tokens that an LLM can process in one go is called the context window size."
  - "If we pass an input that is longer than the context window size, the LLM chooses to ignore all text beyond the size."
  - "Lost in the middle problem: Even in those LLMs which have a long context window (Claude 3 by Anthropic has a context window of up to 200,00 tokens), an issue with accurately reading the information has been observed."
  - "It has been noticed that accuracy declines dramatically if the relevant information is somewhere in the middle of the prompt."
  - "This problem can be addressed by passing only the relevant information to the LLM instead of the entire document."
* **Technical Entities (Classes/Functions/APIs):** `Claude 3` (Anthropic)
* **Code Snippet:** None

---

## Basic Chunking Methods
### Fixed Size Chunking
* **Key Points:**
  - "A very common approach is to pre-determine the size of the chunk and the amount of overlap between the chunks."
  - "There are several chunking methods that follow a fixed size chunking approach."
  - "Character-Based Chunking: Chunks are created based on a fixed number of characters"
  - "Token-Based Chunking: Chunks are created based on a fixed number of tokens."
  - "Sentence-Based Chunking: Chunks are defined by a fixed number of sentences"
  - "Paragraph-Based Chunking: Chunks are created by dividing the text into a fixed number of paragraphs."
  - "While character and token based chunking methods are useful to maintain consistency of chunk sizes, they may lead to loss of meaning by abruptly splitting words or sentences."
  - "On the other hand, while sentence and paragraph based chunking methods are helpful in preserving the context, they introduce a variability in chunk sizes."
* **Technical Entities (Classes/Functions/APIs):** `CharacterTextSplitter` (LangChain)
* **Code Snippet:**
```python
#import libraries
from langchain_text_splitters import CharacterTextSplitter
#Set the CharacterTextSplitter parameters
text_splitter = CharacterTextSplitter(
    separator="n",    #The character that should be used to split
    chunk_size=1000,   #Number of characters in each chunk
    chunk_overlap=200, #Number of overlapping characters between chunks
)

#Create Chunks
chunks=text_splitter.create_documents([data_transformed[0].page_content])

#Show the number of chunks created
print(f"The number of chunks created : {len(chunks}")

>>The number of chunks created : 64
```

---

### Structure-Based Chunking
* **Key Points:**
  - "The aim of chunking is to keep meaningful data together."
  - "If we are dealing with data in form of HTML, Markdown, JSON or even computer code, it makes more sense to split the data based on the structure rather than a fixed size."
  - "Another approach for chunking is to take into consideration the format of the extracted and loaded data."
  - "A markdown file, for example is organised by headers, a code written in a programming language like python or java is organized by classes and functions and HTML, likewise, is organised in headers and sections."
  - "For such formats a specialised chunking approach can be employed."
* **Technical Entities (Classes/Functions/APIs):** `HTMLHeaderTextSplitter` (LangChain)
* **Code Snippet:**
```python
# Import the HTMLHeaderTextSplitter library
from langchain_text_splitters import HTMLHeaderTextSplitter

# Set url as the Wikipedia page link
url="https://en.wikipedia.org/wiki/2023_Cricket_World_Cup"

# Specify the header tags on which splits should be made
headers_to_split_on=[
    ("h1", "Header 1"),
    ("h2", "Header 2"),
    ("h3", "Header 3"),
    ("h4", "Header 4")
]

# Create the HTMLHeaderTextSplitter function
html_splitter = HTMLHeaderTextSplitter(headers_to_split_on=headers_to_split_on)

# Create splits in text obtained from the url
html_header_splits = html_splitter.split_text_from_url(url)
```

---

## Advanced Chunking Techniques
### Context-Enriched Chunking
* **Key Points:**
  - "This method adds the summary of the larger document to each chunk to enrich the context of the smaller chunk."
  - "This makes more context available to the LLM without adding too much noise."
  - "It also improves the retrieval accuracy and maintains semantic coherence across chunks."
  - "This is particularly useful in scenarios where a more holistic view of the information is crucial."
  - "While this approach enhances the understanding of the broader context, it adds a level of complexity and comes at the cost of higher computational requirements, increased storage needs and possible latency in retrieval."
* **Technical Entities (Classes/Functions/APIs):** `GPT-4o-mini`, `OpenAI embeddings`, `FAISS`, `AsyncHtmlLoader` (LangChain), `Html2TextTransformer` (LangChain), `RecursiveCharacterTextSplitter` (LangChain)
* **Code Snippet:**
```python
#Loading text from Wikipedia page
from langchain_community.document_loaders import AsyncHtmlLoader
from langchain_community.document_transformers import Html2TextTransformer
url=https://en.wikipedia.org/wiki/2023_Cricket_World_Cup
loader = AsyncHtmlLoader (url)
data = loader.load()
html2text = Html2TextTransformer()
data_transformed = html2text.transform(data)
document_text=data_transformed[0].page_content

# Generating summary of the text using GPT-4o-mini model
summary_prompt = f"Summarize the given document in a single /
paragraphndocument: {document_text}" 
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages= [
    {"role": "user", "content": summary_prompt}
      ]
)

summary=response.choices[0].message.content

# Creating Chunks using Recursive Character Splitter
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
chunk_size=1000,
chunk_overlap=200)
chunks=text_splitter.split_text(data_transformed[0].page_content)

# Enriching Chunks with Summary Data
context_enriched_chunks = [summary + "n" + chunk for chunk in chunks]

# Creating embeddings and storing in FAISS index
embedding = OpenAIEmbeddings(openai_api_key=api_key) #E
vector_store = FAISS.from_texts(context_enriched_chunks, embedding)
```

---

### Agentic Chunking
* **Key Points:**
  - "In agentic chunking, chunks from the text are created based on a goal or a task."
  - "Consider an e-commerce platform wanting to analyse customer reviews. The best way for the reviews to be chunked is if the reviews pertaining to a particular topic are put in the same chunk."
  - "Similarly, the critical reviews and positive reviews may be put in different chunks."
  - "To achieve this kind of chunking, we will need to do sentiment analysis, entity extraction and some kind of clustering. This can be achieved by a multi-agent system."
  - "Agentic chunking is still an active area of research and improvement."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Semantic Chunking
* **Key Points:**
  - "This idea proposed by Greg Kamradt questions two aspects of the fixed size chunking methods. Why should we have a pre-defined fixed size of chunks? Why don't chunking methods take into consideration the actual meaning of content?"
  - "To address these issues, a method that looks at semantic similarity (or similarity in the meaning) between sentences is called semantic chunking."
  - "It first creates groups of three sentences and then merges groups that are similar in meaning."
  - "To find out the similarity in meaning, this method uses Embeddings (We will discuss Embeddings in the next section 3.3)."
  - "This is still an experimental chunking technique."
* **Technical Entities (Classes/Functions/APIs):** `SemanticChunker` (langchain_experimental.text_splitter)
* **Code Snippet:** None

---

## Small-to-Big & Sliding Window Techniques
* **Key Points:**
  - "Small to big chunking is a hierarchical chunking method where the text is first broken down into very small units (e.g., sentences, paragraphs), and the small chunks are merged into larger ones until the chunk size is achieved."
  - "Sliding window chunking uses overlap between chunks to maintain context across chunk boundaries."
  - "The process can be understood in three steps – Divide the longer text into compact, meaningful units like sentences or paragraphs."
  - "Merge the smaller units into larger chunks until a specific size is achieved. Once the size is achieved, this chunk is treated as an independent segment of text."
  - "When a new chunk is being created include a part of the previous chunk at the start of the new chunk. This overlap is necessary to maintain contextual continuity."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Chunk Size
* **Key Points:**
  - "The size of the chunks can have a significant impact on the quality of the RAG system."
  - "While large sized chunks provide better context, they also carry a lot of noise."
  - "Smaller chunks, on the other hand, have precise information but they might miss important information."
  - "For instance, consider a legal document that's 10,000 words long. If we chunk it into 1,000-word segments, each chunk might contain multiple legal clauses, making it hard to retrieve specific information. Conversely, chunking it into 200-word segments allows for more precise retrieval of individual clauses, but may lose the context provided by surrounding clauses."
  - "Experimenting with chunk sizes can help find the optimal balance for accurate retrieval."
  - "The processing time also depends on the size of the chunk."
  - "Chunk size, therefore, has a significant impact on retrieval accuracy, processing speed and storage efficiency."
  - "The ideal chunk size varies with the use case and depends on balancing factors like document types and structure, complexity of user query and the desired response time."
  - "There is no one size fits all approach towards optimizing chunk sizes."
  - "Experimentation and evaluation of different chunk sizes on metrics like faithfulness, relevance, response time can help in identifying the optimal chunk size for the RAG system."
  - "Chunk size optimisation may require periodic reassessment as data, or requirements change."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Enhancing Chunking with Metadata
* **Key Points:**
  - "A common way of defining metadata is 'data about data'. Metadata describes other data."
  - "It can provide information like a description of the data, time of creation, author, etc."
  - "While metadata is useful for managing and organising data, in the context of RAG, metadata enhances the searcheability of data."
  - "Metadata filtering: Adding metadata like timestamp, author, category, etc. can enhance the chunks. While retrieving, chunks can first be filtered by relevant metadata information before doing a similarity search. This improves retrieval efficiency and reduces noise in the system."
  - "For example, using the timestamp filters can help avoid outdated information in the knowledge base. If a user searches for 'latest COVID-19 travel guidelines,' metadata filtering by timestamp ensures that only the most recent guidelines are retrieved, avoiding outdated information."
  - "Metadata enrichment: Time stamp, author, category, chapter, page number etc. are common metadata elements that can be extracted from documents. However, even more valuable metadata items can be constructed. This can be a summary of the chunk, extracting tags from the chunk."
  - "One particularly useful technique is Reverse Hypothetical Document Embeddings. It involves using a language model to generate potential queries that could be answered by each document or chunk. These synthetic queries are then added to the metadata. During retrieval, the system compares the user's query with these synthetic queries to find the most relevant chunks."
* **Technical Entities (Classes/Functions/APIs):** `Reverse Hypothetical Document Embeddings`
* **Code Snippet:** None

---

## Parent-child Structure
* **Key Points:**
  - "In a parent-child document structure, documents are organised hierarchically."
  - "The parent document contains overarching themes or summaries, while child documents delve into specific details."
  - "During retrieval, the system can first locate the most relevant child documents and then refer to the parent documents for additional context if needed."
  - "This approach enhances the precision of retrieval while maintaining the broader context."
  - "At the same time, this hierarchical structure can present challenges in terms of memory requirements and computational load."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Choice of Chunking Strategy
### Nature of the Content
* **Key Points:**
  - "The type of data that you're dealing with can be a guide for the chunking strategy."
  - "If your application uses a data in a specific format like code or HTML, a specialized chunking method is recommended."
  - "Not only that, whether you're working with long documents like whitepapers and reports or short form content like social media posts, tweets, etc. can also guide the chunk size and overlap limits."
  - "If you're using a diverse set of information sources, then you might have you use different methods for different sources."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Expected Length and Complexity of User Query
* **Key Points:**
  - "The nature of the query that your RAG enabled system is likely to receive is also a determinant of the chunking strategy."
  - "If your system expects a short and straightforward query, then the size of your chunks should be different when compared to a long and complex query."
  - "Matching long queries to short chunks may prove inefficient in certain cases."
  - "Similarly, short queries matching with large chunks may yield partially irrelevant results."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Application & Use Case Requirements
* **Key Points:**
  - "The nature of the use case you're addressing may also determine the optimal chunking strategy."
  - "For a direct question answering system, it is likely that shorter chunks are used for precise results, while for summarisation tasks, longer chunks may make more sense."
  - "If the results of your system need to serve as an input to another downstream application, that may also influence the choice of the chunking strategy."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Embeddings Model Being Used
* **Key Points:**
  - "Embeddings are vector representations of unstructured data."
  - "Once the text is chunked, it is converted into a numeric form."
  - "Embeddings are popular data patterns because they help in establishing semantic relationships between words, phrases, and documents."
  - "The choice of embeddings model is also influential in how the chunking should be done."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Common Pitfalls and Cautions
### Sub-optimal Overlap
* **Key Points:**
  - "Breaking text mid-sentence or mid-paragraph leads to loss of meaning in the chunks."
  - "To mitigate this, overlapping techniques like sliding window is used. This ensures that contextual information at the boundary is present in consecutive chunks."
  - "However, too much overlap can introduce redundancies and result in processing inefficiency and inconsistent results."
  - "Balancing the overlap window size is therefore crucial to maintain."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Inconsistent Chunk Size
* **Key Points:**
  - "Fixed size chunking is sometimes criticised for not being context aware."
  - "On the other hand, techniques like semantic and agentic chunking can create chunks of inconsistent sizes."
  - "This results in inconsistency of information."
  - "Smaller chunks might not carry enough information and others may overwhelm the model with too much text."
  - "It is advisable to have guardrails around the chunk size even when employing variable size chunking strategies."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Underchunking and Overchunking
* **Key Points:**
  - "Large chunks can exceed processing limits (e.g., token limits for language models) and may result in ignoring parts of the data, while overly small chunks prevent the model to see enough context and can cause latency in retrieval."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

### Task Specific Needs
* **Key Points:**
  - "Chunking is not a one size fits all approach."
  - "Smaller chunks are necessary for entity recognition and larger ones may be needed for summarisation."
  - "Using the same chunking method for different tasks can lead to sub-optimal results."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Emerging Trends in Chunking
* **Key Points:**
  - "Chunking is a sub-field of generative AI and is evolving as fast as the technology."
  - "A few trends to look out for in the near future are – Enhancements in the context awareness of chunks"
  - "Adaptive chunking that adjusts chunks dynamically based on the use case."
  - "Evolution of agentic chunking into a more sophisticated task-oriented process."
  - "Multimodal chunking is designed to segment not only text but other unstructured form of data like images, audio and video."
  - "Deeper integration with knowledge graphs to maintain connections to broader concepts."
  - "Real-time and responsive chunking for edge devices and low-latency systems."
  - "Chunking is a crucial step towards creating production grade RAG systems."
  - "While fixed width chunking methods are good for prototyping and developing simpler systems, more advanced strategies are required from production grade applications."
  - "As chunking evolves with the evolution of generative AI, it is important to develop points of view on chunking strategies and experiment with the ones available for different use cases."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None