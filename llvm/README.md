# CHERI-LLVM Dockerfile

Prebuilt Docker images for `linux/arm64` and `linux/amd64` host architectures are published to The Capable Hub's [Dockerhub](https://hub.docker.com/u/capablehub) and [GitHub Container Registry](https://github.com/orgs/The-Capable-Hub/packages?repo_name=tch-containers) pages.

These images contain CHERI capable versions of Clang and LLD, targeting `riscv64` only.

### Available Variants
The workflow builds docker images a

| Image Name | Branch | LLVM Version | Tags |
| :--- | :--- | :--- | :--- |
| `cheri-ca-llvm` | `codasip-cheri-riscv` | 17 | `17-riscv64-latest`, `17-riscv64-<commit-hash>` |
| `cheri-ca-llvm` | `codasip-cheri-riscv-18` | 18 |`18-riscv64-latest`, `18-riscv64-<commit-hash>` |
| `cheri-ca-llvm` | `codasip-cheri-riscv-19` | 19 |`19-riscv64-latest`, `19-riscv64-<commit-hash>` |
| `cheri-ctsrd-llvm` | `master` | 17 |`17-riscv64-latest`, `17-riscv64-<commit-hash>` |

**`-latest`**: Tracks the `HEAD` of the respective branch.

**`-<commit-hash>`**: A pinned version corresponding to the first 7 characters of the git commit hash (SHA) of the source repo.

### Example Usage

```bash
# Pull the latest CHERI Alliance LLVM 17 from Dockerhub:
docker pull capablehub/cheri-ca-llvm:17-riscv64-latest

# Pull the latest University of Cambridge LLVM 17 from ghcr:
docker pull ghcr.io/the-capable-hub/cheri-ctsrd-llvm:17-riscv64-latest

# Pull the a specific commit hash of University of Cambridge LLVM 17 from ghcr:
docker pull ghcr.io/the-capable-hub/cheri-ctsrd-llvm:17-riscv64-74d200f
```

## Building 

Building an llvm image requires you to specify some build arguments:

| Build Arguments | Default | Description |
|---|---|---|
| REPO_URL | https://github.com/CHERI-Alliance/llvm-project.git | The URL of the repo to fetch the LLVM source from |
| BRANCH | main | The branch to use |
| COMMIT_HASH | HEAD | HEAD or the specific commit hash you want to checkout before building |
| INSTALL_DIR | /root/llvm-cheri/install | The location of where LLVM should be installed. This will be added to the containers PATH variable |

For example, to build a docker image of the latest Cambridge LLVM repo:
```bash
docker buildx build \
    --build-arg REPO_URL=https://github.com/CTSRD-CHERI/llvm-project.git \
    --build-arg BRANCH=master \
    -f llvm/Dockerfile \
    --tag cheri-ctsrd-llvm:17-riscv64-latest .
```

Or to build a specific commit:
```bash
docker buildx build \
    --build-arg REPO_URL=https://github.com/CTSRD-CHERI/llvm-project.git \
    --build-arg BRANCH=master \
    --build-arg COMMIT_HASH=238a953f5c7ca3de3cff67bf2433a0f59c9677ee
    -f llvm/Dockerfile \
    --tag cheri-ctsrd-llvm:17-riscv64-238a953 .
```

## Usage from CLI

The image can be used to call clang directly:

```bash
docker run --rm cheri-ctsrd-llvm:17-riscv64-latest clang --version

clang version 17.0.0 (https://github.com/CTSRD-CHERI/llvm-project.git 74d200f2bb65b06633ca87f86b1b9145ebf63a46)
Target: riscv64-unknown-unknown-elf
Thread model: posix
InstalledDir: /root/llvm-cheri/install/bin

```

To compile a file in your local directory (assuming its called main.c):
```bash
docker run --rm -v $(pwd):/work -w /work cheri-ctsrd-llvm:17-riscv64-latest \
    clang -c main.c -o main.o
```

## Usage in Dockerfiles

This docker image can be used to speedup building other images that require llvm

Within your project's Dockerfile use the COPY instruction to copy in the llvm install directory. 

```bash
COPY --from=cheri-ca-llvm:17-riscv64-latest --chown=myuser:mygroup /root/llvm-cheri/install /opt/cheri-tools/llvm-cheri-codasip
```
> [!NOTE]
> If you need to change ownership of the copied files, it is best to do it as part of the COPY instruction as this save image disk size.