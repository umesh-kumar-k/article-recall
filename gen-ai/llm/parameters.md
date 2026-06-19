---
aliases:
  - LLM Parameters
Source 1: https://www.ibm.com/think/topics/llm-parameters
---
# What are LLM parameters?



* **Key Points:**
  - LLM parameters are the settings that control and optimize a large language model's (LLM) output and behavior. Trainable parameters include weights and biases and are configured as a large language model (LLM) learns from its training dataset. Hyperparameters are external to the model, guiding its learning process, determining its structure and shaping its output.
* **Technical Entities (Classes/Functions/APIs):** `LLM`
* **Code Snippet:** None

## Types of LLM parameters
* **Key Points:**
  - LLM parameters can be sorted into three primary categories: Weights; Biases; Hyperparameters
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Weights
* **Key Points:**
  - Weights are numerical values that represent the importance that the LLM assigns to a specific input. Not all inputs are treated equally by the artificial intelligence (AI) model when generating responses. The higher an input's weight, the more relevant it is to the model's output.
  - Trainable parameter settings such as weights are configured by a model's learning algorithm during the training process. The learning algorithm measures the machine learning (ML) model's performance with a loss function, which attempts to minimize error through optimizing the model's parameters.
  - Within neural networks, weights are multipliers that determine the signal strength from one neuron layer to the next. Signals must meet the activation function's strength threshold to advance through the network. As such, weights directly affect how a network propagates data forward through its layers.
  - Backpropagation is used to calculate how a change to weight values affects model performance.
* **Technical Entities (Classes/Functions/APIs):** `backpropagation`
* **Code Snippet:** None

### Biases
* **Key Points:**
  - Like weights, biases are also configured automatically during AI model training. Biases are constant values added to a signal's value from the previous layers. Models use biases to allow neurons to activate under conditions where the weights alone might not be sufficient to pass through the activation function.
  - Biases enable models to be more flexible. Models can learn from data even if the weighted inputs do not meet the activation threshold. Like weights, biases are adjusted with backpropagation during training to optimize model performance and minimize errors.
  - The combination of weights and biases in LLMs can result in models with billions of parameters. During the fine-tuning process—when a pretrained LLM is further trained for downstream tasks—its weights and biases are tweaked with domain-specific training data.
* **Technical Entities (Classes/Functions/APIs):** `backpropagation`
* **Code Snippet:** None

### Hyperparameters
* **Key Points:**
  - Hyperparameters are external settings that determine a model's behavior, shape, size, resource use and other characteristics. The process of hyperparameter tuning or model tuning uses algorithms to uncover the optimal combination of hyperparameters for better performance. Along with prompt engineering, hyperparameter tuning is one of the primary LLM customization methods.
  - Architecture hyperparameters, such as the number of layers and dimension of the hidden layers, configure a model's size and shape.
  - Training hyperparameters, such as the learning rate and batch size, guide the model's training process. Training hyperparameters strongly affect model performance and whether a model meets the required LLM benchmarks.
  - Inference hyperparameters, such as temperature and top-p sampling, decide how a generative AI model produces its outputs.
  - Memory and compute hyperparameters, such as the context window, maximum number of tokens in an output sequence and stop sequences, balance model performance and capabilities with resource requirements.
  - Output quality hyperparameters, such as presence penalty and frequency penalty, help LLMs generate more varied and interesting outputs while controlling costs.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Notable LLM parameters
* **Key Points:**
  - The number of parameters in larger models—complex neural networks such as GPT-4 and GPT-3, Llama, Gemini and other transformer models—can reach into the billions. Smaller models have fewer parameters, making them less compute-hungry but also less able to discern complex patterns and relationships.
  - All parameters help determine how the model makes sense of the real-world data it encounters. But the parameters most directly affecting the model's output are its hyperparameters. One benefit to open source models is that their hyperparameter settings are visible.
  - Hyperparameter tuning is a significant pillar of LLM customization: tweaking a model for specific tasks.
  - Among the most significant of a model's hyperparameters are: Number of layers; Context window; Temperature; Top-p (nucleus sampling); Top-k; Token number (max tokens); Learning rate; Frequency penalty; Presence penalty; Stop sequence
* **Technical Entities (Classes/Functions/APIs):** `GPT-4`, `GPT-3`, `Llama`, `Gemini`
* **Code Snippet:** None

### Number of layers
* **Key Points:**
  - The number of layers in a neural network is a crucial hyperparameter for setting model size and complexity. Neural networks are made of layers of neurons or nodes. The more layers between the initial input layer and final output layer, the more complex the model.
  - But complexity isn't always good. A model that has too many layers for a task that doesn't need them can suffer from overfitting and waste computational resources. Meanwhile, a model with insufficient layers will fail to capture the patterns, relationships and distributions in complex datasets.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Context window
* **Key Points:**
  - The context window hyperparameter is relevant to any model built on the transformer architecture, such as the open source LLM Llama-2. The context window is the maximum number of tokens the model can field while maintaining coherency across the entire input sequence.
  - Context window also determines the length of the conversation that a model can maintain without losing track of previous content. Larger context windows lead to greater accuracy, fewer hallucinations and the ability to process larger documents or have longer conversations.
  - However, large context windows also require a greater degree of computational resources and can extend the processing time for response generation.
* **Technical Entities (Classes/Functions/APIs):** `Llama-2`
* **Code Snippet:** None

### Temperature
* **Key Points:**
  - The LLM temperature hyperparameter is akin to a randomness or creativity dial. Raising the temperature increases the probability distribution for the next words that appear in the model's output during text generation.
  - A temperature setting of 1 uses the standard probability distribution for the model. Temperatures higher than 1 flatten the probability distribution, encouraging the model to select a wider range of tokens. Conversely, temperatures lower than 1 widen the probability distribution, making the model more likely to select the most probable next token.
  - A temperature value closer to 1.0, such as 0.8, means that the LLM gets more creative in its responses, but with potentially less predictability. Meanwhile, a lower temperature of 0.2 will yield more deterministic responses. A model with low temperature delivers predictable, if staid, outputs. Higher temperatures closer to 2.0 can begin to produce nonsensical output.
  - The use case informs the ideal temperature value for an LLM. A chatbot designed to be entertaining and creative, such as ChatGPT, needs a higher temperature to create human-like text. A text summarization app in a highly regulated field such as law, health or finance requires the inverse—its generated text summaries must adhere to strict requirements.
* **Technical Entities (Classes/Functions/APIs):** `ChatGPT`
* **Code Snippet:** None

### Top-p (nucleus sampling)
* **Key Points:**
  - Like temperature, top-p sampling also affects word diversity in generated text outputs. Top-p works by setting a probability threshold p for the next token in an output sequence. The model is allowed to generate responses by using tokens within the probability limit.
  - With top-p sampling, tokens are ranked in order of probability. Tokens with a greater likelihood of appearing next in the sequence have a higher score, with the opposite being true for less-likely tokens. The model assembles a group of potential next tokens until the cumulative p score reaches the set threshold, then randomly selects a token from that group.
  - Higher p thresholds result in more diverse outputs, while lower thresholds preserve accuracy and coherency.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Temperature versus top-p sampling
* **Key Points:**
  - The difference between temperature and top-p sampling is that while temperature adjusts the probability distribution of potential tokens, top-p sampling limits token selection to a finite group.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Top-k
* **Key Points:**
  - The top-k hyperparameter is another diversity-focused setting. The k value sets the limit for the number of terms that can be considered as the next in the sequence. Terms are ordered based on probability, and the top k terms are chosen as candidates.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Top-p versus top-k
* **Key Points:**
  - Top-p limits the token pool up to a set p probability total, while top-k limits the pool to the top k most likely terms.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Token number (max tokens)
* **Key Points:**
  - The token number or max tokens hyperparameter sets an upper limit for output token length. Smaller token number values are ideal for quick tasks such as chatbot conversations and summarization—tasks that can be handled by small language models as well as LLMs.
  - Higher token number values are better for when longer outputs are needed, such as if attempting to use an LLM for vibe coding.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Learning rate
* **Key Points:**
  - Learning rate is a critical hyperparameter that affects the speed at which the model adjusts its weights and biases during training and fine-tuning. These processes often use a learning algorithm known as gradient descent.
  - A gradient descent algorithm attempts to minimize a loss function that measures the error of a model's predictions. At each iteration of training, the algorithm updates the model's weights to ideally improve performance with the next batch of data.
  - Learning rate controls the degree to which the weights are updated. A higher learning rate leads to bigger increases, speeding up training at the risk of overshooting a local minimum. Lower learning rates make more subtle adjustments but require more iterations to reach a minimum and can even stall out.
  - One effective method for managing learning rate is to start training with a higher value and lower the learning rate as the model approaches a local minimum of its loss function.
* **Technical Entities (Classes/Functions/APIs):** `gradient descent`
* **Code Snippet:** None

### Frequency penalty
* **Key Points:**
  - The frequency penalty hyperparameter helps prevent models from overusing terms within the same outputs. Once a term appears in the output, the frequency penalty dissuades the model from reusing it again later.
  - Models assign scores to each token known as logits and use logits to calculate probability values. Frequency penalties linearly lower the logit value of a term each time it is repeated, making it progressively less likely to be chosen next time. Higher frequency penalty values lower the logit by a greater amount per application.
  - Because the model is dissuaded from repeating terms, it must instead choose other terms, resulting in more diverse word choices in generated text.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Repetition penalty
* **Key Points:**
  - Repetition penalty is similar to frequency penalty except that it is exponential rather than linear. Repetition penalty lowers a term's logit exponentially each time it is reused, making it a stronger discouragement than the frequency penalty. For this reason, lower repetition penalty values are recommended.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Presence penalty
* **Key Points:**
  - Presence penalty is a related hyperparameter that works similarly to the frequency penalty, except it only applies once. The presence penalty lowers a term's logit value by the same amount regardless of how often that term is present in the output, so long as it appears at least once.
  - If the term bear appears in the output 10 times, and the term fox appears once, bear has a higher frequency penalty than fox. However, both bear and fox share the same presence penalty.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Stop sequence
* **Key Points:**
  - The stop sequence is a preset string of tokens that, when it appears, causes the model to end the output sequence. For example, if a model is designed to output a single sentence at a time, the stop sequence might be a period.
  - Stop sequences maintain response conciseness without affecting the way the model generates output up to the stopping point. Because they truncate model responses, stop sequences also help save on token costs when connecting to LLMs through APIs.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Optimizing LLM parameters
* **Key Points:**
  - Optimizing a model's internal, trainable parameters—its weights and biases—is essential to strong performance. Once a model has been outfitted with the optimal hyperparameters, its designers have a range of methods at their disposal to help shape the internal LLM parameters.
  - Fine-tuning adjusts a model's weights and biases for specific tasks. Parameter-efficient fine-tuning (PEFT) freezes most parameters while changing a small relevant subset.
  - Transfer learning is a broad school of model optimization techniques that all center around using a model's prior knowledge to improve performance on new tasks.
  - Quantization simplifies all the math inside a model, making it smaller and more efficient while still representing the same data.
  - Early stopping prevents overfitting by aborting the training process when it stops making noticeable performance gains.
* **Technical Entities (Classes/Functions/APIs):** `PEFT`
* **Code Snippet:** None