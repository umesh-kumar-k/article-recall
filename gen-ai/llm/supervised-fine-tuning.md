---
aliases:
  - Supervised Fine Tuning
Source 1: https://medium.com/mantisnlp/supervised-fine-tuning-customizing-llms-a2c1edbf22c3
---
# Understanding and Using Supervised Fine-Tuning (SFT) for Language Models

* **Key Points:**
  - "Although pretraining is expensive (i.e., several hundred thousand dollars in compute), fine-tuning an LLM (or performing in-context learning) is cheap in comparison (i.e., several hundred dollars, or less)."
  - "Given that high-quality, pretrained LLMs (e.g., MPT, Falcon, or LLAMA-2) are widely available and free to use (even commercially), we can build a variety of powerful applications by fine-tuning LLMs on relevant tasks."
  - "One of the most widely-used forms of fine-tuning for LLMs within recent AI research is supervised fine-tuning (SFT). This approach curates a dataset of high-quality LLM outputs over which the model is directly fine-tuned using a standard language modeling objective."
  - "SFT is simple/cheap to use and a useful tool for aligning language models, which has made is popular within the open-source LLM research community and beyond."
* **Technical Entities (Classes/Functions/APIs):** `MPT`, `Falcon`, `LLAMA-2`

## Useful Background Information
* **Key Points:**
  - "The training process for language models typically proceeds in three phases; see above. First, we pretrain the language model, which is (by far) the most computationally-expensive step of training. From here, we perform alignment, typically via the three-step framework (see below) with supervised fine-tuning (SFT) and reinforcement learning from human feedback (RLHF)."
  - "SFT and RLHF are computationally cheap compared to pretraining, but they require the curation of a dataset—either of high-quality LLM outputs or human feedback on LLM outputs—which can be difficult and time consuming."
  - "Sometimes we have to do a bit more when applying an LLM to solve a downstream task. In particular, we can further specialize a language model (if needed) either via domain-specific fine-tuning or in-context learning."
  - "Domain-specific fine-tuning simply trains the model further—usually via a language modeling objective, similarly to pretraining/SFT—on data that is relevant to the downstream task, while in-context learning adds extra context or examples into the language model's prompt to be used as context for solving a problem."
  - "A pretrained language model is usually not useful. If we generate output with this model, the results will probably be repetitive and not very helpful. To create a more useful language model, we have to align this model to the desires of the human user. In other words, instead of generating the most likely textual sequence, our language model learns to generate the textual sequence that is desired by a user."
  - "Such alignment, which is accomplished via the three-step framework with SFT and RLHF outlined above, can be used to encourage a variety of behaviors and properties within an LLM."
* **Technical Entities (Classes/Functions/APIs):** `transformers library`, `transformers`

## What is SFT?
* **Key Points:**
  - "Supervised fine-tuning (SFT) is the first training step within the alignment process for LLMs, and it is actually quite simple. First, we need to curate a dataset of high-quality LLM outputs—these are basically just examples of the LLM behaving correctly; see below. Then, we directly fine-tune the model over these examples."
  - "The 'supervised' aspect of fine-tuning comes from the fact that we are collecting a dataset of examples that the model should emulate. Then, the model learns to replicate the style of these examples during fine-tuning."
  - "SFT is not much different from language model pretraining—both pretraining and SFT use next token prediction as their underlying training objective! The main difference arises in the data that is used. During pretraining, we use a massive corpus of raw textual data to train the model. In contrast, SFT uses a supervised dataset of high-quality LLM outputs."
  - "Typically, the next token prediction objective is only applied to the portion of each example that corresponds to the LLM's output."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Where did this come from?
* **Key Points:**
  - "The three-step alignment process—including both SFT and RLHF—was originally proposed by InstructGPT [2] (though it was previously explored for summarization models in [21]), the precursor and sister model to ChatGPT."
  - "Due to the success of both InstructGPT and ChatGPT, this three-step framework has become standardized and quite popular, leading to its use in a variety of subsequent language models (e.g., Sparrow [4] and LLaMA-2 [6]). Alignment via SFT and RLHF is now used heavily in both research and practical applications."
  - "SFT is slightly different than generic fine-tuning. Typically, fine-tuning a deep learning model is done to teach the model how to solve a specific task, but it makes the model more specialized and less generic—the model becomes a 'narrow expert'. The model will likely solve the task on which it is fine-tuned more accurately compared to a generic model, but it may lose its ability to solve other tasks. In contrast, SFT is a core component of aligning language models, including generic foundation models. Because we are fine-tuning the model to emulate a correct style or behavior, rather than to solve a particular task, it does not lose its generic problem solving abilities."
* **Technical Entities (Classes/Functions/APIs):** `InstructGPT`, `ChatGPT`, `Sparrow`, `LLaMA-2`, `GPT`, `BERT`, `LaMDA`, `Codex`, `GOAT`

## Pros and Cons of SFT
* **Key Points:**
  - "SFT is simple to use—the training process and objective are very similar to pretraining. Plus, the approach is highly effective at performing alignment and—relative to pretraining—is computationally cheap (i.e, 100X less expensive, if not more)."
  - "Using SFT alone (i.e., without any RLHF) yields a clear benefit in terms of the model's instruction following capabilities, correctness, coherence, and overall performance."
  - "Creating a dataset. The results of SFT are heavily dependent upon the dataset that we curate. If this dataset contains a diverse set of examples that accurately capture all relevant alignment criteria and characterize the language model's expected output, then SFT is a great approach. However, how can we guarantee that the dataset used for SFT comprehensively captures all of the behaviors that we want to encourage during the alignment process? This can only be guaranteed through careful manual inspection of data, which is i) not scalable and ii) usually expensive."
  - "Recent research has explored automated frameworks of generating datasets for SFT (e.g., self instruct [12]; see above), but there is no guarantee on the quality of data."
  - "Adding RLHF is beneficial. Even after curating a high-quality dataset for SFT, recent research indicates that further benefit can be gained by performing RLHF. In other words, fine-tuning a language model via SFT alone is not enough."
  - "The current consensus within the research community seems to be that the optimal approach to alignment is to i) perform SFT over a moderately-sized dataset of examples with very high quality and ii) invest remaining efforts into curating human preference data for fine-tuning via RLHF."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`

## Using SFT in Practice
### Implementation of SFT
* **Key Points:**
  - "Under the hood, any implementation of SFT will use a next token prediction (also known as standard language modeling) objective."
  - "One of the best tools that we can use for training an LLM with SFT is the transformer reinforcement learning (TRL) Python library, which contains an implementation of SFT that can be used to fine-tune an existing language model with only a few lines of code."
  - "Due to the simplicity, fine-tuning models via SFT has been incredibly popular within the open-source LLM research community."
  - "Beyond this basic definition of SFT, there are a few useful (and more advanced) techniques that we might want to use, such as applying supervision only on model responses (as opposed to the full dialogue or example), augmenting all response examples with shared prompt template, or even adopting a parameter efficient fine-tuning (PEFT) approach (e.g., LoRA [13])."
  - "The SFTTrainer class defined by TRL is adaptable and extensible enough to handle each of these cases."
* **Technical Entities (Classes/Functions/APIs):** `TRL` (transformer reinforcement learning), `transformers library`, `OPT` (Meta's model), `SFTTrainer`, `PEFT`, `LoRA`

### SFT use cases in AI Research
#### InstructGPT
* **Key Points:**
  - "The three part alignment process—including SFT and RLHF—used by most language models was first used by InstructGPT [2], though it was previously explored for text summarization models in [21]. This publication laid the foundation for a lot of recent LLM advancements and contains many valuable insights into the alignment process."
  - "Unlike recent models proposed by OpenAI, the details of InstructGPT's training process and architecture are fully-disclosed within the publication. As such, this model offers massive insight into the creation of powerful language models, and reading the blog posts for ChatGPT and GPT-4 with this added context is much more informative."
* **Technical Entities (Classes/Functions/APIs):** `InstructGPT`, `ChatGPT`, `GPT-4`

#### Imitation models
* **Key Points:**
  - "During the recent explosion of open-source language models that followed the release of LLaMA, SFT was utilized heavily within the imitation learning context."
  - "We could: Start with an open-source base model; Collect a dataset of dialogue sessions from a proprietary language model (e.g., ChatGPT or GPT-4); Train the model (using SFT) over the resulting dataset."
  - "These models (e.g., Alpaca, Koala, and Vicuna) were cheap to train and performed quite well, highlighting that impressive results can be obtained with SFT using relatively minimal compute."
  - "Although early imitation models were later revealed to perform poorly compared to proprietary models, recent variants that are trained over larger imitation datasets (e.g., Orca [15]) perform well. Combining SFT with imitation learning is an cheap and easy way to make a decent LLM."
* **Technical Entities (Classes/Functions/APIs):** `LLaMA`, `ChatGPT`, `GPT-4`, `Alpaca`, `Koala`, `Vicuna`, `Orca`

#### LIMA
* **Key Points:**
  - "Research in imitation learning revealed that using proprietary language models to generate large datasets for SFT is a useful approach. In contrast, parallel research explored whether alignment could be achieved via smaller, carefully curated datasets. In LIMA [16], authors curate a dataset of only 1K examples for SFT, and the resulting model is quite competitive with top open-source and proprietary LLMs."
  - "In this case, the key to success is manual inspection of data to ensure both quality and diversity, which are found to be more important than the raw size of dataset used for SFT."
  - "These results are corroborated by LLaMA-2, where authors find that a moderately-sized dataset with high quality and diversity standards yields the best results for SFT."
* **Technical Entities (Classes/Functions/APIs):** `LIMA`, `LLaMA-2`

#### Open-source alignment
* **Key Points:**
  - "Until the recent proposal of LLaMA-2 (and even afterwards), open-source LLMs were aligned using primarily SFT with minimal RLHF (if any)."
  - "Several variants of the MPT models, as well as the Instruct versions of Falcon and LLaMA are created using SFT over a variety of different datasets (many of which are publicly available on HuggingFace!)."
  - "A variety of the top models are versions of popular base models (e.g., LLaMA-2 or Falcon) that have been fine-tuned via SFT on a mix of different data. Notable examples of this include Platypus, WizardLM, Airoboros, Guanaco, and more."
* **Technical Entities (Classes/Functions/APIs):** `LLaMA-2`, `MPT`, `Falcon`, `HuggingFace`, `Platypus`, `WizardLM`, `Airoboros`, `Guanaco`

## Concluding Remarks
* **Key Points:**
  - "SFT is a powerful tool for AI practitioners, as it can be used to align a language model to certain human-defined objectives in a data-efficient manner. Although further benefit can be achieved via RLHF, SFT is simple to use (i.e., very similar to pretraining), computationally inexpensive, and highly effective."
  - "Such properties have led SFT to be adopted heavily within the open-source LLM research community, where a variety of new models are trained (using SFT) and released nearly every day."
  - "Given access to a high-quality base model (e.g., LLaMA-2), we can efficiently and easily fine-tune these models via SFT to handle a variety of different use cases."
* **Technical Entities (Classes/Functions/APIs):** `RLHF`, `LLaMA-2`


# Supervised Fine-tuning: customizing LLMs

* **Key Points:**
  - "Fine-tuning has emerged as a powerful and effective technique to adapt pre-trained Large Language Models (LLMs) to specific downstream tasks."
  - "Fine-tuning bridges this gap by taking advantage of the general language understanding captured during pre-training and adapting it to a target task through supervised learning."
  - "By fine-tuning a pre-trained model on a task-specific dataset, NLP practitioners can achieve impressive results with significantly less training data and computational resources than training a model from scratch."
  - "Specifically, for Large Language Models, finetuning is crucial, as a retraining step with the whole data is computationally prohibitive."
* **Technical Entities (Classes/Functions/APIs):** `GPT`, `transformers library`, `trl module`

## Supervised fine-tuning (SFT)
* **Key Points:**
  - "Supervised fine-tuning, involves adapting a pre-trained Language Model (LLM) to a specific downstream task using labeled data. In supervised fine-tuning, the finetuning data is collected from a set of responses validated before hand. That's the main difference to the unsupervised techniques, where data is not validated before hand. While LLM training is (usually) unsupervised, Finetuning is (usually) supervised."
  - "During supervised fine-tuning, the pre-trained LLM is fine-tuned on this labeled dataset using supervised learning techniques. The model's weights are adjusted based on the gradients derived from the task-specific loss, which measures the difference between the LLM's predictions and the ground truth labels."
  - "The supervised fine-tuning process allows the model to learn task-specific patterns and nuances present in the labeled data. By adapting its parameters according to the specific data distribution and task requirements, the model becomes specialized in performing well on the target task."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Supervised fine-tuning (SFT) using Transformers Library
* **Key Points:**
  - "Hugging Face's transformers library has become by far the most used library to train and finetune models, including LLMs. Finetuning models has always been one of its core capabilities, and is almost transparently included in the Trainer facilitator class."
  - "However, recently, with the creation of the trl module for Reinforcement Learning, a new class called SFTTrainer was released, which is more specific for Supervised Fine-tuning on LLMs."
* **Technical Entities (Classes/Functions/APIs):** `transformers library`, `Trainer`, `trl module`, `SFTTrainer`

### Finetuning using Trainer class
* **Key Points:**
  - "The Trainer class facilitates the pretraining and finetuning of models, including LLMs. It requires the following arguments: a model, loaded with AutoModelWithLMHead.from_pretrained; TrainingArgs; a train dataset and an evaluation dataset; a Data collator, which applies different transformations to the datasets."
  - "In case of LLMs there is another usage though, which makes it mandatory in this type of training — the masking of random tokens for next-token prediction."
* **Technical Entities (Classes/Functions/APIs):** `Trainer`, `AutoModelWithLMHead.from_pretrained`, `TrainingArgs`, `Data collator`, `Dataset`
* **Code Snippet:**
```python
# Preprocessing the data is a required step before training
```

### Finetuning using `trl` SFTTrainer class
* **Key Points:**
  - "There is another class, called SFTTrainer, which was later on included in Hugging Face's trl library dedicated to Reinforcement Learning. As Supervised finetuning is the first step in Reinforcement Learning by Human Feedback (RLHF), the developers found a motivation for it to be isolated in its own class, adding at the same time some facilitating functions you needed to do manually if just using the Trainer library."
  - "The SFTTrainer class inherits from the Trainer function, if you check the source code. Basically, same model, train_dataset, evaluation dataset and collator are required."
  - "Support of peft: In SFTTrainer there is support of the Parameter Efficient Finetuning library, which includes Lora, QLora, etc. Lora allows you to add Adapters with weights, which are the only parameters being fine-tuned, while freezing the rest. QLora is the quantized version. Both approaches improve a lot the finetuning time, which is specially relevant when finetuning LLMs because of the big computational cost."
  - "Batch packing: Instead of using the Tokenizer to pad your sentences to the max length supported by your model, packing allows you to combine inputs one after another which increases batch capabilities."
* **Technical Entities (Classes/Functions/APIs):** `trl library`, `SFTTrainer`, `Trainer`, `peft`, `Lora`, `QLora`

## Conclusions
* **Key Points:**
  - "Fine-tuning has historically been a mechanism to get the best of Transformers architectures. Either if you need to train a Language Model for the next token prediction and/or masking, or if you want to train a sequence, token classifier (among others), finetuning is carried out in a supervised way, since it requires data supervised by human labellers."
  - "In case of Large Language Models, as they are trained with large amounts of open / available data, customize their answers to your needs is usually necessary."
  - "There are many ways you can finetune a Large Language Model. One of the easiest options is the Trainer class from the transformers library, which has been used for quite some time now to finetune any kind of transformer-based model."
  - "Recently, a new library called trl was released by Hugging Face to carry out the training of Reinforcement Learning by Human Feedback. One of the main steps of such a training is precisely Supervised Fine-tuning, so they made available a new class called SFTTrainer which manages the process by also providing parameter-efficient (peft) and packing optimizations."
* **Technical Entities (Classes/Functions/APIs):** `Transformers`, `Trainer`, `trl library`, `SFTTrainer`, `peft`
