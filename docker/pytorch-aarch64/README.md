# Build PyTorch for AArch64 using Docker

A script to build a Docker image containing [PyTorch](https://www.pytorch.org/) and dependencies for the [Armv8-A architecture](https://developer.arm.com/architectures/cpu-architecture/a-profile) with AArch64 execution.
For more information, see this Arm Developer Community [blog post](https://community.arm.com/developer/tools-software/tools/b/tools-software-ides-blog/posts/aarch64-docker-images-for-pytorch-and-tensorflow).

Before using this project run the uname command to confirm the machine is aarch64. Other architectures will not work.

```
> uname -m
aarch64
```


## What's in the final image?
  * OS: Ubuntu 18.04
  * Compiler: GCC 9.3
  * Maths libraries: [Arm Optimized Routines](https://github.com/ARM-software/optimized-routines) and [OpenBLAS](https://www.openblas.net/) 0.3.10
  * [oneDNN](https://github.com/oneapi-src/oneDNN) 2.2.
    - [Compute Library for the Arm architecture](https://developer.arm.com/ip-products/processors/machine-learning/compute-library) 21.02, providing AArch64 optimised primitives for oneDNN.>
  * Python3 environment built from CPython 3.8 and containing:
    - NumPy 1.19.5
    - SciPy 1.5.2
    - PyTorch 1.8.1
  * [ML Commons :TM: (MLPerf)](https://mlperf.org/)
  * [Example scripts](./examples/README.md) that demonstrate how to run ML models
A user account with username 'ubuntu' is created with sudo privaleges and password of 'Portland'.

In addition to the Dockerfile, please look at the files in the scripts/ directory and the patches/ directory too see how the software is built.


## Installing Docker
The [Docker Engine](https://docs.docker.com/get-docker/) is used. Instructions on how to install Docker are available for various Linux distributions such as [CentOS](https://docs.docker.com/engine/install/centos/) and [Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

Confirm Docker is working:

``` > docker run hello-world ```

If there are any problems make sure the service is running:

``` > systemctl start docker ```

and make sure you are in the Docker group:

```  > usermod -aG docker $USER ```

These steps may require root privlages and usermod requires logout and login to take effect.

See https://docs.docker.com for more information.


## Building the Docker image
Use the build.sh script to build the image. This script implements a multi-stage build to minimise the size of the finished image:
  * Stage 1: 'base' image including Ubuntu with core packages and GCC9.
  * Stage 2: 'libs' image including essential tools and libraries such as Python and OpenBLAS.
  * Stage 3: 'tools' image, including a Python3 virtual environment in userspace and a build of NumPy against OpenBLAS, as well as other Python essentials.
  * Stage 4: 'dev' image, including PyTorch with sources
  * Stage 5: 'pytorch' image, including only the Python3 virtual environment, the PyTorch module, and the example scripts. PyTorch sources are not included in this image.

To see the command line options for build.sh use:

``` > ./build.sh -h ```

The image to build is selected with the '--build-type' flag. The options are base, libs, tools, dev, pytorch, or full. Selecting full builds all of the images. The default value is 'pytorch'


For example:
  * To build the final pytorch image:

    ``` > ./build.sh --build-type pytorch ```

  * For a full build:

    ``` > ./build.sh --build-type full ```

  * For a base build:

    ```  > ./build.sh --build-type base ```

    This will generate an image named 'DockerTest/ubuntu/base'.

PyTorch can optionally be built with oneDNN, using the '--onednn' or '--dnnl' flag. By default this will use AArch64 optimised primitives from Arm Compute Library where available. Specifying `--onednn reference` will disable Compute Library primitives and use oneDNN's reference C++ kernels throughout.

## Running the Docker image
To run the finished image:

  ``` > docker run -it --init <image name> ```

where <image name> is the name of the finished image, for example 'pytorch'.

  ``` > docker run -it --init pytorch```

To display available images use the Docker command:

  ``` > docker images ```