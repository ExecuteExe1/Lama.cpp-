# Llama.cpp + CUDA Setup Guide

A quick guide to installing **[llama.cpp](https://github.com/ggml-org/llama.cpp)** and building it with **CUDA support** on WSL.

> Follow the steps below in order, then you're ready to start experimenting with local LLMs!

## Prerequisites

This guide assumes you are using:

* 🐧 **WSL2 / Ubuntu**
* 🎮 An **NVIDIA GPU**
* 🛠️ NVIDIA drivers with CUDA support

## Step1 Install Dependencies

First, update your package lists and install the required build tools:

```bash
sudo apt update
sudo apt install -y git cmake build-essential
sudo apt install -y libssl-dev
```

### Step 2 Check Your GPU & CUDA

You can verify that your NVIDIA GPU is visible from WSL with:

```bash
nvidia-smi
```

Check whether the CUDA compiler is installed:

```bash
nvcc --version
```

If `nvcc` is not available, you can install the CUDA toolkit with:

```bash
sudo apt install -y nvidia-cuda-toolkit
```

>  **Note:** For newer NVIDIA/WSL setups, the recommended CUDA installation method may differ from the Ubuntu repository package. Make sure your NVIDIA driver and CUDA toolkit versions are compatible.

---

## Step3 Clone llama.cpp

Navigate to the directory where you want to store the project:

```bash
cd ~/Work
```

Clone the repository:

```bash
git clone https://github.com/ggml-org/llama.cpp.git
```

Enter the project directory:

```bash
cd llama.cpp
```

---

## Step4 ⚙️ Configure the Build with CUDA

Configure `llama.cpp` with CUDA support enabled:

```bash
cmake -B build -DGGML_CUDA=ON
```

If CMake completes successfully, you're ready to build.

---

## Step5 Build llama.cpp

You can speed up compilation by allowing CMake to use multiple CPU cores.

First, check how many processors WSL can see:

```bash
nproc
```

For example, if WSL reports **4 cores**, build using:

```bash
cmake --build build --config Release -j4
```

You can replace `4` with the number returned by `nproc`:

```bash
cmake --build build --config Release -j$(nproc)
```

---

## Step6 Verify the Build

Once compilation finishes, you should have the llama.cpp executables inside:

```text
build/bin/
```

For example:

```bash
ls build/bin
```

You can then run the CLI tools from there.

---

## Finish

You now have **llama.cpp built with CUDA support** and are ready to run GGUF models locally.

### What's next?

* Download a **GGUF** model
*  Run a model with `llama-cli`
*  Experiment with GPU offloading
*  Profile performance with tools such as `perf` or **NVIDIA Nsight Systems**
*  Experiment with different quantization levels

Good Luck on your studies/experiments!










 

