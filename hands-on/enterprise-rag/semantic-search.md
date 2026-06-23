---
aliases:
  - Semantic Search
Source 1: https://www.instaclustr.com/education/vector-database/vector-search-vs-semantic-search-4-key-differences-and-how-to-choose/
Source 2: https://www.elastic.co/search-labs/blog/introduction-to-vector-search
Source 3: https://cloud.google.com/discover/what-is-semantic-search
---
[Code Examples](./code/semantic-search.md) 


## What is semantic search?
* **Key Points:**
  - Semantic search is a data searching technique that focuses on understanding the contextual meaning and intent behind a user's search query, rather than only matching keywords.
  - Instead of merely looking for literal matches between search queries and indexed content, it aims to deliver more relevant search results by considering various factors, including the relationships between words, the searcher's location, any previous searches, and the context of the search.
  - Traditional search engines typically focus on matching keywords within a search query to corresponding keywords in indexed web pages.
  - In contrast, semantic search aims to comprehend the deeper meaning and intent behind a user's search, much like a human would.
  - By understanding the meaning and context of words, phrases, and entities within a search query, semantic search strives to deliver highly relevant search results that satisfy the user's information needs.
  - Google Cloud products that can be used to help build a semantic search solution include Agent Search on Gemini Enterprise Agent Platform, BigQuery, and AppSheet.
* **Technical Entities (Classes/Functions/APIs):** `Agent Search on Gemini Enterprise Agent Platform`, `BigQuery`, `AppSheet`

## AI's relationship to semantic search
* **Key Points:**
  - Natural language processing (NLP), a subset of artificial intelligence, plays a crucial role in semantic search by enabling search engines to understand and process human language.
  - Machine learning algorithms, another core aspect of AI, helps power the identifying of patterns and relationships in data that ultimately inform semantic search.
* **Technical Entities (Classes/Functions/APIs):** `Natural language processing (NLP)`, `Machine learning`

## How does semantic search work?
* **Key Points:**
  - Semantic search engines employ various techniques from natural language processing (NLP), knowledge representation, and machine learning to understand the semantics of search queries and web content.
  - Here's a breakdown of the process:
    - Query analysis: The search engine analyzes the user's query to identify keywords, phrases, and entities. It also attempts to interpret the user's search intent by analyzing the relationships between these elements.
    - Knowledge graph integration: Semantic search engines often leverage knowledge graphs, vast databases containing information about entities and their relationships. This information helps the engine understand the context of the search query.
    - Content analysis: Similar to how a search engine analyzes queries, it also examines the content of web pages to determine their relevance to a particular search. This analysis goes beyond keyword matching and considers factors such as the overall topic, sentiment, and entities mentioned within the content.
    - Result return and retrieval: Based on the analysis of the query and the content, the search engine could return web pages according to their relevance and semantic similarity to the search query. It then retrieves and displays the most relevant results to the user.
* **Technical Entities (Classes/Functions/APIs):** `NLP`, `knowledge graphs`

## Why is semantic search important?
* **Key Points:**
  - Semantic search is important for several reasons:
    - Improved relevance: By understanding the meaning behind a search query, especially complex or ambiguous ones, search engines can deliver more relevant results. This means users are more likely to find exactly what they're looking for on the first try.
    - Enhanced user experience: When search results are highly relevant, users have a more satisfying experience. They can quickly find the information they need without wading through pages of irrelevant links.
    - Increased engagement: Relevance is key to engagement. When users find what they are looking for, they are more likely to spend time interacting with the content since they more quickly find what they're looking for.

## Search type comparisons
* **Key Points:**
  - Let's delve into how semantic search differs from other search methodologies.

### Keyword search vs. semantic search
* **Key Points:**
  - While semantic search aims to understand the meaning and intent behind a search, keyword search focuses more so on finding exact matches between the keywords in a query and the keywords in a document.
  - Semantic search does a better job at capturing the user's true information needs, especially with complex queries involving synonyms, ambiguous terms, or implied relationships between concepts.

### Lexical search vs. semantic search
* **Key Points:**
  - Lexical search, similar to keyword search, relies on matching words and phrases based on their literal form without considering their underlying meaning, whereas semantic search, again, aims to understand the meaning and relationships between words and phrases.

### Contextual search vs. semantic search
* **Key Points:**
  - Contextual search expands upon traditional search by taking into account the user's context, such as their location, and past interactions.
  - Semantic search, while it can leverage contextual cues, primarily focuses on understanding the meaning of words and phrases within the search query itself.
  - Think of contextual search as using external clues about the user, while semantic search focuses on deciphering the intrinsic meaning of the query.

### Vector search vs. semantic search
* **Key Points:**
  - Vector search relies on representing text as mathematical vectors in a high-dimensional space.
  - It then calculates the distance between the query vector and document vectors to find the most similar content.
  - While semantic search can use vector representations, it is a broader concept that encompasses various techniques to understand the meaning and relationships between words.

## Examples of semantic search
* **Key Points:**
  - Let's illustrate semantic search with a few examples:

### Understanding related terms
* **Key Points:**
  - A search for " running shoes," for example, on a large e-commerce website, can illustrate how a semantic search engine operates.
  - The engine understands that "running shoes" are related to terms like "sneakers," "athletic footwear," and "jogging shoes."
  - It might also consider brands like Nike, Adidas, or Brooks, known for producing running shoes.

### Considering context
* **Key Points:**
  - The search "trail maps" on a national park's website can demonstrate how location context impacts results.
  - A semantic search engine, using the user's IP address or a previously provided location, might prioritize results for trail maps near their location.
  - If the user is near the park's northern entrance, for example, the engine may prioritize maps for trails accessible from that point.

### Interpreting natural language
* **Key Points:**
  - Semantic search excels at understanding natural language queries.
  - For instance, a search like "what's the weather like in Paris next week?" on a search engine would be interpreted correctly, retrieving a weather forecast for Paris for the following week.
  - The engine breaks down the query and understands the intent despite it being phrased conversationally.

## Applications of semantic search
* **Key Points:**
  - Semantic search can have a wide range of applications across various industries:

### E-commerce search: Improving product search
* **Key Points:**
  - Semantic search may enhance e-commerce platforms by enabling more accurate and relevant product discovery.
  - For example, a user searching for "warm winter gloves" could see results that include gloves made from wool, fleece, or other warm materials, even if the product descriptions don't explicitly mention "warm."

### Enterprise search: Help employees find company information
* **Key Points:**
  - Within an enterprise setting, semantic search can help employees quickly and efficiently find relevant information within company databases, intranets, and knowledge repositories.
  - This may improve productivity and decision-making by providing employees with the information they need when they need it.

---


# A quick introduction to vector search

* **Key Points:**
  - This article is the first in a series of three that will dive into the intricacies of vector search, also known as semantic search, and how it is implemented in Elasticsearch.
  - This first part focuses on providing a general introduction to the basics of embedding vectors and how vector search works under the hood.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch`

## Vectors are not new
* **Key Points:**
  - Research on the subject began in the mid-1960s, and the first research papers were published in 1978 by Gerard Salton, an information retrieval pundit, and his colleagues at Cornell University.
  - Salton's work on dense and sparse vector models constitutes the root of modern vector search technology.
  - These include Elasticsearch powered by the Apache Lucene project, which started working on vector search in 2019.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch`, `Apache Lucene`

## Vector search vs. lexical search
* **Key Points:**
  - Vector search, also commonly known as semantic search, and lexical search work very differently.
  - Lexical search is the kind of search that we've all been using for years in Elasticsearch.
  - To summarize it very briefly, it doesn't try to understand the real meaning of what is indexed and queried, instead, it makes a big effort to lexically match the literals of the words or variants of them (think stemming, synonyms, etc.) that the user types in a query with all the literals that have been previously indexed into the database using similarity algorithms, such as TF-IDF
  - The resulting terms are indexed in an inverted index, which simply maps the analyzed terms to the document IDs containing them.
  - When dealing with unstructured data, semantics comes into play. What does semantics mean? Very simply, the meaning!
  - The main reason why lexical search is inadequate in such situations is that unstructured data can neither be indexed nor queried the same way as structured data.
  - That's what vector search enables and helps to achieve.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch`, `TF-IDF`, `inverted index`

## Embedding vectors
* **Key Points:**
  - Deep Learning is a specific area of machine learning that relies on models based on artificial neural networks made of multiple layers of processing that can progressively extract the true meaning of the data.
  - The true feat of neural networks is that they are capable of turning a single piece of unstructured data into a sequence of floating point values, which are known as embedding vectors or simply embeddings.
  - However, the embedding vectors on which neural network models work can have several hundreds or even thousands of dimensions and simply represent a point in a multi-dimensional space.
  - Each vector dimension represents a feature, or a characteristic, of the unstructured data.
  - Hence, the value of each element in the embedding vector denotes the similarity of that input to a specific dimension.
  - In contrast to lexical search, where a term can either be matched or not, with vector search we can get a much better sense of how similar a piece of unstructured data is to each of the dimensions supported by the model.
  - As such, embedding vectors serve as a fantastic semantic representation of unstructured data.
* **Technical Entities (Classes/Functions/APIs):** `Deep Learning`, `neural networks`, `embedding vectors`

### The secret sauce
* **Key Points:**
  - Embedding vectors that are close to one another represent semantically similar pieces of data.
  - So, when we query a vector database, the search input (image, text, etc.) is first turned into an embedding vector using the same model that has been used for indexing all the unstructured data, and the ultimate goal is to find the nearest neighboring vectors to that query vector.
  - Hence, all we need to do is figure out how to measure the "distance" or "similarity" between the query vector and all the existing vectors indexed in the database, that's pretty much it.

### Distance and similarity
* **Key Points:**
  - Luckily for us, measuring the distance between two vectors is an easy problem to solve thanks to vector arithmetics.
  - So, let's look at the most popular distance and similarity functions that are supported by modern vector search databases, such as Elasticsearch.
* **Technical Entities (Classes/Functions/APIs):** `Elasticsearch`

### L1 distance
* **Key Points:**
  - The L1 distance, also called the Manhattan distance, of two vectors x and y is measured by summing up the pairwise absolute difference of all their elements.
  - Obviously, the smaller the distance d, the closer the two vectors are.

### L2 distance
* **Key Points:**
  - The L2 distance, also called the Euclidean distance, of two vectors x and y is measured by first summing up the square of the pairwise difference of all their elements and then taking the square root of the result.
  - It's basically the shortest path between two points (also called hypotenuse).
  - Similarly to L1, the smaller the distance d, the closer the two vectors are.

### Linf distance
* **Key Points:**
  - The Linf (for L infinity) distance, also called the Chebyshev or chessboard distance, of two vectors x and y is simply defined as the longest distance between any two of their elements or the longest distance measured along one of the axis/dimensions.

### Cosine similarity
* **Key Points:**
  - In contrast to L1, L2, and Linf, cosine similarity does not measure the distance between two vectors x and y, but rather their relative angle, i.e., whether they are both pointing in roughly the same direction.
  - The higher the similarity s, the "closer" the two vectors are.
  - Furthermore, as cosine values are always in the [-1, 1] interval, -1 means opposite similarity (i.e., a 180° angle between both vectors), 0 means unrelated similarity (i.e., a 90° angle), and 1 means identical (i.e., a 0° angle).

### Dot product similarity
* **Key Points:**
  - One drawback of cosine similarity is that it only takes into account the angle between two vectors but not their magnitude (i.e., length), which means that if two vectors point roughly in the same direction but one is much longer than the other, both will still be considered similar.
  - Dot product similarity, also called scalar or inner product, improves that by taking into account both the angle and the magnitude of the vectors, which provides for a much more accurate similarity metric.
  - Two equivalent formulas are used to compute dot product similarity.
  - One thing worth noting is that if all the vectors are normalized first (i.e., their length is 1), then the dot product similarity becomes exactly the same as the cosine similarity (because |x| |y| = 1), i.e., the cosine of the angle between both vectors.
  - As we'll see later, normalizing vectors is a good practice to adopt in order to make the magnitude of the vector irrelevant so that the similarity simply focuses on the angle.
  - It also speeds up the distance computation at indexing and query time, which can be a big issue when operating on billions of vectors.

### Quick recap
* **Key Points:**
  - …semantic search is based on deep learning neural network models that excel at transforming unstructured data into multi-dimensional embedding vectors.
  - …each dimension of the model represents a feature or characteristic of the unstructured data.
  - …an embedding vector is a sequence of similarity values (one for each dimension) that represent how similar to each dimension a given piece of unstructured data is.
  - …the "closer" two vectors are (i.e., the nearest neighbors), the more they represent semantically similar concepts.
  - …distance functions (L1, L2, Linf) allow us to measure how close two vectors are.
  - …similarity functions (cosine and dot product) allow us to measure how much two vectors are heading in the same direction.
  - The brute-force approach of measuring the distance or similarity between the query vector and all vectors in the database can work for small data sets but quickly falls short as the number of vectors increases.
  - Put differently, how can we index millions, billions, or even trillions of vectors and find the nearest neighbors of the query vector in a reasonable amount of time?
  - That's where we need to get smart and figure out optimal ways of indexing vectors so we can zero in on the nearest neighbors as fast as possible without degrading precision too much.

## Vector search algorithms and techniques
* **Key Points:**
  - Over the years, many different research teams have invested a lot of effort into developing very clever vector search algorithms.
  - Here, we're going to briefly introduce the main ones.
  - Depending on the use case, some are better suited than others.

### Linear search
* **Key Points:**
  - We briefly touched upon linear search, or flat indexing, earlier when we mentioned the brute-force approach of comparing the query vector with all vectors present in the database.
  - While it might work well on small datasets, performance decreases rapidly as the number of vectors and dimensions increase (O(n) complexity).
  - Luckily, there are more efficient approaches called approximate nearest neighbor (ANN) where the distances between embedding vectors are pre-computed and similar vectors are stored and organized in a way that keeps them close together, for instance using clusters, trees, hashes, or graphs.
  - Such approaches are called "approximate" because they usually do not guarantee 100% accuracy.
  - The ultimate goal is to either reduce the search scope as much and as quickly as possible in order to focus only on areas that are most likely to contain similar vectors or to reduce the vectors' dimensionality.
* **Technical Entities (Classes/Functions/APIs):** `approximate nearest neighbor (ANN)`

### K-Dimensional trees
* **Key Points:**
  - A K-Dimensional tree, or KD tree, is a generalization of a binary search tree that stores points in a k-dimensional space and works by continuously bisecting the search space into smaller left and right trees where the vectors are indexed.
  - The biggest advantage of the KD tree algorithm is that it allows us to quickly focus only on some localized tree branches, thus eliminating most of the vectors from consideration.
  - However, the efficiency of this algorithm decreases as the number of dimensions increases because many more branches need to be visited than in lower-dimensional spaces.
* **Technical Entities (Classes/Functions/APIs):** `K-Dimensional tree`, `KD tree`

### Inverted file index
* **Key Points:**
  - The inverted file index (IVF) approach is also a space-partitioning algorithm that assigns vectors close to each other to their shared centroid.
  - This algorithm suffers from the same issue as KD trees when used in high-dimensional spaces.
  - This is called the curse of dimensionality, and it occurs when the volume of the space increases so much that all the data seems sparse and the amount of data that would be required to get more accurate results grows exponentially.
  - When the data is sparse, it becomes harder for these space-partitioning algorithms to organize the data into clusters.
  - Luckily for us, there are other algorithms and techniques that alleviate this problem, as detailed below.
* **Technical Entities (Classes/Functions/APIs):** `inverted file index (IVF)`, `curse of dimensionality`

### Quantization
* **Key Points:**
  - Quantization is a compression-based approach that allows us to reduce the total size of the database by decreasing the precision of the embedding vectors.
  - This can be achieved using scalar quantization (SQ) by converting the floating point vector values into integer values.
  - This not only reduces the size of the database by a factor of 8 but also decreases memory consumption and speeds up the distance computation between vectors at search time.
  - Another technique is called product quantization (PQ), which first divides the space into lower-dimensional subspaces, and then vectors that are close together are grouped in each subspace using a clustering algorithm (similar to k-means).
  - Note that quantization is different from dimensionality reduction, where the number of dimensions is reduced, i.e., the vectors simply become shorter.
* **Technical Entities (Classes/Functions/APIs):** `scalar quantization (SQ)`, `product quantization (PQ)`, `k-means`

### Hierarchical Navigable Small Worlds (HNSW)
* **Key Points:**
  - In short, Hierarchical Navigable Small Worlds is a multi-layer graph-based algorithm that is very popular and efficient.
  - It is used by many different vector databases, including Apache Lucene.
  - On the top layer, we can see a graph of very few vectors that have the longest links between them, i.e., a graph of connected vectors with the least similarity.
  - The more we dive into lower layers, the more vectors we find and the denser the graph becomes, with more and more vectors closer to one another.
  - At the lowest layer, we can find all the vectors, with the most similar ones being located closest to one another.
  - At search time, the algorithm starts from the top layer at an arbitrary entry point and finds the vector that is closest to the query vector (shown by the gray point).
  - Then, it moves one layer below and repeats the same process, starting from the same vector that it left in the above layer, and so on, one layer after another, until it reaches the lowest layer and finds the nearest neighbor to the query vector.
* **Technical Entities (Classes/Functions/APIs):** `Hierarchical Navigable Small Worlds (HNSW)`, `Apache Lucene`

### Locality-sensitive hashing (LSH)
* **Key Points:**
  - In the same vein as all the other approaches presented so far, locality-sensitive hashing seeks to drastically reduce the search space in order to increase the retrieval speed.
  - With this technique, embedding vectors are transformed into hash values, all by preserving the similarity information, so that the search space ultimately becomes a simple hash table that can be looked up instead of a graph or tree that needs to be traversed.
  - The main advantage of hash-based methods is that vectors containing an arbitrary (big) number of dimensions can be mapped to fixed-size hashes, which enormously speeds up retrieval time without sacrificing too much precision.
  - While conventional hashing methods try to minimize hashing collisions between similar data pieces, the main objective of locality-sensitive hashing is to do exactly the opposite, i.e., to maximize hashing collisions so that similar data falls within the same bucket with a high probability.
  - By doing so, embedding vectors that are close together in a multi-dimensional space will be hashed to a fixed-size value falling in the same bucket.
  - Since LSH allows those hashed vectors to retain their proximity, this technique comes in very handy for data clustering and nearest neighbor searches.
  - All the heavy lifting happens at indexing time when the hashes need to be computed, while at search time we only need to hash the query vector in order to look up the bucket that contains the closest embedding vectors.
  - Once the candidate bucket is found, a second round usually takes place to identify the nearest neighboring vectors to the query vector.
* **Technical Entities (Classes/Functions/APIs):** `locality-sensitive hashing (LSH)`

## Let's conclude
* **Key Points:**
  - After comparing the differences between lexical search and vector search, we've learned how deep learning neural network models manage to capture the semantics of unstructured data and transcode their meaning into high-dimensional embedding vectors, a sequence of floating point numbers representing the similarity of the data along each of the dimensions of the model.
  - It is also worth noting that vector search and lexical search are not competing but complementary information retrieval techniques (as we'll see in the third part of this series when we'll dive into hybrid search).
  - After that, we introduced a fundamental building block of vector search, namely the distance (and similarity) functions that allow us to measure the proximity of two vectors and assess the similarity of the concepts they represent.
  - Finally, we've reviewed different flavors of the most popular vector search algorithms and techniques, which can be based on trees, graphs, clusters, or hashes, whose goal is to quickly narrow in on a specific area of the multi-dimensional space in order to find the nearest neighbors without having to visit the entire space like a linear brute-force search would do.

## Frequently Asked Questions
### What's the difference between vector search and lexical search?
* **Key Points:**
  - Lexical search doesn't try to understand the real meaning of what is indexed and queried- it matches the literals of the words or their variants.
  - In contrast, vector search indexes data in a way that allows it to be searched based on the meaning it represents.