# Copyright 2021, Proofcraft Pty Ltd
#
# SPDX-License-Identifier: BSD-2-Clause

# The context of this Dockerfiles is the repo root (../)

ARG BASE_IMG=capablehub/cheri-sel4-ci-riscv-base:latest
ARG WORKSPACE=/workspace

FROM $BASE_IMG

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3-venv \
    && apt-get clean autoclean \
    && apt-get autoremove --yes \
    && rm -rf /var/lib/{apt,dpkg,cache,log}/

RUN curl https://sh.rustup.rs -sSf | env RUSTUP_HOME=/opt/rust/rustup CARGO_HOME=/opt/rust/cargo sh -s -- --profile default --no-modify-path -y
ENV PATH="/opt/rust/cargo/bin:$PATH"

ARG WORKSPACE
RUN mkdir -p ${WORKSPACE}
WORKDIR ${WORKSPACE}
