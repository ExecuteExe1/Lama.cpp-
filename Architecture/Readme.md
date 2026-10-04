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
├── build 
│       
├── build-xcframework.sh 
├── build_error.txt
├── ci
├── cmake
├── common
├── conversion
├── convert_hf_to_gguf.py 
├── convert_hf_to_gguf_update.py
├── convert_llama_ggml_to_gguf.py
├── convert_lora_to_gguf.py
├── docs
├── examples
├── flake.nix
├── ggml
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
