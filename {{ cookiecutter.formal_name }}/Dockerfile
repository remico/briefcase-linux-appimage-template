FROM ubuntu:16.04

# Set the working directory
WORKDIR /app

# Set up Docker build arguments
ARG PY_VERSION
ARG SYSTEM_REQUIRES

# Install the deadsnakes PPA so we can get arbitrary Python versions
RUN apt-get update -y && \
    apt-get install -y \
        software-properties-common \
        dirmngr \
        apt-transport-https \
        lsb-release \
        ca-certificates
RUN apt-add-repository ppa:deadsnakes/ppa

# Install git, Python, and any packages required
RUN apt-get update -y && \
    apt-get install -y \
        git \
        python${PY_VERSION}-dev \
        python${PY_VERSION}-venv \
        ${SYSTEM_REQUIRES}

# Install (and update) pip
RUN python${PY_VERSION} -m ensurepip
RUN python${PY_VERSION} -m pip install --upgrade pip
RUN python${PY_VERSION} -m pip install --upgrade setuptools
RUN python${PY_VERSION} -m pip install --upgrade wheel

# restrict pycairo version for ubuntu 16.04
RUN CONSTRAINTS="/constraints.txt"; pip config --global set global.constraint $CONSTRAINTS; echo "pycairo<1.20" > $CONSTRAINTS

# Ensure Docker user UID:GID matches host user UID:GID (beeware/briefcase#403)
ARG HOST_UID
ARG HOST_GID
RUN groupadd --gid $HOST_GID briefcase || true
RUN useradd --uid $HOST_UID --gid $HOST_GID brutus --home /home/brutus

# setup pip cache & pre-install system packages for PySide development
RUN mkdir -p /home/brutus/.cache/pip
RUN chown -R $HOST_UID:$HOST_GID /home/brutus
RUN apt install -y apt-file && apt-file update
RUN apt install -y vim
RUN apt install -y libgl1-mesa-glx libfreetype6 libxcb-render0 libxkbcommon0 libxcb-render-util0 libxcb-icccm4 libfontconfig1 \
    libxcb-image0 libxcb-keysyms1 libxcb-xinerama0 libxcb-shape0 libxcb-xfixes0 libxcb-xkb1 libxcb-randr0 libxcb-shm0 libxkbcommon-x11-0

USER brutus

# setup additional vars & pre-install python packages for PySide development
ENV PATH=/home/brutus/.local/bin:$PATH
RUN python${PY_VERSION} -m pip install pudb PySide2 requests json2xml
RUN python${PY_VERSION} -m pip install --extra-index-url https://remico.github.com/pypi trim-ps2

CMD ln -s "$(find /app -name python3)" /home/brutus/.local/bin/; \
    SHIBOKEN2="$(find /app -name shiboken2.abi3.so)"; \
    export LD_LIBRARY_PATH=${SHIBOKEN2%/*}; \
    /bin/bash
