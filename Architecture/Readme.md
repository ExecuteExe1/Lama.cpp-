## Repository Architecture

The `llama.cpp` repository is organized into several components, each responsible for a different part of the inference stack.

```text
llama.cpp/
│
├── AGENTS.md                 
├── AUTHORS 
├── CLAUDE.md   
├── CMakepresets.json  
├── CODEOWNERS
│
├── CONTRIBUTING.md       
├── LICENSE   
├── Makefile                        
├── README.md
├── SECURITY.md
├── annotate.txt 
├── app
│    ├── CMakeLists.txt  It tells CMake what source files to compile, what libraries to link, and what the resulting executable should be called.
|    ├── download.cpp  Parse the user's model source → download/resolve the model → verify that it exists → print the resulting model path
|    └── llama.cpp  So this file is basically a dispatcher/router.
│     
├── benches               
│    ├── dgx-spark      Contains files that are benchmark results from llama.cpp, specifically related to running the AIME 2025 benchmark with GPT-OSS-120B on an NVIDIA DGX Spark.
|    ├── mac-m2-ultra  It is benchmarking llama.cpp's Metal backend on Apple Silicon
|    └── nemotron This is a benchmark report for llama.cpp running on an NVIDIA DGX Spark, whose GPU is the NVIDIA GB10, using the CUDA backend
|
├── build   COntains all the dependencies and everything we built when we ran the set up and running commands!
│       
├── build-xcframework.sh  This script takes the llama.cpp source code and builds an Apple-compatible llama.xcframework, so developers can use llama.cpp inside iOS/macOS/visionOS/tvOS applications
├── build_error.txt  Self-explainatory
├── ci   This CI implements heavy-duty workflows that run on self-hosted runners. Typically the purpose of these workflows is to
|        cover hardware configurations that are not available from Github-hosted runners and/or require more computational
|        resource than normally available.
|        It is a good practice, before publishing changes to execute the full CI locally on your machine. For example:
|
├── cmake  That cmake/ directory contains CMake configuration modules and templates used by llama.cpp to configure builds for different platforms and architectures.
├── common   The common/ directory contains the shared, higher-level functionality used by llama.cpp applications. It sits above the core llama/ggml libraries and provides |            reusable code for things like command-line arguments, chat handling, sampling, JSON, grammar processing, downloading models, logging, and speculative decoding.
|
├── conversion   This directory is essentially the Hugging Face → GGUF conversion layer of llama.cpp.Handles different model architectures and transform their Hugging  
|                  Face/PyTorch representation into the GGUF format that llama.cpp can load at runtime
|
├── convert_hf_to_gguf.py    This file is the orchestrator that decides which converter to use, what format to produce, where to save it, and which options to apply.
├── convert_hf_to_gguf_update.py  This file  makes sure llama.cpp tokenizes text the same way as the original Hugging Face model.
├── convert_llama_ggml_to_gguf.py  This script converts legacy GGML model files into the newer GGUF format, reading the old model’s hyperparameters, vocabulary, and 
|                                  tensors, then rewriting them with GGUF metadata and tensor structure so they can be used by modern llama.cpp
|
├── convert_lora_to_gguf.py  This script converts a Hugging Face PEFT/LoRA adapter into a GGUF adapter file, reading the LoRA A/B weight matrices and base-model  
|                            configuration, then storing them with the necessary metadata so llama.cpp can apply the adapter to a compatible base model
|
├── docs            Primarily documentation that explains how the corresponding parts of llama.cpp work
├── examples        Contains small programs, scripts, and demonstrations showing how to use or test different llama.cpp capabilities
├── flake.nix       The flake interface to llama.cpp's Nix expressions. The flake is used as a more discoverable entry-point, as well as a way to pin the dependencies and
|                   expose default outputs, including the outputs built by the CI.
|
├── ggml            Cuda Magic
├── gguf-py
├── grammars
├── include
├── licenses
├── media
├── models
├── mypy.ini
├── perf.data
├── pocs
├── pyproject.toml
├── pyrightconfig.json
├── requirements
├── requirements.txt
├── scripts
├── skills
├── src
├── test_cude.nsys-rep
├── tests
├── tools
├── ty.toml
└── vendor
