# Global ARGs that are re-used in multiple stages
ARG ENABLED_TOOLS="rvy"
ARG BUILD_TMP_DIR="/tmp/cheri"
ARG FINAL_INSTALL_DIR="/opt/cheri-tools"

# Define these stages and ARGs globally so COPY --from can use a static name
ARG LLVM_ISAV9_IMAGE="capablehub/cheri-ctsrd-llvm:17-riscv64-74d200f"
ARG LLVM_RVY_IMAGE="capablehub/cheri-ca-llvm:17-riscv64-678272e"
FROM ${LLVM_ISAV9_IMAGE} AS llvm-source-isav9
FROM ${LLVM_RVY_IMAGE} AS llvm-source-rvy

FROM ubuntu:24.04 AS cheri-builder-base

ARG ENABLED_TOOLS
ARG BUILD_TMP_DIR
ARG FINAL_INSTALL_DIR

# Validation Step
RUN if [ "$ENABLED_TOOLS" != "isav9" ] && [ "$ENABLED_TOOLS" != "rvy" ]; then \
        echo "ERROR: Docker ARGs are case-sensitive."; \
        echo "ENABLED_TOOLS: ${ENABLED_TOOLS}" ; \
        echo "Please use: --build-arg ENABLED_TOOLS=isav9"; \
        echo "or: --build-arg ENABLED_TOOLS=rvy"; \
        exit 1; \
    fi

# Set non-interactive frontend for apt to skip any user confirmations
ENV DEBIAN_FRONTEND=noninteractive

#Install build dependencies
RUN apt update -y && \
    apt upgrade -y && \
    apt install -y \
    git \
    cmake \
    ninja-build \
    gperf \
    ccache \
    dfu-util \
    device-tree-compiler \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-tk \
    python3-wheel \
    xz-utils \
    file \
    make \
    gcc \
    libsdl2-dev \
    libmagic1 \
    python3-venv \
    autoconf \
    automake \
    libtool \
    pkg-config \
    clang \
    bison \
    samba \
    flex \
    texinfo \
    time \
    libglib2.0-dev \
    libpixman-1-dev \
    libarchive-dev \
    libarchive-tools \
    libbz2-dev \
    libattr1-dev \
    libcap-ng-dev \
    libexpat1-dev \
    libgmp-dev \
    libmpfr-dev \
    libmpc-dev \
    libncurses-dev \
    libdebuginfod-dev \
    bzip2

# Create build space
RUN mkdir -p ${BUILD_TMP_DIR}
WORKDIR ${BUILD_TMP_DIR}


FROM cheri-builder-base AS cheri-builder-isav9

ARG BUILD_TMP_DIR
ARG FINAL_INSTALL_DIR

ARG QEMU_REPO="https://github.com/CTSRD-CHERI/qemu.git"
ARG QEMU_BRANCH="qemu-cheri"
ARG QEMU_COMMIT_HASH="HEAD"

ARG GDB_REPO="https://github.com/CTSRD-CHERI/gdb.git"
ARG GDB_BRANCH="cheri-14"
ARG GDB_COMMIT_HASH="HEAD"

# Build QEMU from CTSRD repo
RUN git clone --no-checkout --branch ${QEMU_BRANCH} ${QEMU_REPO} qemu && \
    cd qemu && \
    git checkout ${QEMU_COMMIT_HASH} && \
    mkdir build install && \
    cd build && \
    ../configure \
        --disable-sdl --disable-gtk --disable-opengl --disable-virglrenderer \
        --disable-stack-protector --disable-strip --smbd=/usr/sbin/smbd \
        --disable-xen --disable-docs --disable-rdma --disable-werror \
        --cxx=/usr/bin/clang++ --cc=/usr/bin/clang \
        --python=/usr/bin/python3 \
        --extra-cflags="-O3 -Wno-error" \
        --disable-capstone --disable-bsd-user --disable-linux-user --enable-slirp=git \
        --target-list=riscv64-softmmu,riscv64cheri-softmmu \
        --prefix=${BUILD_TMP_DIR}/qemu/install && \
    ninja && \
    ninja install

# Build GDB from CTSRD repo 
RUN git clone --no-checkout --branch ${GDB_BRANCH} ${GDB_REPO} gdb && \
    cd gdb && \
    git checkout ${GDB_COMMIT_HASH} && \
    mkdir build install && \
    cd build && \
    ../configure \
        --disable-nls --enable-tui --disable-ld --disable-gold --disable-sim \
        --enable-64-bit-bfd --without-gnu-as \
        --disable-werror \
        --disable-libstdcxx --with-guile=no --enable-targets=all --with-expat \
        CC=/usr/bin/clang CXX=/usr/bin/clang++ \
        CFLAGS="-O2 -fcommon" CXXFLAGS="-O2 -fcommon" \
        LDFLAGS="" --prefix=${BUILD_TMP_DIR}/gdb/install && \
    make -j$(nproc) && \
    make install

COPY --from=llvm-source-isav9 --chown=builder:builder /root/llvm-cheri/install ${BUILD_TMP_DIR}/llvm/install

FROM cheri-builder-base AS cheri-builder-rvy

ARG BUILD_TMP_DIR
ARG FINAL_INSTALL_DIR

ARG QEMU_REPO="https://github.com/CHERI-Alliance/qemu.git"
ARG QEMU_BRANCH="codasip-cheri-riscv_v3"
ARG QEMU_COMMIT_HASH="2a2e882b"

ARG GDB_REPO="https://github.com/CHERI-Alliance/gdb.git"
ARG GDB_BRANCH="codasip-cheri-riscv"
ARG GDB_COMMIT_HASH="df929d4d"

ARG LLVM_IMAGE="capablehub/cheri-ca-llvm:17-riscv64-678272e"

# Build QEMU from CHERI Alliance
RUN git clone --no-checkout --branch ${QEMU_BRANCH} ${QEMU_REPO} qemu && \
    cd qemu && \
    git checkout ${QEMU_COMMIT_HASH} && \
    mkdir build install && \
    cd build && \
    ../configure \
        --prefix=${BUILD_TMP_DIR}/qemu/install \
        --target-list="riscv32cheri-softmmu riscv64cheri-softmmu" \
        --disable-gtk --audio-drv-list="" --disable-brlapi --disable-libiscsi \
        --disable-libnfs --disable-rbd --disable-sdl --disable-snappy \
        --disable-vnc --disable-vnc-jpeg --disable-vnc-sasl --disable-l2tpv3 \
        --disable-oss --disable-alsa --disable-tpm --disable-werror --meson=git && \
    ninja && \
    ninja install

# Build GDB from CHERI Alliance
RUN git clone --no-checkout --branch ${GDB_BRANCH} ${GDB_REPO} gdb && \
    cd gdb && \
    git checkout ${GDB_COMMIT_HASH} && \
    mkdir build install && \   
    cd build && \
    ../configure \
        --target=riscv64-unknown-elf \
        --prefix=${BUILD_TMP_DIR}/gdb/install && \
    make -j$(nproc) && \
    make install &&\
    cd ~/

COPY --from=llvm-source-rvy --chown=builder:builder /root/llvm-cheri/install ${BUILD_TMP_DIR}/llvm/install

FROM cheri-builder-${ENABLED_TOOLS} AS selected-cheri-builder

FROM ubuntu:24.04 AS cheri-final

ENV DEBIAN_FRONTEND=noninteractive

# These can can be set with --build-args when building the image
ARG ZEPHYR_REPO_URL=https://github.com/CHERI-Alliance/CHERI-zephyr.git
ARG ZEPHYR_BRANCH=CHERI-RISCV64-v2.0
ARG ZEPHYR_COMMIT_HASH=HEAD

ARG BUILD_TMP_DIR
ARG FINAL_INSTALL_DIR

# Install runner dependencies
RUN apt update -y && \
    apt upgrade -y && \
    apt install -y --no-install-recommends \
    git \
    cmake \
    ninja-build \
    gperf \
    ccache \
    dfu-util \
    device-tree-compiler \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-tk \
    python3-wheel \
    xz-utils \
    file \
    make \
    gcc \
    libsdl2-dev \
    libmagic1 \
    libpixman-1-dev \
    python3-venv && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user - cheribuild causes fatal error if run as root
RUN useradd -m -s /bin/bash builder && echo "builder ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Copy the tools into the image
RUN mkdir -p /opt/cheri-tools/
COPY --from=selected-cheri-builder --chown=builder:builder ${BUILD_TMP_DIR}/qemu/install ${FINAL_INSTALL_DIR}/qemu
COPY --from=selected-cheri-builder --chown=builder:builder ${BUILD_TMP_DIR}/llvm/install ${FINAL_INSTALL_DIR}/llvm
COPY --from=selected-cheri-builder --chown=builder:builder ${BUILD_TMP_DIR}/gdb/install ${FINAL_INSTALL_DIR}/gdb
RUN chown builder:builder /opt/cheri-tools


WORKDIR /home/builder
USER builder

ENV ZEPHYR_TOOLCHAIN_VARIANT=llvm-cheri
ENV LLVM_CHERI_TOOLCHAIN_PATH=${FINAL_INSTALL_DIR}/llvm
ENV QEMU_BIN_PATH=${FINAL_INSTALL_DIR}/qemu/bin
RUN echo "source ~/.venv/bin/activate" >> ~/.bashrc

# Clone the CHERI-Zephyr release & Install west
RUN git clone --no-checkout --branch ${ZEPHYR_BRANCH} ${ZEPHYR_REPO_URL} zephyr && \
    cd zephyr && \
    git checkout ${ZEPHYR_COMMIT_HASH} && \
    cd .. && \
    python3 -m venv .venv && . .venv/bin/activate && \
    pip install --no-cache-dir west && \
    west init -l zephyr && \
    west update --narrow --fetch-opt=--depth=1 && \
    west zephyr-export && \
    pip install --no-cache-dir -r zephyr/scripts/requirements.txt && \
    find . -name ".git" -type d -exec rm -rf {} +

RUN echo "source ~/.venv/bin/activate" >> ~/.bashrc

CMD ["bash"]
