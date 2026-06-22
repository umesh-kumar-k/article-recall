---
aliases:
  - Langchain Advanced
Source 1: https://medium.com/@mehtameet115/langchain-advanced-rag-your-complete-guide-to-document-processing-and-intelligent-retrieval-part-12e71d81e30e
---
# LangChain Advanced RAG: Your Complete Guide to Document Processing and Intelligent Retrieval (Part 2)

## 1. Document Loaders & Data Ingestion: Your AI's Reading List
* **Key Points:**
  - Before your AI can be smart about your content, you need to feed it knowledge. Document loaders are like different types of readers — some are great with PDFs, others excel at web scraping, and some can even understand YouTube videos.
  - Think of document loaders as specialized librarians. Each one knows exactly how to handle specific types of content.
* **Technical Entities (Classes/Functions/APIs):** `PyPDFLoader`, `UnstructuredLoader`, `WebBaseLoader`, `YoutubeLoader`

### 1.1 The PDF Whisperer — PyPDFLoader
* **Technical Entities (Classes/Functions/APIs):** `PyPDFLoader`, `load_and_split`
* **Code Snippet:**
```python
from langchain_community.document_loaders import PyPDFLoader
loader = PyPDFLoader("path/to/your/research_paper.pdf")
pages = loader.load_and_split()
print(f"Extracted {len(pages)} pages of pure knowledge!")
```

### 1.2 The Universal Translator — UnstructuredLoader
* **Key Points:**
  - This is your Swiss Army knife for documents. It handles PDFs, Word docs, HTML, and even images with text.
* **Technical Entities (Classes/Functions/APIs):** `UnstructuredLoader`
* **Code Snippet:**
```python
from langchain_community.document_loaders import UnstructuredLoader
# Handles almost any file format
loader = UnstructuredLoader("mystery_document.whatever")
docs = loader.load()
```

### 1.3 The Web Crawler — WebBaseLoader
* **Key Points:**
  - Turn any website into your knowledge base.
* **Technical Entities (Classes/Functions/APIs):** `WebBaseLoader`
* **Code Snippet:**
```python
from langchain_community.document_loaders import WebBaseLoader
# Scrape educational content
loader = WebBaseLoader([
    "https://www.coursera.org/learn/machine-learning",
    "https://cs229.stanford.edu/syllabus.html"
])
web_docs = loader.load()
```

### 1.4 The YouTube Scholar — YouTubeLoader
* **Key Points:**
  - Extract transcripts from educational videos.
* **Technical Entities (Classes/Functions/APIs):** `YoutubeLoader`, `from_youtube_url`
* **Code Snippet:**
```python
from langchain_community.document_loaders import YoutubeLoader
# Learn from the best YouTube educators
loader = YoutubeLoader.from_youtube_url(
    "https://www.youtube.com/watch?v=aircAruvnKk",  # 3Blue1Brown neural networks
    add_video_info=True
)
transcript = loader.load()
```

## 2. Text Splitting Strategies: The Art of Smart Chunking
* **Key Points:**
  - Raw documents are like giant textbooks — too big for your AI to digest in one bite. Text splitting is like creating perfectly sized study cards that preserve context while being manageable.

### 2.1 The Chunking Hierarchy: From Crude to Clever
* **Key Points:**
  - Level 1: The Brute Force Approach
  - Level 2: The Smart Splitter (Recommended) — This is your go-to splitter
  - Level 3: The Token-Aware Splitter — Perfect for staying under model limits
* **Technical Entities (Classes/Functions/APIs):** `CharacterTextSplitter`, `RecursiveCharacterTextSplitter`, `TokenTextSplitter`
* **Code Snippet:**
```python
from langchain.text_splitter import CharacterTextSplitter
# Simple but effective
basic_splitter = CharacterTextSplitter(
    separator="\n\n",  # Split on paragraphs
    chunk_size=1000,
    chunk_overlap=200
)

from langchain.text_splitter import RecursiveCharacterTextSplitter
# This is your go-to splitter
smart_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    separators=["\n\n", "\n", ". ", " ", ""]  # Try these in order
)
chunks = smart_splitter.split_documents(documents)
print(f"Created {len(chunks)} perfectly sized chunks!")

from langchain.text_splitter import TokenTextSplitter
# Perfect for staying under model limits
token_splitter = TokenTextSplitter(
    chunk_size=512,  # Exact token count
    chunk_overlap=50
)
```

### 2.2 Semantic Splitting: The Future is Here
* **Key Points:**
  - Instead of splitting based on length, semantic chunkers understand meaning.
  - When semantic splitting shines: Research papers with multiple topics; Long tutorials with different concepts; Mixed content documents.
* **Technical Entities (Classes/Functions/APIs):** `SemanticChunker`, `OpenAIEmbeddings`, `langchain_experimental.text_splitter`
* **Code Snippet:**
```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings
# Split when topics change, not when length limits hit
embeddings = OpenAIEmbeddings()
semantic_splitter = SemanticChunker(
    embeddings,
    breakpoint_threshold_type="percentile",  # How sensitive to topic changes
    breakpoint_threshold_amount=95
)
semantic_chunks = semantic_splitter.split_text(your_long_document)
```

### 2.3 The Chunking Playground
* **Key Points:**
  - Want to see how different strategies affect your documents? Check out the interactive Chunk Visualizer at chunkviz.up.railway.app. Upload your document and watch how different splitters create chunks — it's like seeing your AI's thought process!
  - Pro chunking tips: Start with 1000 characters, 200 overlap — works for most use cases; Adjust based on content: Longer chunks for academic papers, shorter for quick facts.

## 3. Vector Stores vs Vector Databases: Your AI's Memory System
* **Key Points:**
  - Once you have perfect chunks, you need somewhere to store them that enables lightning-fast similarity search. Think of this as choosing between a filing cabinet (vector store) and a library system (vector database).

### 3.1 Vector Stores: Quick and Dirty
* **Key Points:**
  - Perfect for getting started and smaller projects.
* **Technical Entities (Classes/Functions/APIs):** `FAISS`, `HuggingFaceEmbeddings`, `save_local`, `load_local`
* **Code Snippet:**
```python
from langchain_community.vectorstores import FAISS
from langchain_huggingface import HuggingFaceEmbeddings
# Lightning fast, runs on your laptop
embeddings = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")
vector_store = FAISS.from_documents(chunks, embeddings)
# Save for later (because nobody likes waiting)
vector_store.save_local("my_knowledge_base")
# Load it back
loaded_store = FAISS.load_local("my_knowledge_base", embeddings)
```

### 3.2 Vector Databases: The Big Leagues
* **Key Points:**
  - When you need enterprise features and massive scale.
* **Technical Entities (Classes/Functions/APIs):** `Pinecone`, `pinecone`
* **Code Snippet:**
```python
from langchain_community.vectorstores import Pinecone
import pinecone
# Fully managed, scales to billions of vectors
pinecone.init(api_key="your-api-key", environment="us-west1-gcp")
vector_store = Pinecone.from_documents(
    chunks, 
    embeddings, 
    index_name="my-knowledge-base"
)
```

### 3.3 The Storage Decision Tree
* **Key Points:**
  - Choose FAISS if: You're prototyping or learning; Your data fits in memory (< 1GB); You want maximum speed for small datasets.
  - Choose Chroma if: You need persistence without complexity; You're building a personal project; You want something between FAISS and enterprise solutions.
  - Choose Pinecone if: You want zero maintenance; You need to scale quickly; Budget allows for managed services.
  - Choose Weaviate/Qdrant if: You need full control; You have complex metadata requirements; You're building for production.

## 4. Advanced Retrievers: The Art of Finding Needles in Haystacks
* **Key Points:**
  - Retrievers are the bridge between your stored knowledge and your AI model. The right retrieval strategy can be the difference between "I don't know" and "Here's exactly what you need."

### 4.1 Basic Vector Retriever: The Foundation
* **Key Points:**
  - Simple similarity search.
* **Technical Entities (Classes/Functions/APIs):** `as_retriever`, `get_relevant_documents`
* **Code Snippet:**
```python
# Simple similarity search
basic_retriever = vector_store.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 5}  # Top 5 most similar chunks
)
# Test it out
results = basic_retriever.get_relevant_documents("What is machine learning?")
```

### 4.2 MMR Retriever: The Diversity Champion
* **Key Points:**
  - Maximal Marginal Relevance - reduces redundancy.
* **Technical Entities (Classes/Functions/APIs):** `as_retriever`, `mmr`, `lambda_mult`
* **Code Snippet:**
```python
# Maximal Marginal Relevance - reduces redundancy
mmr_retriever = vector_store.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 5,
        "lambda_mult": 0.7  # Balance between relevance (1.0) and diversity (0.0)
    }
)
```

### 4.3 Multi-Query Retriever: The Perspective Shifter
* **Key Points:**
  - Generates multiple query angles automatically. One question becomes many perspectives.
* **Technical Entities (Classes/Functions/APIs):** `MultiQueryRetriever`, `from_llm`, `langchain.retrievers.multi_query`
* **Code Snippet:**
```python
from langchain.retrievers.multi_query import MultiQueryRetriever
# Generates multiple query angles automatically
multi_query_retriever = MultiQueryRetriever.from_llm(
    retriever=vector_store.as_retriever(),
    llm=your_llm,
    verbose=True  # See the generated queries
)
# One question becomes many perspectives
results = multi_query_retriever.get_relevant_documents(
    "How do neural networks learn?"
)
```

### 4.4 Contextual Compression Retriever: The Noise Filter
* **Key Points:**
  - Removes irrelevant parts from retrieved documents. Only get the relevant parts.
* **Technical Entities (Classes/Functions/APIs):** `ContextualCompressionRetriever`, `LLMChainExtractor`, `langchain.retrievers`
* **Code Snippet:**
```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
# Removes irrelevant parts from retrieved documents
compressor = LLMChainExtractor.from_llm(your_llm)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=vector_store.as_retriever()
)
# Only get the relevant parts
compressed_results = compression_retriever.get_relevant_documents(
    "Explain backpropagation algorithm"
)
```

### 4.5 Ensemble Retriever: The Best of Both Worlds
* **Key Points:**
  - Combine semantic and keyword search.
* **Technical Entities (Classes/Functions/APIs):** `EnsembleRetriever`, `BM25Retriever`, `langchain.retrievers`
* **Code Snippet:**
```python
from langchain.retrievers import EnsembleRetriever, BM25Retriever
# Combine semantic and keyword search
bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 5
ensemble_retriever = EnsembleRetriever(
    retrievers=[vector_store.as_retriever(), bm25_retriever],
    weights=[0.7, 0.3]  # 70% semantic, 30% keyword
)
```

### 4.6 Building a Smart Retrieval System
* **Key Points:**
  - Here's how to create a retrieval system that adapts to different types of questions.
* **Technical Entities (Classes/Functions/APIs):** `SmartRetriever`
* **Code Snippet:**
```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain.retrievers import EnsembleRetriever

class SmartRetriever:
    def __init__(self, vector_store, llm):
        self.vector_store = vector_store
        self.llm = llm
        
        # Set up different retrieval strategies
        self.basic_retriever = vector_store.as_retriever(search_kwargs={"k": 5})
        self.mmr_retriever = vector_store.as_retriever(
            search_type="mmr", 
            search_kwargs={"k": 5, "lambda_mult": 0.7}
        )
        self.multi_query_retriever = MultiQueryRetriever.from_llm(
            retriever=self.basic_retriever,
            llm=llm
        )
    
    def retrieve(self, query, strategy="auto"):
        if strategy == "auto":
            # Simple heuristics for strategy selection
            if len(query.split()) > 10:
                strategy = "multi_query"  # Complex questions
            elif "compare" in query.lower() or "difference" in query.lower():
                strategy = "mmr"  # Need diverse results
            else:
                strategy = "basic"  # Simple questions
        
        if strategy == "basic":
            return self.basic_retriever.get_relevant_documents(query)
        elif strategy == "mmr":
            return self.mmr_retriever.get_relevant_documents(query)
        elif strategy == "multi_query":
            return self.multi_query_retriever.get_relevant_documents(query)
        
        return self.basic_retriever.get_relevant_documents(query)

# Use it
smart_retriever = SmartRetriever(vector_store, llm)
results = smart_retriever.retrieve("What's the difference between supervised and unsupervised learning?")
```

## Complete RAG Implementation: Building Your AI Assistant
* **Key Points:**
  - Now let's put everything together into a production-ready RAG system.
* **Technical Entities (Classes/Functions/APIs):** `DirectoryLoader`, `PyPDFLoader`, `RecursiveCharacterTextSplitter`, `FAISS`, `HuggingFaceEmbeddings`, `ChatGoogleGenerativeAI`, `RetrievalQA`, `PromptTemplate`, `StreamingStdOutCallbackHandler`
* **Code Snippet:**
```python
from langchain_community.document_loaders import DirectoryLoader, PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate
from langchain.callbacks import StreamingStdOutCallbackHandler

class RAGAssistant:
    def __init__(self, docs_path="./documents"):
        self.docs_path = docs_path
        self.vector_store = None
        self.llm = None
        self.qa_chain = None
        
    def load_documents(self):
        """Load and process documents"""
        print("📚 Loading documents...")
        loader = DirectoryLoader(
            self.docs_path, 
            glob="**/*.pdf", 
            loader_cls=PyPDFLoader,
            show_progress=True
        )
        documents = loader.load()
        print(f"✅ Loaded {len(documents)} documents")
        return documents
    
    def create_chunks(self, documents):
        """Split documents into chunks"""
        print("✂️ Creating chunks...")
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len,
            separators=["\n\n", "\n", ". ", " ", ""]
        )
        chunks = text_splitter.split_documents(documents)
        print(f"✅ Created {len(chunks)} chunks")
        return chunks
    
    def create_vector_store(self, chunks):
        """Create vector store from chunks"""
        print("🧠 Creating vector store...")
        embeddings = HuggingFaceEmbeddings(
            model_name="sentence-transformers/all-MiniLM-L6-v2"
        )
        vector_store = FAISS.from_documents(chunks, embeddings)
        print("✅ Vector store created")
        return vector_store
    
    def setup_llm(self):
        """Initialize the language model"""
        self.llm = ChatGoogleGenerativeAI(
            model="gemini-2.0-flash-001",
            temperature=0.1,
            streaming=True,
            callbacks=[StreamingStdOutCallbackHandler()]
        )
    
    def create_qa_chain(self):
        """Create the QA chain"""
        template = """You are a helpful AI assistant. Use the following context to answer the question.
        If you don't know the answer based on the context, say so honestly.
        
        Context: {context}
        
        Question: {question}
        
        Answer: Let me help you with that based on the information I have."""
        
        prompt = PromptTemplate(
            template=template, 
            input_variables=["context", "question"]
        )
        
        self.qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            chain_type="stuff",
            retriever=self.vector_store.as_retriever(search_kwargs={"k": 5}),
            chain_type_kwargs={"prompt": prompt},
            return_source_documents=True
        )
    
    def setup(self):
        """Initialize the entire RAG system"""
        documents = self.load_documents()
        chunks = self.create_chunks(documents)
        self.vector_store = self.create_vector_store(chunks)
        self.setup_llm()
        self.create_qa_chain()
        print("🚀 RAG Assistant ready!")
    
    def ask(self, question):
        """Ask a question to the RAG system"""
        if not self.qa_chain:
            print("❌ Please run setup() first!")
            return
        
        print(f"\n🤔 Question: {question}")
        print("🤖 Answer: ", end="")
        
        result = self.qa_chain.invoke({"query": question})
        
        print(f"\n\n📖 Sources used:")
        for i, doc in enumerate(result["source_documents"], 1):
            print(f"{i}. {doc.metadata.get('source', 'Unknown')}")
        
        return result

# Use your RAG assistant
assistant = RAGAssistant("./my_study_materials")
assistant.setup()

# Start asking questions!
assistant.ask("What are the key concepts in machine learning?")
assistant.ask("How does gradient descent work?")
assistant.ask("What's the difference between classification and regression?")
```

## 7 RAG Improvements: Production-Ready Enhancements

### 1. UI Based Enhancements
* **Key Points:**
  - Frontend Frameworks: Streamlit — Rapid prototyping for data science apps; Gradio — Quick demo interfaces with sharing capabilities; Chainlit — Chat-focused UI specifically for LLM applications; Reflex — Full-stack Python web framework; Next.js + Vercel AI SDK — Production-ready web applications.

### 2. Evaluation
* **Key Points:**
  - RAG-Specific Evaluation Tools: RAGAS — Comprehensive RAG evaluation framework (Metrics: Faithfulness, Answer Relevancy, Context Precision, Context Recall); LangSmith — LangChain's evaluation and monitoring platform (Features: Tracing, debugging, dataset management, A/B testing).

### 3. Indexing
* **Key Points:**
  - Document Ingestion: Unstructured.io — Advanced document parsing; LlamaIndex — Data framework for LLM applications.
  - Text Splitting: LangChain Text Splitters — Comprehensive splitting strategies; Semantic Chunking — Topic-aware document segmentation.
  - Vector Stores: Pinecone — Managed vector database; Weaviate — Open-source vector search engine; Qdrant — High-performance vector database; Chroma — AI-native open-source embedding database.

### 4. Retrieval
* **Key Points:**
  - Pre-Retrieval: Query Rewriting: HyDE (Hypothetical Document Embeddings); Multi-Query Generation: LangChain MultiQueryRetriever; Domain Routing: LangChain RouterChain.
  - During Retrieval: MMR (Maximal Marginal Relevance): Diversity optimization; Hybrid Retrieval: BM25 + Vector search combination; Reranking: Cohere Rerank API, sentence-transformers cross-encoders.
  - Post-Retrieval: Contextual Compression: LangChain ContextualCompressionRetriever; Lost in the Middle mitigation strategies; Context window optimization techniques.

### 5. Augmentation
* **Key Points:**
  - Prompt Engineering: LangChain Hub — Prompt template repository; PromptLayer — Prompt management and versioning; Weights & Biases Prompts — Experiment tracking for prompts.
  - Answer Grounding: Citation generation frameworks; Source attribution systems; Fact verification pipelines.

### 6. Generation
* **Key Points:**
  - Answer with Citation: LangChain Citation Tools; Custom citation formatters; Source tracking mechanisms.
  - Guard Railing: Guardrails AI — Validation framework for LLM outputs; NeMo Guardrails — NVIDIA's safety framework; Content filtering and safety checks.

### 7. System Design
* **Key Points:**
  - Multimodal: LangChain Multimodal — Text, image, and document processing; Unstructured — Multi-format document handling.
  - Agentic: LangGraph — Agent workflow orchestration; CrewAI — Multi-agent collaboration framework; AutoGen — Microsoft's multi-agent conversation framework.
  - Memory-Based: LangChain Memory — Conversation and long-term memory; Mem0 — Personalized AI memory layer; Zep — Long-term memory for AI assistants.