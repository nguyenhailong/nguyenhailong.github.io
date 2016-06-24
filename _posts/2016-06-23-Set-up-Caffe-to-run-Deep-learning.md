---
layout: post
title: "Set up Caffe to run Deep learning"
author: long_nguyen
modified:
excerpt: "Set up Caffe to run Deep learning"
tags: []
---
Caffe is a fast high performance Deep neural network library. It requires Nvidia GPU with CUDA support. 

# Install CUDA
Download CUDA for Linux from here:
(https://developer.nvidia.com/cuda-downloads). For me, I download runfile (local)

Run `sudo sh cuda_7.5.18_linux.run`. Follow instructions.

Then, you may encounter this error: "You appear to be running an X server".
Follow this link to fix it: (http://askubuntu.com/questions/149206/how-to-install-nvidia-run)

1. Hit CTRL+ALT+F1 and login using your credentials.
2. Kill your current X server session by typing sudo service lightdm stop or sudo stop lightdm
3. Enter runlevel 3 by typing sudo init 3 and install your *.run file.
4. You might be required to reboot when the installation finishes. If not, run sudo service lightdm start or sudo start lightdm to start your X server again.

# Install cuDNN

cuDNN is used by caffe for GPU acceleration, provided by CUDA, very much recommended for speed. Go to (https://developer.nvidia.com/cuDNN), download .tar.gz file and extract.
There are 2 things to do with this.

1. Copy all the files, (except cudnn.h) to /usr/local/cuda-6.5/lib64
2. Copy the cudnn.h to /usr/local/cuda-6.5/include

Again, you may have an error here: "ImportError: libcudart.so.7.0: cannot open shared object file: No such file or directory".

Run the following command:

1. `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64`
2. `sudo ldconfig /usr/local/cuda/lib64`

# Install Caffe
1. Clone Caffe from GitHub (https://github.com/BVLC/caffe) and follow the installation instructions (http://caffe.berkeleyvision.org/installation.html)
2. Install all the dependencies and libraries
- `sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev`
- `sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler`
3. Edit config file
- `USE_CUDNN := 1`
4. Compile caffe
- `make all`
- `make test`
- `make runtest`
5. Build also python module and distribution libraries.
- `make pycaffe`
- `make distribute`




