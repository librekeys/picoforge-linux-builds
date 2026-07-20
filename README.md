# PicoForge Linux AppImage Builds (Podman)

Containerized, Podman-native AppImage build infrastructure for [PicoForge](https://github.com/librekeys/picoforge).

This repository provides container environment definitions (`Containerfile`s) and build scripts for producing **4 AppImage release variants**:

| Variant | C Library Target | Baseline / Target OS | Architecture | Output Filename Pattern |
|---|---|---|---|---|
| `glibc-x86_64` | `glibc` | Rocky Linux 8 (glibc 2.28+) | `x86_64` | `picoforge_<version>_glibc-2.28_x86-64.AppImage` |
| `glibc-aarch64` | `glibc` | Rocky Linux 8 (glibc 2.28+) | `aarch64` | `picoforge_<version>_glibc-2.28_aarch64.AppImage` |
| `musl-x86_64` | `musl` | Alpine Linux (musl libc) | `x86_64` | `picoforge_<version>_musl_x86-64.AppImage` |
| `musl-aarch64` | `musl` | Alpine Linux (musl libc) | `aarch64` | `picoforge_<version>_musl_aarch64.AppImage` |

---

## Architecture & Design

- **Podman-Native**: All build environments are defined via dedicated `Containerfile`s (`Containerfile.glibc-x86_64`, `Containerfile.glibc-aarch64`, `Containerfile.musl-x86_64`, `Containerfile.musl-aarch64`).
- **glibc Compatibility Floor**: Built on Rocky Linux 8 (`glibc 2.28`) with `gcc-toolset-13`, ensuring wide backward compatibility across standard desktop distributions (Ubuntu 20.04+, Debian 10+, RHEL 8+, Fedora, Arch).
- **musl Compatibility**: Built on Alpine Linux with dynamically linked `musl` libraries for musl-based distributions (Alpine, Void Linux, Chimera Linux, etc.).
- **Host Dependencies**: All required shared libraries are bundled inside the AppImage by `linuxdeploy`, with the exception of `libpcsclite.so.1` (the smartcard daemon library, which talks to `pcscd` on the host system).

---

## Local Building with Podman

To build any variant locally, clone `picoforge` inside this directory (or mount your local copy of `picoforge`):

```bash
# 1. Build the Podman container image
podman build -t picoforge-builder:glibc-x86_64 -f Containerfile.glibc-x86_64 .

# 2. Run the build script inside the container
podman run --rm -v $(pwd)/..:/workspace:z picoforge-builder:glibc-x86_64 /workspace/picoforge-linux-builds/build.sh glibc-x86_64
```

The resulting AppImage will be placed in `dist/`.

---

## GitHub Actions CI/CD

The workflow `.github/workflows/build-appimages.yml` automatically triggers on push and `workflow_dispatch`. It uses native GitHub runner architectures (`ubuntu-24.04` for x86_64, `ubuntu-24.04-arm` for ARM64) to build all 4 AppImage variants using `podman build` and `podman run`.
