# Firstly you will need a model,you may search on:

https://huggingface.co/spaces

It offers a wide sections of different local llms supported by llama.cpp so feel free to search what fits better your needs.
For my study and documentation i chose the following:

https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/blob/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf

Now once we place our model into the folder /models it is time to run it with different ways.

# CPU only

 ./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 0

My own device uses the AMD Ryzen 7 260 w/ Radeon 780M Graphics CPU and a simple question such as 
"What is a cpu cache" took :
 Prompt: 18.5 t/s → llama.cpp processed your input prompt at 18.5 tokens per second.
 Generation: 11.2 t/s → once it started answering, it generated 11.2 new tokens per second.

 To see the real time of the model all you need to do is add time on the original command, with only the use of my CPU it took the LLM 45seconds to Load and answer the promt

 # GPU Accelaration 

  ./build/bin/llama-cli \
    -m models/mistral/mistral-7b-instruct-v0.2.Q4_K_M.gguf \
    -ngl 99

My own device uses an RTX 5050 with an 8GB Ram as a result with the same simple question we get:
 Prompt: 187.9 t/s → llama.cpp processed your input prompt at 18.5 tokens per second.
 Generation: 56.7 t/s → once it started answering, it generated 11.2 new tokens per second.

 When it comes to real time of course it outperformed our previous run,adding time on our previous command we get the astonishing time of 10seconds.
 Almost 5 times faster than our original CPU based run

 Clarification the  -ngl flag shows how many GPU layers the llama.cpp uses and 99 is automatically maxing out the capabilities of your model or GPU while 0 means no use of the GPU 

 # Flash Attention & Context size

  introducing a new flag -fa on

It basically activated the model's attention computation

What does this mean?

Well Mistral is a transformer which means it repeatedly performs attentions
Q[Query]  K[Key]    V[Value]
|           |          |
------------|-----------
            |
            Attention ----->New Hidden States

The straightforward attention implementation may require a lot of memory especially if the context gets longer.
Flash attention can
Improve inference speed
Reduce memory/VRAM usage
Allow larger context sized

 Let's run an experiment again with a more complex question such as
 
 "Explain in detail how a modern computer CPU works. Cover instruction fetching, decoding, execution, registers, ALUs, caches, branch prediction, pipelining, out-of-order execution, and memory access. Explain how these components interact during the execution of a program and provide a concrete example of executing a simple instruction sequence."
 
 With flag -fa on and -c 4096 [where c is the context size]
  Prompt: 40.3 t/s | Generation: 55.0 t/s

with flag -fa off and -c 4096
 Prompt: 179.0 t/s | Generation: 51.8 t/s 

We notice that FA ON slightly improves generation throughput (+6.2%), but dramatically decreases prompt-processing throughput (~77.5% slower).

What if we increase the size? 

With flag -fa on and -c 16384 [where c is the context size]
[ Prompt: 115.1 t/s | Generation: 55.2 t/s ]

With flag -fa off and -c 16384 [where c is the context size]
[ Prompt: 77.6 t/s | Generation: 51.8 t/s ]

Here is the beauty of the Flash Attention,with the increase of context the prompt-processing throughput improved 
 
