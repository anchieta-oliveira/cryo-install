# cryo-ao Base Image
# ===========================

FROM ubuntu:24.04

LABEL maintainer="Anchieta Oliveira"
LABEL description="Cryo-EM environment with Micromamba"

# Evita perguntas durante a instalação
ENV DEBIAN_FRONTEND=noninteractive

# Atualiza o sistema e instala ferramentas básicas e dependências gráficas/compilação do Xmipp
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
    cmake \
    libopenmpi-dev \
    openmpi-bin \
    openjdk-11-jdk \
    libx11-dev \
    libxext-dev \
    libxft-dev \
    libfftw3-dev \
    libssl-dev \
    libhdf5-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Baixa e instala o Miniconda silenciosamente
RUN wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
    -O /tmp/miniconda.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

ENV PATH=/opt/conda/bin:$PATH
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

# Instala o Scipion base
RUN /opt/conda/bin/pip install scipion-installer && \
    mkdir -p /opt/scipion3 && \
    /opt/conda/bin/python -m scipioninstaller \
        -conda \
        -noAsk \
        /opt/scipion3

# Prepara o ambiente interno do Conda com as ferramentas de desenvolvimento do Python
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    conda install -c conda-forge -y python-dev

# Define as variáveis para que o CMake do instalador encontre os cabeçalhos do Python corretos
ENV PATH="/opt/conda/envs/scipion3/bin:$PATH"
ENV CONDA_PREFIX="/opt/conda/envs/scipion3"
ENV CMAKE_PREFIX_PATH="/opt/conda/envs/scipion3"

# Instala o plugin do Xmipp
RUN /opt/scipion3/scipion3 install -p scipion-em-xmipp