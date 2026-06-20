---
aliases:
  - Embedding Use Cases Beyond Search
Source 1: https://machinelearningmastery.com/text-embedding-generation-with-transformers/
Source 2: https://machinelearningmastery.com/example-applications-of-text-embedding/
---

# Example Applications of Text Embedding

## Overview
* **Key Points:**
  - This post is divided into five parts; they are:
    - Recommendation Systems
    - Cross-Lingual Applications
    - Text Classification
    - Zero-Shot Classification
    - Visualizing Text Embeddings

## Recommendation Systems
* **Key Points:**
  - A simple recommendation system can be created by finding a few of the most similar items to the target item. In the example of natural language processing, you can find some similar articles as "you may also like" while the user is reading an article.
  - There are many ways to implement this. But the easiest way is to check how similar are the two articles. You can just convert all the articles into a context embedding. The two articles with the highest similarity in the context embedding are similar in content. It may not be what you expect for a recommendation, but it is sometimes useful and it is a good starting point.
  - These recommendations will be based on semantic similarity rather than just keyword matching, so you will get articles about neural networks or machine learning even if they don't contain the exact phrase "deep learning." This approach can be extended to more complex recommendation systems by incorporating user preferences, collaborative filtering, or hybrid approaches.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `all-MiniLM-L6-v2`, `cosine_similarity()`, `np.argsort()`
* **Code Snippet:**
```python
import numpy as np
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
 
# Define a corpus of articles (title and content)
articles = [
  {
    "title": "Understanding Deep Learning",
    "content": ("Deep learning is a subset of machine learning where artificial neural networks, "
                "algorithms inspired by the human brain, learn from large amounts of data.")
  },
  {
    "title": "Introduction to Natural Language Processing",
    "content": ("Natural Language Processing (NLP) is a field of AI that gives machines the "
                "ability to read, understand, and derive meaning from human languages.")
  },
  {
    "title": "The Future of Computer Vision",
    "content": ("Computer vision is an interdisciplinary field that deals with how computers can "
                "gain high-level understanding from digital images or videos.")
  },
  {
    "title": "Reinforcement Learning Explained",
    "content": ("Reinforcement learning is an area of machine learning concerned with how "
                "software agents ought to take actions in an environment so as to maximize some "
                "notion of cumulative reward.")
  },
  {
    "title": "Neural Networks and Their Applications",
    "content": ("Neural networks are a set of algorithms, modeled loosely after the human brain, "
                "that are designed to recognize patterns in data.")
  }
]
 
model = SentenceTransformer("all-MiniLM-L6-v2")
 
def create_article_embeddings(articles, model):
    """create embeddings for articles"""
    texts = [f"{article["title"]}. {article["content"]}" for article in articles]
    embeddings = model.encode(texts)
    return embeddings
 
def get_recommendations(article_id, articles, embeddings, top_n=2):
    """get recommendations for a given article ID based on cosine similarity"""
    similarities = cosine_similarity([embeddings[article_id]], embeddings)[0]
    similar_indices = np.argsort(similarities)[::-1][1:top_n+1]
    return [articles[idx] for idx in similar_indices]
 
# Create embeddings for all articles, and get recommendation for first article
embeddings = create_article_embeddings(articles, model)
recommendations = get_recommendations(0, articles, embeddings)
 
# Print the recommendations
print(f'Recommendations for "{articles[0]["title"]}":')
for i, rec in enumerate(recommendations):
    print(f"{i+1}. {rec["title"]}")
```

## Cross-Lingual Applications
* **Key Points:**
  - One of the powerful features of modern transformer models is their ability to generate embeddings for text in multiple languages. This enables cross-lingual applications where you can compare or process text across different languages.
  - This works because the embedding vector represents the semantic meaning of the text, regardless of the language. This cross-lingual capability is particularly useful for applications like multilingual search engines.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `paraphrase-multilingual-MiniLM-L12-v2`, `cosine_similarity()`, `np.argsort()`
* **Code Snippet:**
```python
import numpy as np
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
 
corpus = [
  {
    "language": "English",
    "text": ("Machine learning is a field of study that gives computers the ability to learn "
             "without being explicitly programmed.")
  },
  {
    "language": "Spanish",
    "text": ("El aprendizaje automático es un campo de estudio que da a las computadoras la "
             "capacidad de aprender sin ser programadas explícitamente.")
  },
  {
    "language": "French",
    "text": ("L'apprentissage automatique est un domaine d'étude qui donne aux ordinateurs "
             "la capacité d'apprendre sans être explicitement programmés.")
  },
  {
    "language": "German",
    "text": ("Maschinelles Lernen ist ein Studienbereich, der Computern die Fähigkeit gibt, "
             "zu lernen, ohne explizit programmiert zu werden.")
  },
  {
    "language": "Italian",
    "text": ("Il machine learning è un campo di studio che conferisce ai computer la capacità "
             "di apprendere senza essere esplicitamente programmati.")
  },
  {
    "language": "English",
    "text": ("Natural language processing is a subfield of linguistics, computer science, "
             "and artificial intelligence.")
  },
  {
    "language": "English",
    "text": ("Computer vision is an interdisciplinary field that deals with how computers can "
             "gain high-level understanding from digital images or videos.")
  }
]
 
model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")
 
# Generate embeddings for the corpus
texts = [doc["text"] for doc in corpus]
embeddings = model.encode(texts)
 
# Define a query in English and generate an embedding
query = "What is machine learning?"
query_embedding = model.encode(query)
 
# Sort the embeddings of the corpus by descending similarity
similarities = cosine_similarity([query_embedding], embeddings)[0]
ranked_indices = np.argsort(similarities)[::-1]
 
# Print ranked results
print(f"Query: {query}\n")
for i, idx in enumerate(ranked_indices[:3]):  # Show top 3 results
    print(f"{i+1}. [{corpus[idx]["language"]}] {corpus[idx]["text"]} (Similarity: {similarities[idx]:.4f})")
```

## Text Classification
* **Key Points:**
  - Imagine you have a lot of text data, and it is growing every day. This may be because you are collecting new articles or emails. You want to classify them into different categories. This can be done by using text embeddings.
  - This is a task similar to "topic modeling". Topic modeling is an unsupervised learning task that groups text documents into different topics. It uses algorithms like Latent Dirichlet Allocation (LDA) to find the signature keywords for classification. Here is a supervised approach: You have a predefined set of categories and some examples (maybe you do the classification manually). Then you add new text to the collection with the classification done automatically.
  - Text embeddings can help by extracting the semantic meaning of the text into vectors. Then you can train a machine learning model to classify the vectors into categories. It works better this way because the vector represents the meaning of the text rather than the text itself. Hence, it is better than using bag-of-words or TF-IDF features.
  - There are many ways to implement the machine learning classifier. A simple one is a logistic regression from scikit-learn.
  - Note that you can use other classifiers like random forest or K-Nearest Neighbors. You can try them and see which one works better.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `all-MiniLM-L6-v2`, `LogisticRegression`, `classification_report`, `train_test_split`, `StandardScaler`
* **Code Snippet:**
```python
from sentence_transformers import SentenceTransformer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
 
articles = [
    # Business articles
    {"text": "The stock market reached a new high today, with technology stocks leading the gains.", "category": "Business"},
    {"text": "The government announced a new tax policy that will affect small businesses.", "category": "Business"},
    {"text": "The central bank has decided to keep interest rates unchanged.", "category": "Business"},
    {"text": "Quarterly earnings reports exceeded expectations for most Fortune 500 companies.", "category": "Business"},
    {"text": "Inflation rates have decreased for the third consecutive month.", "category": "Business"},
    {"text": "The merger between two major corporations has been approved by regulators.", "category": "Business"},
    {"text": "Unemployment rates have fallen to a five-year low according to new data.", "category": "Business"},
    {"text": "The cryptocurrency market experienced significant volatility this week.", "category": "Business"},
 
    # Health articles
    {"text": "A new study shows that regular exercise can reduce the risk of heart disease.", "category": "Health"},
    {"text": "A clinical trial for a new cancer treatment has shown promising results.", "category": "Health"},
    {"text": "A balanced diet and regular sleep are essential for maintaining good health.", "category": "Health"},
    {"text": "Medical researchers have identified a new gene linked to Alzheimer's disease.", "category": "Health"},
    {"text": "The WHO has issued new guidelines for managing diabetes in elderly patients.", "category": "Health"},
    {"text": "A new technique for early detection of breast cancer has been developed.", "category": "Health"},
    {"text": "Studies show that mindfulness meditation can help reduce stress and anxiety.", "category": "Health"},
    {"text": "Public health officials warn of a potential flu outbreak this winter season.", "category": "Health"},
 
    # Technology articles
    {"text": "The latest smartphone from Apple features a better camera and longer battery life.", "category": "Technology"},
    {"text": "The new electric car from Tesla has a range of over 400 miles.", "category": "Technology"},
    {"text": "The latest update to the operating system includes new security features.", "category": "Technology"},
    {"text": "A new artificial intelligence system can detect diseases from medical images.", "category": "Technology"},
    {"text": "The tech company unveiled its new virtual reality headset at the annual conference.", "category": "Technology"},
    {"text": "Researchers have developed a quantum computer that can solve complex problems.", "category": "Technology"},
    {"text": "The new social media platform has gained millions of users in just a few months.", "category": "Technology"},
    {"text": "Cybersecurity experts warn of a new type of malware targeting smart home devices.", "category": "Technology"},
 
    # Science articles
    {"text": "Scientists have discovered a new species of frog in the Amazon rainforest.", "category": "Science"},
    {"text": "Astronomers have observed a supernova in a distant galaxy.", "category": "Science"},
    {"text": "Researchers have developed a new method for measuring ocean temperatures.", "category": "Science"},
    {"text": "A fossil discovery suggests that dinosaurs may have been warm-blooded.", "category": "Science"},
    {"text": "Climate scientists report that Arctic ice is melting at an unprecedented rate.", "category": "Science"},
    {"text": "Physicists have confirmed the existence of a new subatomic particle.", "category": "Science"},
    {"text": "A study of coral reefs shows signs of recovery in protected marine areas.", "category": "Science"},
    {"text": "Biologists have sequenced the genome of an endangered species of tiger.", "category": "Science"}
]
 
 
# Prepare data for classification training
model = SentenceTransformer("all-MiniLM-L6-v2")
texts = [article["text"] for article in articles]
X = model.encode(texts)
y = [article["category"] for article in articles]
 
# Normalize features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
 
# Split data into training and testing sets with stratification
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.2, random_state=42, stratify=y
)
 
# Train a logistic regression classifier with regularization
classifier = LogisticRegression(C=1.0, class_weight="balanced", max_iter=1000)
classifier.fit(X_train, y_train)
 
# Evaluate the classifier
y_pred = classifier.predict(X_test)
print(classification_report(y_test, y_pred))
 
# Classify new articles
new_articles = [
    "The company reported a 20% increase in quarterly profits.",
    "A new vaccine has been approved for use against the flu.",
    "The new laptop features a faster processor and more memory.",
    "The Mars rover has sent back new images of the planet\"s surface."
]
new_embeddings = model.encode(new_articles)
new_embeddings_scaled = scaler.transform(new_embeddings)
new_predictions = classifier.predict(new_embeddings_scaled)
for article, prediction in zip(new_articles, new_predictions):
    print(f"Article: {article}\nPredicted Category: {prediction}\n")
```

## Zero-Shot Classification
* **Key Points:**
  - In the previous example, you trained a classifier to classify the text into one of the predefined categories. If the category labels are meaningful text, why can't you use the meaning of the label for classification? In this way, you can simply convert the text into embeddings and then compare it with the category labels' embeddings. The text is then tagged with the most similar category label.
  - This is the idea of zero-shot learning. It is not a supervised learning task. Indeed, you never train a new model, but the classification and information retrieval tasks can still be done.
  - Zero-shot learning is particularly useful for tasks where labeled training data is scarce or unavailable. It can be applied to a wide range of NLP tasks, including classification, entity recognition, and question-answering.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `all-MiniLM-L6-v2`, `util.cos_sim()`, `torch.argmax()`
* **Code Snippet:**
```python
import torch
from sentence_transformers import SentenceTransformer, util
 
texts = [
    "The stock market reached a new high today, with technology stocks leading the gains.",
    "A new study shows that regular exercise can reduce the risk of heart disease.",
    "The latest smartphone from Apple features a better camera and longer battery life.",
    "Scientists have discovered a new species of frog in the Amazon rainforest."
]
categories = ["Business", "Health", "Technology", "Science"]
 
# Load a pre-trained Sentence Transformer model
model = SentenceTransformer("all-MiniLM-L6-v2")
text_embeddings = model.encode(texts, convert_to_tensor=True)
category_embeddings = model.encode(categories, convert_to_tensor=True)
 
# Calculate cosine similarity between texts and categories
similarities = util.cos_sim(text_embeddings, category_embeddings)
 
# Get the most similar category for each text
best_categories = torch.argmax(similarities, dim=1)
for i, text in enumerate(texts):
    category = categories[best_categories[i]]
    similarity = similarities[i][best_categories[i]].item()
    print(f"Text: {text}")
    print(f"Category: {category} (Similarity: {similarity:.4f})\n")
```

## Visualizing Text Embeddings
* **Key Points:**
  - Not a particular application, but visualizing text embeddings can sometimes provide insights into the semantic relationships between texts. Since embeddings typically have hundreds of dimensions, you need dimensionality reduction techniques to visualize them in 2D or 3D.
  - PCA is probably the most popular dimensionality reduction technique. However, for visualization, t-SNE (t-Distributed Stochastic Neighbor Embedding) usually works better.
  - The visualization puts texts with similar meanings together; this means the labels are useful to represent the semantic meaning of the texts. You can look at the plot and check if the points from the same category are clustered close enough to tell if your embeddings are good.
  - Other dimensionality reduction techniques exist, such as PCA (Principal Component Analysis) or UMAP (Uniform Manifold Approximation and Projection). You can try these to see if the visualization still makes sense.
* **Technical Entities (Classes/Functions/APIs):** `SentenceTransformer`, `all-MiniLM-L6-v2`, `TSNE`, `plt.scatter()`, `plt.annotate()`
* **Code Snippet:**
```python
import matplotlib.pyplot as plt
import numpy as np
from sentence_transformers import SentenceTransformer
from sklearn.manifold import TSNE
 
texts_with_categories = [
    {"text": "The stock market reached a new high today.", "category": "Business"},
    {"text": "Investors are optimistic about the economy.", "category": "Business"},
    {"text": "The company reported strong quarterly earnings.", "category": "Business"},
    {"text": "The central bank has decided to keep interest rates unchanged.", "category": "Business"},
    {"text": "A new study shows that regular exercise can reduce the risk of heart disease.", "category": "Health"},
    {"text": "A balanced diet is essential for maintaining good health.", "category": "Health"},
    {"text": "The new vaccine has been approved for use against the flu.", "category": "Health"},
    {"text": "Sleep is important for physical and mental health.", "category": "Health"},
    {"text": "The latest smartphone features a better camera and longer battery life.", "category": "Technology"},
    {"text": "The new laptop has a faster processor and more memory.", "category": "Technology"},
    {"text": "The software update includes new security features.", "category": "Technology"},
    {"text": "5G networks promise faster internet speeds for mobile devices.", "category": "Technology"},
    {"text": "Scientists have discovered a new species in the Amazon rainforest.", "category": "Science"},
    {"text": "Astronomers have observed a supernova in a distant galaxy.", "category": "Science"},
    {"text": "The Mars rover has sent back new images of the planet's surface.", "category": "Science"},
    {"text": "Researchers have developed a new method for measuring ocean temperatures.", "category": "Science"}
]
 
 
# Extract texts and categories
texts = [item["text"] for item in texts_with_categories]
categories = [item["category"] for item in texts_with_categories]
 
# Generate embeddings, then reduce dimension with t-SNE
model = SentenceTransformer("all-MiniLM-L6-v2")
embeddings = model.encode(texts)
 
tsne = TSNE(n_components=2, perplexity=5, random_state=42)
reduced_embeddings = tsne.fit_transform(embeddings)
 
# Define colors for categories
unique_categories = list(set(categories))
colors = plt.cm.rainbow(np.linspace(0, 1, len(unique_categories)))
category_to_color = {category: color for category, color in zip(unique_categories, colors)}
 
# Create a scatter plot
plt.figure(figsize=(10, 8))
for i, (x, y) in enumerate(reduced_embeddings):
    category = categories[i]
    color = category_to_color[category]
    plt.scatter(x, y, color=color, alpha=0.7)
    plt.annotate(texts[i][:20] + "...", (x, y), fontsize=8)
 
# Add legend, mark the axes
for category, color in category_to_color.items():
    plt.scatter([], [], color=color, label=category)
plt.legend()
plt.xlabel("t-SNE Dimension 1")
plt.ylabel("t-SNE Dimension 2")
plt.title("t-SNE Visualization of Text Embeddings")
plt.tight_layout()
plt.show()
```

## Further Readings
* **Key Points:**
  - Below are some further readings that you may find useful:
    - Pretrained Models for Text Embeddings
    - t-SNE in Scikit-Learn
    - PCA in Scikit-Learn
    - UMAP
* **Technical Entities (Classes/Functions/APIs):** `PCA`, `t-SNE`, `UMAP`

## Summary
* **Key Points:**
  - In this tutorial, you learned a few applications of text embeddings. Particularly, you learned how to:
    - Build a recommendation system using similarity in the embedding space
    - Implement cross-lingual applications with multilingual embeddings
    - Train a text classification system using embedding as features
    - Develop a zero-shot text labeling application using similarity metrics in embedding space
    - Visualize and analyze text embeddings
  - Text embeddings are simple yet powerful tools for a wide range of NLP tasks. They enable machines to understand and process text in a way that captures semantic meaning.

---


# Text Embedding Generation with Transformers

## Overview

* **Key Points:**
  - This post is divided into three parts; they are:
    - Understanding Text Embeddings
    - Other Techniques to Generate Embedding
    - How to Get a High-Quality Text Embedding?

## Understanding Text Embeddings
* **Key Points:**
  - Text embeddings are to use numerical vectors to represent text. A trivial way to represent text is to find all words in the dictionary and assign a unique number to each word. Then, you can represent each word as a one-hot vector or a sentence as a bag-of-words vector: the number in each position means how many times the word appears in the sentence.
  - A dictionary has thousands of words, so one hot vector is too large and sparse. A dense vector, in which each element is a floating point number instead of a boolean, can make it more compact. However, what value should each element in the vector be? This isn't easy to decide. But it can be trained. Examples include Word2Vec, GloVe, and FastText. The interesting property of dense word vectors is that they place semantically similar words closer together in the vector space. You can use the vector to measure the semantic similarity between words and perform word math, such as "king – man + woman = queen".
  - One step further, you want to represent a sentence as a vector. It is more difficult than simply adding the word vectors from the words in a sentence because you need to identify the context of a word. For example, "bear" can be a verb or a noun; the word vector cannot tell their difference but is important for the context. Representing the semantic meaning of a sentence into a vector is very useful for many NLP tasks.
  - Transformer models can generate such contextual embeddings by processing the entire sequence of words at once. The representation of a word in the embedding depends on its context within the text. This allows for much richer representations that can capture nuances like polysemy (words with multiple meanings).
  - Training a transformer model to generate embeddings is computationally expensive and difficult because it requires a high-quality dataset and a complex training process. Fortunately, we can use pre-trained models to generate embeddings if we merely want to create a vector to represent a text's semantic meaning.
* **Technical Entities (Classes/Functions/APIs):** `Word2Vec`, `GloVe`, `FastText`, `AutoTokenizer`, `AutoModel`, `bert-base-uncased`
* **Code Snippet:**
```python
from transformers import AutoTokenizer, AutoModel
import torch
import numpy as np
 
# Load pre-trained model and tokenizer
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")
 
# Define some example sentences
sentences = [
    "The cat sat on the mat.",
    "The dog slept on the floor.",
    "I love natural language processing."
]
 
def get_embeddings(sentences, model, tokenizer):
    "Function to get embeddings for a batch of sentences"
 
    # Tokenize input and get model output
    encoded_input = tokenizer(sentences, padding=True, truncation=True, return_tensors="pt")
    with torch.no_grad():
        model_output = model(**encoded_input)
 
    # Use the CLS token embedding as the sentence embedding
    sentence_embeddings = model_output.last_hidden_state[:, 0, :]
 
    # Convert torch tensor to numpy array for easier handling
    return sentence_embeddings.numpy()
 
# Get embeddings for our example sentences
embeddings = get_embeddings(sentences, model, tokenizer)
print(f"Embedding shape: {embeddings.shape}")
print(f"First 5 dimensions of the sentences' embeddings:\n{np.round(embeddings[:, :5], 3)}")
```

## Other Techniques to Generate Embedding
* **Key Points:**
  - While using the [CLS] token embedding is a common approach, it is not the only one.
  - If you can use the [CLS] prefix token for embedding, you can also take the average of all output tokens. This is the technique of mean pooling. It may provide a better representation of the sentence.
  - The mean pooling method is believed to provide better sentence embeddings than just the [CLS] token, especially for tasks like semantic similarity and information retrieval.
* **Technical Entities (Classes/Functions/APIs):** `AutoTokenizer`, `AutoModel`, `bert-base-uncased`
* **Code Snippet:**
```python
from transformers import AutoTokenizer, AutoModel
import torch
import numpy as np
 
# Load pre-trained model and tokenizer
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")
 
# Define some example sentences
sentences = [
    "The cat sat on the mat.",
    "The dog slept on the floor.",
    "I love natural language processing."
]
 
def get_embeddings(sentences, model, tokenizer):
    "Function to get embeddings for a batch of sentences with mean pooling"
 
    # Tokenize input and get model output
    encoded_input = tokenizer(sentences, padding=True, truncation=True, return_tensors="pt")
    with torch.no_grad():
        model_output = model(**encoded_input)
 
    # Extract the attention mask and output sequence
    attention_mask = encoded_input["attention_mask"]
    output_seq = model_output.last_hidden_state
 
    # Mean pooling: take the average of all token embeddings
    mask = attention_mask.unsqueeze(-1).expand(output_seq.size()).float()
    sum_embeddings = (output_seq * mask).sum(1)
    sum_mask = torch.clamp(mask.sum(1), min=1e-9)
    mean_pooled = sum_embeddings / sum_mask
 
    # Convert torch tensor to numpy array for easier handling
    return mean_pooled.numpy()
 
# Get embeddings with mean pooling
embeddings = get_embeddings(sentences, model, tokenizer)
print(f"Embedding shape: {embeddings.shape}")
print(f"First 5 dimensions of the sentences' embeddings:\n{np.round(embeddings[:, :5], 3)}")
```

## How to Get a High-Quality Text Embedding?
* **Key Points:**
  - All sentence embeddings are generated from a deep learning model, especially transformer models. The quality of the embedding highly depends on the quality of the model and the training data.
  - Larger models, such as BERT and RoBERTa, are generally better than smaller ones, such as DistilBERT, trading off speed and memory usage for quality. A model trained or fine-tuned for a specific task will also likely provide better embedding than a general-purpose model if used for a specialized domain. For example, a model trained with a corpus from the medical domain will likely provide better embedding of medical text than a general-purpose model.
  - Also, note that the tokenizer plays an important role in the embedding quality. Transformer models operate on a sequence of tokens. A tokenizer that splits the sentence into sub-words that retain the semantic meaning helps the model generate better embedding. At one extreme, the tokenizer can emit every single character as a token, but that will lose a lot of information when the sequence is fed into the model. A tokenizer with a larger vocabulary so that tokens are more likely to be meaningful words will help the model understand the context. Still, it will also make the model larger.
* **Technical Entities (Classes/Functions/APIs):** `BERT`, `RoBERTa`, `DistilBERT`, `SentenceTransformer`, `all-MiniLM-L6-v2`, `cosine_similarity()`
* **Code Snippet:**
```python
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np
 
# Define some example sentences
sentences = [
    "The cat sat on the mat.",
    "The dog slept on the floor.",
    "I love natural language processing."
]
 
# Load a pre-trained model and generate embeddings
model = SentenceTransformer("all-MiniLM-L6-v2")
embeddings = model.encode(sentences)
 
# Get embeddings with mean pooling
print(f"Embedding shape: {embeddings.shape}")
print(f"First 5 dimensions of the sentences' embeddings:\n{np.round(embeddings[:, :5], 3)}")
 
# Calculate cosine similarity between the first two sentences
similarity = cosine_similarity([embeddings[0]], [embeddings[1]])
print(f"Cosine similarity between '{sentences[0]}' and '{sentences[1]}': {np.round(similarity[0][0], 3)}")
```