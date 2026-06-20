---
aliases:
  - Reinforcement Learning From Human Feedback
Source 1: https://huggingface.co/blog/rlhf
Source 2: https://www.superannotate.com/blog/rlhf-for-llm
---


# Reinforcement learning with human feedback (RLHF) for LLMs
* **Key Points:**
  - "RLHF is about fine-tuning LLMs to grasp the subtle nuances of human communication. It's a move towards making language models not only mimic human interactions but also understand and adapt to them. By integrating human feedback directly into the learning process, RLHF aims to make interactions with AI as natural and intuitive as talking to another person."
* **Technical Entities (Classes/Functions/APIs):** `GPT-3`

## What is RLHF?
* **Key Points:**
  - "Reinforcement learning with human feedback (RLHF) is a technique where AI improves by learning directly from human feedback. This way, you enrich AI's learning process with real human insights. In RLHF, AI doesn't just produce what it thinks is best based on data alone but also considers what people actually find useful or relevant."
  - "RLHF is especially handy for natural language processing tasks requiring a human touch, like creating content that genuinely resonates with us. By integrating our feedback, language models become more adept at delivering results that align with human goals and preferences, marking a significant step forward in generative AI applications, including large language models."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`

### Understanding RLHF meaning
* **Key Points:**
  - "Given the variety in language and human choice, it's clear that preferences in summaries can vary widely among individuals. This variability is exactly why summarization isn't a one-size-fits-all task. While specific natural language processing tasks have straightforward answers, summarization is subjective, often leading to multiple 'correct' summaries based on individual preferences. By collecting human feedback, the RLHF model crafts the data needed for later LLM processing."
  - "RLHF can be useful even if you're not training an LLM from scratch."
  - "RLHF is a method for gathering human feedback on which responses they prefer to train a model to generate responses that humans prefer."
  - "In a nutshell, RLHF helps us improve LLM's ability to solve complex tasks where the desired output is difficult to explain or describe. In other words, problems with no single correct answer, which is the case for many LLM problems, RLHF doesn't solve all of the problems of truthfulness and toxicity in language models, but it's been a key part of improving LLM's quality."
* **Technical Entities (Classes/Functions/APIs):** None specified

## RLHF for LLMs
* **Key Points:**
  - "Human feedback is used in various generative AI projects, essentially supporting multimodal applications. A significant portion of RLHF's application in business is focused on developing language models. RLHF for LLMs is straightforward—it involves using human feedback to evaluate the responses of language models, then collecting this feedback to refine and improve those responses."
  - "In business contexts, RLHF for LLMs is particularly useful for improving customer interaction tools, like chatbots or virtual assistants. By training these tools through RLHF, companies can ensure more natural and effective communication, leading to improved customer satisfaction and engagement. This approach is also used to ensure AI systems perform well across various languages and cultural contexts, which is vital for global businesses."
* **Technical Entities (Classes/Functions/APIs):** None specified

## How does RLHF work
* **Key Points:**
  - "RLHF is an evolving area of research, and there are many variations of how we may implement it, but the high-level themes are the same. RLHF consists of three stages: We first create a preference dataset. Then, we use this preference dataset to train a reward function with supervised learning. Afterwards, we use the reward learning in the reinforcement learning loop to fine-tune our base LLM."
* **Technical Entities (Classes/Functions/APIs):** None specified

### RLHF phases
#### Stage 1: Preference dataset
* **Key Points:**
  - "We start by choosing the large language model (LLM) that needs refinement. The process starts by giving the pre-trained model various prompts, such as requests to summarize specific texts, setting the stage for further tuning. Human labelers play a crucial role at this point; they evaluate pairs of model-generated responses to each prompt and select the more suitable option. This comparison builds our LLM evaluation or preference dataset, which captures human preferences among the model's outputs. Creating this dataset is essential but requires clear goals for the model's tuning, like enhancing accuracy, reducing bias, or increasing user engagement."
* **Technical Entities (Classes/Functions/APIs):** None specified

#### Stage 2: Reward model
* **Key Points:**
  - "Next, we take the preference data we've gathered and get down to training a reward model. This model's job is essentially to act as the judge during the training process, scoring the LLM's responses using a reward function based on how well they align with what our human labelers prefer. This step turns qualitative judgments into quantifiable scores, offering a way to measure how close an LLM's response is to the ideal."
  - "Training this model involves feeding it examples of prompts paired with two different responses—the preferred one and the not-so-preferred one. From there, it learns to assign scores that reflect the preferences it's been trained on. The reward function score isn't about right or wrong but aligning closer with human values and preferences."
* **Technical Entities (Classes/Functions/APIs):** None specified

#### Stage 3: Fine-tuning
* **Key Points:**
  - "The final step involves fine-tuning the base language model with the insights from the reward model. The aim here is to adjust the LLM's output to reflect human preferences better, as indicated by higher scores from the reward model."
  - "This step uses a different dataset, filled with prompts, and applies reinforcement learning to improve the model's output, guiding it toward generating responses that humans favor."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Reinforcement learning component
* **Key Points:**
  - "Reinforcement learning (RL) comes into play when you have a complex and not strictly defined task. Imagine trying to teach someone a game without explicitly telling them the rules but instead rewarding them for good moves. That's the essence of RL—it's about guiding the model towards making a series of decisions that lead to the best outcome, even when the 'best' isn't clearly defined from the start."
  - "In reinforcement learning, the model, or 'agent,' learns by doing. It interacts with its environment, makes decisions (or 'actions'), sees how the environment responds and receives rewards or penalties. This process helps the agent figure out the environment's rules. A famous example is AlphaGo, which mastered the game of Go by experimenting with different strategies and learning from the outcomes."
  - "This learning process differs from what we see in supervised learning, where the model learns from clear examples of what to do. In reinforcement learning, there's no set path. The agent explores, tries different actions, and learns from the results. It keeps track of what actions lead to better rewards in different situations, storing this information in a policy. Like the agent's decision-making brain, this policy maps the current state of the environment to the actions the agent should take next, aiming to maximize rewards."
  - "For instance, when tuning a large language model with RL, the 'current state' might include the prompt given to the model and any text it has generated up until that point. The 'actions' are the next tokens or words the model chooses to generate. Each choice the model makes is evaluated by a reward model, which scores how well the generated text aligns with what we're looking for. The goal is to learn a policy that gets the LLM to produce highly scored completions, effectively teaching the model to generate text that matches human preferences more closely."
* **Technical Entities (Classes/Functions/APIs):** `AlphaGo`

## RLHF example: Summary comparison
* **Key Points:**
  - "The first thing you do is collect text samples and have people summarize them. There's rarely just one way to summarize a text – language inherently requires personal touch and perception."
  - "To address this, we focus on understanding what people actually like. By showing data trainers two different summaries and asking which they prefer, we shift from finding a single 'correct' answer to aligning AI outputs with human guidance. This approach is key to RLHF."
  - "In this process, you start with an LLM that's already been trained with instructions and learned to follow them. You then gather a dataset that indicates a human labeler's preferences between multiple completions of the same prompts and use this dataset as a reward signal to fine-tune an instruction-tuned LLM. The result is a tuned LLM that generates outputs that better align with human guidance."
  - "Note that we discussed text in this case but it can be any other data type – you may use images (diffusion models), videos, audio, PDF and so on. The idea is that no matter the data type and use case, feedback data is crucial to improve the model and align it with preferences."
* **Technical Entities (Classes/Functions/APIs):** `SuperAnnotate`

## Why is RLHF important?
* **Key Points:**
  - "It makes AI more human-friendly. Think of it like training a pet. You reward good behavior, right? Similarly, RLHF rewards AI for responses that align with human expectations, not just those that are technically correct. This approach trains AI systems to understand and prioritize what people really need and want."
  - "AI systems can be incredibly smart yet still fail at basic social interactions. RLHF teaches AI the subtle, often unspoken rules of human behavior, making it a more social creature."
  - "It makes AI safer and more ethical. As more people worry about the ethical side of AI, it's important to make sure these systems don't cause harm. RLHF is crucial because it uses feedback from people to push AI away from biased or harmful actions, making sure it acts in ways that meet our ethical standards."
  - "It's scalable. As AI systems get more complex, RLHF provides a practical way to improve and grow their abilities without having to start over from scratch. This makes it an essential tool for AI applications."
* **Technical Entities (Classes/Functions/APIs):** None specified

## RLHF in SuperAnnotate
* **Key Points:**
  - "SuperAnnotate helps companies build RLHF datasets to fine-tune and improve their models. Through its customizable editor, users can build templates for any multimodal use case."
  - "We've put time and effort into gathering a world-class team of experts who meticulously work with clients' data. They ensure top-quality human feedback – the gem for any RLHF project."
  - "The interface is fully customizable – you can build your own use case aside from the ready-made templates!"
  - "Our platform offers analytics and insights that allow the clients to control and understand their data fully."
  - "API integrations make it easy to set up a model in the loop, AI feedback, and much more."
* **Technical Entities (Classes/Functions/APIs):** `SuperAnnotate`

## Alternatives to RLHF
* **Key Points:**
  - "While reinforcement learning from human feedback offers a robust way to align LLM outputs with human preferences, it's not the only method on the table. In fact, it has some notable drawbacks that make people think of more efficient alternatives. Some challenges include the scalability of gathering human feedback, potential biases introduced by the feedback providers, and the complexity of effectively integrating this feedback into the AI training process."
* **Technical Entities (Classes/Functions/APIs):** None specified

### RLHF vs. DPO
* **Key Points:**
  - "RLHF is a complicated process. You first fit a reward model based on human feedback, then fine-tune the unsupervised language model using RL to maximize the reward score while staying close to the original method. Stanford researchers recently came up with a new parametrization method of the reward model that enables optimal policy extraction in closed form. This allows solving the RLHF problem with only a simple classification loss. The resulting algorithm is called direct preference optimization (DPO) and is computationally lightweight, stable, and performant. DPO eliminates the need for sampling from the language model during fine-tuning or performing significant hyperparameter tuning."
  - "DPO shows some impressive results, setting itself as a method that fine-tunes an LM to fit human feedback as well as or even better than existing methods. Results show that DPO performs better than PPO-based RLHF in terms of controlling the sentiment of generations. In summarization and single-return dialogue tasks, it matches or improves RLHF while being substantially simple to implement and train."
* **Technical Entities (Classes/Functions/APIs):** `DPO` (direct preference optimization), `PPO`

### RLHF vs. RLAIF
* **Key Points:**
  - "Human labor in RLHF training is time-consuming and extensive. That's a significant motivation for a technique that gained popularity last year – RLAIF, which uses a ready-made LLM to mimic the job of human annotators, creating AI-generated preferences instead."
  - "When it comes to tasks like summarization and crafting helpful or non-offensive dialogue, RLAIF keeps up and sometimes races ahead of RLHF. It beats the standard approach of fine-tuning with supervision, impressively doing so with a preference labeler the same size as the policy model it's training."
  - "Intriguingly, simply asking the LLM directly for reward scores can lead to better results than the typical RLAIF approach, which involves turning LLM-generated preferences into a reward model first. Through thoroughly exploring different methods to generate AI preferences that align with human values, the findings hint at RLAIF's capacity to outperform human annotators. This breakthrough points to a way around the tricky issue of scaling RLHF, offering a glimpse into a future where aligning AI with human preferences might not be so daunting after all."
* **Technical Entities (Classes/Functions/APIs):** `RLAIF`

### RLHF vs. ReST
* **Key Points:**
  - "Reinforced self-training (ReST) method offers a twist on the typical RLHF, focusing instead on aligning LLMs more closely with human preferences."
  - "What ReST does differently is it uses a sampling strategy to craft a better training dataset. It picks out high-quality data snippets over several rounds, which helps refine its reward function gradually. The key perk here is that ReST prepares its training set offline, which is a departure from the usual online RLHF methods like those used with proximal policy optimization (PPO) in popular models like InstructGPT or Llama 2."
  - "However, the ReST paper points out a gap—it doesn't directly compare ReST's efficiency to these traditional RLHF PPO methods. So, while ReST seems promising in theory, it's a bit like saying it's potentially more efficient without showing the full homework to prove it matches or outdoes the current standards."
* **Technical Entities (Classes/Functions/APIs):** `ReST` (Reinforced self-training), `PPO`, `InstructGPT`, `Llama 2`

### Fine-grained RLHF
* **Key Points:**
  - "Language models sometimes mess up by creating content that's misleading, harmful, or just plain irrelevant. To make these models better listeners and speakers, researchers have implemented our well-known RLHF."
  - "But, traditional RLHF is like getting a single report card for an entire year's subjects—it's too broad and doesn't pinpoint where the model needs to improve. Fine-grained RLHF is a sophisticated approach that breaks down feedback into more detailed, bite-sized pieces. It's like getting a report card that not only tells you how you did in each subject but also gives you feedback on every assignment and test."
  - "Fine-grained RLHF enables training and learning from reward functions that are fine-grained in two respects: Density, providing a reward after every segment (e.g., a sentence) is generated. Incorporating multiple reward models associated with different feedback types (e.g., factual incorrectness, irrelevance, and information incompleteness)."
  - "You can find all related data, collected human feedback, and codes on GitHub."
  - "Note that RLHF and its alternative approaches are also widely used to train small language models (SLMs). This is because fine-tuning the model on RLHF data is a lot easier, faster and cheaper for smaller models rather than the large ones."
* **Technical Entities (Classes/Functions/APIs):** `Fine-grained RLHF`, `GitHub`, `SLMs`

## Wrapping up
* **Key Points:**
  - "The arrival of language models like GPT-3 opened up a world of possibilities for making AI understand and generate human-like language. But here's the real challenge: fine-tuning AI to grasp the nuances of what we really mean or prefer. RLHF comes in here, blending the best of LLM's learning abilities with the irreplaceable insights from human feedback. It's all about making AI not just smart but also sensitive to our preferences."
  - "RLHF shines by steering AI towards outcomes that resonate more authentically with us, especially in scenarios without clear-cut answers. However, perfecting this approach has its hurdles, like ensuring we can scale up without losing the personal touch or introducing biases. That's why alternatives like direct performance optimization (DPO) and reinforcement learning from AI feedback (RLAIF) are getting attention. DPO simplifies the fine-tuning process, and RLAIF introduces a clever workaround for RLHF's scalability challenge by using AI to simulate human feedback, both showing promising strides toward achieving nuanced AI interactions."
  - "As we explore these paths, the end goal is crystal clear: to evolve artificial intelligence systems that are not only efficient but deeply aligned with human values and thoughts. RLHF's journey and its alternatives showcase our drive towards creating AI that genuinely understands and interacts with us on a human level. It's an exciting time, with each step forward bringing us closer to seamlessly integrated AI-human interactions."
* **Technical Entities (Classes/Functions/APIs):** `GPT-3`, `DPO`, `RLAIF`

## Frequently asked questions
### What is RLHF?
* **Key Points:**
  - "Reinforcement learning with human feedback (RLHF) is a technique where AI improves by learning directly from human feedback."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`

### How is Human Feedback Used in RLHF for Language Models (LLMs)?
* **Key Points:**
  - "RLHF improves language models by using human feedback to rank model responses. These rankings train a reward model, which helps the LLM generate better, more aligned outputs over time."
* **Technical Entities (Classes/Functions/APIs):** None specified

### How does RLHF work?
* **Key Points:**
  - "RLHF (Reinforcement Learning with Human Feedback) works in three steps: first, human feedback is collected to create a preference dataset. Second, a reward model is trained using this dataset. Third, the reward model is used in a reinforcement learning loop to fine-tune a language model, improving its alignment with human preferences."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`

### Why is RLHF important?
* **Key Points:**
  - "RLHF is important because it helps AI systems align better with human values. It trains models to prioritize helpful and socially appropriate responses, making them more human-friendly and ethically aware. RLHF also improves AI safety by reducing harmful or biased outputs and offers a scalable way to enhance AI performance as systems grow more complex."
* **Technical Entities (Classes/Functions/APIs):** None specified

### What is an example of RLHF?
* **Key Points:**
  - "An example of RLHF is using human feedback to improve AI-generated text summaries. Instead of searching for a single 'correct' summary, data trainers compare multiple summaries and select the one they prefer. This preference data trains a reward model, which then fine-tunes a language model to produce outputs more aligned with human expectations. This method can be applied to other data types too, like images, audio, or video, making RLHF a versatile tool for aligning AI with human preferences."
* **Technical Entities (Classes/Functions/APIs):** None specified


---


# Illustrating Reinforcement Learning from Human Feedback (RLHF)


* **Key Points:**
  - "Reinforcement Learning from Human Feedback (RLHF); use methods from reinforcement learning to directly optimize a language model with human feedback. RLHF has enabled language models to begin to align a model trained on a general corpus of text data to that of complex human values."
  - "RLHF's most recent success was its use in ChatGPT."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`, `ChatGPT`

## RLHF: Let's take it step by step
* **Key Points:**
  - "Reinforcement learning from Human Feedback (also referenced as RL from human preferences) is a challenging concept because it involves a multiple-model training process and different stages of deployment. In this blog post, we'll break down the training process into three core steps: Pretraining a language model (LM), gathering data and training a reward model, and fine-tuning the LM with reinforcement learning."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Pretraining language models
* **Key Points:**
  - "As a starting point RLHF use a language model that has already been pretrained with the classical pretraining objectives."
  - "OpenAI used a smaller version of GPT-3 for its first popular RLHF model, InstructGPT. In their shared papers, Anthropic used transformer models from 10 million to 52 billion parameters trained for this task. DeepMind has documented using up to their 280 billion parameter model Gopher."
  - "This initial model can also be fine-tuned on additional text or conditions, but does not necessarily need to be."
  - "Core to starting the RLHF process is having a model that responds well to diverse instructions."
  - "In general, there is not a clear answer on 'which model' is the best for the starting point of RLHF. This will be a common theme in this blog – the design space of options in RLHF training are not thoroughly explored."
* **Technical Entities (Classes/Functions/APIs):** `GPT-3`, `InstructGPT`, `Gopher`

### Reward model training
* **Key Points:**
  - "Generating a reward model (RM, also referred to as a preference model) calibrated with human preferences is where the relatively new research in RLHF begins. The underlying goal is to get a model or system that takes in a sequence of text, and returns a scalar reward which should numerically represent the human preference."
  - "The system can be an end-to-end LM, or a modular system outputting a reward (e.g. a model ranks outputs, and the ranking is converted to reward). The output being a scalar reward is crucial for existing RL algorithms being integrated seamlessly later in the RLHF process."
  - "These LMs for reward modeling can be both another fine-tuned LM or a LM trained from scratch on the preference data."
  - "Anthropic has used a specialized method of fine-tuning to initialize these models after pretraining (preference model pretraining, PMP) because they found it to be more sample efficient than fine-tuning, but no one base model is considered the clear best choice for reward models."
  - "The training dataset of prompt-generation pairs for the RM is generated by sampling a set of prompts from a predefined dataset. The prompts are passed through the initial language model to generate new text."
  - "Human annotators are used to rank the generated text outputs from the LM. One may initially think that humans should apply a scalar score directly to each piece of text in order to generate a reward model, but this is difficult to do in practice. The differing values of humans cause these scores to be uncalibrated and noisy. Instead, rankings are used to compare the outputs of multiple models and create a much better regularized dataset."
  - "One method that has been successful is to have users compare generated text from two language models conditioned on the same prompt. By comparing model outputs in head-to-head matchups, an Elo system can be used to generate a ranking of the models and outputs relative to each-other. These different methods of ranking are normalized into a scalar reward signal for training."
  - "An interesting artifact of this process is that the successful RLHF systems to date have used reward language models with varying sizes relative to the text generation (e.g. OpenAI 175B LM, 6B reward model, Anthropic used LM and reward models from 10B to 52B, DeepMind uses 70B Chinchilla models for both LM and reward). An intuition would be that these preference models need to have similar capacity to understand the text given to them as a model would need in order to generate said text."
* **Technical Entities (Classes/Functions/APIs):** `RM` (reward model), `PMP` (preference model pretraining), `Elo`, `Chinchilla`

### Fine-tuning with RL
* **Key Points:**
  - "Training a language model with reinforcement learning was, for a long time, something that people would have thought as impossible both for engineering and algorithmic reasons. What multiple organizations seem to have gotten to work is fine-tuning some or all of the parameters of a copy of the initial LM with a policy-gradient RL algorithm, Proximal Policy Optimization (PPO)."
  - "Some parameters of the LM are frozen because fine-tuning an entire 10B or 100B+ parameter model is prohibitively expensive (for more, see Low-Rank Adaptation (LoRA) for LMs or the Sparrow LM from DeepMind) -- depending on the scale of the model and infrastructure being used. The exact dynamics of how many parameters to freeze, or not, is considered an open research problem."
  - "PPO has been around for a relatively long time – there are tons of guides on how it works. The relative maturity of this method made it a favorable choice for scaling up to the new application of distributed training for RLHF. It turns out that many of the core RL advancements to do RLHF have been figuring out how to update such a large model with a familiar algorithm."
  - "First, the policy is a language model that takes in a prompt and returns a sequence of text (or just probability distributions over text). The action space of this policy is all the tokens corresponding to the vocabulary of the language model (often on the order of 50k tokens) and the observation space is the distribution of possible input token sequences, which is also quite large given previous uses of RL (the dimension is approximately the size of vocabulary ^ length of the input token sequence). The reward function is a combination of the preference model and a constraint on policy shift."
  - "The reward function is where the system combines all of the models we have discussed into one RLHF process. Given a prompt, x, from the dataset, the text y is generated by the current iteration of the fine-tuned policy. Concatenated with the original prompt, that text is passed to the preference model, which returns a scalar notion of 'preferability', rθ. In addition, per-token probability distributions from the RL policy are compared to the ones from the initial model to compute a penalty on the difference between them. In multiple papers from OpenAI, Anthropic, and DeepMind, this penalty has been designed as a scaled version of the Kullback–Leibler (KL) divergence between these sequences of distributions over tokens, rKL. The KL divergence term penalizes the RL policy from moving substantially away from the initial pretrained model with each training batch, which can be useful to make sure the model outputs reasonably coherent text snippets. Without this penalty the optimization can start to generate text that is gibberish but fools the reward model to give a high reward. In practice, the KL divergence is approximated via sampling from both distributions. The final reward sent to the RL update rule is r = rθ − λrKL."
  - "Some RLHF systems have added additional terms to the reward function. For example, OpenAI experimented successfully on InstructGPT by mixing in additional pre-training gradients (from the human annotation set) into the update rule for PPO. It is likely as RLHF is further investigated, the formulation of this reward function will continue to evolve."
  - "Finally, the update rule is the parameter update from PPO that maximizes the reward metrics in the current batch of data (PPO is on-policy, which means the parameters are only updated with the current batch of prompt-generation pairs). PPO is a trust region optimization algorithm that uses constraints on the gradient to ensure the update step does not destabilize the learning process. DeepMind used a similar reward setup for Gopher but used synchronous advantage actor-critic (A2C) to optimize the gradients, which is notably different but has not been reproduced externally."
  - "Optionally, RLHF can continue from this point by iteratively updating the reward model and the policy together. As the RL policy updates, users can continue ranking these outputs versus the model's earlier versions. Most papers have yet to discuss implementing this operation, as the deployment mode needed to collect this type of data only works for dialogue agents with access to an engaged user base. Anthropic discusses this option as Iterated Online RLHF (see the original paper), where iterations of the policy are included in the ELO ranking system across models. This introduces complex dynamics of the policy and reward model evolving, which represents a complex and open research question."
* **Technical Entities (Classes/Functions/APIs):** `PPO` (Proximal Policy Optimization), `LoRA`, `Sparrow`, `KL divergence`, `A2C` (synchronous advantage actor-critic), `Iterated Online RLHF`

## Open-source tools for RLHF
* **Key Points:**
  - "The first code released to perform RLHF on LMs was from OpenAI in TensorFlow in 2019."
  - "Today, there are already a few active repositories for RLHF in PyTorch that grew out of this. The primary repositories are Transformers Reinforcement Learning (TRL), TRLX which originated as a fork of TRL, and Reinforcement Learning for Language models (RL4LMs)."
  - "TRL is designed to fine-tune pretrained LMs in the Hugging Face ecosystem with PPO."
  - "TRLX is an expanded fork of TRL built by CarperAI to handle larger models for online and offline training. At the moment, TRLX has an API capable of production-ready RLHF with PPO and Implicit Language Q-Learning ILQL at the scales required for LLM deployment (e.g. 33 billion parameters). Future versions of TRLX will allow for language models up to 200B parameters. As such, interfacing with TRLX is optimized for machine learning engineers with experience at this scale."
  - "RL4LMs offers building blocks for fine-tuning and evaluating LLMs with a wide variety of RL algorithms (PPO, NLPO, A2C and TRPO), reward functions and metrics. Moreover, the library is easily customizable, which allows training of any encoder-decoder or encoder transformer-based LM on any arbitrary user-specified reward function. Notably, it is well-tested and benchmarked on a broad range of tasks in recent work amounting up to 2000 experiments highlighting several practical insights on data budget comparison (expert demonstrations vs. reward modeling), handling reward hacking and training instabilities, etc. RL4LMs current plans include distributed training of larger models and new RL algorithms."
  - "There is a large dataset created by Anthropic available on the Hub."
* **Technical Entities (Classes/Functions/APIs):** `TRL` (Transformers Reinforcement Learning), `TRLX`, `RL4LMs` (Reinforcement Learning for Language models), `Hugging Face`, `PPO`, `ILQL` (Implicit Language Q-Learning), `NLPO`, `A2C`, `TRPO`

## What's next for RLHF?
* **Key Points:**
  - "While these techniques are extremely promising and impactful and have caught the attention of the biggest research labs in AI, there are still clear limitations. The models, while better, can still output harmful or factually inaccurate text without any uncertainty. This imperfection represents a long-term challenge and motivation for RLHF – operating in an inherently human problem domain means there will never be a clear final line to cross for the model to be labeled as complete."
  - "When deploying a system using RLHF, gathering the human preference data is quite expensive due to the direct integration of other human workers outside the training loop. RLHF performance is only as good as the quality of its human annotations, which takes on two varieties: human-generated text, such as fine-tuning the initial LM in InstructGPT, and labels of human preferences between model outputs."
  - "Generating well-written human text answering specific prompts is very costly, as it often requires hiring part-time staff (rather than being able to rely on product users or crowdsourcing). Thankfully, the scale of data used in training the reward model for most applications of RLHF (~50k labeled preference samples) is not as expensive. However, it is still a higher cost than academic labs would likely be able to afford. Currently, there only exists one large-scale dataset for RLHF on a general language model (from Anthropic) and a couple of smaller-scale task-specific datasets (such as summarization data from OpenAI). The second challenge of data for RLHF is that human annotators can often disagree, adding a substantial potential variance to the training data without ground truth."
  - "With these limitations, huge swaths of unexplored design options could still enable RLHF to take substantial strides. Many of these fall within the domain of improving the RL optimizer. PPO is a relatively old algorithm, but there are no structural reasons that other algorithms could not offer benefits and permutations on the existing RLHF workflow. One large cost of the feedback portion of fine-tuning the LM policy is that every generated piece of text from the policy needs to be evaluated on the reward model (as it acts like part of the environment in the standard RL framework). To avoid these costly forward passes of a large model, offline RL could be used as a policy optimizer. Recently, new algorithms have emerged, such as implicit language Q-learning (ILQL), that fit particularly well with this type of optimization. Other core trade-offs in the RL process, like exploration-exploitation balance, have also not been documented. Exploring these directions would at least develop a substantial understanding of how RLHF functions and, if not, provide improved performance."
* **Technical Entities (Classes/Functions/APIs):** `PPO`, `ILQL` (implicit language Q-learning)

## Further reading
* **Key Points:**
  - List of prevalent papers on RLHF:
  - "TAMER: Training an Agent Manually via Evaluative Reinforcement (Knox and Stone 2008): Proposed a learned agent where humans provided scores on the actions taken iteratively to learn a reward model."
  - "Interactive Learning from Policy-Dependent Human Feedback (MacGlashan et al. 2017): Proposed an actor-critic algorithm, COACH, where human feedback (both positive and negative) is used to tune the advantage function."
  - "Deep Reinforcement Learning from Human Preferences (Christiano et al. 2017): RLHF applied on preferences between Atari trajectories."
  - "Deep TAMER: Interactive Agent Shaping in High-Dimensional State Spaces (Warnell et al. 2018): Extends the TAMER framework where a deep neural network is used to model the reward prediction."
  - "A Survey of Preference-based Reinforcement Learning Methods (Wirth et al. 2017): Summarizes efforts above with many, many more references."
  - Key LM papers:
  - "Fine-Tuning Language Models from Human Preferences (Zieglar et al. 2019): An early paper that studies the impact of reward learning on four specific tasks."
  - "Learning to summarize with human feedback (Stiennon et al., 2020): RLHF applied to the task of summarizing text. Also, Recursively Summarizing Books with Human Feedback (OpenAI Alignment Team 2021), follow on work summarizing books."
  - "WebGPT: Browser-assisted question-answering with human feedback (OpenAI, 2021): Using RLHF to train an agent to navigate the web."
  - "InstructGPT: Training language models to follow instructions with human feedback (OpenAI Alignment Team 2022): RLHF applied to a general language model."
  - "GopherCite: Teaching language models to support answers with verified quotes (Menick et al. 2022): Train a LM with RLHF to return answers with specific citations."
  - "Sparrow: Improving alignment of dialogue agents via targeted human judgements (Glaese et al. 2022): Fine-tuning a dialogue agent with RLHF."
  - "ChatGPT: Optimizing Language Models for Dialogue (OpenAI 2022): Training a LM with RLHF for suitable use as an all-purpose chat bot."
  - "Scaling Laws for Reward Model Overoptimization (Gao et al. 2022): studies the scaling properties of the learned preference model in RLHF."
  - "Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback (Anthropic, 2022): A detailed documentation of training a LM assistant with RLHF."
  - "Red Teaming Language Models to Reduce Harms: Methods, Scaling Behaviors, and Lessons Learned (Ganguli et al. 2022): A detailed documentation of efforts to 'discover, measure, and attempt to reduce [language models] potentially harmful outputs.'"
  - "Dynamic Planning in Open-Ended Dialogue using Reinforcement Learning (Cohen at al. 2022): Using RL to enhance the conversational skill of an open-ended dialogue agent."
  - "Is Reinforcement Learning (Not) for Natural Language Processing?: Benchmarks, Baselines, and Building Blocks for Natural Language Policy Optimization (Ramamurthy and Ammanabrolu et al. 2022): Discusses the design space of open-source tools in RLHF and proposes a new algorithm NLPO (Natural Language Policy Optimization) as an alternative to PPO."
  - "Llama 2 (Touvron et al. 2023): Impactful open-access model with substantial RLHF details."
* **Technical Entities (Classes/Functions/APIs):** `TAMER`, `COACH`, `PPO`, `NLPO` (Natural Language Policy Optimization), `Llama 2`