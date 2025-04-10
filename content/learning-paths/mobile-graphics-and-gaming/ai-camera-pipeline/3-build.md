---
title: Build the pipelines
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---

## Download the AI Camera Pipelines project

```BASH
git clone https://git.gitlab.arm.com/kleidi/kleidi-examples/ai-camera-pipelines.git ai-camera-pipelines.git
```

Checkout the data files:

```BASH
cd ai-camera-pipelines.git
git lfs install
git lfs pull
```

## Create a build container

The pipelines will be build from a container, so you first need to build the container:

```BASH
docker build -t ai-camera-pipelines -f docker/Dockerfile --build-arg DOCKERHUB_MIRROR=docker.io --build-arg CI_UID=$(id -u) .
```

## Build the AI Camera Pipelines

Start a shell in the container you have just built with:

```BASH
docker run --rm --volume $PWD:/home/cv-examples/example -it ai-camera-pipelines
```

And execute the following commands:

```BASH
ENABLE_SME2=1
TENSORFLOW_GIT_TAG=ddceb963c1599f803b5c4beca42b802de5134b44

# Build flatbuffers
git clone https://github.com/google/flatbuffers.git
cd flatbuffers
git checkout v24.3.25
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../install
cmake --build . -j16
cmake --install .
cd ../..

# Build the pipelines
mkdir build
cd build
cmake -GNinja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../install -DARMNN_TFLITE_PARSER=0 -DTENSORFLOW_GIT_TAG=$TENSORFLOW_GIT_TAG -DTFLITE_HOST_TOOLS_DIR=../flatbuffers/install/bin -DENABLE_SME2=$ENABLE_SME2 -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake -S ../example -B .
cmake --build . -j16
cmake --install .
cd ../install
mkdir lib
cp /usr/lib/llvm-19/lib/libomp.so.5 lib/

# Package and export the pipelines.
cd ..
tar cfz example/install.tar.gz install

# Leave the container
ctrl+D
```

## Install the pipelines

```BASH
cd $HOME
tar xfz ai-camera-pipelines.git/install.tar.gz
mv install ai-camera-pipelines
export LD_LIBRARY_PATH=$HOME/ai-camera-pipelines/lib:$LD_LIBRARY_PATH
```
