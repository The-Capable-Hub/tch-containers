# The Capable Hub Container Repo

This repo contains the Dockerfiles and workflows for building and publishing docker images for CHERI-enabled tools / projects.

Each CHERI project is separated into individual folders. Where the Dockerfile and accompanying documentation can be found.

Docker images are published to The Capable Hub [Dockerhub](https://hub.docker.com/u/capablehub) and [GitHub Container Registry](https://github.com/orgs/The-Capable-Hub/packages?repo_name=tch-containers)


Currently Supported:
- [seL4](seL4/Dockerfile)
- [qemu](qemu/Dockerfile)
- [yocto](yocto/Dockerfile-morello)
- [zephyr](zephyr/README.md)