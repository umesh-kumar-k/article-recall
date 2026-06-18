---
aliases:
  - Graph RAG
highlights: Combines knowledge graphs with vector search to capture entity relationships and multi hop reasoning paths
tags:
  - rag
  - graph-rag
Source 1: https://neo4j.com/blog/knowledge-graph/what-is-knowledge-graph/
Source 2: https://neo4j.com/blog/genai/what-is-graphrag/
---
# What is a knowledge graph?

## What is a knowledge graph?
* **Key Points:**
  - "A knowledge graph is an organized representation of real-world entities and their relationships. It is typically stored in a graph database, which natively stores the relationships between data entities. Entities in a knowledge graph can represent objects, events, situations, or concepts. The relationships between these entities capture the context and meaning of how they are connected."
  - "A knowledge graph stores data and relationships alongside frameworks known as organizing principles. They can be thought of as rules or categories around the data that provide a flexible, conceptual structure to drive deeper data insights."
  - "The usefulness of a knowledge graph lies in the way it organizes the principles, data, and relationships to surface new knowledge for your user or business."
  - "The design is useful for many usage patterns, including real-time applications, search and discovery, and grounding generative AI for question-answering."
  - "Sometimes, people overcomplicate the concept of a knowledge graph. You might hear about enterprise-wide structures that consolidate and connect information across data silos and various sources. While that does describe a knowledge graph (one that can underpin a data integration use case), it describes one with a wide scope."
  - "Thinking only in terms of bridging large datasets and multiple data sources can make creating and implementing knowledge graphs seem complicated and time-consuming. But knowledge graphs don't need to be broad or elaborate. You can build one with a much smaller scope to solve a use-case-specific problem."
* **Technical Entities (Classes/Functions/APIs):** `knowledge graph`, `graph database`
* **Code Snippet:** None

---

## How knowledge graphs work
* **Key Points:**
  - "You may have heard of knowledge graphs in the context of search engines. The Google Knowledge Graph changed how we search for and find information on the Web. It amasses facts about people, places, and things into an organized network of entities. When you do a Google search for information, it uses the connections between entities to surface the most relevant results in context, for example, in the box Google calls the 'knowledge panel.'"
  - "The entities in the Google knowledge graph represent the world as we know it, marking a shift from 'strings to things.' Behind this simple phrase is the profound concept of treating information on the web as entities rather than a bunch of text. Since information is organized as a network of entities, Google can tap into the collective intelligence of the knowledge graph to return results tailored to the meaning of your query rather than a simple keyword match."
* **Technical Entities (Classes/Functions/APIs):** `Google Knowledge Graph`
* **Code Snippet:** None

---

## Key characteristics
### Nodes
* **Key Points:**
  - "Nodes denote and store details about entities, such as people, places, objects, or institutions. Each node has a (or sometimes several) label to identify the node type and may optionally have one or more properties (attributes). Nodes are also sometimes called vertices."
  - "For example, the nodes in an e-commerce knowledge graph typically represent entities such as people (customers and prospects), products, and orders:"
* **Technical Entities (Classes/Functions/APIs):** `Nodes`, `vertices`
* **Code Snippet:** None

---

### Relationships
* **Key Points:**
  - "Relationships link two nodes together: they show how the entities are related. Like nodes, each relationship has a label identifying the relationship type and may optionally have one or more properties. Relationships are also sometimes called edges."
  - "In the e-commerce example, relationships exist between the customer and order nodes, capturing the 'placed order' relationship between customers and their orders:"
* **Technical Entities (Classes/Functions/APIs):** `Relationships`, `edges`
* **Code Snippet:** None

---

### Organizing principle(s)
* **Key Points:**
  - "Organizing Principles are a framework, or schema, that organizes nodes and relationships according to fundamental concepts essential to the use cases at hand. Unlike many data designs, knowledge graphs easily incorporate multiple organizing principles."
  - "Organizing principles range from simple (product line -> product category -> product taxonomy) to complex (a complete business vocabulary that explains the data in the graph). Think of an organizing principle as a conceptual map or metadata layer overlaying the data and relationships in the graph."
  - "The model uses the same node-and-relationship structure as the rest of the knowledge graph to describe the organizing principles – which means you can write queries that draw from both instance data and organizing principles."
  - "In the e-commerce example, an organizing principle might be product types and categories:"
* **Technical Entities (Classes/Functions/APIs):** `Organizing Principles`
* **Code Snippet:** None

---

### What about ontologies?
* **Key Points:**
  - "When learning about knowledge graphs, you might come across articles on ontologies and wonder where they fit in. An ontology is a formal specification of the concepts and the relationships between them for a given subject area; semantic networks are a common way to represent ontologies. Put simply, ontologies are a type of organizing principle."
  - "Ontologies can be complex and require a great deal of effort to define and maintain. When deciding whether an ontology is needed, it's critical to consider the problems you're trying to solve with a knowledge graph. In many cases, it won't be necessary. In the e-commerce example, using a product taxonomy as the organizing principle is sufficient for a product recommendation use case."
  - "Think of the knowledge graph as a growing, evolving system to simplify your design in the early stages and deliver value sooner. If you pick the right technology to implement your knowledge graph, you can expand and evolve the graph as your needs change. In this way, you can add ontologies when your use case requires them rather than forcing yourself to build them up-front."
* **Technical Entities (Classes/Functions/APIs):** `ontologies`, `semantic networks`
* **Code Snippet:** None

---

## Knowledge graph example
* **Key Points:**
  - "Let's see what a knowledge graph might look like. Below is a simple knowledge graph of the e-commerce example that shows nodes as circles and relationships between them as arrows. The organizing principles are also stored as nodes and relationships, so the illustration uses some color shading to show which nodes and relationships are the instance data and which are the organizing principles:"
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Knowledge graphs and graph databases
### Property graphs
* **Key Points:**
  - "Native property graph databases, such as Neo4j, are a logical choice for implementing knowledge graphs. They natively store information as nodes, relationships, and properties, allowing for an intuitive visualization of highly interconnected data structures. The physical database matches the conceptual data model, making designing and developing the knowledge graph easier."
  - "When you use property graphs, you get: Simplicity and ease of design: Property graphs allow for straightforward data modeling when designing the knowledge graph. Because the conceptual and physical models are very similar (often the same), the transition from design to implementation is more straightforward (and easy to explain to non-technical users)."
  - "Flexibility: It's easy to add new data, properties, relationship types, and organizing principles without extensive refactoring or code rewrites. As needs change, you can iterate and incrementally expand the knowledge graph's data, relationships, and organization."
  - "Performance: Property graphs offer superior query performance compared to alternatives like RDF databases or relational databases, especially for complex traversals and many-to-many relationships. This performance comes from storing the relationships between entities directly in the database rather than re-generating them using joins in queries. A native property graph database traverses relationships by following pointers in memory, making queries that traverse even complex chains of many relationships very fast."
  - "Developer-friendly Code: Property graphs support an intuitive and expressive ISO query language standard, GQL, which means you have less code to write, debug, and maintain than SQL or SPARQL. Neo4j's Cypher is the most widely used implementation of GQL."
* **Technical Entities (Classes/Functions/APIs):** `Neo4j`, `property graphs`, `GQL`, `Cypher`, `SQL`, `SPARQL`
* **Code Snippet:** None

---

### Property graph vs. triple stores (RDF)
* **Key Points:**
  - "People sometimes think of property graphs and triple stores as equally viable options for building a knowledge graph, but triple stores (also known as RDF databases) have considerable disadvantages."
  - "Based on the Resource Description Framework (RDF), triple stores use a granular approach to design and storage. Triple stores express all data in the form of subject-predicate-object 'triples.' This model does not support relationships with properties or multiple same-typed relationships between entities. To accommodate real-world use cases, you will need to implement workarounds. Common workarounds include turning relationships into objects (called reification) or using singleton properties to capture properties using extra 'type-of' relationships. These workarounds mean larger databases, additional complexity in the physical model, and poor query performance."
  - "Because reification and singleton properties force tough decisions about the design up front, triple stores don't lend themselves to solving real-world problems that involve messy data domains. Knowledge graphs built on a triple store are more challenging to design, time-consuming to implement, and difficult to change."
* **Technical Entities (Classes/Functions/APIs):** `triple stores`, `RDF databases`, `Resource Description Framework (RDF)`, `reification`, `singleton properties`
* **Code Snippet:** None

---

### Property graph vs. relational databases
* **Key Points:**
  - "Relational databases and other non-native graph approaches suffer similar design friction. Neither relational nor document databases store relationships – they must be synthesized at runtime with joins or value lookups in query code. Since the relationships reside in the code rather than with the dataset, each application and data use must have its own implementation. SQL (the relational database query language) forces you to define every join in the query itself. As a result, the knowledge graph becomes more difficult to manage and yields poor runtime performance as the number of relationships expands."
* **Technical Entities (Classes/Functions/APIs):** `relational databases`, `SQL`
* **Code Snippet:** None

---

## Knowledge graph use cases
### Generative AI for enterprise search applications
* **Key Points:**
  - "In generative AI applications, knowledge graphs capture and organize key domain-specific or proprietary company information. Knowledge graphs are not limited to structured data; they can handle less organized data as well."
  - "GraphRAG, a technique that grounds large language models with knowledge graphs, is emerging as the foundation of AI applications that use proprietary domain data (these are known as RAG applications). A knowledge graph grounding increases response accuracy and improves explainability with the context provided by data relationships."
  - "Industry leaders such as Deloitte highlight the critical role of knowledge graphs for building enterprise-grade GenAI. Gartner places knowledge graphs having a 'high mass,' being an impactful technology for GenAI today:"
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG`, `RAG applications`, `GenAI`
* **Code Snippet:** None

---

### Fraud detection and analytics in financial services, banking, and insurance
* **Key Points:**
  - "In Fraud Detection and Analytics, the knowledge graph represents a network of transactions, their participants, and relevant information about them. Companies can use this knowledge graph to quickly identify suspicious activity, investigate suspected fraud, and evolve their knowledge graph to keep up with changing fraud patterns. Algorithms such as pathfinding and community detection provide key signals to machine learning algorithms that can uncover more sophisticated fraud networks."
* **Technical Entities (Classes/Functions/APIs):** `pathfinding`, `community detection`
* **Code Snippet:** None

---

### Master data management
* **Key Points:**
  - "In Master Data Management (e.g., for Customer 360 use cases), the knowledge graph provides an organized, resolved (i.e., 'de-duped'), and comprehensive database of a company's customers and the company's interactions with them."
  - "This organized view of customers is especially important for companies with multiple divisions or applications interacting with customers. Without a knowledge graph, it can be difficult or impossible to obtain an accurate view of the customer. A knowledge graph links customer behaviors across multiple applications through an organizing principle that identifies them as coming from the same customer."
* **Technical Entities (Classes/Functions/APIs):** `Master Data Management`, `Customer 360`
* **Code Snippet:** None

---

### Supply chain management
* **Key Points:**
  - "In Supply Chain Management, a knowledge graph represents the network of suppliers, raw materials, products, and logistics that work together to supply a company's operations and customers. This end-to-end supply chain visibility allows managers to identify weak points and predict where disruptions may occur. Graph algorithms such as shortest path optimize the supply chain in real time by finding the most direct route between A and B."
* **Technical Entities (Classes/Functions/APIs):** `shortest path`
* **Code Snippet:** None

---

### Investigative journalism
* **Key Points:**
  - "In Investigative Journalism, knowledge graphs capture key entities (companies, people, bank accounts, etc.) and activities under investigation. Organizing these entities in relation to one another makes it possible to find hidden patterns, such as distant relationships between entities that shouldn't be present."
  - "Investigators may use techniques such as entity resolution to reveal entities hiding behind fake or shell identities to mask their activities. Algorithms like community detection and link prediction also provide insight and areas for further investigation."
* **Technical Entities (Classes/Functions/APIs):** `entity resolution`, `community detection`, `link prediction`
* **Code Snippet:** None

---

### Drug discovery in healthcare research
* **Key Points:**
  - "Knowledge graphs store information about the research subject in medical and other research use cases. For example, the knowledge graph could have protein and genome sequences together with environmental and chemical data, revealing intricate patterns and expanding our knowledge of proteins."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Getting started with knowledge graphs
* **Key Points:**
  - "Knowledge graphs are organized representations of real-world entities and their relationships, overlaid with one or more organizing principles that frame the information in context to drive insight from the data. Knowledge graphs underpin insightful applications and artificial intelligence solutions across many use cases."
  - "The Developer's Guide: How to Build a Knowledge Graph: Master the concepts and get hands-on experience with building your first knowledge graph."
  - "The guide covers how to build, manage, query, analyze, and visualize your knowledge graph for applications and analytics."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None


---

# What is GraphRAG?

## Introduction to retrieval-augmented generation (RAG)
* **Key Points:**
  - "Before diving into the specifics of GraphRAG, it's essential to understand the basic concepts of RAG. Let's take a closer look at the three key phases of RAG: Retrieval: In this phase, the RAG system retrieves relevant information from external data sources, such as documents or databases, based on the user's query. The retrieval process can use different techniques to identify the most pertinent data, such as similarity searches or database queries. The results are then ranked and scored based on their relevance to the query."
  - "Augmentation: During the augmentation phase, the retrieved information is combined with the original user question, along with any additional instructions or context. This augmented prompt provides a richer context for the language model to generate a response. The goal is to force the model to only use this relevant information to produce an accurate and useful output."
  - "Generation: In the final phase, the augmented prompt is processed by an LLM, which generates an answer in a requested format using only the provided context rather than relying on its pre-trained knowledge. The response can also link source information and additional metadata."
  - "By augmenting the language model with external knowledge and using the model's natural language understanding capabilities to retrieve and process this information, RAG systems can produce more accurate and informative responses compared to standalone language models that rely solely on their pre-trained knowledge."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval-Augmented Generation)`, `LLM`
* **Code Snippet:** None

---

## Limitations of vector-only RAG
* **Key Points:**
  - "Many baseline RAG systems rely solely on vector search over text embeddings (numerical vector representations) for information retrieval. To accurately capture the cohesive semantic meaning of a piece of text, the source documents are often chunked into smaller fragments, which are then embedded, indexed, and stored for retrieval."
  - "However, this approach has its limitations. By relying solely on vector search, the response's content is confined to the text fragments in the retrieved chunks. This can lead to incomplete or fragmented answers."
  - "For example, if a user asks a question about a specific product feature, a vector-only RAG system might retrieve chunks that mention the product but fail to include relevant information from other parts of the documentation that provide a more comprehensive answer."
  - "Moreover, due to the black-box nature of vector representations and vector search, such methods cannot explain the sources of the gathered information. This means that users and developers have limited visibility into why certain chunks were retrieved and how they contribute to the generated response. This lack of explainability can be a significant drawback, particularly in domains where transparency and accountability are crucial, such as healthcare or finance."
  - "To address these shortcomings, new techniques are emerging to improve different phases of the RAG process (retrieval, augmentation, and generation)."
  - "Since the information provided to the LLM is crucial to answer quality, improving the retrieval mechanism often has the most significant impact. GraphRAG is a common approach to enhance retrieval by incorporating structured domain knowledge stored in a knowledge graph. By tapping into the rich connections and semantic relationships in a knowledge graph, GraphRAG aims to overcome the limitations of vector-only RAG and provide more accurate and explainable responses."
* **Technical Entities (Classes/Functions/APIs):** `vector search`, `text embeddings`, `GraphRAG`, `knowledge graph`
* **Code Snippet:** None

---

## Knowledge graph for data representation
* **Key Points:**
  - "A knowledge graph model is especially suitable for representing structured and unstructured data with connected elements. Unlike traditional databases, they don't require a rigid schema but are more flexible in the data model. The graph model allows efficient storage, management, querying, and processing of the richness of real-world information. In a RAG system, the knowledge graph serves as the flexible memory companion to the language skills of LLMs, such as summarization, translation, and extraction."
* **Technical Entities (Classes/Functions/APIs):** `knowledge graph`, `LLMs`
* **Code Snippet:** None

---

## Knowledge graph components
* **Key Points:**
  - "In a knowledge graph, facts and entities are represented as nodes with attributes connected with typed relationships, which also carry attributes for qualification. This graph model can scale from a simple family tree to the complete digital twin of a company encompassing employees, customers, processes, products, partnerships, and resources, with millions or billions of connections."
  - "Graph structures can originate from various sources, from a structured business domain, (hierarchical) document representations, and signals computed by graph algorithms."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Graph querying for GraphRAG retrievers
* **Key Points:**
  - "Graphs can be navigated (traversed) by following simple patterns like (node:Type)-[relationship:TYPE]->(node:Type) or more complex variants expressed in Graph query languages like Cypher or GQL. Pattern matching results in paths whose nodes, relationships, and attributes can be filtered, aggregated, and sorted like in other query languages like SQL."
  - "Here is an example of a graph query that returns neighborhood information from a vector embedding search:"
* **Technical Entities (Classes/Functions/APIs):** `Cypher`, `GQL`, `db.index.vector.queryNodes`
* **Code Snippet:**
```cypher
CALL db.index.vector.queryNodes(docs, 5, $embedding) yield node as doc, score
RETURN score, doc, COLLECT { 
MATCH path = (doc)-[rel]-(neighbor)
RETURN path
} as paths
ORDER BY score DESC LIMIT 10
```

---

## How GraphRAG improves retrieval
* **Key Points:**
  - "A GraphRAG retrieval can find starting points in this network of data via vector, fulltext, spatial, or other searches and then follow relevant relationships to gather additional information to satisfy the user queries. The context of the user and task is considered to increase relevance. All captured nodes, relationships and their attributes can be filtered and ranked before being returned as context in the augmentation phase."
  - "This approach offers several advantages over vector-only RAG systems: By navigating the graph structure and following relevant relationships, GraphRAG can retrieve information that may not be directly mentioned in the initial set of retrieved chunks, providing a more comprehensive and contextually relevant response."
  - "The ability to filter and rank the retrieved information based on the user's context and task allows GraphRAG to prioritize the most pertinent information, improving the overall quality of the generated response."
  - "GraphRAG enables better explainability by capturing the relationships between the retrieved information, making it easier to trace the sources and reasoning behind the generated response."
  - "By using the knowledge graph's ability to integrate structured and unstructured data, as well as computed signals, GraphRAG can provide more informed and nuanced responses that draw from a wider range of information sources."
  - "These improvements in the retrieval phase contribute to GraphRAG's ability to generate more accurate, relevant, and traceable responses compared to vector-only RAG systems."
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG`, `vector`, `fulltext`, `spatial`
* **Code Snippet:** None

---

## Types of GraphRAG retrievers
* **Key Points:**
  - "The actual graph retrieval depends on the use case and domain. Different types of retrievers can be combined, and their results ranked, combined, or sequenced. In an agentic setup, retrievers can become tools that the LLM selects and runs iteratively, passing parameters and results until the necessary information to answer the question is collected."
  - "Examples of GraphRAG retriever types include: Vector (Embedding), Fulltext, Spatial, or other Search Indexes: Using index searches with information from the user question to determine starting points in the graph for further exploration."
  - "Neighborhood Traversal: Access direct or indirect neighbors of a node to put a piece of information into context."
  - "Path Traversals: Find paths between starting entities, expand relationships to their neighborhood, and retrieve additional related documents, claims, and other entities."
  - "Global Queries: Using pre-computed, cross-topic summarization and other global representations of insights to answer general questions (see Microsoft's GraphRAG with Query Focused Summarization)."
  - "Query Templates: Use case-specific queries for categories of questions are provided by a domain expert, can have the same starting points but explore different sub-graphs, and can be selected by categorizing questions."
  - "Dynamic Cypher Generation (Text2Cypher): A (fine-tuned) LLM generates a Cypher query from the user question and the graph schema description to answer specific and structural questions."
  - "Agentic Traversal: Using different retrievers, an LLM selects and executes them in a planned sequence to collect all information to answer the question."
  - "Graph Embedding Retrievers: Using embeddings to represent the 'essence' of a node's neighborhood and allow fuzzy topological search by matching candidate embeddings."
  - "You can find more examples in the GraphRAG Pattern Catalog on graphrag.com."
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG retrievers`, `Vector (Embedding)`, `Fulltext`, `Spatial`, `Neighborhood Traversal`, `Path Traversals`, `Global Queries`, `Query Templates`, `Dynamic Cypher Generation (Text2Cypher)`, `Agentic Traversal`, `Graph Embedding Retrievers`, `Cypher`, `LLM`
* **Code Snippet:** None

---

## Knowledge graph construction
* **Key Points:**
  - "For GraphRAG to work well, we need to ensure that our data has a shape that accurately represents the highly relevant, connected pieces of information. To create this knowledge graph, we need to follow two steps, which can be repeated for refinement: Model the relevant nodes and relationships to represent our domain data."
  - "Import, create, or compute the graph structures to fit this graph model."
  - "We can combine different sources of data: Import existing structured data from databases, files or APIs."
  - "Turn unstructured data (text, audio, video) into a graph representation of document structures/hierarchies and add vector embeddings and full-text indexes for chunks."
  - "Construct or connect structured entities (with optional embeddings) and their relationships from textual information."
  - "Enhance existing graphs with additional computation or algorithms, such as topic-clustering summaries (like in Microsoft Query Focused Summarization), similarity relationships, and personalized page rank (PPR) scores."
  - "These graph models and sources are also described in more detail in the GraphRAG pattern catalog."
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG`, `vector embeddings`, `full-text indexes`, `personalized page rank (PPR)`
* **Code Snippet:** None

---

## A GraphRAG example with Neo4j
* **Key Points:**
  - "A frequent use case for GraphRAG is analyzing research information in more detail than just 'chatting with your PDF.' In a vector-only semantic search approach, the data returned from the retrievers are just scored chunks of text with little or no information on how they relate to concepts from the domain or each other."
  - "In contrast, a GraphRAG approach allows us to extract entities such as Person, Organization, Article, Paper, BiologicalProcess, Condition, Disease, Drug, Gene, Expression, Exposure, and Pathway that appear in our documents and create a rich network of information."
  - "To demonstrate this, let's walk through an example of constructing a knowledge graph using the open source neo4j-graphrag package. You can also use LangChain, LlamaIndex, or other integrations."
  - "In this example, we use the SimpleKGPipeline, which comes with a number of defaults and executes the steps depicted below:"
  - "To run this extraction, we configure the Pipeline with the following components: LLM (e.g., gpt-4o-mini from OpenAI), Embedding model, Document splitter, Graph schema"
  - "Once configured, we can execute the pipeline on our dataset of biomedical research papers:"
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG`, `neo4j-graphrag`, `LangChain`, `LlamaIndex`, `SimpleKGPipeline`, `OpenAILLM`, `OpenAIEmbeddings`, `FixedSizeSplitter`, `Neo4j`
* **Code Snippet:**
```python
driver = neo4j.GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USERNAME, NEO4J_PASSWORD))

ex_llm=OpenAILLM(
    model_name="gpt-4o-mini",
    model_params={
        "response_format": {"type": "json_object"},
        "temperature": 0
    }
)
embedder = OpenAIEmbeddings()

node_labels = ["Anatomy", "BiologicalProcess", ...]
rel_types = ["ACTIVATES", "AFFECTS", "ASSESSES",..."TREATS", "USED_FOR"]

kg_builder_pdf = SimpleKGPipeline(
    llm=ex_llm,
    driver=driver,
    text_splitter=FixedSizeSplitter(chunk_size=500, chunk_overlap=100),
    embedder=embedder,
    entities=node_labels,
    relations=rel_types,
    prompt_template=prompt_template,
    from_pdf=True
)

pdf_file_paths = ['biomolecules-11-00928-v2.pdf', 'GAP-between-patients-and-clinicians_2023_Best-Practice.pdf','pgpm-13-39.pdf']

for path in pdf_file_paths:
    graph_data = await kg_builder_pdf.run_async(file_path=path)
```

---

## A GraphRAG example with Neo4j (continued)
* **Key Points:**
  - "After storing the chunked document data and graph data in Neo4j, we can visualize it using our Query tool."
  - "Now, we can execute a GraphRAG retriever and compare its results with a vector RAG retriever."
  - "This retriever first executes a vector search for the indexed text chunks and then follows non-chunk relationships up to 2 hops out, retrieving not only the directly extracted entities but also their first- and second-degree neighbors. It returns the chunk texts and entity-relationship-entity pairs as context for use in the final phases of prompt augmentation and answer generation."
  - "Next, we build the vector and GraphRAG pipelines using each retriever with a suitable LLM (here, using the better OpenAI gpt-4o) and prompt for question answering:"
  - "The comparison of answers shows that the GraphRAG response is much more comprehensive and covers more of the relevant context."
* **Technical Entities (Classes/Functions/APIs):** `VectorCypherRetriever` (neo4j_graphrag.retrievers), `RagTemplate`, `GraphRAG`, `vector_retriever`, `graph_retriever`, `search()`
* **Code Snippet:**
```python
from neo4j_graphrag.retrievers import VectorCypherRetriever

graph_retriever = VectorCypherRetriever(
    driver,
    index_name="text_embeddings",
    embedder=embedder,
    retrieval_query="""
//1) Go out 2-3 hops in the entity graph and get relationships
WITH node AS chunk
MATCH (chunk)<-[:FROM_CHUNK]-(entity)-[relList:!FROM_CHUNK]-{1,2}(nb)
UNWIND relList AS rel

//2) collect relationships and text chunks
WITH collect(DISTINCT chunk) AS chunks, collect(DISTINCT rel) AS rels

//3) format and return context
RETURN apoc.text.join([c in chunks | c.text], '\n') + 
  apoc.text.join([r in rels | 
  startNode(r).name+' - '+type(r)+' '+r.details+' -> '+endNode(r).name],  
  '\n') AS info
"""
)
```

```python
llm = LLM(model_name="gpt-4o",  model_params={"temperature": 0.0})

rag_template = RagTemplate(template='''Answer the Question using the following Context. Only respond with information mentioned in the Context. Do not inject any speculative information not mentioned. 

# Question:
{query_text}
 
# Context:
{context}

# Answer:
''', expected_inputs=['query_text', 'context'])

vector_rag  = GraphRAG(llm=llm, retriever=vector_retriever, prompt_template=rag_template)

graph_rag = GraphRAG(llm=llm, retriever=graph_retriever, prompt_template=rag_template)

q = "Can you summarize systemic lupus erythematosus (SLE)? including common effects, biomarkers, and treatments? Provide in detailed list format."

vector_rag.search(q, retriever_config={'top_k':5}).answer
graph_rag.search(q, retriever_config={'top_k':5}).answer
```

---

## Common GraphRAG use cases
* **Key Points:**
  - "GraphRAG is used in applications and domains that require a higher level of trust, as their outputs are used for critical business decision-making. Some examples include: Legal and Compliance: Reviewing and analyzing contracts, cases, laws, and regulations."
  - "Investment Research: Investigating organizations, people, competitors, markets, and trends."
  - "Biotech: Accessing knowledge graphs for drug discovery and repurposing, clinical trials, and research."
  - "Business Process Support: Integrating various business data sources into a cohesive view of an organization."
  - "Supply Chain: Conducting investigations for risk assessment, compliance, and sustainability of products and production processes."
  - "Fraud Detection: Identifying and preventing money laundering (AML), insurance fraud, and other fraudulent activities."
  - "Investigative Journalism: Uncovering connections and patterns in large datasets for news stories and investigations."
  - "Natural Language Search and Chatbots: Democratizing access to pre-existing knowledge bases through user-friendly interfaces."
* **Technical Entities (Classes/Functions/APIs):** `GraphRAG`
* **Code Snippet:** None

---

## GraphRAG: Enabling enterprise-grade AI apps
* **Key Points:**
  - "RAG architectures are currently the most effective way to provide reliable content for GenAI business applications by using data from trusted data sources. GraphRAG takes this a step further, improving upon basic vector-based RAG in both quality and explainability."
  - "By considering more relevant context and using a variety of retrievers that navigate document, domain, and computed graph structures, GraphRAG delivers more accurate, trustworthy, and traceable results. The combination of knowledge graphs, with their rich representation of real-world information, and LLMs, with their advanced language skills, creates a robust and reliable solution for enterprise use cases."
  - "As more organizations adopt GenAI, GraphRAG will be essential in ensuring the accuracy, reliability, and transparency of these systems, paving the way for better decision-making and improved business outcomes."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `GraphRAG`, `knowledge graphs`, `LLMs`
* **Code Snippet:** None