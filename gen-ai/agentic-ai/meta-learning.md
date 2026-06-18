---
aliases:
  - Meta Learning
highlights: Agents improve performance over time by analyzing past task excecutions and adapting strategies to new contexts
Source 1: https://www.ibm.com/think/topics/meta-learning
---
# What is meta learning?



* **Key Points:**
  - Meta learning, also called "learning to learn," is a subcategory of machine learning that trains artificial intelligence (AI) models to understand and adapt to new tasks on their own.
  - Meta learning's primary aim is to provide machines with the skill to learn how to learn.
  - Unlike conventional supervised learning, where models are trained to solve a specific task using a defined training dataset, the meta learning process entails a variety of tasks, each with its own associated dataset.
  - From these multiple learning events, models gain the ability to generalize across tasks, which allow them to adapt swiftly to novel scenarios even with little data.
  - Meta learning algorithms are trained on the predictions and metadata of other machine learning algorithms. Meta learning algorithms then generate predictions of their own and information that can be used to enhance the performance and results of other machine learning algorithms.
* **Technical Entities (Classes/Functions/APIs):** `meta learning`, `machine learning`, `supervised learning`
* **Code Snippet:** None.

---

## How meta learning works

* **Key Points:**
  - Meta learning involves two key stages: meta training and meta testing. For both stages, a base learner model adjusts and updates its parameters as it learns. The dataset used is divided into a support set for meta training and a test set for meta testing.
  - Meta training: In the meta training phase, the base learner model is supplied with a wide array of tasks. The model's goal is to uncover common patterns among these tasks and acquire broad knowledge that can be applied in solving new tasks.
  - Meta testing: During the meta testing phase, the base learner model's performance is assessed by giving it tasks it hasn't encountered when it was trained. The model's effectiveness is measured by how well and how fast it adapts to these new tasks using its learned knowledge and generalized understanding.
* **Technical Entities (Classes/Functions/APIs):** `base learner model`, `support set`, `test set`
* **Code Snippet:** None.

---

## Common meta learning approaches

* **Key Points:**
  - There are three typical approaches to meta learning. Here's how each approach works and their different types:
  - Metric-based meta learning: Metric-based meta learning is centered around learning a function that computes a distance metric, which is a measure of the similarity between two data points. This approach is akin to the k-nearest neighbors (KNN) algorithm, which uses proximity to make classifications or predictions.
  - Model-based meta learning: Model-based meta learning involves learning a model's parameters, which can facilitate rapid learning from sparse data.
  - Optimization-based meta learning: Deep learning typically requires multiple iterative updates of model parameters through backpropagation and the gradient descent optimization algorithm. In optimization-based meta learning, sometimes called gradient-based meta learning, the algorithm learns which initial model parameters or hyperparameters of deep neural networks can be efficiently fine-tuned for relevant tasks. This usually means meta-optimization—that is, optimizing the optimization algorithm itself.
* **Technical Entities (Classes/Functions/APIs):** `Metric-based meta learning`, `k-nearest neighbors (KNN)`, `Model-based meta learning`, `Optimization-based meta learning`, `gradient descent`, `meta-optimization`
* **Code Snippet:** None.

---

## Metric-based meta learning

* **Key Points:**
  - Convolutional Siamese neural network: A convolutional Siamese neural network consists of identical twin convolutional neural networks that share parameters and weights. Parameter updates are mirrored across the two networks. Both networks are joined by a loss function that calculates a distance metric (usually pairwise similarity). The training dataset is composed of pairs of matching and nonmatching samples. Convolutional Siamese neural networks then learn to compute pairwise similarity, maximizing the Euclidean distance between nonmatching or dissimilar pairs and minimizing the distance between matching or similar pairs.
  - Matching networks: Matching networks learn to predict classification by measuring a distance metric known as cosine similarity between two samples.
  - Relation network: A relation network learns a deep nonlinear distance metric for comparing items. The network classifies items by calculating relation scores, which represent the similarity between items.
  - Prototypical networks: Prototypical networks compute the mean of all samples of a class to create a prototype for that class. The network then learns a metric space, where classification tasks are done by calculating the squared Euclidean distance between a particular data point and the prototype representation of a class.
* **Technical Entities (Classes/Functions/APIs):** `Convolutional Siamese neural network`, `convolutional neural networks`, `Euclidean distance`, `Matching networks`, `cosine similarity`, `Relation network`, `Prototypical networks`
* **Code Snippet:** None.

---

## Model-based meta learning

* **Key Points:**
  - Memory-augmented neural networks: A memory-augmented neural network (MANN) is equipped with an external memory module to allow for stable storage and speedy encoding and retrieval of information. In meta learning, MANNs can be trained to learn a general technique for the kinds of representations to store in external memory and a method for using those representations to make predictions. MANNs have been shown to perform well in regression and classification tasks.
  - Meta Networks: MetaNet (short for Meta Networks) is a meta learning model that can be applied in imitation learning and reinforcement learning. Like MANNs, Meta Networks also have external memory. MetaNet is composed of a base learner and a meta learner working in separate space levels. The meta learner acquires general knowledge across different tasks within a meta space. The base learner takes an input task and sends meta information about the current task space to the meta learner. Based on this information, the meta learner conducts fast parameterization to update the weights within both spaces.
* **Technical Entities (Classes/Functions/APIs):** `Memory-augmented neural network (MANN)`, `Meta Networks (MetaNet)`, `base learner`, `meta learner`
* **Code Snippet:** None.

---

## Optimization-based meta learning

* **Key Points:**
  - LSTM meta-learner: This optimization-based meta learning method uses a popular recurrent neural network architecture called long-short term memory (LSTM) networks to train a meta-learner to acquire both long-term knowledge shared among tasks and short-term knowledge from each task. The meta-learner then optimizes another learner neural network classifier. It learns an initialization of the learner's parameters for fast training convergence and how to update those parameters efficiently given a small training set, helping the learner adapt to new tasks quickly.
  - Model-agnostic meta learning (MAML): As its name implies, this optimization-based meta learning algorithm is model-agnostic. This makes it compatible with any model trained using gradient descent and suitable for solving various learning problems, such as classification, regression and reinforcement learning. The core idea behind MAML is to train the model's initial parameters in a way that a few gradient updates will result in rapid learning on a new task. The goal is to determine model parameters that are sensitive to changes in a task such that minor changes to those parameters lead to major improvements in the task's loss function. Meta-optimization across tasks is done using stochastic gradient descent (SGD). Unlike gradient descent, which computes derivatives to optimize a model's parameters for a certain task, MAML computes second derivatives to optimize a model's initial parameters for task-specific optimization. A modified version of model-agnostic meta learning, known as first-order MAML or FOMAML, omits second derivatives for a less computationally expensive process.
  - Reptile: Reptile is a first-order gradient-based meta learning algorithm similar to FOMAML. It repeatedly samples a task, trains on that task through many gradient descent steps and moves the model weight toward the new parameters.
* **Technical Entities (Classes/Functions/APIs):** `LSTM meta-learner`, `long-short term memory (LSTM)`, `Model-agnostic meta learning (MAML)`, `stochastic gradient descent (SGD)`, `first-order MAML (FOMAML)`, `Reptile`
* **Code Snippet:** None.

---

## Meta learning use cases in machine learning

* **Key Points:**
  - To further demonstrate meta learning's versatility, here are a few ways meta learning can be used within the machine learning realm itself:
  - Automated machine learning (AutoML): Automated machine learning (AutoML) allows for the automation of tasks in the machine learning pipeline. Meta learning techniques are well suited for AutoML, especially when it comes to hyperparameter optimization and model selection. Fine-tuning hyperparameters for machine learning models is typically done manually. Meta learning algorithms can help automate this procedure by learning how to optimize hyperparameters or identifying the ideal hyperparameters for a certain task. Meta learning algorithms can also learn how to choose the most appropriate model—and even that model's parameters and architecture—to solve a specific task. This helps automate the model selection process.
  - Few-shot learning: Few-shot learning is a machine learning framework that trains an AI model on a small number of examples. Most few-shot learning methods are built around meta learning, where models adapt to new tasks given scarce training data.
  - Recommendation engines: A recommendation engine relies on machine learning algorithms to find patterns in user behavior data and recommend relevant items based on those patterns. Meta learning systems can learn recommendation models to generate more accurate and more relevant suggestions that better personalize user experiences.
  - Transfer learning: Meta learning can help facilitate transfer learning, which adapts a pretrained model to learn new tasks or previously unseen classes of data.
* **Technical Entities (Classes/Functions/APIs):** `Automated machine learning (AutoML)`, `hyperparameter optimization`, `model selection`, `Few-shot learning`, `recommendation engine`, `Transfer learning`
* **Code Snippet:** None.

---

## Applications of meta learning

* **Key Points:**
  - Meta learning can be applied to different areas of the technology industry, some of which include:
  - Computer vision: Meta learning can be employed for computer vision tasks, which include facial recognition, image classification, image segmentation, object detection and object tracking.
  - Natural language processing: Meta learning can be used for natural language processing tasks, such as language modeling, sentiment classification, speech recognition and text classification.
  - Robotics: Meta learning can help robots rapidly learn new tasks and adapt to dynamic environments. It can be applied in a number of tasks such as grasping, navigation, manipulation and movement.
* **Technical Entities (Classes/Functions/APIs):** `Computer vision`, `Natural language processing`, `Robotics`
* **Code Snippet:** None.

---

## Benefits of meta learning

* **Key Points:**
  - Meta learning holds much potential. Here are a few of its advantages:
  - Adaptability: Meta learning can be used to build more generalized AI models that can learn to do many related tasks. Due to this flexibility, meta learning systems can quickly adapt to new tasks and different domains.
  - Efficient use of data: Meta learning supports learning from just a few samples, which might eliminate the need for huge dataset volumes. This can be particularly helpful for domains where gathering and preparing data might be labor intensive and time consuming.
  - Decreased training time and training cost: Because of its data efficiency and swift learning, meta learning can result in a faster training process and reduced training costs.
* **Technical Entities (Classes/Functions/APIs):** None.
* **Code Snippet:** None.

---

## Challenges of meta learning

* **Key Points:**
  - Despite the promise of meta learning, it also presents challenges. Here are some of them:
  - Lack of data: Sometimes, the amount of data to train AI models is insufficient, especially for niche domains. Or, if data is available, the quality might not be adequate to efficiently train meta learning algorithms.
  - Overfitting: Not having enough variability among tasks in the support set for meta training can lead to overfitting. This means that a meta learning algorithm might only be applicable to specific tasks without being able to effectively generalize across a broad spectrum of tasks.
  - Underfitting: Conversely, having too much variability among tasks in the support set for meta training can result in underfitting. This means that a meta learning algorithm might not be able to use its knowledge in solving another task and might have difficulty adapting to new scenarios. Therefore, a balance in task variability is key.
* **Technical Entities (Classes/Functions/APIs):** `support set`
* **Code Snippet:** None.