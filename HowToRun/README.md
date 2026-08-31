# llama.cpp — How to run and Observations

This repository documents my experiments with **[llama.cpp](https://github.com/ggml-org/llama.cpp)**, focusing on local Large Language Model (LLM) inference, CPU vs GPU acceleration, Flash Attention, and context-size performance.

The goal is to understand how different `llama.cpp` configuration options affect **inference speed, prompt processing, generation speed, and context handling**.

---

##  Table of Contents

* [1. Model](#1-model)
* [2. CPU-Only Inference](#2-cpu-only-inference)
* [3. GPU Acceleration](#3-gpu-acceleration)
* [4. Flash Attention](#4-flash-attention)
* [5. Context Size Experiment](#5-context-size-experiment)
* [6. KV-Cache Quantization](#6-kv-cache-Quantization)
* [7. Increase batch size](#.7-increase-batch-size)


---

# 1. Model

Before running `llama.cpp`, we need a model in **GGUF format**.

A large collection of models can be found on:

 [Hugging Face](https://huggingface.co/)

For this project, I chose:

**Mistral 7B Instruct v0.2 — Q4_K_M**

[Model on Hugging Face](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF)

The downloaded model should be placed inside the repository's `models` directory.

For example:

```text
llama.cpp/
├── build/
├── models/
│   └── mistral/
│       └── mistral-7b-instruct-v0.2.Q4_K_M.gguf
└── ...
```

---

# 2. CPU-Only Inference

To run the model entirely on the CPU:

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 0
```

The `-ngl` option controls how many model layers are offloaded to the GPU.

|     Value | Meaning                                           |
| --------: | ------------------------------------------------- |
|  `-ngl 0` | CPU only                                          |
| `-ngl 99` | Attempt to offload all possible layers to the GPU |

### My CPU

**AMD Ryzen 7 260 with Radeon 780M Graphics**

Using a simple question such as:

> **"What is a CPU cache?"**

I obtained approximately:

```text
Prompt:      18.5 t/s
Generation:  11.2 t/s
```

### What do these numbers mean?

**Prompt: 18.5 t/s**

`llama.cpp` processed the input prompt at approximately **18.5 tokens per second**.

**Generation: 11.2 t/s**

Once the model started producing its response, it generated approximately **11.2 tokens per second**.

---

## 2.1  Measuring Total Runtime

Linux can measure the execution time of a command using `time`.

Simply add `time` before the command:

```bash
time ./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 0
```

For this experiment, the model took approximately:

```text
~45 seconds
```

to load and answer the prompt using only the CPU.

---

# 3. GPU Acceleration

Now let's take advantage of the GPU.

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99
```

### My GPU

**NVIDIA RTX 5050 — 8 GB VRAM**

Using the same question:

> **"What is a CPU cache?"**

the results were approximately:

```text
Prompt:      187.9 t/s
Generation:   56.7 t/s
```

Compared with the CPU-only experiment:

| Metric            |      CPU |       GPU |
| ----------------- | -------: | --------: |
| Prompt processing | 18.5 t/s | 187.9 t/s |
| Generation        | 11.2 t/s |  56.7 t/s |
| Total runtime     |    ~45 s |     ~10 s |

The GPU therefore provides a **substantial improvement in inference performance**.

For this particular experiment, total execution time decreased from approximately **45 seconds to 10 seconds**, making the GPU run roughly **4.5× faster**.

> ⚠️ These numbers are specific to this hardware, model, prompt, and llama.cpp build. Different systems will produce different results.

---

# 4. Flash Attention

Next, we can experiment with **Flash Attention**.

Enable it using:

```bash
-fa on
```

For example:

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on
```

## 4.1 What is Flash Attention?

Mistral is based on the **Transformer architecture**, which relies heavily on the attention mechanism.

A simplified representation is:

```text
        Q (Query)
          │
          │
K ────────┼─────── V
          │
          ▼
      Attention
          │
          ▼
   New Hidden States
```

The attention mechanism determines how strongly different tokens in the sequence should interact with each other.

The traditional attention implementation can become increasingly expensive as the **context length grows**.

Flash Attention uses a more memory-efficient approach to calculating attention.

Depending on the workload and hardware, it can:

*  Improve inference performance
*  Reduce memory/VRAM usage
*  Make larger context sizes more practical
*  Improve performance when processing long sequences

However, **Flash Attention does not necessarily make every workload faster**.

This is something we can demonstrate experimentally.

---

## 4.2 Context Size Experiment

To test Flash Attention, I used a more demanding prompt:

> **"Explain in detail how a modern computer CPU works. Cover instruction fetching, decoding, execution, registers, ALUs, caches, branch prediction, pipelining, out-of-order execution, and memory access. Explain how these components interact during the execution of a program and provide a concrete example of executing a simple instruction sequence."**

First, we use a context size of:

```text
-c 4096
```

The `-c` option controls the model's **context size**, i.e. the maximum number of tokens the model can keep in its context window.

---

## Context Size: 4096

### Flash Attention ON

```bash
-fa on
-c 4096
```

Result:

```text
Prompt:      40.3 t/s
Generation:  55.0 t/s
```

### Flash Attention OFF

```bash
-fa off
-c 4096
```

Result:

```text
Prompt:      179.0 t/s
Generation:   51.8 t/s
```

### Comparison

| Configuration |        Prompt |   Generation |
| ------------- | ------------: | -----------: |
| FA OFF        | **179.0 t/s** |     51.8 t/s |
| FA ON         |      40.3 t/s | **55.0 t/s** |

At this context size, Flash Attention actually made **prompt processing substantially slower**.

However, generation throughput improved slightly.

Approximately:

```text
Generation improvement:
(55.0 - 51.8) / 51.8 × 100 ≈ +6.2%
```

Prompt processing, on the other hand, was approximately:

```text
(179.0 - 40.3) / 179.0 × 100 ≈ 77.5% slower
```

So at `-c 4096`, Flash Attention was **not beneficial for prompt processing** in this particular experiment.

---

# 5. Increasing the Context Size

Now let's increase the context size from:

```text
4096 → 16384
```

This gives us a much larger context window and allows us to see where Flash Attention becomes more useful.

---

## 5.1 Context Size: 16384

### Flash Attention ON

```bash
-fa on
-c 16384
```

Result:

```text
Prompt:      115.1 t/s
Generation:   55.2 t/s
```

### Flash Attention OFF

```bash
-fa off
-c 16384
```

Result:

```text
Prompt:       77.6 t/s
Generation:   51.8 t/s
```

### Comparison

| Configuration |        Prompt |   Generation |
| ------------- | ------------: | -----------: |
| FA OFF        |      77.6 t/s |     51.8 t/s |
| FA ON         | **115.1 t/s** | **55.2 t/s** |

Now the situation changes significantly.

With a context size of **16,384 tokens**, Flash Attention is faster for both prompt processing and generation.

Prompt processing improves from:

```text
77.6 t/s → 115.1 t/s
```

That's approximately a:

```text
+48.3%
```

improvement in prompt-processing throughput.

Generation also improves:

```text
51.8 t/s → 55.2 t/s
```

or approximately:

```text
+6.6%
```

---

## 5.2 Benchmark Results

The experiments can be summarized as follows.

### CPU vs GPU

| Configuration   |    Prompt | Generation | Runtime |
| --------------- | --------: | ---------: | ------: |
| CPU (`-ngl 0`)  |  18.5 t/s |   11.2 t/s |   ~45 s |
| GPU (`-ngl 99`) | 187.9 t/s |   56.7 t/s |   ~10 s |

### Flash Attention

| Context | Flash Attention |        Prompt |   Generation |
| ------: | :-------------: | ------------: | -----------: |
|    4096 |       OFF       | **179.0 t/s** |     51.8 t/s |
|    4096 |        ON       |      40.3 t/s | **55.0 t/s** |
|   16384 |       OFF       |      77.6 t/s |     51.8 t/s |
|   16384 |        ON       | **115.1 t/s** | **55.2 t/s** |

---

## 5.3 Conclusions

These experiments demonstrate an important aspect of LLM inference:

> **The fastest configuration depends heavily on the workload.**

###  GPU Acceleration

Moving inference from the CPU to the RTX 5050 produced a major performance improvement.

The total runtime for the simple test decreased from approximately:

```text
45 seconds → 10 seconds
```

This demonstrates the importance of GPU acceleration for local LLM inference.

###  Flash Attention

Flash Attention produced very different results depending on context size.

At:

```text
-c 4096
```

Flash Attention significantly reduced prompt-processing throughput.

However, when the context size was increased to:

```text
-c 16384
```

Flash Attention became beneficial and increased prompt-processing throughput by approximately **48%**.

This demonstrates why benchmarking LLM optimizations using only a single configuration can be misleading.

A feature that appears slower under one workload may become significantly more useful as the workload scales.

---

## 6. KV-Cache Quantization

This experiment is particularly interesting when trying to push the context length of a Large Language Model on a GPU with limited VRAM.

My GPU has **8 GB of VRAM**, so reducing the memory consumed by the KV cache can make larger context windows more practical.

For this experiment, I used:

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    -c 8192 \
    -ctk q8_0 \
    -ctv q8_0
```

### Configuration

| Option      | Description                                         |
| ----------- | --------------------------------------------------- |
| `-ngl 99`   | Offload as many model layers as possible to the GPU |
| `-fa on`    | Enable Flash Attention                              |
| `-c 8192`   | Use an 8,192-token context window                   |
| `-ctk q8_0` | Quantize the Key cache using Q8_0                   |
| `-ctv q8_0` | Quantize the Value cache using Q8_0                 |

The important parameters for this experiment are:

```bash
-ctk q8_0
-ctv q8_0
```

These control the precision used to store the **Key (K)** and **Value (V)** tensors inside the KV cache.

---

### Test Prompts

I tested the configuration using both a simple and a more demanding prompt.

#### Simple Question

> **"What is a CPU cache?"**

Result:

```text
Prompt:      58.6 t/s
Generation:  54.9 t/s
```

#### Complex Question

> **"Explain in detail how a modern computer CPU works. Cover instruction fetching, decoding, execution, registers, ALUs, caches, branch prediction, pipelining, out-of-order execution, and memory access. Explain how these components interact during the execution of a program and provide a concrete example of executing a simple instruction sequence."**

Result:

```text
Prompt:      203.2 t/s
Generation:   52.4 t/s
```

---

### Results

| Prompt           | Prompt Processing |   Generation |
| ---------------- | ----------------: | -----------: |
| Simple question  |          58.6 t/s | **54.9 t/s** |
| Complex question |     **203.2 t/s** |     52.4 t/s |

An interesting observation is that generation speed remains relatively stable at approximately **52–55 tokens per second**, while prompt processing throughput changes significantly depending on the workload.

The important question now becomes:

> **Why can KV-cache quantization help when working with larger context windows?**

To answer that, we first need to understand what the **KV cache** actually is.

---

##  6.1 What is a KV Cache?

In the context of `llama.cpp`, **KV-cache quantization** means reducing the precision used to store the model's **Key (K)** and **Value (V)** attention cache.

It is particularly useful when running LLMs with large context windows.

During inference, a Transformer model uses an attention mechanism.

For every token, the model calculates three important representations:

```text
Q = Query
K = Key
V = Value
```

Conceptually:

```text
                Token Representation
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
           Query (Q)   Key (K)   Value (V)
```

When generating a new token, the model needs to compare the current token against information from previous tokens.

This is where the attention mechanism becomes computationally expensive.

---

## 6.2 The Problem Without a KV Cache

Imagine the model processing the following sequence:

```text
Token 1
Token 2
Token 3
Token 4
...
```

Without caching, the model would repeatedly need to recompute information for previously processed tokens.

Example:

```text
Token 1 → Compute K₁ and V₁

Token 2 → Recompute Token 1 + Compute Token 2

Token 3 → Recompute Token 1 + Token 2 + Compute Token 3

Token 4 → Recompute Token 1 + Token 2 + Token 3 + Compute Token 4
```

As the sequence grows, repeatedly recalculating this information becomes inefficient.

---

## 6.3 The Solution: KV Cache

Instead of recalculating the Keys and Values for every previous token, the model stores them in memory.

```text
KV Cache

Token 1 → [K₁, V₁]
Token 2 → [K₂, V₂]
Token 3 → [K₃, V₃]
Token 4 → [K₄, V₄]
...
Token N → [Kₙ, Vₙ]
```

When generating the next token, the model calculates a new Query:

```text
Q_new
```

and compares it against the previously stored Keys:

```text
Q_new × K₁
Q_new × K₂
Q_new × K₃
...
Q_new × Kₙ
```

This produces attention scores.

The attention scores are then applied to the stored Values:

```text
Attention Scores
        │
        ▼
Weighted Values
        │
        ▼
Attention Output
```

A simplified representation looks like this:

```text
                        Previous Tokens
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
                 Keys (K)            Values (V)
                    │                   │
                    │                   │
New Token ──► Query (Q)                │
                    │                   │
                    ▼                   │
                Q × Kᵀ                  │
                    │                   │
                    ▼                   │
             Attention Scores           │
                    │                   │
                    └───────────────────┘
                              │
                              ▼
                       Attention Output
```

The KV cache therefore avoids repeatedly recalculating Keys and Values that were already computed.

> **The KV cache does not store the original tokens themselves. It stores the computed Key and Value representations associated with previously processed tokens.**

A useful analogy is a CPU cache.

A CPU cache stores frequently needed data close to the processor to avoid repeatedly fetching it from slower memory.

Similarly, the KV cache stores previously computed attention information so the Transformer does not need to recompute it every time a new token is generated.

---

## 6.4 What is Quantization?

The KV cache solves the problem of repeatedly recomputing attention information.

However, it introduces another problem:

> **The KV cache can consume a significant amount of RAM or VRAM.**

This becomes increasingly important as the context window grows.

For example:

```text
Context length increases
        │
        ▼
More tokens stored
        │
        ▼
More Keys and Values stored
        │
        ▼
Larger KV cache
        │
        ▼
Higher VRAM consumption
```

This is where **quantization** becomes useful.

---

## 6.5 Precision vs Memory

Normally, numerical values inside neural networks can be stored using floating-point formats.

For example:

```text
FP32 → 32 bits per value
FP16 → 16 bits per value
```

KV-cache quantization allows these values to be stored using fewer bits.

For example:

```text
FP16
 │
 ├──► Q8_0 → approximately 8 bits per value
 │
 └──► Q4_0 → approximately 4 bits per value
```

This means we trade numerical precision for reduced memory consumption.

| Format | Approximate Bits per Value | Memory Usage | Precision |
| ------ | -------------------------: | ------------ | --------- |
| FP32   |                     32-bit | Very High    | Very High |
| FP16   |                     16-bit | High         | High      |
| Q8_0   |                     ~8-bit | Lower        | Very Good |
| Q4_0   |                     ~4-bit | Much Lower   | Lower     |

The exact memory usage depends on the quantization format because additional information, such as scaling factors, must also be stored.

---

## What Does Q8_0 Mean?

`Q8_0` is a quantization format used by `llama.cpp`.

```text
Q = Quantized
8 = Approximately 8 bits per value
0 = Quantization format variant
```

Similarly:

```text
Q4_0

Q = Quantized
4 = Approximately 4 bits per value
0 = Quantization format variant
```

Conceptually:

```text
FP16
16 bits per value
        │
        ▼
      Q8_0
~8 bits per value
        │
        ▼
      Q4_0
~4 bits per value
```

The lower the precision, the less memory is required.

However, aggressive quantization can introduce information loss.

---

## 6.6 Why Does This Matter for Large Context Windows?

Consider what happens when the context size increases.

```text
Context: 4,096 tokens

[K₁,V₁] [K₂,V₂] [K₃,V₃] ... [K₄₀₉₆,V₄₀₉₆]
```

Now increase the context:

```text
Context: 16,384 tokens

[K₁,V₁] [K₂,V₂] [K₃,V₃] ... [K₁₆₃₈₄,V₁₆₃₈₄]
```

The model must now store significantly more Key and Value data.

Using FP16:

```text
Large Context
      +
High Precision KV Cache
      =
High VRAM Consumption
```

Using quantization:

```text
Large Context
      +
Quantized KV Cache
      =
Lower VRAM Consumption
```

This is why KV-cache quantization is particularly useful on GPUs with limited memory.

For an **8 GB GPU**, reducing the KV-cache precision can free VRAM and make larger context windows more practical.

---

## 6.7 Model Quantization vs KV-Cache Quantization

It is important not to confuse these two concepts.

The model used in these experiments is:

```text
Mistral 7B Q4_K_M
```

The `Q4_K_M` part refers to **model weight quantization**.

This reduces the memory required to store the model itself.

KV-cache quantization is different:

```bash
-ctk q8_0
-ctv q8_0
```

This controls the precision of the temporary Key and Value data generated during inference.

```text
┌──────────────────────────────────────┐
│              GPU VRAM                │
├──────────────────────────────────────┤
│                                      │
│ Model Weights                        │
│ └── Q4_K_M                           │
│                                      │
├──────────────────────────────────────┤
│                                      │
│ KV Cache                             │
│ ├── Keys   → Q8_0                    │
│ └── Values → Q8_0                    │
│                                      │
├──────────────────────────────────────┤
│                                      │
│ Compute Buffers                      │
│ CUDA Memory                          │
│ Other Runtime Memory                 │
│                                      │
└──────────────────────────────────────┘
```

So:

| Type            | What is Quantized?                               |
| --------------- | ------------------------------------------------ |
| `Q4_K_M`        | Model weights                                    |
| `Q8_0` KV cache | Key and Value tensors generated during inference |

---

## 6.8 Final Takeaway

The KV cache exists to avoid repeatedly recalculating attention information for previously processed tokens.

However, as context length increases, the KV cache itself can consume a large amount of memory.

KV-cache quantization addresses this problem by reducing the precision used to store Keys and Values:

```text
Longer Context
      │
      ▼
More KV Data
      │
      ▼
Higher Memory Usage
      │
      ▼
KV Quantization
      │
      ▼
Reduced VRAM Consumption
      │
      ▼
Larger Context Windows Become More Practical
```

This demonstrates another important principle of LLM inference:

> **Performance optimization is often a trade-off between computation, memory usage, precision, and model quality.**


KV-cache quantization allows us to sacrifice a small amount of numerical precision in exchange for potentially significant memory savings, making it especially useful when experimenting with large context windows on GPUs with limited VRAM.

---

# 7. Increasing Batch Size

Batch size is another important parameter when experimenting with `llama.cpp`, particularly for **prompt processing**.

The `-b` option controls the **logical batch size**, which determines how many tokens can be processed together.

For example:

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    -b 512
```

Here:

```text
-b 512
```

sets the batch size to **512 tokens**.

For these experiments, I used the same two questions as in the previous experiments and changed the batch and micro-batch sizes to observe their effect on performance.

---

### Batch Size: 512

```text
-b 512
```

### Results:

| Prompt  | Prompt Processing | Generation |
| ------- | ----------------: | ---------: |
| Simple  |          55.8 t/s |   56.3 t/s |
| Complex |          39.9 t/s |   53.8 t/s |

---

### Batch Size: 1024

```text
-b 1024
```

### Results:

| Prompt  | Prompt Processing | Generation |
| ------- | ----------------: | ---------: |
| Simple  |          76.7 t/s |   55.9 t/s |
| Complex |          40.2 t/s |   42.4 t/s |

Increasing the batch size from **512 → 1024** improved prompt-processing throughput for the simple question:

```text
55.8 t/s → 76.7 t/s
```

However, the complex prompt showed almost no improvement:

```text
39.9 t/s → 40.2 t/s
```

Generation performance also remained relatively similar for the simple prompt.

---

### Batch Size: 2048

```text
-b 2048
```

### Results:

| Prompt  | Prompt Processing | Generation |
| ------- | ----------------: | ---------: |
| Simple  |          80.8 t/s |   55.6 t/s |
| Complex |          39.9 t/s |   54.5 t/s |

For the simple prompt, increasing the batch size further provided another improvement:

```text
55.8 t/s → 76.7 t/s → 80.8 t/s
```

However, the complex prompt again remained around:

```text
~40 t/s
```

This suggests that simply increasing the logical batch size does not necessarily improve prompt processing for every workload.

---

## 7.1 Micro-Batch Size

So far, the micro-batch size was automatically selected by `llama.cpp` and remained at:

```text
-ub 512
```

The micro-batch size controls how many tokens are actually processed by the computational kernels at a time.

This gives us another variable to experiment with.

Instead of only changing:

```text
-b
```

we can also explicitly control:

```text
-ub
```

---

### Batch 512 + Micro-Batch 256

We now use:

```text
-b 512
-ub 256
```

### Results:

| Prompt  | Prompt Processing | Generation |
| ------- | ----------------: | ---------: |
| Simple  |     **157.2 t/s** |   56.1 t/s |
| Complex |     **118.9 t/s** |   53.7 t/s |

This produces a surprisingly large improvement in prompt-processing throughput compared with the previous `-b 512` experiment:

### Simple prompt

```text
55.8 t/s → 157.2 t/s
```

### Complex prompt

```text
39.9 t/s → 118.9 t/s
```

Generation speed, however, remained almost unchanged.

This is an important observation:

> **Changing the micro-batch size had a much larger effect on prompt processing than on token generation in this experiment.**

---

## 7.2 Batch 2048 + Micro-Batch 1024

Finally, we tested:

```text
-b 2048
-ub 1024
```

### Results:

| Prompt  | Prompt Processing | Generation |
| ------- | ----------------: | ---------: |
| Simple  |     **202.7 t/s** |   56.1 t/s |
| Complex |          40.0 t/s |   54.7 t/s |

The simple prompt achieved the highest prompt-processing throughput of the experiment:

```text
202.7 t/s
```

while generation remained around:

```text
~55 t/s
```

However, the complex prompt remained at approximately:

```text
40 t/s
```

---

## 7.3 Complete Comparison

Putting all of the experiments together:

| Batch | Micro-Batch | Prompt  | Prompt Processing | Generation |
| ----: | ----------: | ------- | ----------------: | ---------: |
|   512 |         512 | Simple  |          55.8 t/s |   56.3 t/s |
|   512 |         512 | Complex |          39.9 t/s |   53.8 t/s |
|  1024 |         512 | Simple  |          76.7 t/s |   55.9 t/s |
|  1024 |         512 | Complex |          40.2 t/s |   42.4 t/s |
|  2048 |         512 | Simple  |          80.8 t/s |   55.6 t/s |
|  2048 |         512 | Complex |          39.9 t/s |   54.5 t/s |
|   512 |         256 | Simple  |     **157.2 t/s** |   56.1 t/s |
|   512 |         256 | Complex |     **118.9 t/s** |   53.7 t/s |
|  2048 |        1024 | Simple  |     **202.7 t/s** |   56.1 t/s |
|  2048 |        1024 | Complex |          40.0 t/s |   54.7 t/s |

---

## 7.4 What Do We Learn?

The results demonstrate that **batch size and micro-batch size can have very different effects on prompt processing and generation**.

For generation, the results are relatively stable:

```text
~53–56 t/s
```

Changing `-b` and `-ub` therefore had comparatively little effect on generation throughput in these experiments.

Prompt processing was much more sensitive.

For example:

```text
-b 512, -ub 512
→ 55.8 t/s
```

while:

```text
-b 512, -ub 256
→ 157.2 t/s
```

This is a significant difference.

Similarly:

```text
-b 2048, -ub 1024
→ 202.7 t/s
```

was the fastest prompt-processing result recorded in this experiment.

---

### The Interesting Part

One of the most interesting observations is that **increasing the logical batch size alone did not produce the same improvement as changing the micro-batch size**.

With the micro-batch fixed at 512:

```text
-b 512  → 55.8 t/s
-b 1024 → 76.7 t/s
-b 2048 → 80.8 t/s
```

There are diminishing returns.

However, explicitly changing the micro-batch size produced much larger changes:

```text
-b 512,  -ub 256  → 157.2 t/s
-b 2048, -ub 1024 → 202.7 t/s
```

This suggests that, for this particular hardware, model, context, and `llama.cpp` build, **micro-batch size is an important factor in prompt-processing performance**.

> **The optimal configuration is not necessarily the configuration with the largest batch size. The interaction between logical batch size, micro-batch size, GPU memory, and the workload determines the final performance.**

These results also reinforce an important principle from the previous experiments:

> **LLM inference performance cannot be evaluated using a single parameter in isolation.**

Batch size, micro-batch size, context length, Flash Attention, KV-cache precision, model quantization, and hardware characteristics can all interact with one another.

For this reason, benchmarking multiple configurations is essential when trying to find the optimal settings for a particular GPU and model.


# 8. How to Benchmark?

Throughout the previous experiments, we introduced several ways to configure `llama.cpp`:

* CPU vs GPU inference
* Different quantization levels
* Flash Attention
* Different context sizes
* Different batch and micro-batch sizes
* KV-cache quantization

But this raises an important question:

> **Which configuration is actually the most efficient for a particular workload?**

This is where **benchmarking** becomes useful.

Instead of manually asking the model questions and observing the results, `llama.cpp` provides a dedicated benchmarking utility called **`llama-bench`**.

It allows us to perform standardized tests and compare different configurations using measurable metrics such as:

* Prompt-processing throughput
* Token-generation throughput
* GPU layer offloading
* Model size
* Quantization
* Hardware capabilities

---

## 8.1 Running `llama-bench`

First, we can verify that the benchmark executable exists:

```bash
find build -type f -name "llama-bench"
```

Then run the benchmark:

```bash
./build/bin/llama-bench \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf
```

The output provides information about both the **hardware** and the **model**, followed by the measured inference performance.

For example:

```text
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 8150 MiB):
  Device 0: NVIDIA GeForce RTX 5050 Laptop GPU,
  compute capability 12.0,
  VMM: yes,
  VRAM: 8150 MiB
```

This tells us that `llama.cpp` detected the NVIDIA GPU and reports:

| Property           | Value                      |
| ------------------ | -------------------------- |
| GPU                | NVIDIA RTX 5050 Laptop GPU |
| Compute Capability | 12.0                       |
| VRAM               | ~8 GB                      |
| VMM                | Yes                        |

---

## 8.2 Understanding the Benchmark Output

A typical result looks like:

```text
| model                  |       size |     params | backend | ngl | test  |          t/s |
| ---------------------- | ---------: | ---------: | ------- | --: | ----- | -----------: |
| llama 7B Q4_K - Medium |   4.07 GiB |    7.24 B  | CUDA    |  -1 | pp512 | 1951.09 ± 59.90 |
| llama 7B Q4_K - Medium |   4.07 GiB |    7.24 B  | CUDA    |  -1 | tg128 |   57.72 ± 0.22 |
```

Let's break down the important columns.

### Model

```text
llama 7B Q4_K - Medium
```

This identifies the model architecture and quantization.

In this experiment, the model has approximately:

```text
7.24 billion parameters
```

and occupies approximately:

```text
4.07 GiB
```

of storage.

---

### Backend

```text
CUDA
```

This indicates that the benchmark is using the **NVIDIA GPU through CUDA**.

If the benchmark were running entirely on the CPU, the backend would be different.

---

### `ngl`

```text
ngl = -1
```

`ngl` refers to the **number of model layers that should be offloaded to the GPU**.

A value of:

```text
-ngl 0
```

means:

> Do not offload model layers to the GPU.

While:

```text
-ngl 99
```

requests that as many layers as possible be offloaded to the GPU.

`llama-bench` may display:

```text
ngl = -1
```

to represent its default configuration for maximum GPU offloading.

---

### `pp512`

```text
pp512
```

stands for:

> **Prompt Processing — 512 tokens**

The benchmark processes a 512-token prompt and measures how quickly the model can process it.

For example:

```text
pp512 → 1951.09 t/s
```

means that the model processed the benchmark's prompt workload at approximately:

```text
1,951 tokens/second
```

---

### `tg128`

```text
tg128
```

stands for:

> **Token Generation — 128 tokens**

The benchmark generates 128 tokens and measures generation throughput.

For example:

```text
tg128 → 57.72 t/s
```

means the model generated approximately:

```text
57.7 tokens/second
```

---

## 8.3 Prompt Processing vs Token Generation

The difference between `pp` and `tg` is important.

### Prompt Processing

The model can process many prompt tokens in parallel:

```text
Token 1 ─┐
Token 2 ─┤
Token 3 ─┤
Token 4 ─┤──► GPU ──► Processed Prompt
Token 5 ─┤
...      ─┘
```

This makes prompt processing highly suitable for GPU parallelism.

### Token Generation

Generation is autoregressive:

```text
Token 1
   ↓
Token 2
   ↓
Token 3
   ↓
Token 4
   ↓
...
```

The next token depends on the previous tokens.

As a result, generation is generally much harder to parallelize and is often considerably slower than prompt processing.

This explains why we can see results such as:

```text
Prompt Processing: ~1951 t/s
Generation:          ~58 t/s
```

without anything being wrong with the benchmark.

---

# 8.4 Benchmarking GPU Layer Offloading

One of the most useful experiments is determining how the number of GPU-offloaded layers affects performance.

We can test multiple values in a single command:

```bash
./build/bin/llama-bench \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 0,10,20,30,40,99
```

This allows us to compare:

```text
0 layers
10 layers
20 layers
30 layers
40 layers
99 layers
```

offloaded to the GPU.

---

## Results

| GPU Layers (`ngl`) |        Prompt Processing |           Generation |
| -----------------: | -----------------------: | -------------------: |
|                  0 |       669.91 ± 10.84 t/s |      7.79 ± 0.76 t/s |
|                 10 |       744.26 ± 40.05 t/s |     10.24 ± 1.30 t/s |
|                 20 |      1081.80 ± 38.71 t/s |     15.93 ± 2.16 t/s |
|                 30 |      1552.03 ± 66.57 t/s |     32.93 ± 7.33 t/s |
|                 40 | **1994.04 ± 146.15 t/s** | **57.47 ± 0.21 t/s** |
|                 99 |     1985.71 ± 137.90 t/s | **57.55 ± 0.15 t/s** |

The complete output from `llama-bench` was:

```text
| model                          |       size |     params | backend    | ngl |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | --------------: | -------------------: |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |   0 |           pp512 |       669.91 ± 10.84 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |   0 |           tg128 |          7.79 ± 0.76 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  10 |           pp512 |       744.26 ± 40.05 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  10 |           tg128 |         10.24 ± 1.30 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  20 |           pp512 |      1081.80 ± 38.71 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  20 |           tg128 |         15.93 ± 2.16 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  30 |           pp512 |      1552.03 ± 66.57 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  30 |           tg128 |         32.93 ± 7.33 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  40 |           pp512 |     1994.04 ± 146.15 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  40 |           tg128 |         57.47 ± 0.21 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  99 |           pp512 |     1985.71 ± 137.90 |
| llama 7B Q4_K - Medium         |   4.07 GiB |     7.24 B | CUDA       |  99 |           tg128 |         57.55 ± 0.15 |
```

---

# 8.5 What Do the Results Tell Us?

The results show a very clear relationship between **GPU layer offloading and inference performance**.

With:

```text
ngl = 0
```

the model does not offload layers to the GPU.

Generation throughput is only:

```text
7.79 t/s
```

As we progressively increase the number of GPU layers:

```text
ngl 0  →  7.79 t/s
ngl 10 → 10.24 t/s
ngl 20 → 15.93 t/s
ngl 30 → 32.93 t/s
ngl 40 → 57.47 t/s
```

Generation performance increases dramatically.

The largest improvement occurs between approximately **30 and 40 GPU layers**.

Once we reach:

```text
ngl = 40
```

generation reaches approximately:

```text
57.47 t/s
```

Increasing the requested number of layers further:

```text
ngl = 99
```

does not provide a meaningful additional improvement:

```text
57.47 t/s → 57.55 t/s
```

This indicates that, for this model and hardware configuration, the model is already effectively fully GPU-offloaded by around **40 layers**.

---

## Prompt Processing

Prompt-processing performance follows a similar trend:

```text
ngl = 0   → 669.91 t/s
ngl = 10  → 744.26 t/s
ngl = 20  → 1081.80 t/s
ngl = 30  → 1552.03 t/s
ngl = 40  → 1994.04 t/s
ngl = 99  → 1985.71 t/s
```

Again, performance increases significantly as more computation is moved to the GPU.

Interestingly, going from:

```text
ngl = 40
```

to:

```text
ngl = 99
```

does **not** improve performance.

In fact, the measured prompt throughput is slightly lower:

```text
1994.04 t/s → 1985.71 t/s
```

This difference is small and is within the benchmark's reported variation.

Therefore, we should not interpret it as `ngl 99` being slower in a meaningful way.

---

# 8.6 Main Conclusion

This experiment demonstrates why benchmarking is important.

Simply assuming:

> **"More GPU layers must always mean better performance."**

is not necessarily correct.

Instead, we want to find the point where the model is sufficiently GPU-offloaded to achieve maximum performance without unnecessarily consuming resources.

For this particular model and GPU:

```text
ngl = 0
      ↓
CPU-heavy execution

ngl = 10
      ↓
Small improvement

ngl = 20
      ↓
Significant improvement

ngl = 30
      ↓
Large improvement

ngl = 40
      ↓
~Maximum performance

ngl = 99
      ↓
No meaningful additional improvement
```

The results therefore suggest that **around 40 GPU layers are sufficient to reach essentially the maximum performance available for this model on this GPU**.

This is a good example of why `llama-bench` is useful: instead of guessing which configuration is optimal, we can systematically test different configurations and measure the results.

> **The optimal configuration is not simply the one with the largest number of GPU layers. It is the configuration that provides the desired performance while fitting within the available hardware resources.**

For future experiments, the same methodology can be applied to other parameters such as:

```text
Batch size
Micro-batch size
Context size
Flash Attention
KV-cache quantization
Model quantization
```

This allows us to build a much clearer picture of how each component affects local LLM inference performance.


> **Note:** All benchmark results in this README are experimental measurements from my own hardware and setup. They should be treated as observations rather than universal performance figures.

