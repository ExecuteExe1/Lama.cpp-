# This is what the directory looks like

```
ggml/
└── src/
    ├── ggml-threading.cpp     CPU threading and parallel execution
    ├── ggml.cpp               Core tensor operations and computation graph
    ├── gguf.cpp               GGUF model format and metadata handling
    ├── ggml-cpu/              CPU backend and CPU optimizations
    └── ggml-cuda/             CUDA backend and GPU kernels/optimizations
```
 # A small and simple diagram to understand how things work

                 llama.cpp
                     │
                     ▼
              GGML computation graph
                     │
                     ▼
                 ggml.cpp
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     CPU backend           CUDA backend
          │                     │
          ▼                     ▼
     ggml-cpu/              ggml-cuda/
          │                     │
          ▼                     ▼
    CPU instructions       CUDA kernels
          │                     │
          ▼                     ▼
         CPU                  GPU
