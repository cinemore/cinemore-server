# Cinemore Server

**English** | [简体中文](./README_zh.md)

[Website](https://cinemore.com.cn/server)

Cinemore Server is a high-performance, cross-platform backend service for personal media library management and playback. It is designed for local storage, NAS, and cloud-based media scenarios, helping users organize scattered video resources into a unified private library and providing a consistent experience across devices.

## Features

- Organize local, NAS, and cloud media into one private library
- Support movies, TV shows, and multiple media directory mappings
- Deploy with Docker, Docker Compose, or a standalone binary
- Run with PostgreSQL or SQLite depending on your setup
- Include FFmpeg in Docker images for easier deployment

## Table of Contents

- [Screenshots](#screenshots)
- [Quick Start](#quick-start)
- [Deployment Options](#deployment-options)
- [Volume Mapping](#volume-mapping)
- [Access the Service](#access-the-service)
- [Run from the Command Line](#run-from-the-command-line)
- [FFmpeg Requirement](#ffmpeg-requirement)
- [Notes](#notes)

## Screenshots

### Home

![Cinemore Home](./screenshots/home.jpg)

### Detail

![Cinemore Detail](./screenshots/detail.jpg)

### Library

![Cinemore Library](./screenshots/resource-library.jpg)

## Quick Start

### Run with Docker

```bash
docker run -d \
  --name cinemore-server \
  -p 8000:8000 \
  -v /path/to/data:/app/data \
  -v /path/to/media:/media/xx \
  cinemore/cinemore-server:latest
```

### Run with Docker Compose

```bash
docker-compose -f docker-compose.yml up -d

# SQLite version
docker-compose -f docker-compose-sqlite.yml up -d
```

## Deployment Options

| Method | Best For | Start Command | Default Access Port | App Data | Media Path | Extra Dependency |
|---|---|---|---|---|---|---|
| Docker | Most users | `docker run ...` | `8000` | `/app/data` | `/media/...` | None |
| Docker Compose (PostgreSQL) | Persistent deployment | `docker-compose -f docker-compose.yml up -d` | `8080` | `./data` | `./media` | None |
| Docker Compose (SQLite) | Lightweight deployment | `docker-compose -f docker-compose-sqlite.yml up -d` | `8080` | `./data` | `./media` | None |
| Standalone Binary | Local development and custom integration | `./cinemore-server-linux-amd64 ...` | `8080` | `./data` | Custom path | FFmpeg required |

Notes:

- `docker-compose.yml` uses PostgreSQL by default
- `docker-compose-sqlite.yml` is a lighter option using SQLite

## Volume Mapping

### Config Directory (`/app/data`)

- **Purpose**: Stores configuration files and database files
- **Mapping**: `/path/to/data:/app/data`
- **Recommendation**: Use persistent storage so data is not lost when the container is removed
- **Compose default**: `./data:/app/data`

### Media Directory (`/media`)

- **Purpose**: Stores movies, TV shows, and other media files
- **Mapping**: `/path/to/media:/media/xx`
- **Recommendation**: Map this to your media storage path
- **Compose default**: `./media:/media`

### Multiple Media Paths Example

```yaml
volumes:
  - /path/to/1:/media/movies1
  - /path/to/2:/media/movies2
  - /path/to/3:/media/tv1
  - /path/to/4:/media/tv2
```

Important:

1. If you only have one media directory, you can map it as `/volume/medias:/media`
2. When using multiple media paths, each container-side subdirectory name under `/media` must be unique
3. `docker-compose.yml` stores PostgreSQL data in `./data/postgresql`

## Access the Service

After startup, open:

- With `docker run`: `http://ip:8000`
- With `docker-compose.yml` or `docker-compose-sqlite.yml`: `http://ip:8080`

## Run from the Command Line

If you are using the standalone binary, you can start it like this:

```bash
./cinemore-server-linux-amd64 --data="./data" --port="8080" --media="/path/to/media"
```

You can also provide an explicit config file:

```bash
./cinemore-server-linux-amd64 --config="/xxx/config.yaml"
```

### Parameters

Based on the current flag definitions in the code, the available parameters are:

```text
--data <path>       Set the config/data directory, default: ./data
--port <port>       Set the HTTP listening port, default: 8080
--media <path>      Set the local media root path, default: /
--config <path>     Set the full path to config.yaml
--version           Show version information
```

Notes:

- `--data` is used for configuration files, database files, and other app data
- `--config` points to a specific config file, for example `/xxx/config.yaml`
- `--media` points to the root directory for local media scanning or access
- To print version info only, run `./cinemore-server-linux-amd64 --version`

The default access URL for command-line startup is:

`http://ip:8080`

If you use a custom port via `--port`, replace the port in the URL accordingly.

## FFmpeg Requirement

- FFmpeg is already included in the Docker image, so no extra installation is needed for Docker deployments
- If you run the binary directly on the host, you need to install FFmpeg yourself first
- `FFmpeg 8.1` or newer is recommended

If you want to install FFmpeg from source, you can refer to the following example:

```bash
export FFMPEG_VERSION=8.1

wget -q https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz
tar -xzf ffmpeg-${FFMPEG_VERSION}.tar.gz
cd ffmpeg-${FFMPEG_VERSION}

ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
    export CFLAGS="-march=x86-64 -mtune=generic"
    export CXXFLAGS="-march=x86-64 -mtune=generic"
elif [ "$ARCH" = "aarch64" ]; then
    export CFLAGS="-march=armv8-a"
    export CXXFLAGS="-march=armv8-a"
fi

./configure \
  --prefix=/usr/local \
  --enable-shared \
  --disable-static \
  --disable-doc \
  --disable-programs
make -j$(nproc)
make install

echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/local.conf > /dev/null
sudo ldconfig
```

Additional note:

- The example above includes `--disable-programs`, which is better suited for installing FFmpeg runtime libraries
- If you also need the `ffmpeg` executable command on the system, install FFmpeg with your OS package manager or remove `--disable-programs` when compiling manually

## Notes

- Docker is the recommended deployment method for most users because it is simpler and includes all required dependencies
- The command-line binary is more suitable for local development, debugging, or custom integration
- Docker should remain the default entry point for public-facing documentation
