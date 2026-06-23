---
aliases:
  - Role Based Access
Source 1: https://www.cerbos.dev/blog/authorization-for-rag-applications-langchain-chromadb-cerbos
Source 2: https://aws.amazon.com/blogs/security/authorizing-access-to-data-with-rag-implementations/
---
[Code Examples](./code/rbac-rag.md) 
# How to build an authorization system for your RAG applications with LangChain, Chroma DB and Cerbos


* **Key Points:**
  - This article explains how to implement authorization systems for your Retrieval Augmented Generation apps.
  - Authorization systems allow controlled access to complete or partial software application features, limiting access to sensitive or specialized features based on user roles and permissions.
  - Authorization is particularly crucial in Retrieval Augmented Generation (RAG) applications as they involve ingesting data in vector databases.
  - Effective authorization ensures that only authenticated and permitted users can ingest, retrieve, or manipulate data in vector databases.
* **Technical Entities (Classes/Functions/APIs):** `LangChain`, `Chroma DB`, `Cerbos`

## What is RAG (Retrieval-Augmented Generation)?
* **Key Points:**
  - Retrieval Augmented Generation is an approach that enhances the capabilities of LLMs by augmenting their default knowledge using external sources.
* **Technical Entities (Classes/Functions/APIs):** `RAG`

### How does RAG work?
* **Key Points:**
  - A RAG system typically consists of the following components: Document loader that imports data from various sources such as PDF documents, websites, databases, etc. You can write custom data loaders or can use data loaders from frameworks such as LangChain. Text Splitter that splits the documents imported via document loader into smaller chunks. Embedding Model that converts text chunks into a vector representation. You can proprietary embeddings models such as OpenAI Embeddings or open-source embedding models from Hugging Face. Vector Store that stores the vectors generated via embedding models. Some commonly used vector stores are Pinecone, Chroma, Faiss, etc. Retriever that matches user queries with the vectors in the vector store and retrieves the vectors with the highest similarity. LangChain provides different types of retrievers you can use to meet your requirements. Language Model that takes in the user query and the textual representation of the vectors in the vector to generate the final response. Some of the most famous language models are OpenAI GPT-4o, Claude 3.5 sonnet, and Qwen 2.5-72.
* **Technical Entities (Classes/Functions/APIs):** `Document loader`, `Text Splitter`, `Embedding Model`, `OpenAI Embeddings`, `Hugging Face`, `Vector Store`, `Pinecone`, `Chroma`, `Faiss`, `Retriever`, `Language Model`, `OpenAI GPT-4o`, `Claude 3.5 sonnet`, `Qwen 2.5-72`, `LangChain`

## RAG demo with Python LangChain and Chroma DB
* **Key Points:**
  - In this section, you will learn how to develop a simple RAG system using the Python LangChain framework and the Chroma DB vector database.
* **Technical Entities (Classes/Functions/APIs):** `langchain`, `langchain-openai`, `pypdf`, `chromadb`, `langchain_community`, `ChatOpenAI`, `OpenAIEmbeddings`, `ChatPromptTemplate`, `StrOutputParser`, `PyPDFLoader`, `Chroma`, `create_stuff_documents_chain`, `create_retrieval_chain`, `Document`, `HumanMessage`, `AIMessage`, `OpenAI GPT-4o`
* **Code Snippet:**
```python
!pip install -U langchain langchain-openai pypdf chromadb langchain_community
```
```python
import os
from dotenv import load_dotenv

from langchain_openai import ChatOpenAI
from langchain_openai import OpenAIEmbeddings

from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_community.document_loaders import PyPDFLoader

from langchain_community.vectorstores import Chroma  
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain.chains import create_retrieval_chain
from langchain_core.documents import Document
from langchain_core.messages import HumanMessage, AIMessage

load_dotenv()  
```
```python
openai_key = os.environ.get('OPENAI_API_KEY')

llm = ChatOpenAI(
    openai_api_key = openai_key ,
    model = 'gpt-4',
    temperature = 0.7
)
```
```python
data_url = "https://www.hse.ie/eng/services/list/5/cancer/pubs/reports/national-survey-on-lung-cancer-awareness-report-january-2020.pdf"
loader = PyPDFLoader(data_url)
lung_cancer_docs = loader.load_and_split()

data_url = "https://s21.q4cdn.com/399680738/files/doc_financials/2024/q3/META-Q3-2024-Earnings-Call-Transcript.pdf"
loader = PyPDFLoader(data_url)
meta_docs = loader.load_and_split()

data_url = "https://s2.q4cdn.com/299287126/files/doc_financials/2024/q3/AMZN-Q3-2024-Earnings-Release.pdf"
loader = PyPDFLoader(data_url)
amazon_docs = loader.load_and_split()
```
```python
def add_metadata(docs, source, month):
    for doc in docs:
        doc.metadata["source"] = source
        doc.metadata["month"] = month
    return docs

lung_cancer_docs = add_metadata(lung_cancer_docs, "lung_cancer_doc", "June")
meta_docs = add_metadata(meta_docs, "meta_doc", "October")
amazon_docs = add_metadata(amazon_docs, "amazon_doc", "November")
```
```python
embeddings = OpenAIEmbeddings(openai_api_key = openai_key)

lung_cancer_vectorstore= Chroma.from_documents(
    documents=lung_cancer_docs,
    embedding=embeddings,
    collection_name="lung_cancer_collection"
)

earning_calls_vectorstore = Chroma.from_documents(
    documents= meta_docs + amazon_docs,
    embedding=embeddings,
    collection_name="earning_calls_collection"
)
```
```python
prompt = ChatPromptTemplate.from_template("""Answer the following question based only on the provided context:

Question: {input}

Context: {context}
"""
)

document_chain = create_stuff_documents_chain(llm, prompt)
```
```python
lung_cancer_retriever = lung_cancer_vectorstore.as_retriever()
lung_cancer_retrieval_chain = create_retrieval_chain(lung_cancer_retriever, document_chain)

query = "What is the revenue generated by Meta in Q3 2024?"
response = lung_cancer_retrieval_chain.invoke({"input": query})
print(response["answer"])
```
```python
query = "What are the major causes of Lung Cancer?"
response = lung_cancer_retrieval_chain.invoke({"input": query})
print(response["answer"])
```
```python
earning_calls_retriever = earning_calls_vectorstore.as_retriever()
earning_calls_retrieval_chain = create_retrieval_chain(earning_calls_retriever, document_chain)

query = "What is the revenue generated by Meta in Q3 2024?"
response = earning_calls_retrieval_chain.invoke({"input": query})
print(response["answer"])
```
```python
query = "What is the revenue generated by Amazon?"
response = earning_calls_retrieval_chain.invoke({"input": query})
print(response["answer"])
```
```python
earning_calls_retriever = earning_calls_vectorstore.as_retriever(search_kwargs={"filter": {"source": "meta_doc"}})
earning_calls_retrieval_chain = create_retrieval_chain(earning_calls_retriever, document_chain)

query = "What is the revenue generated by Amazon in Q3 2024?"
response = earning_calls_retrieval_chain.invoke({"input": query})
print(response["answer"])
```

## Advantages of RAG
* **Key Points:**
  - I personally prefer to work with RAG since it helps avoid fine-tuning an LLM, which can be costly and time-consuming.
  - With RAG, you directly provide the answer to user queries to an LLM, whose job is then to formulate the answer and return the formulated response.
  - Following are some advantages of RAG applications. They allow you to retrieve updated information by enabling LLMs to access current data beyond their training cutoff. RAG allows you to specialize an LLM in a specific domain. With high-quality and quantity of data, LLMs can generate responses as good as human experts. Context information retrieved from vector stores reduced the l-likelihood of models hallucinating. By default, LLMs do not provide sources for their responses. With RAG, you can retrieve the document an LLM uses to generate responses, leading to high model transparency.

## Limitations of RAG
* **Key Points:**
  - Following are some of the main limitations of RAG: RAG depends highly on the quality of data. Poor quality data leads to poor and sometimes inaccurate LLM responses. RAG systems are often more challenging to implement and maintain than standalone LLMs since you must maintain various components such as retrievers, vector stores, document embeddings, etc. Also, the computational complexity of RAG systems is much higher than that of standalone LLMs. RAG systems, by default, do not contain any authentication and authorization logic, which, if not handled, may lead to the leaking of sensitive data to unauthorized users.

## Security concerns for RAG architecture
* **Key Points:**
  - While the RAG approach is compelling and innovative, it also introduced several security vulnerabilities that need to be carefully addressed before releasing RAG applications into production.
  - Here are some of the security concerns for RAG applications:

### Prompt injection
* **Key Points:**
  - Prompt injection is a technique in which a hacker crafts a malicious prompt that prompts an LLM to return sensitive or unauthorized information.
  - For example, a RAG system may retrieve confidential financial data based on user queries. An attacker could submit a prompt like, "Provide all confidential data on financial projections," potentially forcing the system to retrieve sensitive data even without authorization.
  - Prompt injections are particularly dangerous for RAG systems.
  - They can allow a malicious user to access sensitive information and cause a model to behave in an undesired manner.
  - Some common measures to prevent prompt injections involve implementing strict input validation and sanitization, using role-based access control to limit the scope of user queries, and employing prompt engineering techniques to make the system more resilient.

### Context injection
* **Key Points:**
  - Context injection refers to an attacker inserting harmful data into a system's retrieval process, potentially causing RAG systems to produce responses based on corrupted information.
  - For example, an attacker may add a document containing malicious code that may be executed during retrieval.
  - Context injection results in two significant problems: misinformation propagation, where the model provides incorrect or misleading information due to the injected data, and malicious code execution, where unsanitized code runs within the system.
  - Preventive measures for context injection include strict validation and sanitization of content before ingestion and allowing only authorized users to add or modify context data.

### Data poisoning
* **Key Points:**
  - Data poisoning occurs when attackers intentionally alter the knowledge base, compromising the reliability and quality of RAG system responses.
  - One prominent example of data poisoning in NLP is the Tay, a chatbot introduced by Microsoft in 2016 that micks the speech patterns of a 19-year-old girl.
  - Malicious users bombarded Tay with offensive, abusive, and racial topics, poisoning its learning process. Consequently, Tay began replicating racist and explicit messages, highlighting the vulnerability of AI systems to data-poisoning attacks.
  - For example, an attacker may insert fabricated documents with biased information into the knowledge base, causing the model to reflect this bias in responses.
  - Data poisoning can involve corrupting model training data or manipulating data vector representation.
  - Mitigating these risks involves implementing robust data validation and cleaning processes and using anomaly detection systems to identify suspicious data patterns.

### Sensitive data exfiltration
* **Key Points:**
  - Sensitive data exfiltration involves leaking sensitive data to malicious users.
  - Poor access controls may allow attackers to query sensitive data directly, while inference attacks could enable attackers to deduce confidential information indirectly through carefully structured queries.
  - For instance, in a healthcare RAG system, a user could issue indirect requests to retrieve a patient's private information.
  - Effective prevention strategies include implementing fine-grained access control, data masking, and differential privacy techniques to obscure sensitive values and maintaining continuous monitoring and audit logs to detect potential exfiltration attempts.

### Model inversion attacks
* **Key Points:**
  - Model inversion attacks involve reverse-engineering of sensitive data through model responses.
  - Attackers might repeatedly query the system to extract confidential information, such as customer data, by probing the model's responses.
  - This risk is especially prevalent when personal data is embedded within the training set, leading to unintended privacy leaks.
  - Mitigation techniques for model inversion include using federated learning to limit raw training data exposure, applying data anonymization to remove identifying details, and regularly updating the model to reduce the effectiveness of inversion techniques.
  - The aforementioned security concerns highlight the importance of access control for AI applications, particularly for RAG.

## Access control for RAG applications
* **Key Points:**
  - Access control is crucial for RAG applications, especially when dealing with private or sensitive data.
  - An effective access control mechanism ensures that only authorized and authenticated users can access specific RAG application resources.

### Authentication and authorization in AI apps
* **Key Points:**
  - Let's first discuss what authentication and authorization mean for AI applications (AI bots, AI companions and agents).

#### What is authentication?
* **Key Points:**
  - Authentication refers to verifying a user's identity to ensure that the person or system attempting to access the application is who they claim to be.
  - Some of the standard authentication techniques for user identification in AI applications include: Password authentication, where a user's identity is verified via a password or user name. It is the most basic form of authentication but is risky if not robustly implemented. Mult-factory authentication, which combines two or more authentication factors (e.g., password and biometric, text message and email, etc.). OAuth/OpenID connect. This approach uses third-party services for user authentication, such as Google OAuth and OAuth 2.0.

#### What is authorization?
* **Key Points:**
  - Authorization is different from authentication in that authorization determines the actions that an authenticated user can perform.
  - For RAG applications, authorization involves controlling access to features such as data ingestion, retrieval, and vector store manipulation.
  - Standard authorization approaches for AI and RAG applications include 3 key concepts: Defining roles and permissions: Clearly specify user roles, e.g. admin, data-ingestor, data-viewer, data-retriever, and the associated resources they can access. Implement fine-grained access control: Implement policies restricting access at a granular level. For example, a finance-retriever role can access only finance-related vectors from the vector store; a health-retriever role may access health-related documents from a vector database. Use third-party authorization tools: Use third-party authorization tools such as Cerbos that offer out of the box authorization functionalities.

### Authorization designs for RAG
* **Key Points:**
  - Depending on the complexity and requirements of your RAG system, you can adopt one or more of the following authorization designs.

#### Access Control Lists (ACL)
* **Key Points:**
  - ACLs are one of the simplest application authorization designs.
  - In ACLs, you define a list of users who can access one or multiple resources, independent of the users' role.
  - Anyone on the list can access the specified resources.
  - An example of an ACL in RAG can be a list of users who can ingest data into a health data store.

#### Role-based Access Control (RBAC)
* **Key Points:**
  - Role-based access control assigns permissions to roles rather than users.
  - Users with the assigned roles can access resources.
  - Users and roles can have a many-to-many relation, where a role can be assigned to multiple users, and a user can have one or multiple roles.
  - For example, all users with the data-ingestor role can ingest data into a vector database.

#### Attribute-based Access Control (ABAC)
* **Key Points:**
  - ABAC allows access to resources based on user and resource attributes.
  - For example you can use ABAC to allow users with department=finance to access resources where vector-store = finance.
  - This fine-grained access control ensures that only the right people can access the right resources under the right conditions, making ABAC particularly suitable for complex systems like RAG.

#### Relationship-based Access Control (ReBAC)
* **Key Points:**
  - ReBAC allows access to resources based on the relationship between entities (users, resources, roles).
  - For example, a team lead can access all documents created by their team members but not those from other teams. In such a case, ReBAC will check who created the document, and if the user who created the document is part of a team leader's team, access will be granted to the team leader.
  - The ReBac approach benefits collaborative RAG applications with dynamic data ownership and relationships.
  - Tools like Cerbos can help you seamlessly implement the access control approaches in your RAG applications.

### Why RAG authorization is critical
* **Key Points:**
  - As discussed earlier, RAG systems deal with sensitive and private data ingestion, retrieval, and manipulation.
  - Without a robust authorization system, RAG applications become vulnerable to unauthorized access, data breaches, and unintended misuse.
  - Following are some factors that highlight the importance of authorization in RAG application.
  - Risk of unauthorized data access: Unauthorized access to private and sensitive data in RAG applications may leak to sensitive data leakage, which a malicious user may exploit. For example, unauthorized access to a company's private earnings data may help competitors develop strategies that can result in financial loss for the company. Implementing ACL or RBAC can help avoid unauthorized data access.
  - Ensuring regulatory compliance: Many industries, such as finance and health care, are governed by strict regulations, such as GDPR and HIPAA. Without secure authorization, RAG applications risk non-compliance with these regulations.
  - Maintaining data integrity: Data integrity is essential to ensuring correct responses in RAG applications. Unauthorized access to vector databases in RAG applications allows malicious users to inject factually wrong or biased information into the database. This results in incorrect and often biased responses from RAG applications.
  - Ensuring users trust: RAG applications with robust authorization foster user trust. Users are more likely to trust applications that demonstrate that user data is handled securely and robustly. For example, a collaborative RAG system for academic research that restricts data ingestion and retrieval based on user roles, e.g., students and faculty, fosters trust among researchers who know that data is ingested by faculty members rather than students.
  - Auditing and accountability: The authorization mechanism in RAG applications allows for better tracking and logging of user actions. This is critical for identifying potential data breaches and maintaining accountability. With authorization, you will have a record of all the actions performed by various users, which will help you identify malicious users in case of data breaches and unauthorized access.
  - Fortunately, all of the aforementioned issues can be handled by using Cerbos, a robust authorization layer that you can use to implement access control in your RAG authorization.

## Implementing authorization in RAG application using Cerbos
* **Key Points:**
  - Cerbos is an open-source, language-agnostic authorization layer that provides a powerful solution for implementing authorization in modern, distributed applications.
  - It offers improved security, scalability, and ease of management for access control policies.
  - Cerbos implements authorization policies using a declarative language, decoupling the authorization logic from your application.
  - It is highly scalable and efficiently handles high-volume authorization requests.
  - In this section, you will see how to use Cerbos to implement various authorization designs such as RBAC and ABAC on the RAG applications we developed in the first section.
* **Technical Entities (Classes/Functions/APIs):** `Cerbos`, `open-source`, `authorization layer`

### Setting up the environment
* **Key Points:**
  - To use Cerbos authorization, you need to install and run the Cerbos server.
  - You can run Cerbos server via Docker as explained in the official documentation.
  - You will also need to install the Cerbos Python SDK to call the Cerbos server from a Python application.
* **Technical Entities (Classes/Functions/APIs):** `cerbos`, `CerbosClient`, `engine_pb2`, `Value`, `load_dotenv()`, `ChatOpenAI`, `OpenAIEmbeddings`
* **Code Snippet:**
```python
pip install cerbos
```
```python
from cerbos.sdk.grpc.client import CerbosClient
from cerbos.engine.v1 import engine_pb2
from google.protobuf.struct_pb2 import Value
```
```python
load_dotenv()

# Cerbos Client Initialization
cerbos_client = CerbosClient("localhost:3593", tls_verify=False)

# OpenAI Client Initialization
openai_key = os.environ.get('OPENAI_API_KEY')
llm = ChatOpenAI(
    openai_api_key=openai_key,
    model="gpt-4",
    temperature=0.7
)

# Initialize OpenAI embeddings
embeddings = OpenAIEmbeddings(openai_api_key=openai_key)
```

### Data loading and metadata addition
* **Technical Entities (Classes/Functions/APIs):** `PyPDFLoader`, `load_and_split()`, `Chroma`
* **Code Snippet:**
```python
data_url = "https://www.hse.ie/eng/services/list/5/cancer/pubs/reports/national-survey-on-lung-cancer-awareness-report-january-2020.pdf"
loader = PyPDFLoader(data_url)
lung_cancer_docs = loader.load_and_split()

data_url = "https://s21.q4cdn.com/399680738/files/doc_financials/2024/q3/META-Q3-2024-Earnings-Call-Transcript.pdf"
loader = PyPDFLoader(data_url)
meta_docs = loader.load_and_split()

data_url = "https://s2.q4cdn.com/299287126/files/doc_financials/2024/q3/AMZN-Q3-2024-Earnings-Release.pdf"
loader = PyPDFLoader(data_url)
amazon_docs = loader.load_and_split()

def add_metadata(docs, source, month):
    for doc in docs:
        doc.metadata["source"] = source
        doc.metadata["month"] = month
    return docs

lung_cancer_docs = add_metadata(lung_cancer_docs, "lung_cancer_doc", "June")
meta_docs = add_metadata(meta_docs, "meta_doc", "October")
amazon_docs = add_metadata(amazon_docs, "amazon_doc", "November")


# Create empty vector stores
lung_cancer_vectorstore = Chroma(
    collection_name="lung_cancer_collection",
    embedding_function=embeddings
)

earning_calls_vectorstore = Chroma(
    collection_name="earning_calls_collection",
    embedding_function=embeddings
)
```

### RBAC authorization in RAG with Cerbos
* **Key Points:**
  - You should use the RBAC approach to allow access to a RAG resource, such as a vector store, based on user role.
  - For example, you want only the users with the role data_ingestor or admin to ingest data into a vector store.
  - The first step in implementing Cerbos authorization is to create policies that define the resource, the roles of the principals who can access it, and the actions that can be performed on it.
  - You can also specify additional conditions that further define the scope of the principals who can perform an action on a resource.
  - You need to define policies in a .yaml and specify the directory containing the .yaml file while starting the Cerbos server.
* **Technical Entities (Classes/Functions/APIs):** `RBAC`, `Cerbos`, `is_allowed()`, `add_documents()`, `engine_pb2.Principal`, `engine_pb2.Resource`
* **Code Snippet:**
```yaml
apiVersion: "api.cerbos.dev/v1"
resourcePolicy:
  resource: "vector_store"
  version: "default"
  rules:
    - actions: ["ingest"]
      effect: EFFECT_ALLOW
      roles: ["data_ingestor", "admin"]
```
```bash
docker run --name cerbos -d -v $(pwd)/cerbos-quickstart/policies:/policies -p 3592:3592 -p 3593:3593 ghcr.io/cerbos/cerbos:0.39.0
```
```python
def ingest_data_with_rbac(vector_store, docs, principal, resource):
    
    with CerbosClient("localhost:3592", tls_verify=False) as client:
        
        if client.is_allowed("ingest", principal, resource):
            print(f"Access granted for {principal.id} to ingest data in resource {resource.id}.")
            vector_store.add_documents(docs)
            return True
            
        else:
            print(f"Access denied for {principal.id}.")
            return False
```
```python
# Define Principals with different roles 
principal_user1 = engine_pb2.Principal(
    id="user1",
    roles=["data_retriever"],  
    policy_version= "default",
)


principal_user2 = engine_pb2.Principal(
    id="user2",
    roles=["data_ingestor"], 
    policy_version= "default",
)


principal_user3 = engine_pb2.Principal(
    id= "admin",
    roles=["admin"], 
    policy_version= "default",
)
```
```python
resource_rbac = engine_pb2.Resource(
    id="lung_cancer_vectorstore",
    kind="vector_store",
)
```
```python
for principal in [principal_user1, principal_user2, principal_user3]:
    print("=====================")
    result = ingest_data_with_rbac(lung_cancer_vectorstore,
                                   lung_cancer_docs,
                                   principal, 
                                   resource_rbac)
    
    if result:
        print("Operation successfull - data ingested")
    else:
        print("You do not have permission to ingest the data")
```

### ABAC authorization on RAG with Cerbos
* **Key Points:**
  - ABAC approach is useful when you want to allow users with certain attributes to access a resource in RAG.
  - For instance, if you want users from a specific department to ingest or retrieve data from a particular vector store, you can use the ABAC approach.
* **Technical Entities (Classes/Functions/APIs):** `ABAC`, `Cerbos`, `is_allowed()`, `plan_resources()`, `engine_pb2.PlanResourcesInput.Resource`

#### Data ingestion with ABAC
* **Key Points:**
  - For ingestion with ABAC, let's add a rule that allows principals with department_data_ingestor roles to perform department_ingest on vector_store type resources if the department attribute of a principal matches the type attribute of a resource.
* **Technical Entities (Classes/Functions/APIs):** `ingest_data_with_abac()`, `is_allowed()`, `add_documents()`
* **Code Snippet:**
```yaml
apiVersion: "api.cerbos.dev/v1"
resourcePolicy:
  resource: "vector_store"
  version: "default"
  rules:

    - actions: ["ingest"]
      effect: EFFECT_ALLOW
      roles: ["data_ingestor", "admin"]


    - actions: ["department_ingest"]
      effect: EFFECT_ALLOW
      roles: ["department_data_ingestor"]
      condition:
        match:
          expr: request.principal.attr.department == request.resource.attr.type
```
```python
def ingest_data_with_abac(vector_store, docs, principal, resource):

    with CerbosClient("localhost:3592", tls_verify=False) as client:
        
        if client.is_allowed("department_ingest", principal, resource):
            print(f"Access granted for {principal.id} to ingest data in resource {resource.id}.")
            vector_store.add_documents(docs)
            return True
            
        else:
            print(f"Access denied for {principal.id}.")
            return False
```
```python
principal_user4 = engine_pb2.Principal(
    id="user4",
    roles=["department_data_ingestor"],
    policy_version= "default",
    attr={"department": Value(string_value="finance")}
)

principal_user5 = engine_pb2.Principal(
    id="user5",
    roles=["department_data_ingestor"],
    policy_version= "default",
    attr={"department": Value(string_value="health")}
)

resource_abac_finance = engine_pb2.Resource(
    id="finance_vectorstore",
    kind="vector_store",
    attr={"type": Value(string_value="finance")}
)

resource_abac_health = engine_pb2.Resource(
    id= "health",
    kind="vector_store",
    attr={"type": Value(string_value="health")}
)
```

#### Data retrieval with ABAC
* **Key Points:**
  - Data retrieval with ABAC involves retrieving data from a vector store using principal and resource attributes.
  - For example, we will add a new rule to our policy that allows principals with the role doc_retriever to perform a retrieve action if the doc_type attributes of the principal and resource match.
  - Notice that here, in addition to the is_allowed() call, we retrieve the principal's plan using the plan_resource() API call.
  - This method dynamically returns the principal's information for accessing a particular resource.
  - We retrieve the string value of the only operand, doc_type, for the principal and use this information to filter the documents in the vector store.
* **Technical Entities (Classes/Functions/APIs):** `generate_response()`, `create_stuff_documents_chain`, `as_retriever()`, `create_retrieval_chain`, `retrieve_data_with_abac()`, `is_allowed()`, `plan_resources()`, `plan_resource`, `filter`, `condition`, `expression`, `operands`
* **Code Snippet:**
```yaml
apmatchn: "api.cerbos.dev/v1"
resourcePolicy:
  resource: "vector_store"
  version: "default"
  rules:

    - actions: ["ingest"]
      effect: EFFECT_ALLOW
      roles: ["data_ingestor", "admin"]


    - actions: ["department_ingest"]
      effect: EFFECT_ALLOW
      roles: ["department_data_ingestor"]
      condition:
        match:
          expr: request.principal.attr.department == request.resource.attr.type
          
          
    - actions: ["retrieve"]
      effect: EFFECT_ALLOW
      roles: ["doc_retriever"]
      condition:
        match:
          expr: request.principal.attr.doc_type == request.resource.attr.doc_type
```
```python
def generate_response(vector_store, query, doc_type) :

    prompt = ChatPromptTemplate.from_template("""Answer the following question based only on the provided context:
    
    Question: {input}
    
    Context: {context}
    """
    )
    
    document_chain = create_stuff_documents_chain(llm, prompt)
    
    
    retriever = vector_store.as_retriever(search_kwargs={"filter": {"source": doc_type}})
    retrieval_chain = create_retrieval_chain(retriever, document_chain)
    
    response = retrieval_chain.invoke({"input": query})
    return response['answer']
```
```python
def retrieve_data_with_rebac(vector_store, query, principal, resource, resource_plan):
    
    with CerbosClient("localhost:3592", tls_verify=False) as client:
        
        if client.is_allowed("retrieve", principal, resource):
            print(f"Access granted for {principal.id} to access the resource {resource.id}.")

            plan = client.plan_resources(action="retrieve", 
                     principal=principal, 
                     resource=plan_resource)

            doc_type = plan.filter.condition.expression.operands[0].value.string_value
            
            response = generate_response(vector_store, 
                                         query, 
                                         doc_type)
            return response
            
        else:
            return (f"Access denied for {principal.id} to access the resource {resource.id}.")        
```
```python
principal_user6 = engine_pb2.Principal(
    id="user6",
    roles=["doc_retriever"],
    policy_version= "default",
    attr={"doc_type": Value(string_value="meta_doc")}
)

principal_user7 = engine_pb2.Principal(
    id="user7",
    roles=["doc_retriever"],
    policy_version= "default",
    attr={"doc_type": Value(string_value="amazon_doc")}
)

resource_abac_meta = engine_pb2.Resource(
    id="meta_docs_vectorstore",
    kind="vector_store",
    attr={"doc_type": Value(string_value="meta_doc")}
)

resource_abac_amazon = engine_pb2.Resource(
    id="amazon_docs_vectorstore",
    kind="vector_store",
    attr={"doc_type": Value(string_value="amazon_doc")}
)
```
```python
plan_resource = engine_pb2.PlanResourcesInput.Resource(
    kind="vector_store",
)

with CerbosClient("localhost:3592", tls_verify=False) as client:
    plan = plan = client.plan_resources(action="retrieve", 
                             principal=principal_user7, 
                             resource=plan_resource)

    print(plan)
```
* **Key Points:**
  - We will define a policy that allows the retrieve action to be performed by the principal with the role team_leader, where the principal's team name is equal to the resource's team name.
  - We will add another condition using the && operator that specifies that the principal's doc_type must match the resource's doc_type.
  - Note: The query plan is designed to adapt dynamically to policy changes. However, using a hardcoded value here can cause issues if the policy changes in the future.
* **Technical Entities (Classes/Functions/APIs):** `team_leader`
* **Code Snippet:**
```yaml
apiVersion: "api.cerbos.dev/v1"
resourcePolicy:
  resource: "vector_store"
  version: "default"
  rules:

    - actions: ["ingest"]
      effect: EFFECT_ALLOW
      roles: ["data_ingestor", "admin"]


    - actions: ["department_ingest"]
      effect: EFFECT_ALLOW
      roles: ["department_data_ingestor"]
      condition:
        match:
          expr: request.principal.attr.department == request.resource.attr.type
          
          
    - actions: ["retrieve"]
      effect: EFFECT_ALLOW
      roles: ["doc_retriever"]
      condition:
        match:
          expr: request.principal.attr.doc_type == request.resource.attr.doc_type


    - actions: ["retrieve"]
      effect: EFFECT_ALLOW
      roles: ["team_leader"]
      condition:
        match:
          expr: request.principal.attr.team_name == request.resource.attr.team_name && request.principal.attr.doc_type == request.resource.attr.doc_type
```
```python
def retrieve_data_with_abac(vector_store, query, principal, resource, resource_plan):
    
    with CerbosClient("localhost:3592", tls_verify=False) as client:
        
        if client.is_allowed("retrieve", principal, resource):
            print(f"Access granted for {principal.id} to access the resource {resource.id}.")

            plan = client.plan_resources(action="retrieve", 
                     principal=principal, 
                     resource=plan_resource)

            doc_type = plan.filter.condition.expression.operands[0].value.string_value
            
            response = generate_response(vector_store, 
                                         query, 
                                         doc_type)
            return response
            
        else:
            return (f"Access denied for {principal.id} to access the resource {resource.id}.")
```
```python
principal_user8 = engine_pb2.Principal(
    id="user8",
    roles=["team_leader"],
    policy_version= "default",
    attr={"team_name": Value(string_value="finance"),
         "doc_type": Value(string_value="meta_doc")}
)

principal_user9 = engine_pb2.Principal(
    id="user9",
    roles=["team_leader"],
    policy_version= "default",
    attr={"team_name": Value(string_value="health"),
         "doc_type": Value(string_value="lung_cancer_doc")}
)

resource_rebac_finance_meta = engine_pb2.Resource(
    id="finance_vectorstore_meta",
    kind="vector_store",
    attr={"team_name": Value(string_value="finance"),
         "doc_type": Value(string_value="meta_doc")}
)

resource_rebac_finance_amazon = engine_pb2.Resource(
    id="finance_vectorstore_amazon",
    kind="vector_store",
    attr={"team_name": Value(string_value="finance"),
         "doc_type": Value(string_value="amazon_doc")}
)

resource_rebac_health = engine_pb2.Resource(
    id="health_vectorstore",
    kind="vector_store",
    attr={"team_name": Value(string_value="health"),
         "doc_type": Value(string_value="lung_cancer_doc")}
)
```

## Final thoughts
* **Key Points:**
  - Security and efficient access are paramount for RAG applications, as they may store sensitive and private data.
  - Implementing a robust access control design improves data security, preventing malicious users or attackers from corrupting the data or accessing private information.
  - Various tools exist that allow you to implement access control mechanisms on RAG applications.
  - Cerbos is one such tool that implements a highly flexible, scalable, and dynamic policy-based approach to access control.
  - You can use Cerbos to implement various access control designs, such as RBAC and ABAC on your RAG applications, as you saw in this artice.
  - I have used Cerbos in my RAG applications, and I can confidently say it is one of the best and easiest-to-implement access control tools for RAG.