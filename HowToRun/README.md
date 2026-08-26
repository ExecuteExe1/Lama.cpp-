# llama.cpp — Local LLM Experimentation & Benchmarking

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

## ⏱️ Measuring Total Runtime

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

# 5. Context Size Experiment

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

# 6. Increasing the Context Size

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

# 7. Benchmark Results

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

# 8. Conclusions

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

##  What I Want to Explore Next

Some possible future experiments:

* [ ] Test larger context sizes such as `32768`
* [ ] Compare different quantization formats
* [ ] Compare different Mistral models
* [ ] Measure VRAM usage
* [ ] Measure CPU utilization
* [ ] Profile llama.cpp with `perf`
* [ ] Profile CUDA execution with NVIDIA Nsight Systems
* [ ] Investigate individual `ggml` operations
* [ ] Compare different GPU layer counts with `-ngl`
* [ ] Test batch size effects
* [ ] Compare different sampling configurations

---

##  Useful Resources

* [llama.cpp](https://github.com/ggml-org/llama.cpp)
* [Hugging Face](https://huggingface.co/)
* [Mistral 7B Instruct v0.2 GGUF](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF)

---

> **Note:** All benchmark results in this README are experimental measurements from my own hardware and setup. They should be treated as observations rather than universal performance figures.

