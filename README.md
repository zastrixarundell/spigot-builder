# Spigot-Builder

Containerized builder for SpigotMC.

[![Docker Pulls](https://img.shields.io/docker/pulls/zastrix/spigot-builder?logo=docker)
](https://hub.docker.com/r/zastrix/spigot-builder)

## Summary

This is a container which, during runtime, downloads the BuildTools.jar file to the latest version and builds the specified version of spigot. Due to the different requirements of the JRE for specific version the appropriate image needs to be set.

## Required version for a specific SpigotMC version

Here's a table with the required version of the images for a specific SpigotMC version. 

|Spigot version|Image version|
|------|------|
|<1.17|openjdk-8-alpine|
|=1.17.1|openjdk-16-alpine|
|>1.17.1|openjdk-17-alpine|

## Usage

For multiple builds at the same time, ideally a [`docker-compose.yml`](./docker-compose-example.yml) file would be used. The two important directories are `/release` into which the final `.jar` file is copied to and the `/build` directory which builds the server. `/build` can be used in a volume to reduce the build time after the initial build, but in case that the build corrups the volume needs to be dropped otherwise it won't build. 

### docker-compose.yml

An example of the `docker-compose.yml` file would be:

```yml
version: '3'

volumes:
  build_1_20:
  build_1_17_1:
  build_1_11_1:

services:
  '1.20':
    image: zastrix/spigot-builder:openjdk-17-alpine
    environment:
      VERSION: '1.20'
    volumes:
      - build_1_20:/build:Z
      - ./release:/release:z
    restart: on-failure
  '1.17.1':
    image: zastrix/spigot-builder:openjdk-16-alpine
    environment:
      VERSION: '1.17.1'
    volumes:
      - build_1_17_1:/build:Z
      - ./release:/release:z
    restart: on-failure
  '1.11.1':
    image: zastrix/spigot-builder:openjdk-8-alpine
    environment:
      VERSION: '1.11.1'
    volumes:
      - build_1_11_1:/build:Z
      - ./release:/release:z
    restart: on-failure
```

With the result of:

```yml
$ tree .
.
├── build.sh
├── docker-compose-example.yml
├── docker-compose.yml
├── Dockerfile16
├── Dockerfile17
├── Dockerfile8
├── README.md
└── release
    ├── spigot-1.11.2.jar
    ├── spigot-1.17.1.jar
    └── spigot-1.20.1.jar

2 directories, 10 files
```


*Note: The `1.11.2` is actually effectively `1.11.1`.*

### CLI

To build spigot `1.17.1` with a single command and no cache you would need to run:

```bash
docker run -v ./release:/release:z -e VERSION='1.17.1' --name spigot-builder-1.11.1 zastrix/spigot-builder:openjdk-16-alpine
```

And in the release folder the file should exist: `./release/spigot-1.17.1.jar`.