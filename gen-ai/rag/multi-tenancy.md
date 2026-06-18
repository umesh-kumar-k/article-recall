---
aliases:
  - Multi Tenancy
highlights: Logical data isolation via metadata filterting or physical isolation via tenant-specific indices; impacts cost/complexity trade offs
tags:
  - rag
  - multi-tenant-rag
Source 1: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/secure-multitenant-rag
Source 2: https://www.llamaindex.ai/blog/building-multi-tenancy-rag-system-with-llamaindex-0d6ab4e0c44b
---
# Building Multi-Tenancy RAG System with LlamaIndex

## Introduction:
* **Key Points:**
  - "The concept of Multi-Tenancy in RAG (Retriever-Augmented Generation) systems has become increasingly vital, especially when it comes to data security and privacy. Multi-Tenancy, in simple terms, refers to a system's ability to serve multiple users ('tenants') independently and securely."
  - "Consider this scenario: In a RAG system, there are two users, User-1 and User-2. Both have their own set of documents which they have indexed into the system. The critical aspect of Multi-Tenancy here is that when User-1 queries the system, they should only receive answers from the documents they have indexed, and not from the documents indexed by User-2, and vice versa. This separation is crucial for maintaining data confidentiality and security, as it prevents the accidental or unauthorized cross-referencing of private information between different users."
  - "In the context of Multi-Tenancy in RAG systems, this means designing a system that not only understands and retrieves information effectively but also strictly adheres to user-specific data boundaries. Each user's interaction with the system is isolated, ensuring that the retriever component of the RAG pipeline accesses only the information relevant and permitted for that particular user. This approach is important in scenarios where sensitive or proprietary data is involved, as it safeguards against data leaks and privacy breaches."
  - "In this blog post, we will look into Building a Multi-Tenancy RAG System with LlamaIndex."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retriever-Augmented Generation)`, `Multi-Tenancy`, `LlamaIndex`
* **Code Snippet:** None

---

## Solving Multi-Tenancy Challenges
* **Key Points:**
  - "The key to managing Multi-Tenancy lies within the metadata. When indexing documents, we incorporate user-specific information into the metadata before adding it to the index. This ensures that each document is uniquely tied to an individual user."
  - "During the query phase, the retriever uses this metadata to filter and only access documents associated with the querying user. Subsequently, it performs a semantic search to retrieve the most relevant information segments, or 'top_k chunks', for that user. By implementing this approach, we effectively prevent the unauthorized cross-referencing of private information between different users, upholding the integrity and confidentiality of each user's data."
  - "Now that we've discussed the concept, let's dive into the construction of a Multi-Tenancy RAG system. For an in-depth step-by-step guide, feel free to follow along with the subsequent instructions in our Google Colab Notebook."
* **Technical Entities (Classes/Functions/APIs):** `metadata`, `retriever`, `top_k chunks`
* **Code Snippet:** None

---

## Download Data:
* **Key Points:**
  - "We will use An LLM Compiler for Parallel Function Calling and Dense X Retrieval: What Retrieval Granularity Should We Use? papers for the demonstrations."
* **Technical Entities (Classes/Functions/APIs):** `wget`
* **Code Snippet:**
```bash
!wget --user-agent "Mozilla" "https://arxiv.org/pdf/2312.04511.pdf" -O "llm_compiler.pdf"
!wget --user-agent "Mozilla" "https://arxiv.org/pdf/2312.06648.pdf" -O "dense_x_retrieval.pdf"
```

---

## Load Data:
* **Key Points:**
  - "We will load the data of LLMCompiler paper for user Jerry and Dense X Retrieval paper for user Ravi"
* **Technical Entities (Classes/Functions/APIs):** `SimpleDirectoryReader`
* **Code Snippet:**
```python
reader = SimpleDirectoryReader(input_files=['dense_x_retrieval.pdf'])
documents_jerry = reader.load_data()

reader = SimpleDirectoryReader(input_files=['llm_compiler.pdf'])
documents_ravi = reader.load_data()
```

---

## Create An Empty Index:
* **Key Points:**
  - "We will initially create an empty index to which we can insert documents, each tagged with metadata containing the user information."
* **Technical Entities (Classes/Functions/APIs):** `VectorStoreIndex`
* **Code Snippet:**
```python
index = VectorStoreIndex.from_documents(documents=[])
```

---

## Ingestion Pipeline:
* **Key Points:**
  - "The IngestionPipeline is useful for data ingestion and performing transformations, including chunking, metadata extraction, and more. Here we utilize it to create nodes, which are then inserted into the index."
* **Technical Entities (Classes/Functions/APIs):** `IngestionPipeline`, `SentenceSplitter`
* **Code Snippet:**
```python
pipeline = IngestionPipeline(
    transformations=[
        SentenceSplitter(chunk_size=512, chunk_overlap=20),
    ]
)
```

---

## Update Metadata and Insert Documents:
* **Key Points:**
  - "We will update the metadata of the documents with each user and insert the documents into the index."
* **Technical Entities (Classes/Functions/APIs):** `document.metadata`, `pipeline.run()`, `index.insert_nodes()`
* **Code Snippet:**
```python
# For user Jerry
for document in documents_jerry:
    document.metadata['user'] = 'Jerry'

nodes = pipeline.run(documents=documents_jerry)
# Insert nodes into the index
index.insert_nodes(nodes)

# For user Ravi
for document in documents_ravi:
    document.metadata['user'] = 'Ravi'

nodes = pipeline.run(documents=documents_ravi)
# Insert nodes into the index
index.insert_nodes(nodes)
```

---

## Define Query Engines:
* **Key Points:**
  - "We will define query engines for both the users with necessary filters."
* **Technical Entities (Classes/Functions/APIs):** `index.as_query_engine()`, `MetadataFilters`, `ExactMatchFilter`
* **Code Snippet:**
```python
# For Jerry
jerry_query_engine = index.as_query_engine(
    filters=MetadataFilters(
        filters=[
            ExactMatchFilter(
                key="user",
                value="Jerry",
            )
        ]
    ),
    similarity_top_k=3
)

# For Ravi
ravi_query_engine = index.as_query_engine(
    filters=MetadataFilters(
        filters=[
            ExactMatchFilter(
                key="user",
                value="Ravi",
            )
        ]
    ),
    similarity_top_k=3
)
```

---

## Querying:
* **Key Points:**
  - "Jerry has Dense X Rerieval paper and should be able to answer following question."
  - "Ravi has LLMCompiler paper"
  - "This should not be answered as Jerry does not have information about LLMCompiler"
  - "As demonstrated, if Jerry queries the system regarding a document indexed by Ravi, the system does not retrieve any answers from that document."
* **Technical Entities (Classes/Functions/APIs):** `jerry_query_engine.query()`, `ravi_query_engine.query()`
* **Code Snippet:**
```python
# Jerry has Dense X Rerieval paper and should be able to answer following question.
response = jerry_query_engine.query(
    "what are propositions mentioned in the paper?"
)
```
```python
# Ravi has LLMCompiler paper
response = ravi_query_engine.query("what are steps involved in LLMCompiler?")
```
```python
# This should not be answered as Jerry does not have information about LLMCompiler
response = jerry_query_engine.query("what are steps involved in LLMCompiler?")
```

---

## What's Next?
* **Key Points:**
  - "We have included a MultiTenancyRAGPack within the LlamaPacks and Replit template which offers a Streamlit interface for hands-on experience. Be sure to explore it."
* **Technical Entities (Classes/Functions/APIs):** `MultiTenancyRAGPack`, `LlamaPacks`, `Replit`, `Streamlit`
* **Code Snippet:** None


---

# Design a secure multitenant RAG inferencing solution

## Single-tenant RAG architecture with an orchestrator
* **Key Points:**
  - "In this single-tenant RAG architecture, an orchestrator fetches relevant proprietary tenant data from the data stores and provides it as grounding data to the foundation model."
  - "A user issues a request to the intelligent web application."
  - "An identity provider authenticates the requestor."
  - "The intelligent application calls the orchestrator API with the user's query and the authorization token for the user."
  - "The orchestration logic extracts the user's query from the request and calls the appropriate data store to fetch relevant grounding data for the query. The grounding data is added to the prompt that's sent to the foundation model, like a model that's exposed in Azure OpenAI, in the next step."
  - "The orchestration logic connects to the foundation model's inferencing API and sends the prompt that includes the retrieved grounding data. The results are returned to the intelligent application."
* **Technical Entities (Classes/Functions/APIs):** `RAG (Retrieval-Augmented Generation)`, `orchestrator`, `foundation model`, `Azure OpenAI`
* **Code Snippet:** None

---

## Single-tenant RAG architecture with direct data access
* **Key Points:**
  - "This variant of the single-tenant RAG architecture uses the On Your Data feature of Azure OpenAI to integrate directly with data stores like Azure AI Search."
  - "Important: Azure OpenAI On Your Data is deprecated and approaching retirement. We recommend that you migrate Azure OpenAI On Your Data workloads to Foundry Agent Service with Foundry IQ to retrieve content and generate grounded answers from your data."
  - "In this architecture, you either don't have your own orchestrator, or your orchestrator has fewer responsibilities. The Azure OpenAI API calls into the data store to fetch the grounding data and passes that data to the language model. This method gives you less control over what grounding data to fetch and the relevancy of that data."
  - "Note: This Azure OpenAI feature integrates with the data store, but the model itself doesn't integrate with the data store. The model receives grounding data in the same way as it does when an orchestrator fetches the data."
  - "In this RAG architecture, the service that provides the foundation model fetches the appropriate proprietary tenant data from the data stores and uses that data as grounding data to the foundation model."
  - "A user issues a request to the intelligent web application."
  - "An identity provider authenticates the requestor."
  - "The intelligent application calls Azure OpenAI with the user's query."
  - "Azure OpenAI connects to supported data stores, such as AI Search and Azure Blob Storage, to fetch the grounding data. The grounding data is used as part of the context when Azure OpenAI calls the OpenAI language model. The results are returned to the intelligent application."
  - "If you want to use this architecture in a multitenant solution, then the service that directly accesses the grounding data, such as Azure OpenAI, must support the multitenant logic that your solution requires."
* **Technical Entities (Classes/Functions/APIs):** `On Your Data` (Azure OpenAI), `Azure AI Search`, `Foundry Agent Service`, `Foundry IQ`, `Azure Blob Storage`, `OpenAI language model`
* **Code Snippet:** None

---

## Multitenancy in RAG architecture
* **Key Points:**
  - "In multitenant solutions, tenant data might exist in a tenant-specific store or coexist with other tenants in a multitenant store. Data might also be in a store that's shared across tenants. Only data that the user is authorized to access should be used as grounding data. The user should see only common or all-tenant data or data from their tenant that's filtered to help ensure that they see only the data that they're authorized to access."
  - "A user issues a request to the intelligent web application."
  - "An identity provider authenticates the requestor."
  - "The intelligent application calls the orchestrator API with the user's query and the authorization token for the user."
  - "The orchestration logic extracts the user's query from the request and calls the appropriate data stores to fetch tenant-authorized, relevant grounding data for the query. The grounding data is added to the prompt that's sent to Azure OpenAI in the next step. Some or all of the following steps are included: The orchestration logic fetches grounding data from the appropriate tenant-specific data store instance and potentially applies security filtering rules to return only the data that the user is authorized to access."
  - "The orchestration logic fetches the appropriate tenant's grounding data from the multitenant data store and potentially applies security filtering rules to return only the data that the user is authorized to access."
  - "The orchestration logic fetches data from a data store that's shared across tenants."
  - "The orchestration logic connects to the foundation model's inferencing API and sends the prompt that includes the retrieved grounding data. The results are returned to the intelligent application."
* **Technical Entities (Classes/Functions/APIs):** `multitenant solution`, `orchestrator`, `Azure OpenAI`, `foundation model`
* **Code Snippet:** None

---

## Design considerations for multitenant data in RAG
### Choose a store isolation model
* **Key Points:**
  - "The two main architectural approaches for storage and data in multitenant scenarios are store-per-tenant and multitenant stores. These approaches are in addition to stores that contain data shared across tenants. Your multitenant solution can use a combination of these approaches."
  - "In store-per-tenant stores, each tenant has its own store. The advantages of this approach include both data and performance isolation. Each tenant's data is encapsulated in its own store. In most data services, the isolated stores aren't susceptible to the noisy neighbor problem of other tenants. This approach also simplifies cost allocation because the entire cost of a store deployment can be attributed to a single tenant."
  - "This approach might present challenges such as increased management and operational overhead and higher costs. You shouldn't use this approach if you have a large number of small tenants, like in business-to-consumer scenarios. This approach might also reach or exceed service limits."
  - "In the context of this AI scenario, a store-per-tenant store means that the necessary grounding data to bring relevancy into the context comes from an existing or new data store that only contains grounding data for the tenant. In this topology, the database instance is the discriminator that's used for each tenant."
  - "In multitenant stores, multiple tenants' data coexists in the same store. The advantages of this approach include the potential for cost optimization, the ability to handle a higher number of tenants than the store-per-tenant model, and lower management overhead because of the lower number of store instances."
  - "The challenges of using shared stores include the need for data isolation and management, the potential for the noisy neighbor antipattern, and more complex cost allocation to tenants. Data isolation is the most important concern when you use this approach. You need to implement secure approaches to help ensure that tenants can only access their data. Data management can also be challenging if tenants have different data lifecycles that require operations such as building indexes on different schedules."
  - "Some platforms have features that you can use when you implement tenant data isolation in shared stores. For example, Azure Cosmos DB has native support for data partitioning and sharding. It's typical to use a tenant identifier as a partition key to provide some isolation between tenants. Azure SQL and Azure Database for PostgreSQL support row-level security. However, these features aren't typically used in multitenant solutions because you have to design your solution around these features if you plan to use them in your multitenant store."
  - "In the context of this AI scenario, grounding data for all tenants commingle in the same data store. Therefore, your query to that data store must include a tenant discriminator to help ensure that responses are restricted to bring back only relevant data within the context of the tenant."
  - "Multitenant solutions often share data across tenants. In an example multitenant solution for the healthcare domain, a database might store general medical information or information that isn't specific to the tenant."
  - "In the context of this AI scenario, the grounding data store is generally accessible and doesn't need filtering based on specific tenants because the data is relevant and authorized for all tenants in the system."
* **Technical Entities (Classes/Functions/APIs):** `store-per-tenant`, `multitenant stores`, `shared stores`, `Azure Cosmos DB`, `Azure SQL`, `Azure Database for PostgreSQL`, `row-level security`, `partition key`
* **Code Snippet:** None

---

## Identity
* **Key Points:**
  - "Identity is a key aspect of multitenant solutions, including multitenant RAG solutions. The intelligent application should integrate with an identity provider to authenticate the identity of the user. The multitenant RAG solution needs an identity directory that stores authoritative identities or references to identities. This identity needs to flow through the request chain and allow downstream services, such as the orchestrator or the data store itself, to identify the user."
  - "You also need a way to map a user to a tenant so that you can grant access to that tenant data."
* **Technical Entities (Classes/Functions/APIs):** `identity provider`, `orchestrator`
* **Code Snippet:** None

---

## Define your tenant and authorization requirements
* **Key Points:**
  - "When you build a multitenant RAG solution, you must define what a tenant is for your solution. The two common models to choose from are business-to-business and business-to-consumer models. The model that you choose helps you determine what other factors you should consider when you build your solution. Understanding the number of tenants is critical for choosing the data store model. A large number of tenants might require a model that has multiple tenants for each store. A smaller number of tenants might allow for a store-per-tenant model. The amount of data for each tenant is also important. Tenants that have large amounts of data might prevent you from using multitenant stores because of size limitations on the data store."
  - "If you intend to expand an existing workload to support this AI scenario, you might have made this decision already. Generally speaking, you can use your existing data storage topology for the grounding data if that data store can provide sufficient relevancy and meet any other nonfunctional requirements. However, if you plan to introduce new components, such as a dedicated vector search store as a dedicated grounding store, then you still need to make this decision. Consider factors such as your current deployment stamp strategy, your application control plane impact, and any per-tenant data lifecycle differences, such as pay-for-performance situations."
  - "After you define what a tenant is for your solution, you need to define your authorization requirements for data. Tenants only access data from their tenant, but your authorization requirements might be more granular. For example, in a healthcare solution, you might have rules such as: A patient can only access their own patient data."
  - "A healthcare professional can access their patients' data."
  - "A finance user can access only finance-related data."
  - "A clinical auditor can see all patients' data."
  - "All users can access basic medical knowledge in a shared data store."
  - "In a document-based RAG application, you might want to restrict users' access to documents based on a tagging scheme or sensitivity levels assigned to the documents."
  - "After you have a definition of what a tenant is and have a clear understanding of the authorization rules, use that information as requirements for your data store solution."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `vector search store`
* **Code Snippet:** None

---

## Data filtering
* **Key Points:**
  - "Restricting access to only the data that users are authorized to access is known as filtering or security trimming. In a multitenant RAG scenario, a user might be mapped to a tenant-specific store. That doesn't mean that the user should be able to access all the data in that store. Define your tenant and authorization requirements discusses the importance of defining authorization requirements for your data. You should use these authorization rules as the basis for filtering."
  - "You can use data platform capabilities like row-level security to implement filtering. Or you might need custom logic, data, or metadata. These platform features aren't typically used in multitenant solutions because you need to design your system around these features."
* **Technical Entities (Classes/Functions/APIs):** `filtering`, `security trimming`, `row-level security`
* **Code Snippet:** None

---

## Encapsulate multitenant data logic
* **Key Points:**
  - "We recommend that you have an API in front of the storage mechanism that you use. The API acts like a gatekeeper that helps ensure that users only get access to information they're authorized to access."
  - "Users' access to data can be limited by: The user's tenant."
  - "Platform features."
  - "Custom security filtering or trimming rules."
  - "The API layer should: Route the query to a tenant-specific store in a store-per-tenant model."
  - "Select only data from the user's tenant in multitenant stores."
  - "Use the appropriate identity for a user to support platform-enabled authorization logic."
  - "Enforce custom security trimming logic."
  - "Store access logs of grounding information for audit purposes."
  - "Code that needs to access tenant data shouldn't be able to query the back-end stores directly. All requests for data should flow through the API layer. This API layer provides a single point of governance or security on top of your tenant data. This approach prevents the tenant and user data access authorization logic from reaching other areas of the application. This logic is encapsulated in the API layer. This encapsulation makes the solution easier to validate and test."
* **Technical Entities (Classes/Functions/APIs):** `API layer`, `gatekeeper`
* **Code Snippet:** None

---

## Summary
* **Key Points:**
  - "When you design a multitenant RAG inferencing solution, you must consider how to architect the grounding data solution for your tenants. Understand the number of tenants and the amount of per-tenant data that you store. This information helps you design your data tenancy solution. We recommend that you implement an API layer that encapsulates the data access logic, including multitenant logic and filtering logic."
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `API layer`
* **Code Snippet:** None