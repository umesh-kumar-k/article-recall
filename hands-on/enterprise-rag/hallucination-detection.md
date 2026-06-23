---
aliases:
  - Hallucination Detection
Source 1: https://machinelearningmastery.com/rag-hallucination-detection-techniques/
Source 2: https://aws.amazon.com/blogs/machine-learning/detect-hallucinations-for-rag-based-systems/
---
## RAG Hallucination Detection Techniques
* **Key Points:**
  - Large language models (LLMs) are useful for many applications, including question answering, translation, summarization, and much more, with recent advancements in the area having increased their potential.
  - As you are undoubtedly aware, there are times when LLMs provide factually incorrect answers, especially when the response desired for a given input prompt is not represented within the model's training data.
  - This leads to what we call hallucinations.
  - To mitigate the hallucination problem, retrieval augmented generation (RAG) was developed.
  - This technique retrieves data from a knowledge base which could help satisfy a user prompt's instructions.
  - While a powerful technique, hallucinations can still manifest with RAG.
  - This is why detecting hallucinations and formulating a plan to alert the user or otherwise deal with them in RAG systems is of the utmost importance.
  - As the foremost point of importance with contemporary LLM systems is the ability to trust their responses, the focus on detecting and handling hallucinations has become more important than ever.
  - In a nutshell, RAG works by retrieving information from a knowledge base using various types of search such as sparse or dense retrieval techniques.
  - The most relevant results will then be passed into LLM alongside the user prompt in order to generate the desired output.
  - However, hallucination can still occur in the output for numerous reasons, including: LLMs acquire accurate information, but they fail to generate correct responses. It often happens if the output requires reasoning within the given information. The retrieved information is incorrect or does not contain relevant information. In this case, LLM might try to answer questions and hallucinate.
  - As we are focusing on hallucinations in our discussion, we will focus on trying to detect the generated responses from RAG systems, as opposed to trying to fix the retrieval aspects.
  - In this article, we will explore hallucination detection techniques that we can use to help build better RAG systems.
* **Technical Entities (Classes/Functions/APIs):** `LLMs`, `RAG`

## Hallucination Metrics
* **Key Points:**
  - The first thing we will try is to use the hallucination metrics from the DeepEval library.
  - Hallucination metrics are a simple approach to determining whether the model generates factual, correct information using a comparison method.
  - It's calculated by adding the number of context contradictions to the total number of contexts.
  - The evaluation will be based on the LLM that evaluates the result.
  - This means we will need the model as an evaluator.
  - For this example, we will use the OpenAI model that is set by default from DeepEval.
* **Technical Entities (Classes/Functions/APIs):** `DeepEval`, `HallucinationMetric`, `LLMTestCase`, `evaluate`, `OpenAI`
* **Code Snippet:**
```python
pip install deepeval
```
```python
import os
os.environ["OPENAI_API_KEY"] = "YOUR-API-KEY"
```
```python
from deepeval import evaluate
from deepeval.metrics import HallucinationMetric
from deepeval.test_case import LLMTestCase
 
context = [
    "The Great Wall of China is a series of fortifications made of stone, brick, tamped earth, wood, and other materials, "
    "generally built along an east-to-west line across the historical northern borders of China to protect the Chinese states "
    "and empires against the raids and invasions of the nomadic groups of the Eurasian Steppe."
]
 
actual_output = ("The Great Wall of China is made entirely of gold and was built in a single year by the Ming Dynasty to store treasures.")
```
```python
test_case = LLMTestCase(
    input="What is the Great Wall of China made of and why was it built?",
    actual_output=actual_output,
    context=context
)
 
halu_metric = HallucinationMetric(threshold=0.5)
```
```python
halu_metric.measure(test_case)
print("Hallucination Metric:")
print("  Score: ", halu_metric.score)
print("  Reason: ", halu_metric.reason)
```

## G-Eval
* **Key Points:**
  - G-Eval is a framework that uses LLM with chain-of-thoughts (CoT) methods to automatically evaluate the LLM output based on a multi-step criteria we decide upon.
  - We will then use DeepEval's G-Eval framework and our criteria to test the RAG's ability to generate output and determine whether they are hallucinating.
  - With G-Eval, we will need to set up the metrics ourselves based on our criteria and the evaluation steps.
* **Technical Entities (Classes/Functions/APIs):** `G-Eval`, `chain-of-thoughts (CoT)`, `GEval`, `LLMTestCaseParams`
* **Code Snippet:**
```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCaseParams
 
correctness_metric = GEval(
    name="Correctness",
    criteria="Determine whether the actual output is factually accurate, logically consistent, and sufficiently detailed based on the expected output.",
    evaluation_steps=[
        "Check if the 'actual output' aligns with the facts in 'expected output' without any contradictions.",
        "Identify whether the 'actual output' introduces new, unsupported facts or logical inconsistencies.",
        "Evaluate whether the 'actual output' omits critical details needed to fully answer the question.",
        "Ensure that the response avoids vague or ambiguous language unless explicitly required by the question."
    ],
    evaluation_params=[LLMTestCaseParams.INPUT, LLMTestCaseParams.ACTUAL_OUTPUT, LLMTestCaseParams.EXPECTED_OUTPUT],
)
```
```python
from deepeval.test_case import LLMTestCase
 
test_case = LLMTestCase(
    input="When did the Apollo 11 mission land on the moon?",
    actual_output="Apollo 11 landed on the moon on July 21, 1969, marking humanity's first successful moon landing.",
    expected_output="Apollo 11 landed on the moon on July 20, 1969, marking humanity's first successful moon landing.",
    retrieval_context=[
        """The Apollo 11 mission achieved the first successful moon landing on July 20, 1969.
        Astronauts Neil Armstrong and Buzz Aldrin spent 21 hours on the lunar surface, while Michael Collins orbited above in the command module."""
    ]
)
```
```python
correctness_metric.measure(test_case)
 
print("Score:", correctness_metric.score)
print("Reason:", correctness_metric.reason)
```

## Faithfulness Metric
* **Key Points:**
  - If you want more quantified metrics, we can try out the RAG-specific metrics to test whether or not the retrieval process is good.
  - The metrics also include a metric to detect hallucination called faithfulness.
  - There are five RAG-specific metrics available in DeepEval to use, which are: Contextual precision to evaluate the reranker; Contextual recall to evaluate the embedding model to capture and retrieve relevant information accurately; Contextual relevancy evaluates the text chunk size and the top-K; Contextual answer relevancy evaluates whether the prompt is able to instruct the LLM to generate a relevant answer; Faithfulness evaluates whether the LLM generates output that does not hallucinate and contradict any information in the retrieval
  - These metrics differ from the hallucination metric previously discussed, as these metrics focus on the RAG process and output.
* **Technical Entities (Classes/Functions/APIs):** `ContextualPrecisionMetric`, `ContextualRecallMetric`, `ContextualRelevancyMetric`, `AnswerRelevancyMetric`, `FaithfulnessMetric`
* **Code Snippet:**
```python
from deepeval.metrics import (
    ContextualPrecisionMetric,
    ContextualRecallMetric,
    ContextualRelevancyMetric,
    AnswerRelevancyMetric,
    FaithfulnessMetric
)
 
contextual_precision = ContextualPrecisionMetric()
contextual_recall = ContextualRecallMetric()
contextual_relevancy = ContextualRelevancyMetric()
answer_relevancy = AnswerRelevancyMetric()
faithfulness = FaithfulnessMetric()
 
contextual_precision.measure(test_case)
print("Contextual Precision:")
print("  Score: ", contextual_precision.score)
print("  Reason: ", contextual_precision.reason)
 
contextual_recall.measure(test_case)
print("\nContextual Recall:")
print("  Score: ", contextual_recall.score)
print("  Reason: ", contextual_recall.reason)
 
contextual_relevancy.measure(test_case)
print("\nContextual Relevancy:")
print("  Score: ", contextual_relevancy.score)
print("  Reason: ", contextual_relevancy.reason)
 
answer_relevancy.measure(test_case)
print("\nAnswer Relevancy:")
print("  Score: ", answer_relevancy.score)
print("  Reason: ", answer_relevancy.reason)
 
faithfulness.measure(test_case)
print("\nFaithfulness:")
print("  Score: ", faithfulness.score)
print("  Reason: ", faithfulness.reason)
```

## Summary
* **Key Points:**
  - This article has explored different techniques for detecting hallucinations in RAG systems, focusing on three main approaches: hallucination metrics using the DeepEval library; G-Eval framework with chain-of-thoughts methods; RAG-specific metrics including faithfulness evaluation
  - We have looked at some practical code examples for implementing each technique, demonstrating how they can measure and quantify hallucinations in LLM outputs, with a particular emphasis on comparing generated responses against known context or expected outputs.
* **Technical Entities (Classes/Functions/APIs):** `DeepEval`, `G-Eval`, `chain-of-thoughts (CoT)`, `faithfulness`