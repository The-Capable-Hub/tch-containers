FROM ubuntu:24.04 AS builder

# Prevent issues with apt prompting
ENV DEBIAN_FRONTEND=noninteractive

# These can can be set with --build-args when building the image
ARG REPO_URL=https://github.com/CHERI-Alliance/llvm-project.git
ARG BRANCH=main
ARG COMMIT_HASH=HEAD
# Internal temporary install path for the builder stage
ARG BUILD_INSTALL_DIR=/tmp/llvm-cheri-install

# Install minimal dependencies to build LLVM
RUN apt update -y && \
    apt install -y \
    git \
    cmake \
    ninja-build \
    python3-dev \
    build-essential \
    libncurses-dev \
    ccache \
    binutils \
    zlib1g-dev

# Fetch the LLVM src
RUN git clone --no-checkout --recurse-submodules --branch ${BRANCH} ${REPO_URL} llvm-cheri && \
    cd llvm-cheri && \
    git checkout ${COMMIT_HASH}

# Build LLVM
RUN cd llvm-cheri && \
    mkdir build install && \
    cd build && \
    cmake \
        -DCMAKE_INSTALL_PREFIX=${BUILD_INSTALL_DIR} \
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

# Generate Build Summary
RUN echo "Generating Build Summary..." && \
    ACTUAL_COMMIT=$(cd llvm-cheri && git rev-parse HEAD) && \
    echo "================= BUILD SUMMARY ==================" > ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "Build Arguments:" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "  REPO_URL:    ${REPO_URL}" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "  BRANCH:      ${BRANCH}" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "  ARG_COMMIT:  ${COMMIT_HASH}" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "Build Artifacts:" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    echo "  Actual Git Hash: ${ACTUAL_COMMIT}" >> ${BUILD_INSTALL_DIR}/build_manifest.txt && \
    cat ${BUILD_INSTALL_DIR}/build_manifest.txt
    
FROM ubuntu:24.04 AS runner

ENV DEBIAN_FRONTEND=noninteractive

# Where the tools live in the final image
ARG INSTALL_DIR=/root/llvm-cheri/install

# Install ONLY runtime dependencies
RUN apt update -y && \
    apt install -y \
    libncurses6 \
    zlib1g \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Copy artifacts from the builder stage
WORKDIR /root
COPY --from=builder /tmp/llvm-cheri-install ${INSTALL_DIR}

# Add to PATH
ENV PATH="${INSTALL_DIR}/bin:${PATH}"

# Verify installation
RUN echo "Verifying installation..." && \
    clang --version && \
    cat ${INSTALL_DIR}/build_manifest.txt
