# A base builder image based on Ubuntu 24.04 that installs some common build dependencies and creates a 'ci' user with sudo permisson
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    ca-certificates \
    software-properties-common \
    apt-utils \
    coreutils \
    sudo \
    locales \
    docker.io \
    podman \
    pkg-config \
    build-essential \
    ccache \
    clang \
    texinfo \
    flex \
    bison \
    bc \
    make \
    autoconf \
    automake \
    libtool \
    cmake \
    ninja-build \
    git-core \
    python3 \
    python3-pip \
    python3-venv \
    fakeroot \
    chrpath \
    file \
    sed \
    gawk \
    lz4 \
    zstd \
    gzip \
    cpio \
    wget \
    curl \
    unzip \
    jq && \
    rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
RUN update-locale
RUN adduser --quiet --disabled-password --gecos 'user' ci
RUN usermod -u 1000 -g 1000 ci -o
RUN echo ci:ci | chpasswd
RUN usermod -aG sudo ci
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER ci
WORKDIR /home/ci
ENV PATH=/home/ci/.local/bin:${PATH}
ENV HOME=/home/ci

# Allow installation of python packages using pip
RUN python3 -m pip config set global.break-system-packages true

ADD --chown=ci:ci awsclikey /tmp/awsclikey

RUN mkdir /tmp/awscli && \
    cd /tmp/awscli && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    curl -o awscliv2.sig https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip.sig && \
    gpg --import /tmp/awsclikey &&\
    gpg --verify awscliv2.sig awscliv2.zip && \
    unzip awscliv2.zip && \
    sudo ./aws/install && \
    rm -rf /tmp/awscli && \
    rm -rf /tmp/awsclikey && \
    aws --version
