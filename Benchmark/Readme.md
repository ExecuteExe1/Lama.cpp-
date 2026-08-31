
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
./build/bin/llama-simple \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    -n 50 \
    -p "What is a CPU cache?"
```
This allows you to take a look into

```text
load time
prompt eval time
prompt eval rate
eval time
eval rate
total time
graphs reused
```

>Note:Pay attention now we used the \bin\llama-simple,which is a function that allows us to see the internal timings.

# 5. Using `perf`

[`perf`](https://perf.wiki.kernel.org/) is a Linux performance-analysis tool that can be used to investigate where `llama.cpp` spends its CPU time and to examine the instructions involved in those operations.

For this experiment, we will use three main `perf` commands:

1. `perf stat` → CPU performance counters and overall execution statistics
2. `perf record` + `perf report` → identify where CPU execution time is being sampled
3. `perf annotate` → inspect the assembly instructions responsible for those samples

---

## 5.1 `perf stat`

First, we can use `perf stat` to collect hardware and software performance counters while running `llama.cpp`.

```bash
perf stat ./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    --perf \
    -st \
    -p "What is a CPU cache?"
```

### Example output

```text
Performance counter stats for './build/bin/llama-cli ...':

       11601707626      task-clock:u
                 0      context-switches:u
                 0      cpu-migrations:u
             55454      page-faults:u

       30469728975      instructions:u
       13777862913      cycles:u
        2104724479      stalled-cycles-frontend:u
        6348974294      branches:u
           6799744      branch-misses:u

      12.573185166 seconds time elapsed

       6.480973000 seconds user
       5.067500000 seconds sys
```

### What do these counters tell us?

`perf stat` provides several different measurements describing how the CPU executed the program.

| Counter                     | Description                                                                 |
| --------------------------- | --------------------------------------------------------------------------- |
| **task-clock**              | Amount of CPU time consumed by the process                                  |
| **context-switches**        | Number of times execution was switched between tasks                        |
| **cpu-migrations**          | Number of times the process moved between CPU cores                         |
| **page-faults**             | Number of page faults generated by the process                              |
| **instructions**            | Number of CPU instructions executed                                         |
| **cycles**                  | Number of CPU clock cycles                                                  |
| **stalled-cycles-frontend** | Cycles where the CPU frontend was unable to supply instructions efficiently |
| **branches**                | Number of branch instructions executed                                      |
| **branch-misses**           | Number of incorrectly predicted branches                                    |
| **time elapsed**            | Wall-clock time from start to completion                                    |
| **user**                    | CPU time spent executing user-space code                                    |
| **sys**                     | CPU time spent executing kernel/system code                                 |

For example, the output above shows:

```text
30,469,728,975 instructions
13,777,862,913 cycles
```

giving approximately:

```text
2.21 instructions per cycle
```

It also shows that approximately **15.28% of frontend cycles were stalled**, while only **0.11% of branch predictions were missed**.

This gives us a useful overview of the CPU's behavior during the `llama.cpp` execution.

> **Note:** These measurements describe CPU activity. Since `-ngl 99` offloads the model computation to the GPU, `perf stat` does not provide detailed CUDA-kernel execution times. For detailed GPU analysis, tools such as NVIDIA Nsight Systems or Nsight Compute are more appropriate.

---

## 5.2 `perf record` and `perf report`

While `perf stat` gives us overall performance counters, `perf record` allows us to **sample where the program is spending its CPU execution time**.

Run:

```bash
perf record ./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99 \
    -fa on \
    --perf \
    -st \
    -p "What is a CPU cache?"
```

This generates a `perf.data` file containing the collected samples.

We can then inspect the samples using:

```bash
perf report
```

### Example

```text
37.32%  llama-cli  libcuda.so.1.1  [.] 0x0000000000587ed6
 7.37%  llama-cli  libcuda.so.1.1  [.] 0x00000000003d64c0
 4.97%  llama-cli  libcuda.so.1.1  [.] 0x0000000000587ee1
 4.04%  llama-cli  libcuda.so.1.1  [.] 0x00000000003d64cc
 3.44%  llama-cli  libcuda.so.1.1  [.] 0x00000000003d64d8
 2.84%  llama-cli  libcuda.so.1.1  [.] 0x0000000000e9d755
 2.40%  llama-cli  libcuda.so.1.1  [.] 0x00000000003d64d4
```

The report tells us where the collected CPU samples occurred.

The main columns can be interpreted as:

```text
Percentage →  Object → Shared library → Symbol/address
```

For example:

```text
37.32%  llama-cli  libcuda.so.1.1  [.] 0x0000000000587ed6
```

means that approximately **37.32% of the collected samples** were observed at that location inside `libcuda.so.1.1`.

### What can we learn?

`perf report` can help identify:

* Which functions consume the most CPU samples
* Whether execution is concentrated inside `llama.cpp`
* Whether significant CPU activity occurs inside CUDA libraries
* Which shared libraries are involved
* Which functions may deserve further investigation

In this particular GPU-offloaded experiment, many samples appear inside:

```text
libcuda.so.1.1
```

This indicates substantial CPU-side activity inside the NVIDIA CUDA driver while `llama.cpp` is interacting with the GPU.

> **Important:** A percentage shown by `perf report` is a **sampling percentage**, not an exact measurement of wall-clock execution time. For example, `37.32%` means approximately 37.32% of the collected samples landed at that location.

---

## 5.3 `perf annotate`

After identifying an interesting function or address with `perf report`, we can use:

```bash
perf annotate
```

to investigate it at a much lower level.

`perf annotate` takes the samples collected by `perf record` and maps them to the corresponding source code or assembly instructions when debugging symbols are available.

### Example

```text
      push     %r15
        │
        │      mov      %rsi,%r15
        │      push     %r14
        │      mov      %edx,%r14d
        │      push     %r13
        │      push     %r12
        │      push     %rbp
        │      push     %rbx
        │      mov      %rdi,%rbx
        │      mov      %rsi,%rdi
```

This allows us to move from:

```text
Program
   ↓
Function
   ↓
Source code
   ↓
Assembly instruction
```

and investigate what is happening at the instruction level.

For example, we can examine instructions such as:

```text
mov
push
pop
call
jmp
cmp
test
```

and identify branches, function calls, register operations, memory accesses, and other low-level operations.

### Important interpretation

The percentages shown by `perf annotate` represent **where the profiler's samples landed**.

They should therefore not be interpreted as:

> "This individual instruction takes exactly X% of the execution time."

Instead, they indicate that the instruction was frequently observed when `perf` sampled the CPU.

---

## 5.4 The relationship between the three tools

The three commands provide progressively more detailed information:

```text
                 perf stat
                     │
                     ▼
          Overall CPU performance
                     │
                     ▼
              perf report
                     │
                     ▼
          Functions / libraries
                     │
                     ▼
             perf annotate
                     │
                     ▼
          Source / Assembly level
```

In other words:

### `perf stat`

Answers:

> **"How did the CPU perform overall?"**

### `perf report`

Answers:

> **"Where were the CPU samples concentrated?"**

### `perf annotate`

Answers:

> **"What instructions are responsible for those samples?"**

Together, these tools allow us to move from a high-level view of CPU performance down to the assembly level.

---


# Using Nvidia Nightshade

Firstly you need to have installed on your laptop/Pc the Nvidia Nightshade to see the analytical reports after running the commands:

```bash
nsys profile     --trace=cuda,nvtx,osrt     --output=llama_fa_on     ./build/bin/llama-cli     -m models/mistral-7b-instruct-v0.2.Q4_K_M.gguf     -ngl 99     -fa on     -st     -n 50     -p "What is a CPU cache?"
```

Then you find your report file and open it in Nvidia Nightshade for a detailed report of your GPU usage



## 5.5 CPU profiling vs GPU profiling

Because our `llama.cpp` command uses:

```bash
-ngl 99
```

most of the neural-network computation is being offloaded to the GPU.

Therefore, `perf` primarily shows us the **CPU side** of the execution, including CPU work performed while interacting with CUDA.

It does **not** provide a detailed breakdown such as:

```text
Flash Attention kernel    0.82 ms
Matrix multiplication     1.74 ms
RoPE kernel               0.12 ms
Softmax                   0.31 ms
```

For this type of GPU-level analysis, NVIDIA profiling tools are more appropriate:

```text
NVIDIA Nsight Systems
        ↓
GPU/CPU timeline and CUDA kernel durations

NVIDIA Nsight Compute
        ↓
Individual CUDA kernel performance
```

Therefore, for the Flash Attention experiment, `perf` and NVIDIA's profiling tools complement each other:

```text
                 llama.cpp
                     │
          ┌──────────┴──────────┐
          │                     │
         CPU                   GPU
          │                     │
        perf                  Nsight
          │                     │
   ┌──────┴──────┐       ┌──────┴──────┐
   │             │       │             │
perf stat    perf report  nsys          ncu
   │             │       │             │
Counters      Functions  Timeline      Kernels
                         & timings     & metrics
```

This combination gives us a much more complete picture of where the execution time is going and how Flash Attention affects the CPU/GPU workload.


