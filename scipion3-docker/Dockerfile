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
    libxcb1 \
    libxcb-render0 \
    libxcb-shm0 \
    libxcb-xfixes0 \
    libxcb-shape0 \
    libxcb-randr0 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-icccm4 \
    libxcb-xinerama0 \
    libxcb-xkb1 \
    libxcb-cursor0 \
    libxkbcommon-x11-0 \
    libxrender1 \
    libxi6 \
    libxrandr2 \
    libxfixes3 \
    libxcursor1 \
    libxinerama1 \
    libgl1 \
    libegl1 \
    libfontconfig1 \
    libfreetype6 \
    libdbus-1-3 \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# ============================================================
# MPI
# ============================================================

ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

ENV MPI_BINDIR=/usr/bin
ENV MPI_INCLUDE=/usr/lib/x86_64-linux-gnu/openmpi/include
ENV MPI_LIBDIR=/usr/lib/x86_64-linux-gnu/openmpi/lib

ENV RELION_MPI_BIN=/usr/bin
ENV RELION_MPI_LIB=/usr/lib/x86_64-linux-gnu/openmpi/lib


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


# ============================================================
# Configuração MPI do Scipion
# ============================================================

RUN echo "" >> /opt/scipion3/config/scipion.conf && \
    echo "MPI_BINDIR = /usr/bin" >> /opt/scipion3/config/scipion.conf && \
    echo "MPI_INCLUDE = /usr/lib/x86_64-linux-gnu/openmpi/include" >> /opt/scipion3/config/scipion.conf && \
    echo "MPI_LIBDIR = /usr/lib/x86_64-linux-gnu/openmpi/lib" >> /opt/scipion3/config/scipion.conf && \
    echo "RELION_MPI_BIN = /usr/bin" >> /opt/scipion3/config/scipion.conf && \
    echo "RELION_MPI_LIB = /usr/lib/x86_64-linux-gnu/openmpi/lib" >> /opt/scipion3/config/scipion.conf && \
    echo "PARALLEL_COMMAND_RELION = /usr/bin/mpirun --allow-run-as-root -np %%(JOB_NODES)d %%(COMMAND)s" >> /opt/scipion3/config/scipion.conf


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





# ============================================================
# VNC / XFCE / noVNC
# ============================================================
ENV DISPLAY=:1

RUN mkdir -p /root/.vnc /root/Desktop


# ------------------------------------------------------------
# XFCE startup
# ------------------------------------------------------------

RUN cat > /root/.vnc/xstartup <<'EOF'
#!/bin/bash


unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

exec startxfce4
EOF

RUN chmod +x /root/.vnc/xstartup


# ------------------------------------------------------------
# VNC + noVNC startup
# ------------------------------------------------------------

RUN cat > /usr/local/bin/start-vnc.sh <<'EOF'
#!/bin/bash
set -e
export DISPLAY=:1

mkdir -p /root/.vnc

# Configura senha VNC caso tenha sido fornecida
if [ -n "${VNC_PASSWORD:-}" ]; then
    echo "$VNC_PASSWORD" | vncpasswd -f > /root/.vnc/passwd
    chmod 600 /root/.vnc/passwd
fi

# Remove locks antigos caso o container tenha parado incorretamente
rm -f /tmp/.X1-lock
rm -rf /tmp/.X11-unix/X1

# Inicia TigerVNC
vncserver :1 \
    -geometry "${VNC_GEOMETRY:-1920x1080}" \
    -depth 24 \
    -localhost yes

# noVNC -> VNC interno
exec websockify \
    --web=/usr/share/novnc/ \
    "${NOVNC_PORT:-6080}" \
    localhost:5901
EOF

RUN chmod +x /usr/local/bin/start-vnc.sh


# ------------------------------------------------------------
# Desktop shortcut - Scipion
# ------------------------------------------------------------

RUN cat > /root/Desktop/scipion.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=Scipion
Comment=Scipion Cryo-EM
Exec=/opt/scipion3/scipion3
Terminal=false
Icon=utilities-terminal
Categories=Science;
EOF

RUN chmod +x /root/Desktop/scipion.desktop


# ------------------------------------------------------------
# Network
# ------------------------------------------------------------

EXPOSE 6080


# ------------------------------------------------------------
# Container startup
# ------------------------------------------------------------

CMD ["/usr/local/bin/start-vnc.sh"]