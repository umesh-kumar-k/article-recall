---
aliases:
  - Query Transformation
highlights: Techniques like HyDE (hypothetical document embeddings) query expansion, step-back prompting to bridge query document vocabulary gaps
tags:
  - rag
  - query-transformation
  - hyde
Source 1: https://www.langchain.com/blog/query-transformations
Source 2: https://ragsimplified.hashnode.dev/hypothetical-document-embeddings-hyde-a-query-transformation-technique-for-advanced-rag
---

# Hypothetical Document Embeddings (HyDE) – A Query Transformation Technique for Advanced RAG


## Hypothetical Document Embedding : What is it ?
* **Key Points:**
  - "It's a technique where instead of directly searching with the query, the system generates a Hypothetical Document using its pre-trained knowledge based on the query, and then uses that fake document's vector embeddings to find most relevant ( semantic search based ) document and use that as context for generating more Accurate Response ."
  - "Note: We need Large LLMs to use this technique, so that they have more knowledge / context - to be able to generate hypothetical Docs based on that."
* **Technical Entities (Classes/Functions/APIs):** `HyDE (Hypothetical Document Embeddings)`
* **Code Snippet:** None

---

## Where do we apply this in RAG ?
* **Key Points:**
  - "RAG contains , three major steps, Indexing Retrieving Generation , now Indexing is storing Data sources in Database by creating Vector embeddings of data chunks , Retrieving process starts, after receiving User Query to get relevant data and we pass it as context with user Query to Generation part, which finally generates Response."
  - "So, this HyDE technique is applicable at second step , i.e RETRIEVAL"
* **Technical Entities (Classes/Functions/APIs):** `RAG`
* **Code Snippet:** None

---

## How does it work ?
* **Key Points:**
  - "On Receiving user query, we ask our LLM to create a hypothetical document, using its Pre-Trained Knowledge / Context ( it already have )"
  - "We process that document by creating its vector embeddings & doing Semantic Search on it , which gives us a more relevant Context."
  - "Now with the Context and user Query, LLM generates more Precise Response for user."
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Semantic Search`
* **Code Snippet:** None

---

## How Response got More Accurate ?
* **Key Points:**
  - "We increased Context ( by providing , more relevant context ) by creating a Relevant fake document from Existing knowledge of LLM, with that augmented context, we got more Precise Response aligned with the user Query."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## How is it different from Normal RAG ?
* **Key Points:**
  - "In Normal RAG, we do retrieval process by creating Vector embeddings on User Query directly, then searching for Semantically related data in Database to find Context, whereas in here, we are creating a hypothetical document, then using it to get Relevant Context - hence getting better Context."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Working Step by Step with Code & Visual :
* **Key Points:**
  - "From user-query, we create a hypothetical document"
  - "We find document's vector embeddings then we do semantic search on those to get relevant context"
  - "Using that context and User Query, our LLM generates more Accurate Response."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Hypothetical Document Embedding Output :
* **Key Points:**
  - (No text provided; image reference present)
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

---

## Conclusion:
* **Key Points:**
  - "Just Explained my learning's on HYPOTHETICAL DOCUMENT EMBEDDINGS Technique ! if you find it useful then don't forget to like this Article & Follow Me for more such informative Articles."
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None



# Query Transformations

## Naive RAG typically splits documents into chunks, embeds them, and retrieves chunks with high semantic similarity to a user question.
* **Key Points:**
  - "But, this present a few problems: (1) document chunks may contain irrelevant content that degrades retrieval, (2) user questions may be poorly worded for retrieval, and (3) structured queries may need to be generated from the user question (e.g., for querying a vectorstore with metadata filtering or a SQL db)."
  - "LangChain has many advanced retrieval methods to help address these challenges. (1) Multi representation indexing: Create a document representation (like a summary) that is well-suited for retrieval (read about this using the Multi Vector Retriever in a blog post from last week). (2) Query transformation: in this post, we'll review a few approaches to transform humans questions in order to improve retrieval. (3) Query construction: convert human question into a particular query syntax or language, which will be covered in a future post."
  - "If you think of a naive RAG pipeline, the general flow is that you take the users question and pass that directly to an embedding model. That embedding is then compared to documents stored in the vectorstore, and the top k most similar ones are returned."
  - "Query transformation deals with transformations of the user's question before passing to the embedding model."
  - "Although this is not a new phenomenon (query expansion has been used in search for years) what is new is the ability to use LLMs to do it."
  - "Below are a few variations of papers and retrieval methods that take advantage of this. They are all using an LLM to generate a new (or multiple new) queries, and the main difference is the prompt they use to do that generation."
* **Technical Entities (Classes/Functions/APIs):** `Multi Vector Retriever`, `LangChain`, `LLMs`
* **Code Snippet:** None

---

## Rewrite-Retrieve-Read
* **Key Points:**
  - "This paper uses an LLM to rewrite a user query, rather than using the raw user query to retrieve directly."
  - "Because the original query can not be always optimal to retrieve for the LLM, especially in the real world... we first prompt an LLM to rewrite the queries, then conduct retrieval-augmented reading."
  - "The prompt used is a relatively simple one (on the Hub here):"
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Rewrite-Retrieve-Read`
* **Code Snippet:** None

---

## Step back prompting
* **Key Points:**
  - "This paper uses an LLM to generate a 'step back' question."
  - "This can be use with or without retrieval."
  - "With retrieval, both the 'step back' question and the original question are used to do retrieval, and then both results are used to ground the language model response."
  - "Here is the prompt used:"
* **Technical Entities (Classes/Functions/APIs):** `LLM`
* **Code Snippet:** None

---

## Follow Up Questions
* **Key Points:**
  - "The most basic and central place query transformation is used is in conversational chains to handle follow up questions."
  - "When dealing with follow up questions, there are essentially three options: Just embed the follow up question. This means that if the follow up question builds on, or references the previous conversation, it will lose that question. For example, if I first ask 'what can I do in Italy' and then ask 'what type of food is there' - if I just embed 'what type of food is there' I will have no context of where 'there' is."
  - "Embed the whole conversation (or last k messages). The problem with this is that if a follow up question is completely unrelated to previous conversation, then it may return completely irrelevant results that would distract during generation."
  - "Use an LLM to do a query transformation! In this last option, you pass the whole conversation to date (including the follow up question) to the LLM and ask it generate a search term. This is what we do in WebLangChain and what most chat based retrieval applications likely do."
  - "The question then becomes: what prompt do I use to transform the whole conversation into a search query? This is where a lot of prompt engineering needs to be done. Below is the prompt we use for WebLangChain (it phrases the 'query generation' bit as constructing a standalone question). See it on the Hub here."
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `WebLangChain`
* **Code Snippet:** None

---

## Multi Query Retrieval
* **Key Points:**
  - "In this strategy, an LLM is used to generate multiple search queries."
  - "These search queries can then be executed in parallel, and the retrieved results passed in altogether."
  - "This is really useful when a single question may rely on multiple sub questions."
  - "For example consider the following question: Who won a championship more recently, the Red Sox or the Patriots? This really requires two sub-questions: 'When was the last time the Red Sox won a championship?' 'When was the last time the Patriots won a championship?'"
* **Technical Entities (Classes/Functions/APIs):** `LLM`, `Multi Query Retrieval`
* **Code Snippet:** None

---

## RAG-Fusion
* **Key Points:**
  - "A recent article builds off the idea of Multi-Query Retrieval."
  - "However, rather than passing in all the documents, they use reciprocal rank fusion to reorder the documents."
* **Technical Entities (Classes/Functions/APIs):** `RAG-Fusion`, `reciprocal rank fusion`
* **Code Snippet:** None

---

## Conclusion
* **Key Points:**
  - "As you can see, there are many different ways to do query transformation."
  - "Again, this is not a new topic - but what is new is using LLMs to do this."
  - "The differences in the methods comes down to the prompts used."
  - "It's very easy to write prompts - almost as easy as it to think of them."
  - "Which begs the question - what query transformations are YOU going to come up with? Let us know!"
* **Technical Entities (Classes/Functions/APIs):** `LLMs`
* **Code Snippet:** None