# PicoForge Linux AppImage Builds (Podman)

Containerized, Podman-native AppImage build infrastructure for [PicoForge](https://github.com/librekeys/picoforge).

This repository provides container environment definitions (`Containerfile`s) and build scripts for producing **4 AppImage release variants**:

| Variant | C Library Target | Baseline / Target OS | Architecture | GHCR Builder Image Tag | Output Filename Pattern |
|---|---|---|---|---|---|
| `glibc-x86_64` | `glibc` | Rocky Linux 8 (glibc 2.28+) | `x86_64` | `glibc-2.28-x86_64` | `picoforge_<version>_glibc-2.28_x86-64.AppImage` |
| `glibc-aarch64` | `glibc` | Rocky Linux 8 (glibc 2.28+) | `aarch64` | `glibc-2.28-aarch64` | `picoforge_<version>_glibc-2.28_aarch64.AppImage` |
| `musl-x86_64` | `musl` | Alpine Linux (musl libc) | `x86_64` | `musl-x86_64` | `picoforge_<version>_musl_x86-64.AppImage` |
| `musl-aarch64` | `musl` | Alpine Linux (musl libc) | `aarch64` | `musl-aarch64` | `picoforge_<version>_musl_aarch64.AppImage` |

---

## Architecture & Container Caching

- **Podman-Native**: All build environments are defined via dedicated `Containerfile`s (`Containerfile.glibc-x86_64`, `Containerfile.glibc-aarch64`, `Containerfile.musl-x86_64`, `Containerfile.musl-aarch64`).
- **Prebuilt Image Caching via GHCR**: Container images are built and pushed to **GitHub Container Registry** (`ghcr.io/${{ github.repository_owner }}/picoforge-appimage-builder:<tag>`). During AppImage builds, Podman pulls the prebuilt image in ~5 seconds instead of building from scratch.
- **glibc Compatibility Floor**: Built on Rocky Linux 8 (`glibc 2.28`) with `gcc-toolset-13`, ensuring wide backward compatibility across standard desktop distributions (Ubuntu 20.04+, Debian 10+, RHEL 8+, Fedora, Arch).
- **musl Compatibility**: Built on Alpine Linux with dynamically linked `musl` libraries for musl-based distributions (Alpine, Void Linux, Chimera Linux, etc.).
- **Host Dependencies**: All required shared libraries are bundled inside the AppImage by `linuxdeploy`, with the exception of `libpcsclite.so.1` (the smartcard daemon library, which talks to `pcscd` on the host system).

---

## Local Building with Podman

To build any variant locally, clone `picoforge` inside this directory (or mount your local copy of `picoforge`):

```bash
# 1. Build the Podman container image
podman build -t picoforge-appimage-builder:glibc-2.28-x86_64 -f Containerfile.glibc-x86_64 .

# 2. Run the build script inside the container
podman run --rm -v $(pwd)/..:/workspace:z picoforge-appimage-builder:glibc-2.28-x86_64 /workspace/picoforge-linux-builds/build.sh glibc-x86_64
```

The resulting AppImage will be placed in `dist/`.

---

## GitHub Actions CI/CD Workflows

1. **[`publish-containers.yml`](.github/workflows/publish-containers.yml)**: Automatically builds and pushes all 4 builder images to GHCR whenever any `Containerfile.*` changes.
2. **[`build-appimages.yml`](.github/workflows/build-appimages.yml)**: Pulls prebuilt builder images from GHCR (with local build fallback), executes `podman run` to compile PicoForge, and uploads/releases the resulting AppImages.
