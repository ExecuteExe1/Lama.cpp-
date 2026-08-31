
# 1 How to Benchmark?

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

## 1.1 Running `llama-bench`

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

## 1.2 Understanding the Benchmark Output

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

## 1.3 Prompt Processing vs Token Generation

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

# 2. Benchmarking GPU Layer Offloading

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

## 2.1 Results

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

## 2.2 What Do the Results Tell Us?

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

## 2.3 Prompt Processing

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

## 2.4 Conclusion

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

---

 # 3 Benchmarking Batch Size

For this experiment, we kept the rest of the configuration constant:

* **Model:** Mistral 7B Q4_K_M
* **GPU:** NVIDIA RTX 5050 Laptop GPU
* **GPU layers:** `ngl 99`
* **Flash Attention:** Enabled
* **Prompt size:** 512 tokens
* **Generation:** Disabled

The command used was:

```bash
./build/bin/llama-bench \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa 1 \
    -p 512 \
    -n 0 \
    -b 128,256,512,1024,2048
```

The only variable we changed was the **batch size**.

---

## 3.1 Results

| Batch Size | Flash Attention | Prompt Size |       Prompt Processing |
| ---------: | :-------------: | ----------: | ----------------------: |
|        128 |        ON       |  512 tokens |    1841.67 ± 101.75 t/s |
|        256 |        ON       |  512 tokens |     1976.93 ± 77.63 t/s |
|        512 |        ON       |  512 tokens |     2004.38 ± 40.27 t/s |
|       1024 |        ON       |  512 tokens | **2006.84 ± 49.75 t/s** |
|       2048 |        ON       |  512 tokens |     2006.51 ± 55.79 t/s |

---

### What Do We Notice?

At first, increasing the batch size produces a noticeable improvement in prompt-processing throughput.

```text
Batch 128
    ↓
1841.67 t/s

Batch 256
    ↓
1976.93 t/s

Batch 512
    ↓
2004.38 t/s
```

However, after approximately **512 tokens**, the performance essentially stops increasing.

```text
512  → 2004.38 t/s
1024 → 2006.84 t/s
2048 → 2006.51 t/s
```

The difference between a batch size of `512` and `2048` is therefore extremely small.

---

## 3.2 Diminishing Returns

We can see the effect more clearly by comparing the relative improvement.

From:

```text
-b 128 → -b 256
```

prompt throughput increases from:

```text
1841.67 → 1976.93 t/s
```

which is approximately:

```text
+7.3%
```

From:

```text
-b 256 → -b 512
```

we get:

```text
1976.93 → 2004.38 t/s
```

or approximately:

```text
+1.4%
```

But from:

```text
-b 512 → -b 1024
```

the improvement is only:

```text
2004.38 → 2006.84 t/s
```

or approximately:

```text
+0.1%
```

Finally:

```text
-b 1024 → -b 2048
```

actually produces an almost negligible decrease:

```text
2006.84 → 2006.51 t/s
```

This difference is tiny and falls within the benchmark's reported variation.

---

## 3.3 Why Doesn't a Larger Batch Keep Increasing Performance?

A larger batch gives the GPU more work to process in parallel.

Initially, this can improve GPU utilization:

```text
Small Batch
     ↓
Less parallel work
     ↓
GPU is not fully utilized
```

Increasing the batch size:

```text
Larger Batch
     ↓
More parallel work
     ↓
Better GPU utilization
     ↓
Higher throughput
```

However, eventually the GPU reaches a point where it is already being utilized efficiently.

At that point, adding more tokens to the batch does not provide much additional performance.


This is known as **diminishing returns**.

---

## 3.5 The Sweet Spot

For this particular configuration, a batch size of approximately:

```text
-b 512
```

already gets us extremely close to the maximum measured prompt-processing throughput.

```text
-b 512  → 2004.38 t/s
-b 1024 → 2006.84 t/s
-b 2048 → 2006.51 t/s
```

Therefore, increasing the batch from `512` to `1024` or `2048` provides essentially no practical performance benefit in this benchmark.

This is useful because a larger batch can require additional memory.

If two configurations provide almost identical throughput:

```text
-b 512   → ~2004 t/s
-b 2048  → ~2007 t/s
```

there is little reason to use the larger batch unless another workload benefits from it.

> ⚠️ These results are specific to this model, GPU, Flash Attention configuration, prompt size, and `llama.cpp` build. Different models and hardware may have a different optimal batch size.

---
# 4. Getting llama.cpp's internal timings:

```bash
./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    --perf
```
This allows you to take a look into

```text
load time
prompt eval time
prompt eval rate
eval time
eval rate
total time
```

This is much more useful than simply watching CPU utilization.





