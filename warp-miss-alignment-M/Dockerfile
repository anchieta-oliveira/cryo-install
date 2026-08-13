# ===========================
# cryo-ao Base Image
# ===========================

#FROM ubuntu:24.04
#FROM nvidia/cuda:12.4.1-devel-ubuntu22.04
FROM nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04

LABEL maintainer="Anchieta Oliveira"
LABEL description="Cryo-EM environment with Micromamba"

# No interactive installation
ENV DEBIAN_FRONTEND=noninteractive

# Apps base
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget \
    curl \
    git \
    nano \
    vim \
    ca-certificates \
    build-essential \
    bzip2 \
    unzip \
    tar \
    sudo \
    libtiff-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ==========================================
# Install Micromamba
# ==========================================

ENV MAMBA_ROOT_PREFIX=/opt/micromamba
ENV PATH=/opt/micromamba/bin:$PATH

RUN mkdir -p /opt/micromamba && \
    curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | \
    tar -xvj -C /opt/micromamba


ENV PATH="/opt/micromamba/bin:${PATH}"

RUN micromamba create -y -n warp \
    warp=2.0.0 \
    -c warpem \
    -c nvidia/label/cuda-11.8.0 \
    -c pytorch \
    -c conda-forge
  
RUN  micromamba run -n warp micromamba update warp -c warpem -c nvidia/label/cuda-11.8.0 -c pytorch -c conda-forge --all --yes

RUN micromamba create -n miss-alignment -c conda-forge python=3.12 cuda-toolkit=12.9 -y

RUN  micromamba run -n miss-alignment python -m pip install torch==2.8.0
RUN  micromamba run -n miss-alignment python -m pip install torch-projectors --index-url https://warpem.github.io/torch-projectors/cu129/simple/
RUN  micromamba run -n miss-alignment python -m pip install miss-alignment
RUN micromamba clean --all --yes

RUN micromamba create -y -n dtp python=3.12 pip -y


RUN micromamba shell init -s bash

# ===========================
# AreTomo3
# ===========================
RUN nvcc --version && \
    echo "CUDAHOME=${CUDAHOME}" && \
    ls -la /usr/local/cuda

WORKDIR /opt

RUN git clone https://github.com/czimaginginstitute/AreTomo3.git

WORKDIR /opt/AreTomo3

# CUDA utilities
ENV CUDAHOME=/usr/local/cuda
ENV PATH="/opt/AreTomo3:/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

# Build AreTomo3
RUN sed -i 's/@$(NVCC) -g -G -m64 $(OBJS)/@$(NVCC) -g -G -m64 -Xcompiler -no-pie $(OBJS)/' makefile12 \
    && make clean -f makefile12 \
    && make exe -f makefile12 CUDAHOME=/usr/local/cuda

RUN ls -lh /opt/AreTomo3

WORKDIR /opt