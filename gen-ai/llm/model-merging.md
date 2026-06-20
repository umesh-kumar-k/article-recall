---
aliases:
  - Model Merging
Source 1: https://huggingface.co/blog/mlabonne/merge-models
---
# Merge Large Language Models with mergekit

* **Key Points:**
  - "Model merging is a technique that combines two or more LLMs into a single model. It's a relatively new and experimental method to create new models for cheap (no GPU required). Model merging works surprisingly well and produced many state-of-the-art models on the Open LLM Leaderboard."
  - "In this tutorial, we will implement it using the mergekit library. More specifically, we will review four merge methods and provide examples of configurations. Then, we will use mergekit to create our own model, Marcoro14-7B-slerp, which became the best-performing model on the Open LLM Leaderboard (02/01/24)."
* **Technical Entities (Classes/Functions/APIs):** `mergekit`, `Marcoro14-7B-slerp`, `Open LLM Leaderboard`, `GitHub`, `Google Colab`, `LazyMergekit`

## 🤝 Merge algorithms
* **Key Points:**
  - "In this section, we will focus on four methods currently implemented in mergekit. Note that there are other methods, such as linear and Task Arithmetic."
  - "If you're interested in papers on model merging, I recommend this excellent collection on Hugging Face."
* **Technical Entities (Classes/Functions/APIs):** `mergekit`, `Hugging Face`

### 1. SLERP
* **Key Points:**
  - "Spherical Linear Interpolation (SLERP) is a method used to smoothly interpolate between two vectors. It maintains a constant rate of change and preserves the geometric properties of the spherical space in which the vectors reside."
  - "There are several reasons to prefer SLERP over a traditional linear interpolation. For example, in high-dimensional spaces, linear interpolation can lead to a decrease in the magnitude of the interpolated vector (i.e., it reduces the scale of weights). Moreover, the change in direction of the weights often represents more meaningful information (like feature learning and representation) than the magnitude of change."
  - "SLERP is currently the most popular merging method, but it is limited to combining only two models at a time. It is still possible to hierarchically combine multiple models, as shown in Mistral-7B-Merge-14-v0.1."
* **Technical Entities (Classes/Functions/APIs):** `SLERP` (Spherical Linear Interpolation), `Mistral-7B-Merge-14-v0.1`
* **Code Snippet:**
```yaml
slices:
  - sources:
      - model: OpenPipe/mistral-ft-optimized-1218
        layer_range: [0, 32]
      - model: mlabonne/NeuralHermes-2.5-Mistral-7B
        layer_range: [0, 32]
merge_method: slerp
base_model: OpenPipe/mistral-ft-optimized-1218
parameters:
  t:
    - filter: self_attn
      value: [0, 0.5, 0.3, 0.7, 1]
    - filter: mlp
      value: [1, 0.5, 0.7, 0.3, 0]
    - value: 0.5
dtype: bfloat16
```

### 2. TIES
* **Key Points:**
  - "Introduced in this paper by Yadav et al., TIES-Merging is designed to efficiently merge multiple task-specific models into a single multitask model."
  - "Redundancy in model parameters: It identifies and eliminates redundant parameters within task-specific models. This is achieved by focusing on the changes made during fine-tuning, identifying the top-k% most significant changes, and discarding the rest."
  - "Disagreement between parameter signs: Conflicts arise when different models suggest opposing adjustments to the same parameter. TIES-Merging resolves these conflicts by creating a unified sign vector that represents the most dominant direction of change across all models."
  - "TIES-Merging is divided into the following three steps: Trim: Reduces redundancy in task-specific models by retaining only a fraction the most significant parameters (density parameter) and resetting the rest to zero. Elect Sign: Resolves sign conflicts across different models by creating a unified sign vector based on the most dominant direction (positive or negative) in terms of cumulative magnitude. Disjoint Merge: Averages parameter values that align with the unified sign vector, excluding zero values."
  - "Unlike SLERP, TIES can merge multiple models at a time."
* **Technical Entities (Classes/Functions/APIs):** `TIES-Merging`
* **Code Snippet:**
```yaml
models:
  - model: mistralai/Mistral-7B-v0.1
    # no parameters necessary for base model
  - model: OpenPipe/mistral-ft-optimized-1218
    parameters:
      density: 0.5
      weight: 0.5
  - model: mlabonne/NeuralHermes-2.5-Mistral-7B
    parameters:
      density: 0.5
      weight: 0.3
merge_method: ties
base_model: mistralai/Mistral-7B-v0.1
parameters:
  normalize: true
dtype: float16
```

### 3. DARE
* **Key Points:**
  - "Introduced by Yu et al. (2023), DARE uses an approach similar to TIES with two main differences: Pruning: DARE randomly reset fine-tuned weights to their original values (those of the base model). Rescaling: DARE rescales the weights to keep the expectations of model outputs approximately unchanged. It adds the rescaled weights of both (or more) models to the weights of the base model with a scale factor."
  - "Mergekit's implementation of this method has two flavours: with the sign election step of TIES (dare_ties) or without (dare_linear)."
* **Technical Entities (Classes/Functions/APIs):** `DARE`, `dare_ties`, `dare_linear`
* **Code Snippet:**
```yaml
models:
  - model: mistralai/Mistral-7B-v0.1
    # No parameters necessary for base model
  - model: samir-fama/SamirGPT-v1
    parameters:
      density: 0.53
      weight: 0.4
  - model: abacusai/Slerp-CM-mist-dpo
    parameters:
      density: 0.53
      weight: 0.3
  - model: EmbeddedLLM/Mistral-7B-Merge-14-v0.2
    parameters:
      density: 0.53
      weight: 0.3
merge_method: dare_ties
base_model: mistralai/Mistral-7B-v0.1
parameters:
  int8_mask: true
dtype: bfloat16
```

### 4. Passthrough
* **Key Points:**
  - "The passthrough method differs significantly from the previous ones. By concatenating layers from different LLMs, it can produce models with an exotic number of parameters (e.g., 9B with two 7B parameter models). These models are often referred to as 'frankenmerges' or 'Frankenstein models' by the community."
  - "This technique is very experimental, but it managed to create impressive models, like goliath-120b using two Llama 2 70B models. The recently released SOLAR-10.7B-v1.0 also uses the same idea, called depth-up scaling in their paper."
* **Technical Entities (Classes/Functions/APIs):** `passthrough`, `goliath-120b`, `SOLAR-10.7B-v1.0`
* **Code Snippet:**
```yaml
slices:
  - sources:
    - model: OpenPipe/mistral-ft-optimized-1218
      layer_range: [0, 32]
  - sources:
    - model: mlabonne/NeuralHermes-2.5-Mistral-7B
      layer_range: [24, 32]
merge_method: passthrough
dtype: bfloat16
```

## 💻 Merge your own models
* **Key Points:**
  - "In this section, we will use mergekit to load a merge configuration, run it, and upload the resulting model to the Hugging Face Hub."
* **Technical Entities (Classes/Functions/APIs):** `mergekit`, `Hugging Face Hub`
* **Code Snippet:**
```python
!git clone https://github.com/cg123/mergekit.git
!cd mergekit && pip install -q -e .
```

```python
import yaml

MODEL_NAME = "Marcoro14-7B-slerp"
yaml_config = """
slices:
  - sources:
      - model: AIDC-ai-business/Marcoroni-7B-v3
        layer_range: [0, 32]
      - model: EmbeddedLLM/Mistral-7B-Merge-14-v0.1
        layer_range: [0, 32]
merge_method: slerp
base_model: AIDC-ai-business/Marcoroni-7B-v3
parameters:
  t:
    - filter: self_attn
      value: [0, 0.5, 0.3, 0.7, 1]
    - filter: mlp
      value: [1, 0.5, 0.7, 0.3, 0]
    - value: 0.5
dtype: bfloat16
"""

# Save config as yaml file
with open('config.yaml', 'w', encoding="utf-8") as f:
    f.write(yaml_config)
```

```python
# Merge models
!mergekit-yaml config.yaml merge --copy-tokenizer --allow-crimes --out-shard-size 1B --lazy-unpickle
```

```python
!pip install -qU huggingface_hub

from huggingface_hub import ModelCard, ModelCardData
from jinja2 import Template

username = "mlabonne"

template_text = """
---
license: apache-2.0
tags:
- merge
- mergekit
- lazymergekit
{%- for model in models %}
- {{ model }}
{%- endfor %}
---

# {{ model_name }}

{{ model_name }} is a merge of the following models using [mergekit](https://github.com/cg123/mergekit):

{%- for model in models %}
* [{{ model }}](https://huggingface.co/{{ model }})
{%- endfor %}

## 🧩 Configuration

\```yaml
{{- yaml_config -}}
\```
"""

# Create a Jinja template object
jinja_template = Template(template_text.strip())

# Get list of models from config
data = yaml.safe_load(yaml_config)
if "models" in data:
    models = [data["models"][i]["model"] for i in range(len(data["models"])) if "parameters" in data["models"][i]]
elif "parameters" in data:
    models = [data["slices"][0]["sources"][i]["model"] for i in range(len(data["slices"][0]["sources"]))]
elif "slices" in data:
    models = [data["slices"][i]["sources"][0]["model"] for i in range(len(data["slices"]))]
else:
    raise Exception("No models or slices found in yaml config")

# Fill the template
content = jinja_template.render(
    model_name=MODEL_NAME,
    models=models,
    yaml_config=yaml_config,
    username=username,
)

# Save the model card
card = ModelCard(content)
card.save('merge/README.md')
```

```python
from google.colab import userdata
from huggingface_hub import HfApi

username = "mlabonne"

# Defined in the secrets tab in Google Colab
api = HfApi(token=userdata.get("HF_TOKEN"))

api.create_repo(
    repo_id=f"{username}/{MODEL_NAME}",
    repo_type="model"
)
api.upload_folder(
    repo_id=f"{username}/{MODEL_NAME}",
    folder_path="merge",
)
```

```python
!pip install -qU transformers accelerate

from transformers import AutoTokenizer
import transformers
import torch

model = "mlabonne/Marcoro14-7B-slerp"
messages = [{"role": "user", "content": "What is a large language model?"}]

tokenizer = AutoTokenizer.from_pretrained(model)
prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
pipeline = transformers.pipeline(
    "text-generation",
    model=model,
    torch_dtype=torch.float16,
    device_map="auto",
)

outputs = pipeline(prompt, max_new_tokens=256, do_sample=True, temperature=0.7, top_k=50, top_p=0.95)
print(outputs[0]["generated_text"])
```

## Conclusion
* **Key Points:**
  - "In this article, we introduced the concept of merging LLMs with four different methods. We detailed how SLERP, TIES, DARE, and passthrough work and provided examples of configurations. Finally, we ran SLERP with mergekit to create Marcoro14-7B-slerp and upload it to the Hugging Face Hub. We obtained excellent performance on two benchmark suites: Open LLM Leaderboard (best-performing 7B model) and NousResearch."
  - "If you want to create your own merges, I recommend using my automated notebook 🥱 LazyMergekit."
* **Technical Entities (Classes/Functions/APIs):** `SLERP`, `TIES`, `DARE`, `passthrough`, `mergekit`, `Marcoro14-7B-slerp`, `Hugging Face Hub`, `Open LLM Leaderboard`, `NousResearch`, `LazyMergekit`