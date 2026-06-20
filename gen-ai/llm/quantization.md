---
aliases:
  - Quantization
Source 1: https://mljourney.com/quantization-techniques-for-llm-inference-int8-int4-gptq-and-awq/
---
# Quantization Techniques for LLM Inference: INT8, INT4, GPTQ, and AWQ

## Understanding Quantization Fundamentals

* **Key Points:**
  - Before examining specific techniques, it's crucial to understand what quantization actually does to neural network weights and how different approaches trade off precision for efficiency. At its core, quantization maps high-precision floating-point numbers to lower-precision representations, typically integers.
  - Floating-Point Representation in standard LLM training uses FP32 (32-bit) or FP16 (16-bit) formats. FP16 has become the de facto standard for modern LLMs, providing sufficient precision while halving memory requirements compared to FP32. Each FP16 number uses 16 bits to represent a wide range of values with good precision, but this precision comes at a cost—memory bandwidth and storage.
  - Integer Quantization converts these floating-point weights to integer representations using significantly fewer bits. The basic formula involves scaling and shifting: quantized_value = round(float_value / scale) + zero_point
  - The scale factor determines the range of values that can be represented, while the zero_point handles asymmetric distributions. During inference, dequantization reverses this process before mathematical operations, though sophisticated implementations can perform operations directly in quantized space.
  - Per-Tensor vs Per-Channel Quantization represents a critical architectural decision. Per-tensor quantization uses a single scale factor for an entire weight matrix, while per-channel quantization uses different scale factors for each output channel. Per-channel quantization provides finer granularity, typically preserving accuracy better but requiring more metadata storage and slightly more complex implementation.
  - The accuracy impact of quantization stems from the quantization error—the difference between the original value and its quantized representation. With fewer bits available, this error increases. The art of quantization lies in minimizing this error's impact on the model's overall behavior, particularly on important weights that significantly affect outputs.
* **Technical Entities (Classes/Functions/APIs):** `FP32`, `FP16`, `INT8`, `INT4`

## INT8 Quantization: The Foundation
* **Key Points:**
  - INT8 quantization represents the most mature and widely supported quantization technique, reducing weights and activations from FP16 (16 bits) to INT8 (8 bits), achieving a 2x memory reduction. This seemingly modest compression provides substantial practical benefits while maintaining remarkably good accuracy across most models and tasks.
  - Post-Training Quantization (PTQ) with INT8 can be applied after training without modifying the training process. The simplest approach, dynamic quantization, quantizes weights ahead of time but quantizes activations dynamically during inference based on their actual range in each forward pass. This adds computational overhead but ensures activations use their full INT8 range effectively.
  - Static quantization goes further by calibrating quantization parameters on representative data before inference. By passing calibration data through the model and observing the actual ranges of activations, you can set scale factors that minimize quantization error for typical inputs. This calibration process typically requires only hundreds to thousands of examples—a small sample of the training data suffices.
  - Accuracy Preservation with INT8 is generally excellent for LLMs. Modern transformer architectures prove remarkably robust to INT8 quantization, with perplexity increases typically under 1-2% and minimal degradation on downstream tasks. This robustness stems partly from the redundancy in large models—not every weight carries equal importance, and moderate quantization error in less critical weights has minimal impact.
  - However, certain operations prove more sensitive to quantization. Attention mechanisms, particularly the softmax operation, can accumulate quantization errors that affect model outputs. Some implementations use higher precision (FP16) for specific sensitive operations while quantizing the bulk of matrix multiplications to INT8, creating mixed-precision inference that balances accuracy and efficiency.
  - Hardware Support for INT8 operations is widespread and highly optimized. Modern GPUs from NVIDIA (Tensor Cores), AMD, and Intel provide dedicated INT8 acceleration offering 2-4x speedup over FP16 for certain operations. CPUs similarly benefit from INT8 quantization through SIMD instructions like AVX-512 VNNI, making INT8 quantization particularly attractive for CPU deployment.
* **Technical Entities (Classes/Functions/APIs):** `INT8`, `FP16`, `BitsAndBytesConfig`, `load_in_8bit`, `llm_int8_threshold`, `llm_int8_has_fp16_weight`, `LLM.int8()`, `bitsandbytes`, `AVX-512 VNNI`
* **Code Snippet:**
```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

# Load model in FP16
model = AutoModelForCausalLM.from_pretrained(
    "model_name",
    torch_dtype=torch.float16,
    device_map="auto"
)

# Apply INT8 quantization
from transformers import BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(
    load_in_8bit=True,
    llm_int8_threshold=6.0,  # Threshold for outlier detection
    llm_int8_has_fp16_weight=False
)

quantized_model = AutoModelForCausalLM.from_pretrained(
    "model_name",
    quantization_config=quantization_config,
    device_map="auto"
)

# Memory reduced by ~2x, modest speedup on supported hardware
```

## INT4 Quantization: Pushing the Limits
* **Key Points:**
  - INT4 quantization represents a more aggressive compression strategy, reducing each weight to just 4 bits—achieving 4x memory reduction compared to FP16. This dramatic compression comes with greater challenges in preserving accuracy, requiring more sophisticated techniques to remain viable.
  - The 4-Bit Challenge stems from the limited representational capacity. With only 16 possible values (2^4), INT4 quantization must carefully allocate this narrow range to minimize impact on model behavior. Naive quantization to INT4 typically results in unacceptable accuracy degradation—perplexity increases of 10-50% or more, rendering models nearly useless for many tasks.
  - The breakthrough came from recognizing that not all weights are equally important. In transformer models, weights follow a distribution where most values cluster near zero with occasional outliers that have disproportionate impact on outputs. Simple uniform quantization wastes representational capacity on unimportant weights while inadequately representing critical outliers.
  - Group Quantization addresses this by dividing weight matrices into smaller groups and quantizing each group independently with its own scale factor. Instead of a single scale for an entire layer's weights, you might use groups of 128 weights, each with optimized quantization parameters. This finer granularity allows the quantization to adapt to local weight distributions, dramatically improving accuracy preservation.
  - The trade-off involves increased metadata—storing scale factors for each group adds overhead. With group size 128, a matrix with 4096 weights requires 32 scale factors (typically stored as FP16), adding about 1% memory overhead. This is well worth the accuracy improvement, with group-quantized INT4 often matching or exceeding naive INT8 accuracy while using half the memory.
  - Mixed-Precision Strategies combine INT4 quantization for most weights with higher precision for sensitive operations. Embedding layers, which map discrete tokens to continuous representations, and the final language modeling head, which produces output probabilities, often benefit from FP16 precision. Similarly, some normalization layers may remain in FP16 to maintain numerical stability.
  - A typical configuration might quantize 95% of weights to INT4 (the large feedforward and attention projection matrices) while keeping 5% in FP16 (embeddings, layer norms, and output head). This achieves approximately 3.5x overall compression with significantly better accuracy than full INT4 quantization.
* **Technical Entities (Classes/Functions/APIs):** `INT4`, `FP16`

## GPTQ: Optimal Weight Quantization
* **Key Points:**
  - GPTQ (Generative Pre-trained Transformer Quantization) represents a significant advancement in post-training quantization, using optimization techniques to minimize the impact of quantization on model outputs. Rather than simply rounding weights to nearest quantized values, GPTQ solves an optimization problem to find quantized weights that best preserve the layer's behavior on calibration data.
  - The Optimization Approach treats quantization as a layer-wise reconstruction problem. For each layer, GPTQ:
    - Passes calibration data through the unquantized layer, recording outputs
    - Quantizes the layer's weights using an optimization procedure that minimizes the difference between quantized and original outputs
    - Uses the Hessian (second derivative) of the loss with respect to weights to guide the quantization, prioritizing accuracy preservation for weights that significantly affect outputs
    - Applies a technique called "optimal brain quantization" that jointly optimizes all weights in a layer rather than quantizing independently
  - This sophisticated approach requires calibration data—typically 128-1024 samples from the training distribution—and significant computation time. Quantizing a 7B parameter model with GPTQ might take 1-2 hours on a high-end GPU, much longer than simple INT8 or naive INT4 quantization.
  - Accuracy Benefits are substantial. GPTQ-quantized models at INT4 often achieve perplexity within 1-3% of the original FP16 model, dramatically outperforming naive INT4 quantization. For practical tasks like question answering, summarization, or coding, GPTQ enables 4-bit models that remain highly capable, whereas naive quantization would render them nearly unusable.
  - The technique works particularly well with group quantization, where GPTQ optimizes the quantization within each group. GPTQ with group size 128 has become a popular configuration, offering an excellent balance of accuracy, memory efficiency, and inference speed.
  - Implementation and Ecosystem for GPTQ is mature and well-supported. The AutoGPTQ library provides easy access.
  - Pre-quantized GPTQ models are widely available on Hugging Face Hub, eliminating the need to run the time-consuming quantization process yourself. These community-contributed quantized models have made powerful LLMs accessible to users with consumer hardware.
  - Inference Performance with GPTQ shows interesting characteristics. The memory savings are clear—4x reduction compared to FP16. Inference speed improvements are more nuanced. On GPUs with good INT4 support (particularly NVIDIA GPUs with recent CUDA versions), GPTQ can provide 1.5-2x speedup. However, some operations still run in higher precision, limiting speedup compared to the theoretical 4x from reduced computation.
  - The speedup depends heavily on batch size and sequence length. For small batches (1-4), memory bandwidth dominates performance, and GPTQ's reduced memory footprint translates to meaningful speedup. For larger batches where compute becomes the bottleneck, speedup depends more on hardware-specific INT4 acceleration capabilities.
* **Technical Entities (Classes/Functions/APIs):** `GPTQ`, `AutoGPTQ`, `BaseQuantizeConfig`, `bits`, `group_size`, `desc_act`, `quantize()`, `save_quantized()`
* **Code Snippet:**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from auto_gptq import AutoGPTQForCausalLM, BaseQuantizeConfig

# Configure GPTQ quantization
quantize_config = BaseQuantizeConfig(
    bits=4,  # INT4 quantization
    group_size=128,  # Group size for finer granularity
    desc_act=False,  # Activation ordering optimization
)

# Load and quantize model
model = AutoGPTQForCausalLM.from_pretrained(
    "model_name",
    quantize_config=quantize_config
)

# Calibrate on sample data
model.quantize(calibration_data)

# Save quantized model for later use
model.save_quantized("quantized_model_path")
```

## AWQ: Activation-Aware Weight Quantization
* **Key Points:**
  - AWQ (Activation-aware Weight Quantization) represents the current state-of-the-art in post-training LLM quantization, achieving better accuracy than GPTQ with faster calibration. The key insight behind AWQ is that activation magnitudes reveal which weights are most important—weights that consistently produce large activations have greater impact on model behavior and should be quantized more carefully.
  - The Activation-Aware Insight observes that in transformer models, a small percentage of weights (1-5%) correspond to channels that produce consistently large activations across different inputs. These "salient" weights disproportionately influence the model's outputs. Traditional quantization treats all weights equally, but AWQ protects important weights by scaling them up before quantization, effectively allocating more of the quantized range to their values.
  - The process works by:
    - Analyzing calibration data to identify which channels produce large activations
    - Computing scaling factors that protect these important channels
    - Applying these scales to weights before quantization, giving salient weights better quantized representations
    - Storing the scales to be applied during inference to maintain correct computations
  - This approach requires no gradient computation or complex optimization, making calibration much faster than GPTQ—often 5-10x speedup in the quantization process itself.
  - Superior Accuracy Preservation is AWQ's primary advantage. Across standard benchmarks, AWQ-quantized models consistently outperform GPTQ at the same bit-width, with the gap widening for more aggressive quantization (3-bit or 2-bit). For INT4 quantization, AWQ typically achieves perplexity within 0.5-1.5% of the original model—better than GPTQ's 1-3% and dramatically better than naive approaches.
  - This accuracy advantage manifests most clearly on challenging tasks requiring precise reasoning or factual recall. Where GPTQ INT4 models might show noticeable quality degradation on complex question-answering or mathematical reasoning, AWQ models maintain performance much closer to the original FP16 baseline.
  - Practical Implementation with AWQ is straightforward using the AutoAWQ library.
  - Like GPTQ, pre-quantized AWQ models are increasingly available on Hugging Face Hub, providing immediate access without requiring calibration resources.
  - Inference Characteristics for AWQ are promising. The activation-aware scaling integrates cleanly with efficient inference kernels, and recent optimizations have achieved inference speeds competitive with or better than GPTQ. Memory footprint is similar to GPTQ at the same bit-width—both achieve approximately 4x compression with INT4.
  - The primary advantage of AWQ over GPTQ comes down to the accuracy-efficiency frontier. For the same computational budget during quantization and inference, AWQ consistently delivers better model quality. This makes it the preferred choice when you need the most capable quantized model possible, particularly for demanding applications.
* **Technical Entities (Classes/Functions/APIs):** `AWQ`, `AutoAWQ`, `zero_point`, `q_group_size`, `w_bit`, `version`, `GEMM`
* **Code Snippet:**
```python
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

# Load model
model = AutoAWQForCausalLM.from_pretrained("model_name")
tokenizer = AutoTokenizer.from_pretrained("model_name")

# Configure quantization
quant_config = {
    "zero_point": True,
    "q_group_size": 128,
    "w_bit": 4,
    "version": "GEMM"
}

# Quantize with calibration data
model.quantize(tokenizer, quant_config=quant_config, calib_data=calibration_samples)

# Save quantized model
model.save_quantized("quantized_model_path")
```

## Choosing the Right Quantization Strategy
* **Key Points:**
  - With multiple quantization techniques available, selecting the right approach for your specific deployment requires considering several factors beyond just compression ratio and accuracy metrics.
  - Hardware Constraints often dictate the decision. If you're deploying on consumer GPUs with limited VRAM, aggressive quantization becomes necessary regardless of the technique. A 70B model requires INT4 quantization or smaller to fit on a single 24GB GPU. In such scenarios, AWQ offers the best accuracy for the required compression level.
  - For datacenter deployments with ample GPU memory, INT8 quantization might suffice, providing excellent accuracy with broad hardware support and simple implementation. The 2x compression is less dramatic but often adequate when combined with techniques like FlashAttention and other inference optimizations.
  - Inference Volume and Cost calculations influence the choice. If you're running thousands of inferences per second, the calibration time for GPTQ or AWQ amortizes quickly—investing hours in quantization saves on ongoing inference costs. For occasional use or experimentation, simpler INT8 quantization avoids the calibration overhead while still providing meaningful memory savings.
  - Cloud deployment costs often favor more aggressive quantization. A 70B model at INT4 (35GB VRAM) fits comfortably on a single A100 40GB instance, while FP16 (140GB) requires multi-GPU setups with significantly higher hourly costs. The monthly savings from using fewer or smaller instances can be substantial, easily justifying the one-time calibration investment.
  - Model Size Considerations affect quantization strategy. Smaller models (7B-13B parameters) tolerate quantization better than larger models—they have less redundancy, so quantization error has greater relative impact. For 7B models, INT8 often provides the best balance. For 70B+ models, the redundancy enables more aggressive INT4 quantization with acceptable accuracy loss, and the memory savings become essential for feasible deployment.
  - Task Sensitivity matters significantly. Tasks requiring precise factual recall or mathematical reasoning are more sensitive to quantization than creative writing or summarization. If your application involves complex reasoning chains or needs to cite specific facts accurately, lean toward more conservative quantization (INT8 or AWQ INT4). For more open-ended generation where approximate outputs are acceptable, even naive INT4 might suffice.
  - Development vs Production phases benefit from different strategies. During development and experimentation, INT8 quantization provides fast iteration with minimal accuracy concerns. For production deployment after thorough validation, invest in GPTQ or AWQ quantization to maximize efficiency while maintaining quality standards.
* **Technical Entities (Classes/Functions/APIs):** `FlashAttention`, `A100 40GB`

## Advanced Considerations and Mixed Precision
* **Key Points:**
  - Mixed-Precision Quantization applies different bit-widths to different parts of the model based on their sensitivity. A sophisticated approach might use:
    - INT8 for attention projections (Q, K, V matrices) which are somewhat sensitive
    - INT4 for feedforward layers which tolerate aggressive quantization well
    - INT3 or INT2 for less critical layers after careful sensitivity analysis
    - FP16 for embeddings and output heads which benefit from higher precision
  - This creates models with average bit-widths like 3.5 or 4.2, threading the needle between memory efficiency and accuracy preservation. AutoGPTQ and AWQ both support mixed-precision configurations, though they require careful tuning based on your specific model and task requirements.
  - Dynamic vs Static Quantization offers another dimension of choice. Static quantization determines all quantization parameters during calibration and uses them unchanged during inference. Dynamic quantization allows activation quantization ranges to adapt per-batch or per-sample, potentially improving accuracy for inputs that differ significantly from calibration data.
  - The trade-off involves computational overhead—dynamic quantization requires computing ranges during inference, adding latency. For deployments prioritizing throughput over latency, static quantization is preferred. For scenarios where input distribution varies significantly, dynamic quantization's flexibility may justify the overhead.
  - Outlier Management in LLMs presents unique challenges. Recent research has revealed that transformer models develop systematic outlier features—specific channels that produce activations orders of magnitude larger than typical values. These outliers appear consistently across different inputs and models, suggesting they're a fundamental characteristic of how transformers represent information.
  - Standard quantization struggles with outliers because uniform quantization ranges must accommodate both typical values and extreme outliers, effectively wasting quantization range. Advanced techniques like SmoothQuant redistribute magnitude between weights and activations, or LLM.int8() processes outlier channels separately in higher precision. AWQ's activation-aware approach implicitly handles outliers by protecting high-activation channels during quantization.
* **Technical Entities (Classes/Functions/APIs):** `INT8`, `INT4`, `INT3`, `INT2`, `FP16`, `AutoGPTQ`, `AWQ`, `SmoothQuant`, `LLM.int8()`

## Practical Deployment Patterns
* **Key Points:**
  - Quantization + FlashAttention combines two orthogonal optimizations. FlashAttention optimizes the attention mechanism's memory access patterns, reducing memory bandwidth requirements and improving speed. Quantization reduces memory footprint. Together, they enable fitting larger models in available memory while accelerating inference—quantization makes the model fit, FlashAttention makes it fast.
  - Most modern inference frameworks (vLLM, TGI, llama.cpp) integrate both optimizations, applying them automatically when appropriate. The combined effect is multiplicative—a 70B model with INT4 quantization and FlashAttention might fit in 35GB VRAM and run 3x faster than naive FP16 implementation, whereas either optimization alone would provide smaller gains.
  - Batch Processing Considerations affect quantization choices. For serving single-user interactive applications where batch size is 1, quantization's memory savings are crucial but throughput optimizations matter less. For batch inference processing many requests simultaneously, compute efficiency becomes equally important as memory—here, techniques that provide both memory reduction and speedup (like GPTQ on optimized hardware) offer maximum value.
  - Model Caching and Startup Time present practical trade-offs. Quantized models load faster from disk due to smaller file sizes—a 70B model at INT4 (35GB) loads 4x faster than FP16 (140GB), reducing cold-start latency in serverless deployments. However, some quantization formats require additional processing during loading. Pre-quantized models with optimized formats minimize this overhead.
  - Version Control and Reproducibility deserve attention. Quantized models should be versioned alongside their quantization parameters, calibration dataset details, and validation metrics. When debugging quality issues, knowing exactly which quantization technique, group size, and calibration data produced a model is essential. Many teams maintain a model registry tracking these details for all deployed variants.
* **Technical Entities (Classes/Functions/APIs):** `FlashAttention`, `vLLM`, `TGI`, `llama.cpp`

## Conclusion
* **Key Points:**
  - Quantization techniques have matured to the point where 4-bit LLMs can approach the quality of their full-precision counterparts while using a quarter of the memory, democratizing access to powerful models and dramatically reducing deployment costs. INT8 quantization provides a robust baseline with excellent accuracy and wide support, while INT4 approaches like GPTQ and AWQ push compression further with increasingly sophisticated methods that preserve model capabilities. AWQ currently represents the state-of-the-art, offering superior accuracy through activation-aware weight protection, though GPTQ remains valuable for its maturity and ecosystem support.
  - The choice among these techniques ultimately depends on your specific constraints and priorities—hardware limitations, task sensitivity, inference volume, and acceptable quality trade-offs all factor into the decision. Start with INT8 for experimentation and development, moving to GPTQ or AWQ INT4 quantization for production deployments where memory constraints demand more aggressive compression. Always validate quantized models on representative samples of your actual use case, as perplexity metrics alone don't capture the practical impact on downstream task performance. With the right quantization strategy, you can deploy capable LLMs on consumer hardware or dramatically reduce cloud inference costs while maintaining the quality your applications require.
* **Technical Entities (Classes/Functions/APIs):** `INT8`, `INT4`, `GPTQ`, `AWQ`