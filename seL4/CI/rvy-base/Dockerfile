# Copyright 2021, Proofcraft Pty Ltd
#
# SPDX-License-Identifier: BSD-2-Clause

# The context of this Dockerfiles is the repo root (../)

# Builds a Docker image based on trustworthysystems/sel4:latest that includes CHERI LLVM for RISCV64

FROM trustworthysystems/sel4:latest
ARG TARGETPLATFORM=linux/amd64

USER root
ENV HOME=/root/

RUN mkdir -p /build
WORKDIR /build

# Install prerequisites
RUN apt update && \
    apt install -y \
        pkg-config \
        libglib2.0-dev \
        libpixman-1-dev && \
    rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/Kitware/CMake/releases/download/v4.2.3/cmake-4.2.3-linux-x86_64.sh && \
    chmod +x cmake-4.2.3-linux-x86_64.sh

RUN ./cmake-4.2.3-linux-x86_64.sh --prefix=/usr --skip-license

###############################################################################
# Build CA LLVM
ARG LLVM_REPO_URL=https://github.com/CHERI-Alliance/llvm-project.git
ARG LLVM_BRANCH=main
ARG LLVM_COMMIT_HASH=HEAD
# Internal temporary install path for the builder stage
ARG LLVM_BUILD_INSTALL_DIR=/opt/cheri-tools/llvm-cheri-codasip

# Fetch the LLVM src
RUN git clone --no-checkout --recurse-submodules --branch ${LLVM_BRANCH} ${LLVM_REPO_URL} llvm-cheri && \
    cd llvm-cheri && \
    git checkout ${LLVM_COMMIT_HASH}

# Build LLVM
RUN cd llvm-cheri && \
    mkdir build install && \
    cd build && \
    cmake \
        -DCMAKE_INSTALL_PREFIX=${LLVM_BUILD_INSTALL_DIR} \
        -DLLVM_TARGETS_TO_BUILD=RISCV \
        -DLLVM_DEFAULT_TARGET_TRIPLE="riscv64-unknown-elf" \
        -DLLVM_ENABLE_PROJECTS="clang;lld" \
        -DCMAKE_BUILD_TYPE=Release \
        -GNinja ../llvm/ \
        -DBUILD_SHARED_LIBS=ON \
        -DLLVM_USE_SPLIT_DWARF=ON \
        -DLLVM_ENABLE_ASSERTIONS=OFF \
        -Wno-dev && \
    ninja && \
    ninja install/strip && \
    cd ~/

ENV PATH="${LLVM_BUILD_INSTALL_DIR}/bin:${PATH}"

###############################################################################
# Build CA QEMU
ARG QEMU_REPO_URL="https://github.com/CHERI-Alliance/qemu.git"
ARG QEMU_BRANCH="codasip-cheri-riscv_v3"
ARG QEMU_COMMIT_HASH="HEAD"
ARG QEMU_BUILD_TARGET_LIST="riscv32cheri-softmmu riscv64cheri-softmmu"
ARG QEMU_INSTALL_DIR="/opt/qemu/"

RUN git clone --no-checkout --recurse-submodules --branch ${QEMU_BRANCH} ${QEMU_REPO_URL} qemu && \
    cd qemu && \
    git checkout ${QEMU_COMMIT_HASH}

RUN cd qemu && \
    mkdir build && \
    cd build && \
    ../configure \
        --prefix="${QEMU_INSTALL_DIR}" \
        --target-list="${QEMU_BUILD_TARGET_LIST}" \
        --disable-sdl \
        --disable-gtk \
        --disable-opengl \
        --audio-drv-list="" \
        --disable-brlapi \
        --disable-libiscsi \
        --disable-libnfs \
        --disable-rbd \
        --disable-snappy \
        --disable-vnc \
        --disable-vnc-jpeg \
        --disable-vnc-sasl \
        --disable-l2tpv3 \
        --disable-oss \
        --disable-alsa \
        --disable-tpm \
        --disable-werror \
        --meson=git && \
    ninja && \
    ninja install

ENV PATH="${QEMU_INSTALL_DIR}/bin:${PATH}"

###############################################################################
# Build CA OpenSBI
ARG OPENSBI_REPO_URL="https://github.com/qwattash/cheri-alliance-opensbi"
ARG OPENSBI_BRANCH="toooba-fixes"
ARG OPENSBI_COMMIT_HASH="HEAD"
ARG OPENSBI_INSTALL_DIR="/opt/opensbi-riscv-purecap/"

RUN git clone --no-checkout --recurse-submodules --branch ${OPENSBI_BRANCH} ${OPENSBI_REPO_URL} opensbi-riscv-purecap && \
    cd opensbi-riscv-purecap && \
    git checkout ${OPENSBI_COMMIT_HASH}

RUN cd opensbi-riscv-purecap && \
    mkdir build && \
    make \
        LLVM=1 \
        CC=clang \
        O=build \
        I="${OPENSBI_INSTALL_DIR}" \
        CC="clang -target riscv64-unknown-elf -B${LLVM_BUILD_INSTALL_DIR}/bin -march=rv64imafdczcherihybrid_zcherilevels -mabi=l64pc128d -mno-relax -Xclang -target-feature -Xclang +cheri-bounded-vararg -Xclang -target-feature -Xclang +cheri-bounded-memarg-caller -Xclang -target-feature -Xclang +cheri-bounded-memarg-callee -mcmodel=medany -Qunused-arguments" \
        CXX="clang++ -target riscv64-unknown-elf -B${LLVM_BUILD_INSTALL_DIR}/bin -march=rv64imafdczcherihybrid_zcherilevels -mabi=l64pc128d -mno-relax -Xclang -target-feature -Xclang +cheri-bounded-vararg -Xclang -target-feature -Xclang +cheri-bounded-memarg-caller -Xclang -target-feature -Xclang +cheri-bounded-memarg-callee -mcmodel=medany -Qunused-arguments" \
        CPP="clang-cpp -target riscv64-unknown-elf -B${LLVM_BUILD_INSTALL_DIR}/bin -march=rv64imafdczcherihybrid_zcherilevels -mabi=l64pc128d -mno-relax -Xclang -target-feature -Xclang +cheri-bounded-vararg -Xclang -target-feature -Xclang +cheri-bounded-memarg-caller -Xclang -target-feature -Xclang +cheri-bounded-memarg-callee -mcmodel=medany -Qunused-arguments" \
        LD="${LLVM_BUILD_INSTALL_DIR}/bin/ld.lld" \
        AR="llvm-ar" \
        OBJCOPY="llvm-objcopy" \
        FW_PIC=n \
        FW_OPTIONS=0x2 \
        PLATFORM_RISCV_ABI=l64pc128 \
        PLATFORM_RISCV_ISA=rv64imaczcherihybrid_zcherilevels \
        PLATFORM_RISCV_XLEN=64 \
        FW_TEXT_START=0x80000000 \
        PLATFORM=generic \
        -j$(nproc) && \
    make O=build I="${OPENSBI_INSTALL_DIR}" PLATFORM=generic install

ENV OPENSBI_FW_DIR="${OPENSBI_INSTALL_DIR}/share/opensbi/"

# Cleanup
RUN rm -rf /build

WORKDIR /
