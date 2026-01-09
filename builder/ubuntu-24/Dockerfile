# A base builder image based on Ubuntu 24.04 that installs some common build dependencies and creates a 'ci' user with sudo permisson
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    coreutils \
    sudo \
    locales \
    docker.io \
    podman \
    build-essential \
    clang \
    texinfo \
    flex \
    bison \
    autoconf \
    automake \
    libtool \
    cmake \
    ninja-build \
    pkg-config \
    git-core \
    python3 \
    python3-pip \
    python3-venv \
    fakeroot \
    chrpath \
    file \
    gawk \
    lz4 \
    zstd \
    wget \
    curl \
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

