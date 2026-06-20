---
aliases:
  - Few Shot Prompting
Source 1: https://www.ibm.com/think/topics/zero-shot-prompting
Source 2: https://www.ibm.com/think/topics/few-shot-prompting
---
# What is zero-shot prompting?

* **Key Points:**
  - "Zero-shot prompting is a prompt engineering method that relies on the pretraining of a large language model (LLM) to infer an appropriate response."
  - "In contrast to other prompt engineering methods such as few-shot prompting, models aren’t provided examples of output when prompting with the zero-shot technique."
* **Technical Entities (Classes/Functions/APIs):** None specified

## How zero-shot prompting works
* **Key Points:**
  - "In zero-shot prompting, the model is prompted to generate a response without receiving an example of the desired output for the use case."
  - "Zero-shot prompting is an application of zero-shot learning, a machine learning pattern that asks models to make predictions with zero training data."
  - "The zero-shot prompting scenario mimics human learning in applying prior knowledge to solve new problems."
* **Technical Entities (Classes/Functions/APIs):** `granite-3-8b-instruct`, `IBM watsonx.ai Prompt Lab`
* **Code Snippet:**
```
Set the Class name for the issue described to either High, Medium or Low. Predict only the Class name for the last issue. Give a short description for why that Class name was chosen.

Class name: High
Description: An issue that has a high business cost, impacts many users or both.

Class name: Medium
Description: An issue that falls somewhere in between high and low.

Class name: Low
Description: An issue that has an impact on a few users, does not have a high business cost or both.

Issue: Users are reporting that they are unable to upload files.

Class: High
```

### Components of a prompt
* **Key Points:**
  - "Instruction: First, the instructions provided to the model are to 'Set the Class name for the issue described...'"
  - "Context: Next, the context for the model includes a description of class names."
  - "Input data: The model receives input data to run the classification task with the prompt of 'Issue: Users are reporting that they are unable to upload files.'"
  - "Output indicator: Optionally, the model can receive an output indicator, in this case the text 'Class:' which cues the model to respond with the class name of the issue. Output indicators indicate to the model what type of output to produce for a specific type of response."
  - "The customized format of this prompt is for the classification problem at hand. Other use cases might require a different format for the prompt and they might not contain the same instruction, context, input data and output indicator components."
  - "Different models can require different formats for a prompt. Be sure to follow any instructions given for how to format a prompt for a specific model."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Zero-shot prompting vs few-shot prompting
* **Key Points:**
  - "In contrast to zero-shot prompting, few-shot prompting provides the model with examples of expected input and output for the task."
  - "Few-shot prompting, which is a strategy derived from the few-shot learning paradigm, is typically used to improve the performance of the model over zero-shot prompting on a task."
  - "In deciding whether to use zero-shot or few-shot prompting, one should consider the constraints of the problem and the demonstrated performance of both strategies."
  - "Reynolds and McDonell (2021) have found that with improvements in prompt structure, zero-shot prompting can outperform few-shot prompting in some scenarios."
  - "Schulhoff et al. (2024) find different results comparing the performance of several prompting strategies."
* **Technical Entities (Classes/Functions/APIs):** `granite-3-8b-instruct`, `IBM watsonx.ai Prompt Lab`
* **Code Snippet:**
```
Set the Class name for the issue described to either High, Medium or Low. I've provided a few examples of issues and their corresponding Class names. Predict only the Class name for the last issue. Give a short description for why that Class name was chosen.

Class name: High
Description: An issue that has a high business cost, impacts many users or both.

Class name: Medium
Description: An issue that falls somewhere in between high and low.

Class name: Low
Description: An issue that has an impact on a few users, does not have a high business cost or both.

Issue: New users report that they cannot create accounts.
Class: High

Issue: A user reports that labels are rendering overlapping in the app's analytics function.
Class: Low

Issue: Users in the United States are reporting that the app is crashing when trying to make a purchase.
Class: High

Issue: Users report that images are not loading in the app.

Class: Medium
```

## Advantages and limitations of zero-shot prompting
* **Key Points:**
  - "Zero-shot prompting is a popular approach because of its advantages."
  - **Advantages:** "Simplicity: Prompts are simple to construct and easy to understand. This approach allows for users to experiment with different prompts without deep prompt engineering knowledge."
  - "Ease of use: Zero-shot prompting doesn't require any additional data, making it valuable in cases where relevant data is difficult to source or scarce."
  - "Flexibility: Prompts are easy to adapt as needed. Improving a prompt or updating a prompt due to changing circumstances requires low effort."
  - **Limitations:** "Performance variability: While zero-shot prompting can be effective, its performance can vary significantly depending on the complexity and specificity of the task."
  - "Dependence on pretrained model quality: The success of zero-shot prompting heavily depends on the quality and comprehensiveness of the pretrained language model. If the model lacks sufficient exposure to certain topics, languages or contexts during pretraining, its zero-shot performance on related tasks will likely be poor."
  - "Researchers continue to develop techniques to address the limitations of this prompting technique."
  - "Advances in training methods for LLMs have improved model output for zero-shot prompting across various use cases."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Improvements in zero-shot prompting performance
* **Key Points:**
  - "Zero-shot prompting relies on the foundation model's pretrained knowledge and its flexibility to adapt to the requested prompt and to provide an appropriate response."
  - "Two improvements that have resulted in better zero-shot prompting performance are instruction tuning and reinforcement learning with human feedback (RLHF)."
  - "In instruction tuning, a model is fine-tuned by using supervised learning on a dataset that includes instructions for various tasks and outcomes for these tasks."
  - "This strategy of fine-tuning with an instructional dataset has resulted in better zero-shot prompting performance on new tasks in these categories."
  - "Another example of using fine-tuning to improve zero-shot prompting outcomes is RLHF fine-tuning, in which reinforcement learning learns a policy that guides the model to better outputs."
  - "In this 3-step process, the model is first fine-tuned by using an instructional dataset in which humans have provided the target responses. Then, the model projects outputs to several prompts ranked by humans. Lastly, the ranked outputs are used to train a reinforcement learning model that learns a policy to select best outputs based on these human-provided rankings."
  - "The final step uses reinforcement learning's ability to use the consequences (rewards or punishments) of actions (decision or path taken) to learn a strategy (or policy) for making good decisions."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Applications of zero-shot prompting
* **Key Points:**
  - "Artificial intelligence practitioners and data scientists can use the generative AI technology of large language models in the zero-shot prompting scenario for various use cases including:"
  - "Text classification: As shown in the prior example classifying the priority of IT issues with IBM's granite-3-8b-instruct model, the model can achieve classification without prior examples belonging to the different classes. This capability is ideal for situations where limited or no labeled training data exists."
  - "Information extraction: Given a body of text and a question, an LLM can extract the requested information in accordance with a prompt."
  - "Question answering: Using the pretrained knowledge of a model, a user can prompt for a response to a question."
  - "Text summarization: Given text and an instruction for text summarization, large language models can run this task in the zero-shot prompt scenario without requiring example summaries of other texts."
  - "Generation: LLMs generate data in the form of text, code, images and more for specified use cases."
  - "Conversation: Typically using models tuned for chat (such as the well-known chat-GPT series), LLMs can interact with a user in chat mode, accomplishing many of the preceding use cases."
* **Technical Entities (Classes/Functions/APIs):** `granite-3-8b-instruct`

## Other prompt engineering methods
* **Key Points:**
  - "For complex use cases such as multistep reasoning tasks, both zero-shot prompting and few-shot prompting might fail to produce an appropriate response from the model."
  - "Chain-of-thoughts: Chain-of-thought (CoT) prompting is a strategy that poses a task to the model by specifying the larger task as a series of discrete steps to solve."
  - "This prompt engineering technique shows success in areas including improving customer service chatbot performance, helping organize the thoughts of researchers and writers and generating step-by-step descriptions for math and science educational problems."
  - "Tree of thoughts: Tree-of-thought (ToT) prompting generates a branching text tree of potential next steps and corresponding possible solutions to the problem."
  - "It is designed to approximate human reasoning strategies when comparing potential paths to a solution."
  - "Common strategies for exploring solutions are breadth-first-search (BFS) and depth-first-search (DFS) along with the heuristic search and reinforcement learning approaches."
  - "Researchers have used this application to solve puzzles such as sudoku and the Game of 24."
* **Technical Entities (Classes/Functions/APIs):** None specified


# What is few shot prompting?

* **Key Points:**
  - "Few-shot prompting refers to the process of providing an AI model with a few examples of a task to guide its performance."
  - "This method is particularly useful in scenarios where extensive training data is unavailable."
  - "In other techniques like zero-shot prompting, which requires no examples, or one-shot prompting, which relies on a single example, few-shot prompting uses multiple examples to improve accuracy and adaptability."
  - "Few-shot learning is essential in situations for generative AI where gathering large amounts of labelled data is challenging."
* **Technical Entities (Classes/Functions/APIs):** `IBM granite series`, `Meta's Llama models`, `OpenAI's GPT-3`, `GPT-4`

## How few shot prompting works?
* **Key Points:**
  - "Few-shot prompting operates by presenting the model with several examples of the desired task within the prompt."
  - "User query: The process begins with a user query, such as 'This product is very cost effective'."
  - "Vector store: All examples are stored in a vector store, a database optimized for semantic search. When a user query is received, the system performs semantic matching to find the most relevant examples from the vector store."
  - "Retrieving relevant examples: Only the most relevant examples are retrieved and used to form the prompt. In this example, Retrieval-Augmented Generation (RAG) is utilized to retrieve the examples from a vector store, which helps tailor the prompt to the specific query."
  - "While RAG is not universally required for few-shot prompting, it can significantly enhance the process by ensuring that the most contextually relevant examples are used, thereby improving the model's performance in certain scenarios."
  - "Prompt formation: The prompt is constructed with the retrieved examples and the user query."
  - "LLM Processing: The constructed prompt is then fed into the LLM. The model processes the prompt and generates an output, in this case, classifying the sentiment of the user query."
  - "Output: The LLM outputs the classification, such as 'negative' for the given example."
  - "Research has highlighted the effectiveness of a few-shot learning approach that reduces reliance on extensive prompt engineering."
  - "Unlike traditional fine-tuning, which involves adjusting model parameters using a large dataset before prompting, fine-tuning in the few-shot setting refers to the process of adapting pre-trained models with just a few examples provided directly within the prompt."
  - "This approach allows the model to leverage its pre-existing knowledge more effectively without the need for additional training on large datasets."
  - "This study demonstrated that even when using 'null prompts'—prompts that do not contain any task-specific templates or labeled examples—the model could still achieve competitive accuracy across various tasks."
  - "Despite this lack of structure, the model can perform well, showcasing the robustness of few-shot learning."
  - "Overall, the study suggests that few-shot learning is a highly effective strategy, particularly when structured prompts are used. While null prompts can yield good results, adding a few well-chosen examples can further enhance the model's performance, making it a versatile and efficient approach, especially in scenarios with limited labeled data."
* **Technical Entities (Classes/Functions/APIs):** `RAG` (Retrieval-Augmented Generation), `vector store`

## Advantages and limitations of few shot prompting
* **Key Points:**
  - **Advantages:**
    - "Efficiency and Flexibility: Few-shot prompting significantly reduces the amount of labelled data required for training, making it highly efficient and adaptable to new tasks."
    - "Improved Performance in Diverse Applications: Few-shot prompting has demonstrated significant improvements in various applications, from text classification to machine translation and beyond."
    - "Robustness to Different Prompts: The robustness of few-shot prompting to different prompt formulations is another key advantage."
    - "Reduced Computational Overhead: Recent advancements have made few-shot prompting more efficient."
  - **Limitations:**
    - "Dependence on Prompt Quality: The quality and design of prompts significantly impact the performance of few-shot prompting."
    - "Computational Complexity: Large language models used in few-shot prompting require substantial computational resources."
    - "Challenge of Generalization: Generalizing prompts across diverse tasks and datasets remains a significant challenge."
    - "Limited Zero-Shot Capabilities: While few-shot prompting excels with minimal examples, its performance in zero-shot settings can be less reliable."
  - "Thus, few-shot prompting offers substantial benefits in terms of efficiency, flexibility, and performance across various applications. However, its dependence on prompt quality, computational complexity, challenges in generalization, and limited zero-shot capabilities highlight the areas where further advancements are needed to maximize its potential."
* **Technical Entities (Classes/Functions/APIs):** `TransPrompt`, `UPT` (Unified Prompt Tuning), `SetFit`, `Sentence Transformers`, `QaNER`

## Use cases
* **Key Points:**
  - "Few-shot prompting has proven to be a versatile and powerful tool with a number of examples across various applications, leveraging the strengths of large language models to perform complex tasks with limited examples."
  - "Sentiment Analysis: Few-shot prompting is particularly useful in sentiment analysis where models classify the sentiment of a text with limited labelled data."
  - "Action Recognition in Videos: Few-shot prompting has also been applied to action recognition in videos."
  - "Grounded Dialog Generation: In grounded dialog generation or chatbots, few-shot prompting strengthens dialog models by integrating external information sources."
  - "Named Entity Recognition (NER): Few-shot prompting can enhance named entity recognition tasks by providing examples that help the model recognize and classify entities within the text."
  - "Code generation Tasks: Few-shot prompting is also applicable to code-related tasks such as test assertion generation and program repair."
  - "These use cases demonstrate the wide-ranging applicability and effectiveness of few-shot prompting across different fields and tasks, showcasing its potential to drive innovation and efficiency in AI and NLP applications."
* **Technical Entities (Classes/Functions/APIs):** None specified