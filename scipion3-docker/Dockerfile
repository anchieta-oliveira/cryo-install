# Scipion Base Image
# ===========================
FROM nvidia/cuda:12.4.1-devel-ubuntu22.04

LABEL maintainer="Anchieta Oliveira"
LABEL description="Cryo-EM environment with Micromamba"

# Evita perguntas durante a instalação
ENV DEBIAN_FRONTEND=noninteractive


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
    libtiff-dev \
    xfce4 \
    xfce4-terminal \
    tigervnc-standalone-server \
    tigervnc-common \
    novnc \
    websockify \
    dbus-x11 \
    x11-xserver-utils \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

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


# Define as variáveis para que o CMake do instalador encontre os cabeçalhos do Python corretos
ENV PATH="/opt/conda/envs/scipion3/bin:$PATH"
ENV CONDA_PREFIX="/opt/conda/envs/scipion3"
ENV CMAKE_PREFIX_PATH="/opt/conda/envs/scipion3"

# Instala o plugin do Xmipp
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-xmipp

# Instala o plugin do imod
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-imod

# Instala o plugin do Relion
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-relion

# Instala o plugin do SPHIRE
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-sphire

# Instala o plugin do MotionCor2
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-motioncorr

    # Instala o plugin do CisTEM
RUN eval "$(/opt/conda/bin/conda shell.bash hook)" && \
    conda activate scipion3 && \
    /opt/scipion3/scipion3 install -p scipion-em-cistem

    RUN mkdir -p /root/.vnc

RUN printf '#!/bin/bash\n\
unset SESSION_MANAGER\n\
unset DBUS_SESSION_BUS_ADDRESS\n\
startxfce4 &\n' > /root/.vnc/xstartup && \
    chmod +x /root/.vnc/xstartup

RUN printf '#!/bin/bash\n\
set -e\n\
\n\
mkdir -p /root/.vnc\n\
\n\
if [ -n "$VNC_PASSWORD" ]; then\n\
    echo "$VNC_PASSWORD" | vncpasswd -f > /root/.vnc/passwd\n\
    chmod 600 /root/.vnc/passwd\n\
fi\n\
\n\
rm -f /tmp/.X1-lock\n\
rm -rf /tmp/.X11-unix/X1\n\
\n\
vncserver :1 \\\n\
    -geometry ${VNC_GEOMETRY:-1920x1080} \\\n\
    -depth 24 \\\n\
    -localhost no\n\
\n\
websockify --web=/usr/share/novnc/ ${NOVNC_PORT:-6080} localhost:5901\n\
' > /usr/local/bin/start-vnc.sh && \
chmod +x /usr/local/bin/start-vnc.sh

EXPOSE 6080
EXPOSE 5901

CMD ["/usr/local/bin/start-vnc.sh"]


RUN mkdir -p /root/Desktop && \
    printf '[Desktop Entry]\n\
Version=1.0\n\
Type=Application\n\
Name=Scipion\n\
Comment=Scipion Cryo-EM\n\
Exec=/opt/scipion3/scipion3\n\
Terminal=false\n\
Icon=utilities-terminal\n' > /root/Desktop/scipion.desktop && \
    chmod +x /root/Desktop/scipion.desktop