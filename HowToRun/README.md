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
* [6. Benchmark Results](#6-benchmark-results)
* [7. Conclusions](#7-conclusions)

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

##  Measuring Total Runtime

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

##  What is Flash Attention?

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

## Context Size Experiment

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

## Context Size: 16384

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

## Benchmark Results

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

## Conclusions

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

## What is a KV Cache?

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

## The Problem Without a KV Cache

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

## The Solution: KV Cache

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

## What is Quantization?

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

## Precision vs Memory

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

## Why Does This Matter for Large Context Windows?

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

## Model Quantization vs KV-Cache Quantization

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

## Final Takeaway

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




> **Note:** All benchmark results in this README are experimental measurements from my own hardware and setup. They should be treated as observations rather than universal performance figures.

