# CHERI-Zephyr Dockerfiles

Prebuilt Docker images for `linux/arm64` and `linux/amd64` host architectures are published to The Capable Hub's [Dockerhub](https://hub.docker.com/u/capablehub) and [GitHub Container Registry](https://github.com/orgs/The-Capable-Hub/packages?repo_name=tch-containers) pages.

These images contain a release of [CHERI-Zephyr](https://github.com/CHERI-Alliance/CHERI-zephyr) as well as a compatible versions of LLVM, QEMU and GDB for the ISAv9 or RVY architectures.

## Available Variants
The workflow builds these docker images

| Image Tag | CHERI Zephyr Version | Architecture | Variant |
| :--- | :--- | :--- | :--- |
| `cheri-zephyr:v0.2.0-riscv64c-rvy` | v0.2.0 | riscv64c | RVY |
| `cheri-zephyr:v0.2.0-riscv64c-isav9` | v0.2.0 | riscv64c | ISAv9 |


## Getting started

Determine which version of the tools you want to use. This will be different depending on the CHERI architecture you want to target.

For Codasip / RVY tools:
```bash
# For Codasip / RVY tools 
docker run --rm -it capablehub/cheri-zephyr:v0.2.0-riscv64c-rvy

# Move into the zephyr folder
cd zephyr
# Build and run the hello_world sample in QEMU
west build -p always -b qemu_riscv64cheri_zcheripurecap samples/hello_world
west build -t run
```

For Cambridge / ISAv9 tools:
```bash
# For Cambridge / ISAv9 tools 
docker run --rm -it capablehub/cheri-zephyr:v0.2.0-riscv64c-isav9

# Move into the zephyr folder
cd zephyr
# Build and run the hello_world sample in QEMU
west build -p always -b qemu_riscv64cheri_purecap samples/hello_world
west build -t run
```


## Building

The Dockerfile used for building supports a range of `--build-args`.
It will default to including the `rvy` variant of tools.

| Build Arguments 	| Default 	| Required 	| Description 	|
|---	|---	|---	|---	|
| ENABLE_TOOLS 	| rvy 	| false 	| "rvy" or "isav9" to select either RVY (CHERI Alliance) or ISAv9 (Cambridge) sets of LLVM, QEMU and GDB 	|
| BUILD_TMP_DIR 	| /tmp/cheri 	| false 	| The directory to build / install LLVM, QEMU and GDB in the intermediate containers 	|
| FINAL_INSTALL_DIR 	| /opt/cheri-tools 	| false 	| The directory the tools will be installed in the final image 	|
| LLVM_ISAV9_IMAGE 	| capablehub/cheri-ctsrd-llvm:17-riscv64-74d200f 	| false 	| External image used as the source for ISAv9 LLVM binaries 	|
| LLVM_RVY_IMAGE 	| capablehub/cheri-ca-llvm:17-riscv64-678272e 	| false 	| External image used as the source for RVY LLVM binaries 	|
| QEMU_REPO 	| https://github.com/CHERI-Alliance/qemu.git<br>https://github.com/CTSRD-CHERI/qemu.git 	| false 	| Repo to fetch QEMU from. Default will be selected based on `ENABLE_TOOLS` value 	|
| QEMU_BRANCH 	| codasip-cheri-riscv_v3<br>qemu-cheri 	| false 	| Branch to use for QEMU. Default will be selected based on `ENABLE_TOOLS` value 	|
| QEMU_COMMIT_HASH 	| 2a2e882b<br>HEAD 	| false 	| Commit hash to use for QEMU branch. Default will be selected based on `ENABLE_TOOLS` value 	|
| GDB_REPO 	| https://github.com/CHERI-Alliance/gdb.git<br>https://github.com/CTSRD-CHERI/gdb.git 	| false 	| Repo to fetch GDB from. Default will be selected based on `ENABLE_TOOLS` value 	|
| GDB_BRANCH 	| codasip-cheri-riscv<br>cheri-14 	| false 	| Branch to use for GDB. Default will be selected based on `ENABLE_TOOLS` value 	|
| GDB_COMMIT_HASH 	| df929d4d<br>HEAD 	| false 	| Commit hash to use for GDB branch. Default will be selected based on `ENABLE_TOOLS` value 	|
| ZEPHYR_REPO_URL 	| https://github.com/CHERI-Alliance/CHERI-zephyr 	| false 	| Repo to fetch CHERI Zephyr from  	|
| ZEPHYR_BRANCH 	| CHERI-RISCV64-v2.0 	| false 	| Branch to use for CHERI Zephyr 	|
| ZEPHYR_COMMIT_HASH 	| HEAD 	| false 	| Commit hash for CHERI Zephyr 	|

So for example, to build the `RVY` variant:

```bash
docker buildx build \
 -t cheri-zephyr:v0.2.0-riscv64c-rvy \
 -f zephyr/Dockerfile \
 .
```

Or for the `ISAv9` variant:

```bash
docker buildx build \
  --build-arg ENABLE_TOOLS=isav9 \
  -f zephyr/Dockerfile \
  -t cheri-zephyr:v0.2.0-riscv64c-isav9 \
  .
```
