# ===========================
# cryo-ao Base Image
# ===========================

#FROM ubuntu:24.04
#FROM nvidia/cuda:12.4.1-devel-ubuntu22.04
FROM nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04

LABEL maintainer="Anchieta Oliveira"
LABEL description="Cryo-EM environment with Micromamba"

# Evita perguntas durante a instalação
ENV DEBIAN_FRONTEND=noninteractive

# Atualiza o sistema e instala ferramentas básicas
RUN apt-get update && apt-get install -y \
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

RUN find /opt/micromamba
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

# CUDA usado pelo compilador
ENV CUDAHOME=/usr/local/cuda
ENV PATH="/opt/AreTomo3:/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

# Compila AreTomo3
RUN make exe -f makefile11 CUDAHOME=/usr/local/cuda

# ===========================
# Final
# ===========================

RUN ls -lh /opt/AreTomo3

WORKDIR /opt
RUN apt-get update \
    && apt-cache policy libtiff6 libtiff-dev \
    && apt-cache search libtiff

RUN apt-get install -y libtiff6