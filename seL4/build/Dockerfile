FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
	autoconf \
	automake \
	autotools-dev \
	bison \
	cmake \
	curl \
	device-tree-compiler \
	flex \
	g++ \
	gcc \
	git \
	groff-base \
	libarchive-dev \
	libexpat1-dev \
	libglib2.0-dev \
	libmpfr-dev \
	libncurses-dev \
	libpixman-1-dev \
	libtool \
	libxml2-utils \
	lsb-release \
	make \
	ninja-build \
	pkg-config \
	python3 \
	python3-setuptools \
	python3-venv \
	qemu-system-riscv64 \
	samba \
	telnet \
	texinfo \
	texlive-base \
	wget

VOLUME ["/cheribuild"]

WORKDIR /root

RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH /root/.cargo/bin:$PATH
RUN rustup target add x86_64-unknown-linux-musl

RUN git clone https://github.com/CTSRD-CHERI/cheribuild.git -b sel4-microkit

WORKDIR /root/cheribuild

RUN ./cheribuild.py --allow-running-as-root -d --riscv-cheri-isa=experimental_std093 --only-dependencies cheri-microkit-baremetal-riscv64-purecap

WORKDIR /cheribuild

CMD ./cheribuild.py --allow-running-as-root -d --riscv-cheri-isa=experimental_std093 cheri-microkit-baremetal-riscv64-purecap