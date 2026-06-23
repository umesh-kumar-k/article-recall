---
aliases:
  - Graph-based metadata filtering
Source 1: https://www.langchain.com/blog/graph-based-metadata-filtering-for-improving-vector-search-in-rag-applications
---
## Graph-based metadata filtering for improving vector search in RAG applications
* **Key Points:**
  - Text embeddings aren't as effective when sorting information based on specific criteria like dates or categories
  - Metadata filtering or filtered vector search can effectively handle structured filters, allowing users to narrow their search results according to specific attributes
  - The process starts with metadata filtering to narrow documents by structured criteria, followed by vector similarity search on the narrowed subset
  - This two-step process increases the accuracy and relevance of the search results
  - Graph databases can store highly complex and connected structured data alongside unstructured data
  - Extensive structured information enables sophisticated metadata filtering to precisely refine document selection
  - This blog post shows how to implement graph-based metadata filtering using LangChain in combination with OpenAI function-calling agent
* **Technical Entities (Classes/Functions/APIs):** `metadata filtering`, `vector similarity search`, `LangChain`, `Neo4j`, `OpenAI function-calling agent`

## Agenda
* **Key Points:**
  - Dataset: companies graph (public demo server hosted by Neo4j)
  - Schema centers around Organization nodes (suppliers, competitors, location, board members, articles, text chunks)
  - Agent tool has four optional input parameters: topic, organization, country, sentiment
  - Tool dynamically generates Cypher statements based on user input to retrieve relevant text chunks
* **Technical Entities (Classes/Functions/APIs):** `Organization`, `Cypher`, `OpenAI`

## Function Implementation
* **Key Points:**
  - `get_organization_news()` dynamically generates Cypher statements
  - If no pre-filtering parameters, uses existing vector index
  - `CYPHER runtime = parallel parallelRuntimeSupport=all` instructs Neo4j to use parallel runtime where available
  - Organization filter: uses `get_candidates()` with full-text index; if multiple candidates, instructs LLM to ask follow-up; appends existential subquery; uses query parameters to prevent Cypher injection
  - Country filter: uses existential subquery with standardized names
  - Sentiment filter: maps "positive" to `a.sentiment > $sentiment` (0.5) and "negative" to `a.sentiment < $sentiment` (-0.5)
  - Topic parameter: uses `embeddings.embed_query()` for vector similarity search
  - If no topic: returns latest articles, avoiding vector similarity search
* **Technical Entities (Classes/Functions/APIs):** `OpenAIEmbeddings()`, `Neo4jGraph()`, `Neo4jVector.from_existing_index()`, `get_organization_news()`, `get_candidates()`, `vector.similarity.cosine()`, `embed_query()`, `CYPHER runtime = parallel`
* **Code Snippet:**
```python
import os

os.environ["OPENAI_API_KEY"] = "sk-"
os.environ["NEO4J_URI"] = "neo4j+s://demo.neo4jlabs.com"
os.environ["NEO4J_USERNAME"] = "companies"
os.environ["NEO4J_PASSWORD"] = "companies"
os.environ["NEO4J_DATABASE"] = "companies"

embeddings = OpenAIEmbeddings()
graph = Neo4jGraph()
vector_index = Neo4jVector.from_existing_index(
    embeddings, 
    index_name="news"
)
```
```python
def get_organization_news(
    topic: Optional[str] = None,
    organization: Optional[str] = None,
    country: Optional[str] = None,
    sentiment: Optional[str] = None,
) -> str:
    # If there is no prefiltering, we can use vector index
    if topic and not organization and not country and not sentiment:
        return vector_index.similarity_search(topic)
    # Uses parallel runtime where available
    base_query = (
        "CYPHER runtime = parallel parallelRuntimeSupport=all "
        "MATCH (c:Chunk)<-[:HAS_CHUNK]-(a:Article) WHERE "
    )
    where_queries = []
    params = {"k": 5}
```
```python
if organization:
    # Map to database
    candidates = get_candidates(organization)
    if len(candidates) > 1:  # Ask for follow up if too many options
        return (
         "Ask a follow up question which of the available organizations "
         f"did the user mean. Available options: {candidates}"
        )
    where_queries.append(
        "EXISTS {(a)-[:MENTIONS]->(:Organization {name: $organization})}"
    )
    params["organization"] = candidates[0]
```
```python
if country:
    # No need to disambiguate
    where_queries.append(
        "EXISTS {(a)-[:MENTIONS]->(:Organization)-[:IN_CITY]->()-[:IN_COUNTRY]->(:Country {name: $country})}"
    )
    params["country"] = country
```
```python
if sentiment:
    if sentiment == "positive":
        where_queries.append("a.sentiment > $sentiment")
        params["sentiment"] = 0.5
    else:
        where_queries.append("a.sentiment < $sentiment")
        params["sentiment"] = -0.5
```
```python
if topic:  # Do vector comparison
    vector_snippet = (
        " WITH c, a, vector.similarity.cosine(c.embedding,$embedding) AS score "
        "ORDER BY score DESC LIMIT toInteger($k) "
    )
    params["embedding"] = embeddings.embed_query(topic)
else:  # Just return the latest data
    vector_snippet = " WITH c, a ORDER BY a.date DESC LIMIT toInteger($k) "
```
```python
return_snippet = "RETURN '#title ' + a.title + '\n#date ' + toString(a.date) + '\n#text ' + c.text AS output"

complete_query = (
    base_query + " AND ".join(where_queries) + vector_snippet + return_snippet
)

# Retrieve information from the database
data = graph.query(complete_query, params)
print(f"Cypher: {complete_query}\n")
# Safely remove embedding before printing
params.pop('embedding', None)
print(f"Parameters: {params}")
return "###Article: ".join([el["output"] for el in data])
```

## Defining OpenAI agent
* **Key Points:**
  - Topic parameter includes few-shot examples to help LLM understand usage
  - Country parameter specifies "Use full names like United States of America and France"
  - Sentiment parameter uses enum ["positive", "negative"]
  - Tool defined with name "NewsInformation" and description
  - Agent executor uses LCEL with ChatOpenAI, function binding, and OpenAIFunctionsAgentOutputParser
  - chat_history placeholder enables conversational follow-up and clarification
* **Technical Entities (Classes/Functions/APIs):** `NewsInput`, `BaseModel`, `Field`, `NewsTool`, `BaseTool`, `ChatOpenAI`, `ChatPromptTemplate`, `MessagesPlaceholder`, `OpenAIFunctionsAgentOutputParser`, `AgentExecutor`
* **Code Snippet:**
```python
fewshot_examples = """{Input:What are the health benefits for Google employees in the news? Query: Health benefits}
{Input: What is the latest positive news about Google? Query: None}
{Input: Are there any news about VertexAI regarding Google? Query: VertexAI}
{Input: Are there any news about new products regarding Google? Query: new products}
"""

class NewsInput(BaseModel):
    topic: Optional[str] = Field(
        description="Any specific information or topic besides organization, country, and sentiment that the user is interested in. Here are some examples: "
        + fewshot_examples
    )
    organization: Optional[str] = Field(
        description="Organization that the user wants to find information about"
    )
    country: Optional[str] = Field(
        description="Country of organizations that the user is interested in. Use full names like United States of America and France."
    )
    sentiment: Optional[str] = Field(
        description="Sentiment of articles", enum=["positive", "negative"]
    )
```
```python
class NewsTool(BaseTool):
    name = "NewsInformation"
    description = (
        "useful for when you need to find relevant information in the news"
    )
    args_schema: Type[BaseModel] = NewsInput

    def _run(
        self,
        topic: Optional[str] = None,
        organization: Optional[str] = None,
        country: Optional[str] = None,
        sentiment: Optional[str] = None,
        run_manager: Optional[CallbackManagerForToolRun] = None,
    ) -> str:
        """Use the tool."""
        return get_organization_news(topic, organization, country, sentiment)
```
```python
llm = ChatOpenAI(temperature=0, model="gpt-4-turbo", streaming=True)
tools = [NewsTool()]

llm_with_tools = llm.bind(functions=[format_tool_to_openai_function(t) for t in tools])

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that finds information about movies "
            " and recommends them. If tools require follow up questions, "
            "make sure to ask the user for clarification. Make sure to include any "
            "available options that need to be clarified in the follow up questions "
            "Do only the things the user specifically requested. ",
        ),
        MessagesPlaceholder(variable_name="chat_history"),
        ("user", "{input}"),
        MessagesPlaceholder(variable_name="agent_scratchpad"),
    ]
)

agent = (
    {
        "input": lambda x: x["input"],
        "chat_history": lambda x: _format_chat_history(x["chat_history"])
        if x.get("chat_history")
        else [],
        "agent_scratchpad": lambda x: format_to_openai_function_messages(
            x["intermediate_steps"]
        ),
    }
    | prompt
    | llm_with_tools
    | OpenAIFunctionsAgentOutputParser()
)

agent_executor = AgentExecutor(agent=agent, tools=tools)
```

## Summary
* **Key Points:**
  - Graph-based metadata filters enhance vector search accuracy
  - Graph data representation allows virtually limitless structured filters when combined with LLM function-calling to generate Cypher statements dynamically
  - Knowledge graphs can support both unstructured text retrieval and structured information retrieval in RAG applications