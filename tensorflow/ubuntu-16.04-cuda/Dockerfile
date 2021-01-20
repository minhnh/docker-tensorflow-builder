FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04

ARG BAZEL_VERSION=0.19.2
ARG TF_VERSION=1.13.1
ARG NUMPY_VERSION=1.16.4
ARG CUDA=10.0
ARG CUDNN=7.4
ARG COMPUTE_CAP=6.1

# Based on instructions from https://www.tensorflow.org/install/source
RUN apt update && apt upgrade -y && apt install -y \
    # tensorflow
    build-essential \
    git \
    wget \
    python3-dev \
    python3-pip \
    # bazel
    curl gnupg g++ zip unzip openjdk-8-jdk \
    # TF dependencies
    pkg-config \
    software-properties-common

RUN pip3 install -U pip wheel && \
    pip install numpy==$NUMPY_VERSION && \
    pip install -U keras_preprocessing --no-deps

# add symlinks to cudnn in cuda location and to python3 for building
RUN ln -s /usr/lib/x86_64-linux-gnu/libcudnn.so.* /usr/local/cuda/lib64/ && \
    ln -s /usr/bin/python3 /usr/bin/python

# Install Bazel
WORKDIR /workspace
RUN wget https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    chmod +x bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh

# Build TF
RUN git clone --depth 1 --branch v$TF_VERSION https://github.com/tensorflow/tensorflow.git tensorflow
# patch wrong sha256 in .bzl file
COPY icu_workspace.bzl /workspace/tensorflow/third_party/icu/workspace.bzl
WORKDIR /workspace/tensorflow
RUN export TF_NEED_GCP=0 && \
    export TF_NEED_HDFS=0 && \
    export PYTHON_BIN_PATH=`which python3` && \
    export TF_NEED_CUDA=1 && \
    export TF_NEED_OPENCL=0 && \
    export TF_CUDA_COMPUTE_CAPABILITIES=$COMPUTE_CAP && \
    yes "" | ./configure
RUN bazel build //tensorflow/tools/pip_package:build_pip_package
RUN ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /wheels

#COPY build.sh /build.sh
#COPY cuda.sh /cuda.sh

#CMD bash build.sh
