---
aliases:
  - Inference Optimization
Source 1: https://letsbuildsolutions.com/blog/ai-ml/how-llm-inference-engines-work-kv-caches-paged-attention-and-continuous-batching/
Source 2: https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/
---

# Mastering LLM Techniques: Inference Optimization


## Understanding LLM inference
* **Key Points:**
  - Most of the popular decoder-only LLMs (GPT-3, for example) are pretrained on the causal modeling objective, essentially as next-word predictors. These LLMs take a series of tokens as inputs, and generate subsequent tokens autoregressively until they meet a stopping criteria (a limit on the number of tokens to generate or a list of stop words, for example) or until it generates a special <end> token marking the end of generation.
  - This process involves two phases: the prefill phase and the decode phase.
  - In the prefill phase, the LLM processes the input tokens to compute the intermediate states (keys and values), which are used to generate the "first" new token. Each new token depends on all the previous tokens, but because the full extent of the input is known, at a high level this is a matrix-matrix operation that's highly parallelized. It effectively saturates GPU utilization.
  - In the decode phase, the LLM generates output tokens autoregressively one at a time, until a stopping criteria is met. Each sequential output token needs to know all the previous iterations' output states (keys and values). This is like a matrix-vector operation that underutilizes the GPU compute ability compared to the prefill phase. The speed at which the data (weights, keys, values, activations) is transferred to the GPU from memory dominates the latency, not how fast the computation actually happens. In other words, this is a memory-bound operation.
  - Many of the inference challenges and corresponding solutions featured in this post concern the optimization of this decode phase: efficient attention modules, managing the keys and values effectively, and others.
  - Different LLMs may use different tokenizers, and thus, comparing output tokens between them may not be straightforward. When comparing inference throughput, even if two LLMs have similar tokens per second output, they may not be equivalent if they use different tokenizers. This is because corresponding tokens may represent a different number of characters.

## Batching
* **Key Points:**
  - The simplest way to improve GPU utilization, and effectively throughput, is through batching. Since multiple requests use the same model, the memory cost of the weights is spread out. Larger batches getting transferred to the GPU to be processed all at once will leverage more of the compute available.
  - Batch sizes, however, can only be increased up to a certain limit, at which point they may lead to a memory overflow. To better understand why this happens requires looking at key-value (KV) caching and LLM memory requirements.
  - Traditional batching (also called static batching) is suboptimal. This is because for each request in a batch, the LLM may generate a different number of completion tokens, and subsequently they have different execution times. As a result, all requests in the batch must wait until the longest request is finished, which can be exacerbated by a large variance in the generation lengths. There are methods to mitigate this, such as in-flight batching.
  - For example, open-source runtimes like NVIDIA TensorRT-LLM have in-flight batching and related scheduling optimizations for popular open models (for example, Llama and Nemotron). This does not require writing custom schedulers or CUDA kernels.
* **Technical Entities (Classes/Functions/APIs):** `NVIDIA TensorRT-LLM`, `Llama`, `Nemotron`

## Key-value caching
* **Key Points:**
  - One common optimization for the decode phase is KV caching. The decode phase generates a single token at each time step, but each token depends on the key and value tensors of all previous tokens (including the input tokens' KV tensors computed at prefill, and any new KV tensors computed until the current time step).
  - To avoid recomputing all these tensors for all tokens at each time step, it's possible to cache them in GPU memory. Every iteration, when new elements are computed, they are simply added to the running cache to be used in the next iteration. In some implementations, there is one KV cache for each layer of the model.

## LLM memory requirement
* **Key Points:**
  - In effect, the two main contributors to the GPU LLM memory requirement are model weights and the KV cache.
  - Model weights: Memory is occupied by the model parameters. As an example, a model with 7 billion parameters (such as Llama 2 7B), loaded in 16-bit precision (FP16 or BF16) would take roughly 7B * sizeof(FP16) ~= 14 GB in memory.
  - KV caching: Memory is occupied by the caching of self-attention tensors to avoid redundant computation.
  - With batching, the KV cache of each of the requests in the batch must still be allocated separately, and can have a large memory footprint.
  - Size of KV cache per token in bytes = 2 * (num_layers) * (num_heads * dim_head) *  precision_in_bytes
  - The first factor of 2 accounts for the K and V matrices. Commonly, the value of (num_heads * dim_head) is the same as the hidden_size (or dimension of the model, d_model) of the transformer. These model attributes are commonly found in model cards or associated config files.
  - This memory size is required for each token in the input sequence, across the batch of inputs. Assuming half-precision, the total size of KV cache is given by the formula below.
  - Total size of KV cache in bytes = (batch_size) * (sequence_length) * 2 * (num_layers) * (hidden_size) *  sizeof(FP16)
  - For example, with a Llama 2 7B model in 16-bit precision and a batch size of 1, the size of the KV cache will be 1 * 4096 * 2 * 32 * 4096 * 2 bytes, which is ~2 GB.
  - Managing this KV cache efficiently is a challenging endeavor. Growing linearly with batch size and sequence length, the memory requirement can quickly scale. Consequently, it limits the throughput that can be served, and poses challenges for long-context inputs. This is the motivation behind several optimizations featured in this post.
* **Technical Entities (Classes/Functions/APIs):** `Llama 2 7B`, `FP16`, `BF16`

## Scaling up LLMs with model parallelization
* **Key Points:**
  - One way to reduce the per-device memory footprint of the model weights is to distribute the model over several GPUs. Spreading the memory and compute footprint enables running larger models, or larger batches of inputs. Model parallelization is a necessity to train or infer on a model requiring more memory than available on a single device, and to make training times and inference measures (latency or throughput) suitable for certain use cases. There are several ways of parallelizing the model based on how the model weights are split.
  - Note that data parallelism is also a technique often mentioned in the same context as the others listed below. In this, weights of the model are copied over multiple devices, and the (global) batch size of inputs is sharded across each of the devices into microbatches. It reduces the overall execution time by processing larger batches. However, it is a training time optimization that is less relevant during inference.
  - Note that any model-parallel techniques—including pipeline and tensor parallelism—are available in open frameworks such as NVIDIA Megatron-LM and the NVIDIA NeMo framework, which underpin training and inference workflows for a wide range of open models.
* **Technical Entities (Classes/Functions/APIs):** `NVIDIA Megatron-LM`, `NVIDIA NeMo`

### Pipeline parallelism
* **Key Points:**
  - Pipeline parallelism involves sharding the model (vertically) into chunks, where each chunk comprises a subset of layers that is executed on a separate device.
  - The main limitation of this method is that, due to the sequential nature of the processing, some devices or layers may remain idle while waiting for the output (activations, gradients) of previous layers. This results in inefficiencies or "pipeline bubbles" in both the forward and backward passes.
  - Microbatching can mitigate this to some extent. The global batch size of inputs is split into sub-batches, which are processed one by one, with gradients being accumulated at the end. This approach shrinks the size of pipeline bubbles, but it does not completely eliminate them.
* **Technical Entities (Classes/Functions/APIs):** `GPipe: Easy Scaling with Micro-Batch Pipeline Parallelism`

### Tensor parallelism
* **Key Points:**
  - Tensor parallelism involves sharding (horizontally) individual layers of the model into smaller, independent blocks of computation that can be executed on different devices. Attention blocks and multi-layer perceptron (MLP) layers are major components of transformers that can take advantage of tensor parallelism. In multi-head attention blocks, each head or group of heads can be assigned to a different device so they can be computed independently and in parallel.
* **Technical Entities (Classes/Functions/APIs):** `Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism`

### Sequence parallelism
* **Key Points:**
  - Tensor parallelism has limitations, as it requires layers to be divided into independent, manageable blocks. It's not applicable to operations like LayerNorm and Dropout, which are instead replicated across the tensor-parallel group. While LayerNorm and Dropout are computationally inexpensive, they do require a considerable amount of memory to store (redundant) activations.
  - As shown in Reducing Activation Recomputation in Large Transformer Models, these operations are independent across the input sequence, and these ops can be partitioned along that "sequence-dimension," making them more memory efficient. This is called sequence parallelism.
* **Technical Entities (Classes/Functions/APIs):** `Reducing Activation Recomputation in Large Transformer Models`

## Optimizing the attention mechanism
* **Key Points:**
  - The scaled dot-product attention (SDPA) operation maps query and key-value pairs to an output, as described in Attention Is All You Need.
* **Technical Entities (Classes/Functions/APIs):** `Attention Is All You Need`

### Multi-head attention
* **Key Points:**
  - As an enhancement to the SDPA, executing the attention layer multiple times in parallel with different, learned projections of the Q, K, and V matrices, enables the model to jointly attend to information from different representational subspaces at different positions. These subspaces are learned independently, providing the model with a richer understanding of different positions in the input.
  - As depicted in Figure 5, the outputs from the multiple parallel attention operations are concatenated and linearly projected to combine them. Each parallel attention layer is called a 'head,' and this approach is called multi-head attention (MHA).
  - In the original work, each attention head operates on a reduced dimension of the model (such as 𝑑𝑚𝑜𝑑𝑒𝑙/8) when using eight parallel attention heads. This keeps the computational cost similar to single-head attention.
* **Technical Entities (Classes/Functions/APIs):** `MHA`, `scaled dot-product attention (SDPA)`

### Multi-query attention
* **Key Points:**
  - One of the inference optimizations to MHA, called multi-query attention (MQA), as proposed in Fast Transformer Decoding, shares the keys and values among the multiple attention heads. The query vector is still projected multiple times, as before.
  - While the amount of computation done in MQA is identical to MHA, the amount of data (keys, values) read from memory is a fraction of before. When bound by memory-bandwidth, this enables better compute utilization. It also reduces the size of the KV-cache in memory, allowing space for larger batch sizes.
  - The reduction in key-value heads comes with a potential accuracy drop. Additionally, models that need to leverage this optimization at inference need to train (or at least fine-tuned with ~5% of training volume) with MQA enabled.
* **Technical Entities (Classes/Functions/APIs):** `MQA`, `Fast Transformer Decoding`

### Grouped-query attention
* **Key Points:**
  - Grouped-query attention (GQA) strikes a balance between MHA and MQA by projecting key and values to a few groups of query heads. Within each of the groups, it behaves like multi-query attention.
  - Models originally trained with MHA, can be "uptrained" with GQA using a fraction of the original training compute. They attain quality close to MHA while maintaining a computational efficiency closer to MQA. Llama 2 70B is an example of a model that leverages GQA.
  - Optimizations like MQA and GQA help reduce the memory required by KV caches by reducing the number of key and value heads that are stored. There may still be inefficiencies in how this KV cache is managed. Of a different flavor than optimizing the attention module itself, the next section presents a technique for more efficient KV cache management.
* **Technical Entities (Classes/Functions/APIs):** `GQA`, `Llama 2 70B`, `GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints`

### Flash attention
* **Key Points:**
  - Another way of optimizing the attention mechanism is to modify the ordering of certain computations to take better advantage of the memory hierarchy of GPUs. Neural networks are generally described in terms of layers, and most implementations are laid out that way as well, with one kind of computation done on the input data at a time in sequence. This doesn't always lead to optimal performance, since it can be beneficial to do more calculations on values that have already been brought into the higher, more performant levels of the memory hierarchy.
  - Fusing multiple layers together during the actual computation can enable minimizing the number of times the GPU needs to read from and write to its memory and to group together calculations that require the same data, even if they are parts of different layers in the neural network.
  - One very popular fusion is FlashAttention, an I/O aware exact attention algorithm, as detailed in FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness. Exact attention means that it is mathematically identical to the standard multi-head attention (with variants available for multi-query and grouped-query attention), and so can be swapped into an existing model architecture or even an already-trained model with no modifications.
  - I/O aware means it takes into account some of the memory movement costs previously discussed when fusing operations together. In particular, FlashAttention uses "tiling" to fully compute and write out a small part of the final matrix at once, rather than doing part of the computation on the whole matrix in steps, writing out the intermediate values in between.
* **Technical Entities (Classes/Functions/APIs):** `FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness`, `FlashAttention`

### Efficient management of KV cache with paging
* **Key Points:**
  - At times, KV caches are statically "over-provisioned" to account for the largest possible input (the supported sequence length) because the size of inputs is unpredictable. For example, if the supported maximum sequence length of a model is 2,048, then regardless of the size of input and the generated output in a request, a reservation of size 2,048 would be made in memory. This space may be contiguously allocated, and often, much of it remains unused, leading to memory waste or fragmentation. This reserved space is tied up for the lifetime of the request.
  - Inspired by paging in operating systems, the PagedAttention algorithm enables storing continuous keys and values in noncontiguous space in memory. It partitions the KV cache of each request into blocks representing a fixed number of tokens, which can be stored non-contiguously.
  - These blocks are fetched as required during attention computation using a block table that keeps account. As new tokens are generated, new block allocations are made. The size of these blocks is fixed, eliminating inefficiencies arising from challenges like different requests requiring different allocations. This significantly limits memory wastage, enabling larger batch sizes (and, consequently, throughput).
* **Technical Entities (Classes/Functions/APIs):** `PagedAttention`, `Efficient Memory Management for Large Language Model Serving with PagedAttention`

## Model optimization techniques
* **Key Points:**
  - So far, we've discussed the different ways LLMs consume memory, some of the ways memory can be distributed across several different GPUs, and optimizing the attention mechanism and KV cache. There are also several model optimization techniques to reduce the memory use on each GPU by making modifications to the model weights themselves. GPUs also have dedicated hardware for accelerating operations on these modified values, providing even more speedups for models.

### Quantization
* **Key Points:**
  - Quantization is the process of reducing the precision of a model's weights and activations. Most models are trained with 32 or 16 bits of precision, where each parameter and activation element takes up 32 or 16 bits of memory—a single-precision floating point. However, most deep learning models can be effectively represented with eight or even fewer bits per value.
  - Reducing the precision of a model can yield several benefits. If the model takes up less space in memory, you can fit larger models on the same amount of hardware. Quantization also means you can transfer more parameters over the same amount of bandwidth, which can help to accelerate models that are bandwidth-limited.
  - There are many different quantization techniques for LLMs involving reduced precision on either the activations, the weights, or both. It's much more straightforward to quantize the weights because they are fixed after training. However, this can leave some performance on the table because the activations remain at higher precisions. GPUs don't have dedicated hardware for multiplying INT8 and FP16 numbers, so the weights must be converted back into a higher precision for the actual operations.
  - It's also possible to quantize the activations, the inputs of transformer blocks and network layers, but this comes with its own challenges. Activation vectors often contain outliers, effectively increasing their dynamic range and making it more challenging to represent these values at a lower precision than with the weights.
  - One option is to find out where those outliers are likely to show up by passing a representative dataset through the model, and choosing to represent certain activations at a higher precision than others (LLM.int8()). Another option is to borrow the dynamic range of the weights, which are easy to quantize, and reuse that range in the activations.
* **Technical Entities (Classes/Functions/APIs):** `LLM.int8()`, `INT8`, `FP16`

### Sparsity
* **Key Points:**
  - Similar to quantization, it's been shown that many deep learning models are robust to pruning, or replacing certain values that are close to 0 with 0 itself. Sparse matrices are matrices where many of the elements are 0. These can be expressed in a condensed form that takes up less space than a full, dense matrix.
  - GPUs in particular have hardware acceleration for a certain kind of structured sparsity, where two out of every four values are represented by zeros. Sparse representations can also be combined with quantization to achieve even greater speedups in execution. Finding the best way to represent large language models in a sparse format is still an active area of research, and offers a promising direction for future improvements to inference speeds.

### Distillation
* **Key Points:**
  - Another approach to shrinking the size of a model is to transfer its knowledge to a smaller model through a process called distillation. This process involves training a smaller model (called a student) to mimic the behavior of a larger model (a teacher).
  - Successful examples of distilled models include DistilBERT, which compresses a BERT model by 40% while retaining 97% of its language understanding capabilities at a speed 60% faster.
  - While distillation in LLMs is an active field of research, the general approach was first described for neural networks in Distilling the Knowledge in a Neural Network:
    - The student network is trained to mirror the performance of a larger teacher network, using a loss function that measures the discrepancy between their outputs. This objective is in addition to potentially including the original loss function of matching the student's outputs with the ground-truth labels.
    - The teacher's outputs that are matched can be the very last layer (called logits) or intermediate layer activations.
  - An alternative approach to distillation is to use data synthesized by the teacher for supervised training of a student LLM, which is especially useful when human annotations are scarce or not available. Distilling Step by Step! goes one step further by extracting rationales from a teacher LLM in addition to the labels that serve as ground truth. These rationales serve as intermediate reasoning steps to train smaller student LLMs in a data-efficient way.
  - It's important to note that many state-of-the-art LLMs today have restrictive licenses that prohibit using their outputs to train other LLMs, making it challenging to find a suitable teacher model.
* **Technical Entities (Classes/Functions/APIs):** `DistilBERT`, `BERT`, `Distilling the Knowledge in a Neural Network`, `Distilling Step by Step!`

## Model serving techniques
* **Key Points:**
  - Model execution is frequently memory-bandwidth bound—in particular, bandwidth-bound in the weights. Even after applying all the model optimizations previously described, it's still very likely to be memory bound. So you want to do as much as possible with your model weights when they are loaded. In other words, try doing things in parallel. Two approaches can be taken:
    - In-flight batching involves executing multiple different requests at the same time.
    - Speculative inference involves executing multiple different steps of the sequence in parallel to try to save time.

### In-flight batching
* **Key Points:**
  - LLMs have some unique execution characteristics that can make it difficult to effectively batch requests in practice. A single model can be used simultaneously for a variety of tasks that look very different from one another. From a simple question-and-answer response in a chatbot to the summarization of a document or the generation of a long chunk of code, workloads are highly dynamic, with outputs varying in size by several orders of magnitude.
  - This versatility can make it challenging to batch requests and execute them in parallel effectively—a common optimization for serving neural networks. This could result in some requests finishing much earlier than others.
  - To manage these dynamic loads, many LLM serving solutions include an optimized scheduling technique called continuous or in-flight batching. This takes advantage of the fact that the overall text generation process for an LLM can be broken down into multiple iterations of execution on the model.
  - With in-flight batching, rather than waiting for the whole batch to finish before moving on to the next set of requests, the server runtime immediately evicts finished sequences from the batch. It then begins executing new requests while other requests are still in flight. In-flight batching can therefore greatly increase the overall GPU utilization in real-world use cases.

### Speculative inference
* **Key Points:**
  - Also known as speculative sampling, assisted generation, or blockwise parallel decoding, speculative inference is a different way of parallelizing the execution of LLMs. Normally, GPT-style large language models are autoregressive models that generate text token by token.
  - Every token that is generated relies on all of the tokens that come before it to provide context. This means that in regular execution, it's impossible to generate multiple tokens from the same sequence in parallel—you have to wait for the nth token to be generated before you can generate n+1.
  - Speculative sampling offers a workaround. The basic idea of this approach is to use some "cheaper" process to generate a draft continuation that is several tokens long. Then, execute the main "verification" model at multiple steps in parallel, using the cheap draft as "speculative" context for the execution steps where it is needed.
  - If the verification model generates the same tokens as the draft, then you know to accept those tokens for the output. Otherwise, you can throw out everything after the first non-matching token, and repeat the process with a new draft.
  - There are many different options for how to generate draft tokens, and each comes with different tradeoffs. You can train multiple models, or fine-tune multiple heads on a single pretrained model, that predict tokens that are multiple steps in the future. Or, you can use a small model as the draft model, and a larger, more capable model as the verifier.
* **Technical Entities (Classes/Functions/APIs):** `Blockwise Parallel Decoding for Deep Autoregressive Models`

## Key takeaways for optimizing LLM inference
* **Key Points:**
  - This post outlines popular solutions to help optimize and serve LLMs efficiently, be it in the data center, cloud, or at the edge on a PC. Many of these techniques are optimized and available through NVIDIA TensorRT-LLM. It's an open source library consisting of the TensorRT deep learning compiler alongside optimized kernels, preprocessing and postprocessing steps, and multi-GPU/multi-node communication primitives for groundbreaking performance on NVIDIA GPUs.
  - NVIDIA TensorRT-LLM is supported by NVIDIA Dynamo, enabling enterprises to serve multiple AI models concurrently across different AI frameworks, hardware accelerators, and deployment models with peak throughput and minimum latency. Open models like the Nemotron 3 family, alongside leading community models such as Llama, are available with optimized TensorRT-LLM configs. See reference deployments and example scripts on GitHub and NVIDIA AI Models.
  - NVIDIA NIM is the fastest way to leverage the latest inference optimization techniques in NVIDIA TensorRT-LLM, as well as popular community frameworks including vLLM and SGLang. NIM microservice containers for the latest AI models from NVIDIA and the community come prepackaged with everything AI teams need to deploy and scale AI models on NVIDIA GPUs—-optimized inference technology, runtime dependencies, and industry-standard APIs. And they're validated, secured, and maintained by NVIDIA. NIM delivers workload-optimized inference performance with the lowest TCO through a streamlined, consistent workflow that integrates seamlessly into automated software delivery pipelines.
* **Technical Entities (Classes/Functions/APIs):** `NVIDIA TensorRT-LLM`, `TensorRT`, `NVIDIA Dynamo`, `Nemotron 3`, `Llama`, `NVIDIA NIM`, `vLLM`, `SGLang`

---


# How LLM Inference Engines Work: KV Caches, PagedAttention, and Continuous Batching From Prompt to Token

* **Key Points:**
  - Running a large language model in production is not the same as running it in a Jupyter notebook. The gap between “it works locally” and “it handles 200 concurrent requests at acceptable latency” is where inference engine design lives. Understanding what vLLM, TGI, and similar systems actually do internally helps you make informed decisions about deployment architecture, hardware sizing, and which knobs to turn when something is slow.
  - This article covers the full stack: from how autoregressive generation works at the GPU level, through KV cache design and PagedAttention, continuous batching, speculative decoding, multi-GPU parallelism, and quantization tradeoffs.
* **Technical Entities (Classes/Functions/APIs):** `vLLM`, `TGI`

## The Autoregressive Loop
* **Key Points:**
  - Transformer-based language models generate text one token at a time. Each forward pass through the model takes the full sequence of tokens so far and outputs a probability distribution over the vocabulary. You sample or greedily select the next token, append it, and repeat.
  - The problem is quadratic attention. For a sequence of length N, the attention mechanism computes N×N attention scores. Each new token extends the sequence, which grows the computation. For a 2048-token context, this is manageable. For 32K or 128K contexts, it is expensive.
  - The key optimization that makes this tractable is the KV cache.

## KV Caches: What They Are and Why Memory Is the Bottleneck
* **Key Points:**
  - During a forward pass, each attention layer computes key and value tensors for every token in the sequence. On subsequent tokens, those tensors for earlier positions do not change. Instead of recomputing them, the engine stores them in GPU memory and reads them for each new token. This is the KV cache.
  - The memory footprint is significant. For a single inference request with a 4096-token context on a 70B parameter model at float16:
  - KV cache per layer = 2 (K and V) × context_length × num_heads × head_dim × bytes_per_element = 2 × 4096 × 64 × 128 × 2 bytes = 134 MB per layer
  - With 80 layers, that is roughly 10.7 GB just for the KV cache of one request. The model weights for a 70B model at float16 are around 140 GB. A GPU with 80 GB of HBM3 memory has to fit both the weights and the KV caches of all concurrent requests.
  - This is why memory management is the central problem of LLM serving, not compute.

## Memory Fragmentation Under Static Allocation
* **Key Points:**
  - Early inference systems allocated KV cache memory statically: reserve a fixed buffer per request equal to the maximum sequence length. If you set max sequence length to 4096 tokens and get a request that only generates 200 tokens, you wasted 95% of that allocation.
  - Worse, because requests finish at unpredictable times, the free memory between active allocations becomes fragmented. You might have 10 GB of free GPU memory but be unable to accept a new request because no contiguous block is large enough to hold its maximum-length KV cache.

## PagedAttention: Virtual Memory for KV Caches
* **Key Points:**
  - PagedAttention, introduced in the vLLM paper, applies a familiar idea from OS virtual memory to KV cache management. Instead of allocating one contiguous block per request, the cache is divided into fixed-size pages (blocks), typically 16 or 32 tokens per block.
  - Each request has a logical page table mapping its token positions to physical GPU memory pages. As a request generates tokens, the engine allocates physical pages on demand. When the request completes, the pages are returned to a free pool and immediately reusable by any other request.
  - The non-contiguous layout requires the attention kernel to gather K and V tensors from non-adjacent memory locations during computation. vLLM ships custom CUDA kernels that handle this gather operation efficiently.
  - PagedAttention also enables prefix caching: if two requests share a common prefix (a system prompt, for example), the physical pages for that prefix can be shared between them with copy-on-write semantics. One copy of the prompt KV cache can serve hundreds of requests. This is how vLLM’s prefix caching feature achieves dramatic speedups for workloads with shared system prompts.
* **Code Snippet:**
```typescript
// Simplified model of the block manager data structures
interface PhysicalBlock {
  blockId: number;
  refCount: number;
  tokens: number[]; // token IDs stored in this block
}

interface SequenceGroup {
  requestId: string;
  logicalBlocks: number[]; // indices into physical block table
  generatedTokens: number;
}

class BlockManager {
  private freeBlocks: number[] = [];
  private physicalBlocks: Map<number, PhysicalBlock> = new Map();
  private blockSize: number;

  constructor(totalBlocks: number, blockSize: number) {
    this.blockSize = blockSize;
    for (let i = 0; i < totalBlocks; i++) {
      this.freeBlocks.push(i);
    }
  }

  allocate(sequence: SequenceGroup): boolean {
    if (this.freeBlocks.length === 0) return false;
    const blockId = this.freeBlocks.pop()!;
    this.physicalBlocks.set(blockId, {
      blockId,
      refCount: 1,
      tokens: [],
    });
    sequence.logicalBlocks.push(blockId);
    return true;
  }

  free(sequence: SequenceGroup): void {
    for (const blockId of sequence.logicalBlocks) {
      const block = this.physicalBlocks.get(blockId)!;
      block.refCount -= 1;
      if (block.refCount === 0) {
        this.freeBlocks.push(blockId);
        this.physicalBlocks.delete(blockId);
      }
    }
  }
}
```

## Static Batching vs. Continuous Batching
* **Key Points:**
  - The naive approach to batching is to collect N requests, run a forward pass over all of them simultaneously, then return results. This works well when all requests finish at the same time. In practice, they do not.
  - Consider a batch of 8 requests where one request generates 5 tokens and another generates 500 tokens. With static batching, the 7 short requests finish early but their GPU slots sit idle while the long request continues. GPU utilization tracks the slowest request, not the average.
  - Continuous batching, also called iteration-level scheduling, changes when new requests join the batch. Instead of waiting for the entire batch to complete before accepting new requests, the scheduler inserts new requests into the next forward pass the moment a slot frees up.
  - Each forward pass can contain a mix of:
    - Requests in their prefill phase (processing the prompt in parallel)
    - Requests in their decode phase (generating one token at a time)
  - The scheduler runs between every forward pass. It decides which sequence groups to run, which to preempt (swap to CPU memory if GPU is full), and which new requests to admit.
  - The throughput gain from continuous batching is substantial. Benchmarks show 5-10x higher request throughput on typical chat workloads compared to static batching, because GPU utilization is nearly constant rather than pulsing with batch boundaries.
* **Technical Entities (Classes/Functions/APIs):** `ScheduledRequest`, `SchedulerOutput`, `schedule()`
* **Code Snippet:**
```typescript
type RequestPhase = "prefill" | "decode";

interface ScheduledRequest {
  requestId: string;
  phase: RequestPhase;
  promptTokens: number[];
  generatedTokens: number[];
  maxNewTokens: number;
}

interface SchedulerOutput {
  prefillRequests: ScheduledRequest[];
  decodeRequests: ScheduledRequest[];
  preempted: string[];
}

function schedule(
  waiting: ScheduledRequest[],
  running: ScheduledRequest[],
  freeBlocks: number,
  blockSize: number
): SchedulerOutput {
  const blocksNeeded = (req: ScheduledRequest) =>
    Math.ceil(
      (req.promptTokens.length + req.generatedTokens.length + 1) / blockSize
    );

  const output: SchedulerOutput = {
    prefillRequests: [],
    decodeRequests: [],
    preempted: [],
  };

  // Admit new prefill requests if memory allows
  for (const req of waiting) {
    if (blocksNeeded(req) <= freeBlocks) {
      output.prefillRequests.push(req);
      freeBlocks -= blocksNeeded(req);
    }
  }

  // All running requests continue decode
  for (const req of running) {
    output.decodeRequests.push(req);
  }

  return output;
}
```

## Speculative Decoding
* **Key Points:**
  - Decode is memory-bandwidth-bound, not compute-bound. For each token, the model reads all its weights from HBM, does a small matmul, and writes back a single token. The hardware is underutilized.
  - Speculative decoding exploits this slack by using a small draft model to propose multiple tokens ahead, then verifying them in parallel with the large target model.
  - The sequence:
    - A small model (e.g., 1B parameters) generates K candidate tokens speculatively.
    - The large target model runs a single forward pass over the original context plus all K candidates in parallel.
    - The target model’s outputs are compared token-by-token against the candidates using rejection sampling.
    - All accepted tokens are committed. If a token is rejected, the target model’s own distribution is used for that position.
  - Because the verification pass processes K tokens in parallel rather than one at a time, it takes roughly the same time as a single decode step. If K tokens are typically accepted (e.g., 3-4 out of 5 speculated), throughput improves proportionally.
  - The draft model must be aligned with the target model’s distribution for acceptance rates to be high. Common choices: smaller models from the same model family, or n-gram language models that just repeat observed patterns.
  - Speculative decoding reduces latency for individual requests without degrading output quality. The output distribution of accepted tokens is identical to what the target model would have produced alone, by construction of the rejection sampling algorithm.

## Tensor Parallelism and Pipeline Parallelism
* **Key Points:**
  - Single GPUs can no longer fit the largest models. Multi-GPU inference requires partitioning the computation.
  - Tensor parallelism splits individual weight matrices across GPUs. For a transformer’s MLP layer with a weight matrix of shape [d_model, 4d_model], you can split it column-wise across N GPUs so each holds [d_model, 4d_model / N]. Each GPU computes a partial result; an allreduce collective operation sums them.
  - vLLM uses Megatron-style tensor parallelism: the QKV projection and MLP layers are split across the GPU’s head dimension and intermediate dimension respectively. Each GPU holds a complete set of model layers but a fraction of each layer’s width.
  - Tensor parallelism requires tight GPU interconnects. NVLink (600 GB/s on H100 NVLink) makes this practical within a node. PCIe (64 GB/s) adds latency that degrades scaling efficiency.
  - Pipeline parallelism assigns different transformer layers to different GPUs. GPU 0 handles layers 0-19, GPU 1 handles layers 20-39, and so on. The output tensor from each stage is passed to the next GPU.
  - Pipeline parallelism works across nodes with slower interconnects (InfiniBand) but introduces pipeline bubbles: GPUs earlier in the pipeline sit idle while later stages catch up. Micro-batching reduces bubbles by interleaving multiple requests through the pipeline, at the cost of additional latency.
  - In practice, most deployments use tensor parallelism within a node and pipeline parallelism across nodes when serving models too large for one node’s memory.
* **Technical Entities (Classes/Functions/APIs):** `vLLM`, `Megatron-style tensor parallelism`

## Quantization: GPTQ, AWQ, and GGUF
* **Key Points:**
  - Quantization reduces memory usage and increases throughput by representing weights in lower precision. The tradeoffs vary significantly by format.
  - GPTQ quantizes each layer by solving an optimization problem over a calibration dataset, minimizing the reconstruction error of each weight matrix. The result is a 4-bit integer weight matrix plus per-group scaling factors.
  - AWQ improves on GPTQ by observing that not all weights are equally sensitive to quantization. A small fraction of weights (those corresponding to large activation channels) account for most of the reconstruction error. AWQ scales these channels up before quantization and back down during inference, protecting them from the precision loss without changing the bit width.
  - GGUF (formerly GGML) packs weights in formats designed for CPU memory layouts with optional GPU offloading. The Q4_K_M variant uses 4-bit quantization with K-means clustering per block, which improves perplexity compared to naive 4-bit rounding. It is the practical choice for running models on Apple Silicon or CPU-only servers where you need to stay off expensive GPU cloud instances.
* **Technical Entities (Classes/Functions/APIs):** `GPTQ`, `AWQ`, `GGUF`, `GPTQ-int8`, `FP8`, `Q4_K_M`

## Production Considerations
* **Key Points:**
  - Memory planning. Calculate your maximum KV cache memory before deploying. The rule of thumb: subtract model weight memory from total GPU memory, then subtract 1-2 GB for fragmentation overhead. What remains is available for KV pages. Set gpu_memory_utilization in vLLM to 0.90 or lower to leave a safety buffer; hitting 100% triggers OOM kills, not graceful degradation.
  - Prefill and decode are different bottlenecks. Prefill (processing the prompt) is compute-bound; it benefits from large batches and tensor parallelism. Decode (generating tokens) is memory-bandwidth-bound; it benefits from continuous batching to keep the GPU busy across requests. If your P50 latency is high but GPU utilization is low during decode, the answer is more concurrent requests, not more GPUs.
  - Prefix caching for shared prompts. If you have a system prompt used in every request, enable prefix caching in vLLM (enable_prefix_caching=True). The memory savings can be dramatic: a 2000-token system prompt shared across 100 concurrent requests reduces KV cache memory by 200,000 tokens worth of pages.
  - Choosing between vLLM, TGI, and Ollama. This is a deployment context decision, not a quality decision:
  - Use vLLM when throughput per GPU-hour is the primary concern. Use TGI when you are deeply in the Hugging Face ecosystem and want tight integration with model hub tooling. Use Ollama for local development and cases where you are running on Apple Silicon or CPU servers without CUDA.
  - Speculative decoding in production. Speculative decoding helps most for chat-style workloads where output is conversational prose (high acceptance rates on speculated tokens). It helps less for code generation, structured JSON output, or tasks with high token diversity. Benchmark acceptance rates on your specific workload before enabling it in production; a low acceptance rate wastes the overhead of running the draft model.
  - Quantization quality checks. Do not rely on perplexity alone to evaluate quantized models. Perplexity tracks average quality; production failures happen at the tail. Evaluate on task-specific benchmarks, especially for multi-step reasoning, instruction following, and any domain-specific knowledge the model must have. A model that looks fine on perplexity can fail noticeably on structured output tasks at Q4 quantization.
  - Monitoring. The metrics that matter for inference serving: time to first token (TTFT), time per output token (TPOT), token throughput (tokens/sec), KV cache utilization (%), and request queue depth. KV cache utilization above 95% consistently means you are either memory-constrained or receiving more load than your deployment can sustain. GPU utilization below 60% during peak load usually indicates a batching or scheduling configuration problem.
* **Technical Entities (Classes/Functions/APIs):** `vLLM`, `TGI`, `Ollama`, `enable_prefix_caching`

## Closing
* **Key Points:**
  - LLM inference is fundamentally a resource management problem. PagedAttention solves fragmentation. Continuous batching solves GPU idle time. Speculative decoding recovers the wasted memory bandwidth of single-token decode. Quantization buys you more model capacity per GB of memory. Understanding which problem each technique solves makes it straightforward to reason about why your deployment behaves the way it does and which lever to pull next.