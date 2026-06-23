---
aliases:
  - Metadata Filtering
Source 1: https://unstructured.io/insights/how-to-use-metadata-in-rag-for-better-contextual-results
Source 2: https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-metadata-filtering-to-improve-retrieval-accuracy/
Source 3: https://aws.amazon.com/blogs/machine-learning/streamline-rag-applications-with-intelligent-metadata-filtering-using-amazon-bedrock/
---
[Code Examples](code/metadata-filtering-graph.md) 

# Metadata filtering

* **Key Points:**
  - Amazon Bedrock Knowledge Bases now supports metadata filtering to improve retrieval accuracy
  - With Amazon Bedrock Knowledge Bases, you can securely connect foundation models (FMs) in Amazon Bedrock to your company data using a fully managed Retrieval Augmented Generation (RAG) model.
  - For RAG-based applications, the accuracy of the generated responses from FMs depend on the context provided to the model.
  - Contexts are retrieved from vector stores based on user queries.
  - In the recently released feature for Amazon Bedrock Knowledge Bases, hybrid search, you can combine semantic search with keyword search.
  - However, in many situations, you may need to retrieve documents created in a defined period or tagged with certain categories.
  - To refine the search results, you can filter based on document metadata to improve retrieval accuracy, which in turn leads to more relevant FM generations aligned with your interests.
  - In this post, we discuss the new custom metadata filtering feature in Amazon Bedrock Knowledge Bases, which you can use to improve search results by pre-filtering your retrievals from vector stores.
* **Technical Entities (Classes/Functions/APIs):** `Amazon Bedrock Knowledge Bases`, `RAG`, `hybrid search`

## Metadata filtering overview
* **Key Points:**
  - Prior to the release of metadata filtering, all semantically relevant chunks up to the pre-set maximum would be returned as context for the FM to use to generate a response.
  - Now, with metadata filters, you can retrieve not only semantically relevant chunks but a well-defined subset of those relevant chunks based on applied metadata filters and associated values.
  - With this feature, you can now supply a custom metadata file (each up to 10 KB) for each document in the knowledge base.
  - You can apply filters to your retrievals, instructing the vector store to pre-filter based on document metadata and then search for relevant documents.
  - This way, you have control over the retrieved documents, especially if your queries are ambiguous.
  - In addition, by reducing the number of chunks that are being searched over, you achieve performance advantages like a reduction in CPU cycles and cost of querying the vector store, in addition to improvement in accuracy.
  - To use the metadata filtering feature, you need to provide metadata files alongside the source data files with the same name as the source data file and .metadata.json suffix.
  - Metadata can be string, number, or Boolean.
* **Technical Entities (Classes/Functions/APIs):** `metadata filtering`, `.metadata.json`
* **Code Snippet:**
```json
{
    "metadataAttributes" : { 
        "tag" : "project EVE",
        "year" :  2016,
        "team": "ninjas"
    }
}
```
* **Key Points:**
  - The metadata filtering feature of Amazon Bedrock Knowledge Bases is available in AWS Regions US East (N. Virginia) and US West (Oregon).
  - The following are common use cases for metadata filtering:
    - Document chatbot for a software company – This allows users to find product information and troubleshooting guides. Filters on the operating system or application version, for example, can help avoid retrieving obsolete or irrelevant documents.
    - Conversational search of an organization's application – This allows users to search through documents, kanbans, meeting recording transcripts, and other assets. Using metadata filters on work groups, business units, or project IDs, you can personalize the chat experience and improve collaboration. An example would be, "What is the status of project Sphinx and risks raised," where users can filter documents for a specific project or source type (such as email or meeting documents).
    - Intelligent search for software developers – This allows developers to look for information of a specific release. Filters on the release version, document type (such as code, API reference, or issue) can help pinpoint relevant documents.

## Solution overview
* **Key Points:**
  - In the following sections, we demonstrate how to prepare a dataset to use as a knowledge base, and then query with metadata filtering.
  - You can query using either the AWS Management Console or SDK.
* **Technical Entities (Classes/Functions/APIs):** `AWS Management Console`, `SDK`

### Prepare a dataset for Amazon Bedrock Knowledge Bases
* **Key Points:**
  - For this post, we use a sample dataset about fictional video games to illustrate how to ingest and retrieve metadata using Amazon Bedrock Knowledge Bases.
  - If you want to add metadata to your documents in an existing knowledge base, create the metadata files with the expected filename and schema, then skip to the step to sync your data with the knowledge base to start the incremental ingestion.
  - In our sample dataset, each game's document is a separate CSV file (for example, s3://$bucket_name/video_game/$game_id.csv) with the following columns: title, description, genres, year, publisher, score
  - Each game's metadata has the suffix .metadata.json (for example, s3://$bucket_name/video_game/$game_id.csv.metadata.json) with the following schema
* **Technical Entities (Classes/Functions/APIs):** `Amazon S3`, `.metadata.json`
* **Code Snippet:**
```json
{
  "metadataAttributes": {
    "ids": number, 
    "genres": string,
    "year": number,
    "publisher": string,
    "score": number
  }
}
```

### Create a knowledge base for Amazon Bedrock
* **Key Points:**
  - For instructions to create a new knowledge base, see Create a knowledge base.
  - For this example, we use the following settings:
    - On the Set up data source page, under Chunking strategy, select No chunking, because you've already preprocessed the documents in the previous step.
    - In the Embeddings model section, choose Titan G1 Embeddings – Text.
    - In the Vector database section, choose Quick create a new vector store.
  - The metadata filtering feature is available for all supported vector stores.
* **Technical Entities (Classes/Functions/APIs):** `Titan G1 Embeddings – Text`

### Synchronize the dataset with the knowledge base
* **Key Points:**
  - After you create the knowledge base, and your data files and metadata files are in an Amazon Simple Storage Service (Amazon S3) bucket, you can start the incremental ingestion.

### Query with metadata filtering on the Amazon Bedrock console
* **Key Points:**
  - To use the metadata filtering options on the Amazon Bedrock console, complete the following steps:
    - On the Amazon Bedrock console, choose Knowledge bases in the navigation pane.
    - Choose the knowledge base you created.
    - Choose Test knowledge base.
    - Choose the Configurations icon, then expand Filters.
    - Enter a condition using the format: key = value (for example, genres = Strategy) and press Enter.
    - To change the key, value, or operator, choose the condition.
    - Continue with the remaining conditions (for example, (genres = Strategy AND year >= 2023) OR (rating >= 9))
    - When finished, enter your query in the message box, then choose Run.

### Query with metadata filtering using the SDK
* **Key Points:**
  - To use the SDK, first create the client for the Amazon Bedrock Agents runtime
  - Then construct the filter (the following are some examples)
  - Pass the filter to retrievalConfiguration of the Retrieval API or RetrieveAndGenerate API
* **Technical Entities (Classes/Functions/APIs):** `boto3`, `bedrock_agent_runtime`, `Retrieval API`, `RetrieveAndGenerate API`, `retrievalConfiguration`, `vectorSearchConfiguration`, `filter`
* **Code Snippet:**
```python
import boto3

bedrock_agent_runtime = boto3.client(
    service_name = "bedrock-agent-runtime"
)
```
```python
# genres = Strategy
single_filter= {
    "equals": {
        "key": "genres",
        "value": "Strategy"
    }
}

# genres = Strategy AND year >= 2023
one_group_filter= {
    "andAll": [
        {
            "equals": {
                "key": "genres",
                "value": "Strategy"
            }
        },
        {
            "GreaterThanOrEquals": {
                "key": "year",
                "value": 2023
            }
        }
    ]
}

# (genres = Strategy AND year >=2023) OR score >= 9
two_group_filter = {
    "orAll": [
        {
            "andAll": [
                {
                    "equals": {
                        "key": "genres",
                        "value": "Strategy"
                    }
                },
                {
                    "GreaterThanOrEquals": {
                        "key": "year",
                        "value": 2023
                    }
                }
            ]
        },
        {
            "GreaterThanOrEquals": {
                "key": "score",
                "value": "9"
            }
        }
    ]
}
```
```python
retrievalConfiguration={
        "vectorSearchConfiguration": {
            "filter": metadata_filter
        }
    }
```
* **Key Points:**
  - In addition to custom metadata, you can also filter using S3 prefixes (which is a built-in metadata, so you don't need to provide any metadata files).
  - For example, if you organize the game documents into prefixes by publisher (for example, s3://$bucket_name/video_game/$publisher/$game_id.csv), you can filter with the specific publisher (for example, neo_tokyo_games) using the following syntax
* **Technical Entities (Classes/Functions/APIs):** `S3 prefixes`, `x-amz-bedrock-kb-source-uri`
* **Code Snippet:**
```python
publisher_filter = {
    "startsWith": {
                    "key": "x-amz-bedrock-kb-source-uri",
                    "value": "s3://$bucket_name/video_game/neo_tokyo_games/"
                }
}
```

## Clean up
* **Key Points:**
  - To clean up your resources, complete the following steps:
    - Delete the knowledge base: On the Amazon Bedrock console, choose Knowledge bases under Orchestration in the navigation pane. Choose the knowledge base you created. Take note of the AWS Identity and Access Management (IAM) service role name in the Knowledge base overview section. In the Vector database section, take note of the collection ARN. Choose Delete, then enter delete to confirm.
    - Delete the vector database: On the Amazon OpenSearch Service console, choose Collections under Serverless in the navigation pane. Enter the collection ARN you saved in the search bar. Select the collection and chose Delete. Enter confirm in the confirmation prompt, then choose Delete.
    - Delete the IAM service role: On the IAM console, choose Roles in the navigation pane. Search for the role name you noted earlier. Select the role and choose Delete. Enter the role name in the confirmation prompt and delete the role.
    - Delete the sample dataset: On the Amazon S3 console, navigate to the S3 bucket you used. Select the prefix and files, then choose Delete. Enter permanently delete in the confirmation prompt to delete.
* **Technical Entities (Classes/Functions/APIs):** `IAM`, `Amazon OpenSearch Service`, `Amazon S3`

## Conclusion
* **Key Points:**
  - In this post, we covered the metadata filtering feature in Amazon Bedrock Knowledge Bases.
  - You learned how to add custom metadata to documents and use them as filters while retrieving and querying the documents using the Amazon Bedrock console and the SDK.
  - This helps improve context accuracy, making query responses even more relevant while achieving a reduction in cost of querying the vector database.
* **Technical Entities (Classes/Functions/APIs):** `Amazon Bedrock Knowledge Bases`, `SDK`