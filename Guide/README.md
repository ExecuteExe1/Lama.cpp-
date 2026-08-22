# This is a Guide on how to install Lama.cpp and its dependencies.
# Follow each step and be ready to experiment!

# Step 1 "Dependencies"

on WSL run

sudo apt update 
sudo apt install -y git cmake build-essential
sudo apt install -y libssl-dev

In case you need to check your Graphics card and cuda tools:

 nvidia-smi
 nvcc --version

 if the cuda-toolbox is NOT installed you can install it by:
sudo apt install nvidia-cuda-toolkit

# Step 2 Llama.cpp installation
 Go to your desired folder and hit

 git clone https://github.com/ggml-org/llama.cpp.git

 get into the folder with cd llama.cpp

# Step 3 Build with Cuda

Run 

cmake -B build -DGGML_CUDA=ON

Now check how many cores are given to your WSL so you can run the next command much faster.I had 4 cores therefore:

cmake --build build --config Release -j4










 

