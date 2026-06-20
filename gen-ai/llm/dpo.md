---
aliases:
  - Direct Preference Optimization
Source 1: https://www.superannotate.com/blog/direct-preference-optimization-dpo
---
# Direct preference optimization (DPO): Complete overview


* **Key Points:**
  - "A new method called direct preference optimization (DPO) simplifies this process from the beginning."
  - "DPO changes the reward model in RLHF, making the fine-tuning process more stable and straightforward. It does away with the need for repeated sampling and constant adjustments during training."
  - "Tests show that DPO not only matches but often exceeds traditional methods in making LLMs align with human preferences. It's especially good at controlling the tone of outputs and improving the quality of summaries and dialogues while also being much easier to set up and train."
* **Technical Entities (Classes/Functions/APIs):** `DPO` (direct preference optimization), `RLHF`

## What is direct preference optimization (DPO)?
* **Key Points:**
  - "Direct preference optimization (DPO) is a new method that helps large, unsupervised language models better match human preferences using a simple classification approach."
  - "DPO tackles a common problem in AI: training models to generate responses that are aligned to human preferences without getting bogged down by complex methods."
  - "Instead, DPO uses a binary cross-entropy objective to steer models toward producing the types of responses that people actually want to see. This approach cuts out the need for building and tweaking a separate reward model, making the training process more straightforward. Also, DPO generally lowers the amount of computing power needed and simplifies the whole development process."
* **Technical Entities (Classes/Functions/APIs):** `DPO`, `RLHF`

## The traditional approach: RLHF
* **Key Points:**
  - "Language models, especially large ones, are usually pre-trained through self-supervision. This involves feeding them sentences from vast internet text sources, removing the end, and letting the model predict how to complete these sentences."
  - "To steer outputs towards more useful interactions—like maintaining conversational flow or avoiding inappropriate content—models are traditionally trained further with human feedback. The most common method here is RLHF, which involves several steps: Pre-train the base model: Start with the base model trained to autocomplete sentences; Response generation: Let the model generate answer pairs to various prompts; Human feedback: Have humans rank these answers by quality; Reward model: Train a reward model to mimic these human rankings; Lastly, you use feedback from the reward model to encourage high-quality responses while keeping the model's responses close to its original training."
  - "The standard approach with RLHF is constructing the reward function and maximizing it."
  - "Maximizing rewards: The part E[r(φ(x, y))] in the formula is about maximizing the expected reward. This means the model is encouraged to choose responses that earn the highest rewards according to the reward function r, which evaluates how good or suitable the responses y are for the given inputs x."
  - "Minimizing deviation: The term -βD_KL[πθ(y | x) || π_ref(y | x)] is for keeping the model's behavior in check. It calculates the difference, or divergence, between the model's current response strategy (πθ(y | x)) and a reference strategy (π_ref(y | x)). The model is penalized if its strategy strays too far from this reference, helping to ensure that while it learns to improve, it doesn't start producing bizarre or unexpected responses."
  - "The objective of this formula is to fine-tune a model so that it not only improves its responses but also stays within acceptable behavior guidelines. It does this by maximizing rewards for good responses—think of it as giving the model a thumbs-up for each right answer. At the same time, it ensures the model doesn't drift too far from a standard or safe way of responding by penalizing it when it deviates too much. The optimization is typically performed using reinforcement learning, specifically proximal policy optimization (PPO)."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`, `PPO` (proximal policy optimization)

### The problem with RLHF
* **Key Points:**
  - "RLHF helps produce models that can engage in more relevant and appropriate dialogues, but it comes with downsides. It's inherently unstable and costly and requires maintaining a reward model that mimics human judgment—a process prone to errors like reward hacking, where RLHF also requires additional computational resources for maintaining and training separate reward models."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`

## How does DPO work?
* **Key Points:**
  - "Your language model is secretly a reward model. This is the main motto of DPO."
  - "DPO challenges the status quo by eliminating the need for a separate reward model and complex reinforcement learning algorithms. Instead, DPO fine-tunes language models directly using human feedback with a simpler mechanism."
* **Technical Entities (Classes/Functions/APIs):** `DPO`

### DPO loss function: Removing reward model
* **Key Points:**
  - "Here's the loss function for DPO."
  - "πθ(y_u | x) and πθ(y_l | x): These terms represent the probabilities that the model, parameterized by θ, assigns to specific responses (y_u for an upper or preferred response, and y_l for a lower or less preferred response) given an input x."
  - "π_ref(y_u | x) and π_ref(y_l | x): These refer to the probabilities given by a reference policy or model (perhaps a baseline or untrained model) for the same responses to the same input. This helps in gauging how much the current model's predictions deviate from a baseline."
  - "β log(πθ(y_u | x) / π_ref(y_u | x)) - β log(πθ(y_l | x) / π_ref(y_l | x)): This is where the optimization happens. The model updates itself to increase the probability of the preferred response (y_u) relative to the baseline while decreasing the probability of the less preferred response (y_l). The β coefficient here acts as a balancing factor, determining how strongly the model should adhere to or deviate from the reference behavior."
  - "E[...]: This notation indicates that we take the expected value of the inside expression across all possible inputs and outputs. This means we're averaging out the adjustments across all scenarios the model encounters to find the optimal overall behavior."
  - "In simpler terms, the DPO formula helps the model learn by pushing it to favor responses that humans prefer and to avoid those they don't while also keeping the model's outputs within reasonable bounds set by the reference policy."
  - "So, instead of using a reward model to interpret and score these human judgments, DPO applies a direct classification approach. The model is fine-tuned using a binary cross-entropy loss function, which is straightforward yet effective. This function adjusts the model by increasing the probability of generating responses labeled as positive and decreasing the probability of those labeled as negative."
  - "The core innovation in DPO lies in its reparameterization of the loss function used in RLHF. By changing the variables in the loss equation, DPO directly influences the model's policy—essentially, a model's strategy to decide its outputs based on input data. This direct manipulation of the policy through a simple classification loss makes DPO not only easier to implement but also more robust and less prone to the common pitfalls of RLHF, such as reward hacking or drifting too far from the original model's capabilities."
* **Technical Entities (Classes/Functions/APIs):** `DPO`, `RLHF`

## Can DPO scale to a real preference dataset?
* **Key Points:**
  - "Automatic evaluation metrics like ROUGE often fail to align perfectly with human judgments in summarization tasks. Research has shown that language models fine-tuned on human preferences using methods such as PPO can produce more relevant summaries. To see how DPO compares, it was tested alongside PPO and preferred fine-tuning (Preferred-FT) using the TL;DR summarization dataset."
  - "The results showed that DPO achieved a win rate of approximately 61% at a temperature setting of 0.0, slightly higher than PPO's 57% at its optimal temperature. Notably, DPO maintained consistent performance across various temperature settings, showcasing greater stability compared to PPO, which experienced more variability in its results."
* **Technical Entities (Classes/Functions/APIs):** `ROUGE`, `PPO`, `DPO`, `Preferred-FT`

### Single-turn dialogue performance with DPO
* **Key Points:**
  - "The evaluation extended to single-turn dialogues using a segment of the Anthropic HH dataset focused on initial interactions. Here, DPO was benchmarked against several methods, including a reference model trained through Preferred-FT and a 2-shot prompted version of the Pythia-2.8B base model. DPO consistently matched or exceeded the performance of other approaches, including the computationally intensive Best of 128 baseline, which serves as a rough proxy for PPO-level performance."
  - "DPO not only matched the preferred outcomes within the dataset but was the only method to consistently enhance them. It also demonstrated a rapid convergence to optimal performance, which underscores its efficiency and effectiveness."
* **Technical Entities (Classes/Functions/APIs):** `DPO`, `Anthropic HH dataset`, `Preferred-FT`, `Pythia-2.8B`

## Why DPO could change LLM fine-tuning
* **Key Points:**
  - "The significance of DPO lies in its potential to make LLM fine-tuning faster, cheaper, and more stable. By eliminating the need for a separate reward model and sidestepping the complexities of traditional reinforcement learning, DPO offers a cleaner and more direct way to refine language models using real human feedback."
  - "This method is not only simple and effective but also a valuable asset for developers looking to quickly upgrade their language models. Whether the goal is to create more engaging chatbots, enhance customer service interfaces, or build AI-driven applications that respond more intuitively, DPO provides a practical approach. It significantly improves the interaction and responsiveness of these technologies in everyday situations, paving the way for smoother and more efficient user experiences."
* **Technical Entities (Classes/Functions/APIs):** `DPO`

## Conclusion
* **Key Points:**
  - "DPO's introduction marks a significant shift in how we approach language model fine-tuning. It stands out for its efficiency and potential to reduce the technical barriers traditionally associated with improving AI models. As the AI community continues to explore and expand on this method, it could well become a new standard for training language models in various applications."
* **Technical Entities (Classes/Functions/APIs):** `DPO`


