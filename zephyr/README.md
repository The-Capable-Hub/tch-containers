# CHERI-Zephyr Dockerfiles

In this folder are 3 Dockerfiles:

- Dockerfile-cambridge
- Dockerfile-codasip
- Dockerfile-combined

Each of these creates an image that contains the tools (QEMU, GDB, LLVM) for targeting the relevant core. 

These are intended to be build and run on the CI to produce images for others to use. 

## Building 

They can be built:
```bash
docker build -t cheri-zephyr-codasip:latest -f Dockerfile-codasip .
docker build -t cheri-zephyr-cambridge:latest -f Dockerfile-cambridge .
docker build -t cheri-zephyr-combined:latest -f Dockerfile-combined .
```

## Running

The images are designed to be self contained SDKs of all the tools needed. 

For the dedicated tools, the environment variables are already set. 
```bash
docker run -it --rm cheri-zephyr-codasip
cd zephyr
west build -p always -b qemu_riscv64cheri_zcheripurecap samples/hello_world
west build -t run
```

or 
```bash
docker run -it --rm cheri-zephyr-cambridge
cd zephyr
west build -p always -b qemu_riscv64cheri_purecap samples/hello_world
west build -t run
```

For the combined, you'll need to setup the environment variables also.
For Codasip:

```bash
docker run -it --rm cheri-zephyr-combined 

export LLVM_CHERI_TOOLCHAIN_PATH=/opt/cheri-tools/llvm-cheri-codasip
export QEMU_BIN_PATH=/opt/cheri-tools/qemu-codasip/bin

cd zephyr
west build -p always -b qemu_riscv64cheri_zcheripurecap samples/hello_world
west build -t run
```

For Cambridge:
```bash
docker run -it --rm cheri-zephyr-combined 

export LLVM_CHERI_TOOLCHAIN_PATH=/opt/cheri-tools/llvm-cheri-cambridge
export QEMU_BIN_PATH=/opt/cheri-tools/qemu-cambridge/bin

cd zephyr
west build -p always -b qemu_riscv64cheri_purecap samples/hello_world
west build -t run
```

