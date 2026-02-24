# Rust Bazel Cross-Compile Example (musl)

A minimal example of cross-compiling Rust to Linux (x86_64 and ARM64) using Bazel with musl for fully static binaries. Build on macOS or Linux and produce self-contained, statically linked binaries for Linux targets.

## Features

- **Cross-compilation** from macOS (Apple Silicon / Intel) and Linux to Linux x86_64 and AArch64
- **musl libc** for clean, portable static linking — no glibc dependency, no dynamic linker needed
- **Microarchitecture variants** for optimized builds:
  - **x86-64**: baseline, v2, v3, v4 (AVX-512)
  - **AArch64**: baseline, v8.1a through v9.6a (SVE2, etc.)
- **[mimalloc](https://github.com/microsoft/mimalloc)** as a high-performance allocator (works with musl)
- **[Tokio](https://tokio.rs/)** async runtime

## Prerequisites

- [Bazelisk](https://github.com/bazelbuild/bazelisk) (recommended) or [Bazel](https://bazel.build/) 8.x
- macOS (ARM64 or Intel) or Linux host

## Quick Start

Build for the host platform (macOS or Linux native):

```bash
bazel build //rust/hello_world:hello_world
```

Cross-compile a static binary for Linux x86_64:

```bash
bazel build //rust/hello_world:hello_world_amd64
```

Cross-compile a static binary for Linux ARM64:

```bash
bazel build //rust/hello_world:hello_world_arm64
```

Build all targets:

```bash
bazel build //...
```

## Architecture Variants

Build for specific x86-64 microarchitecture levels:

```bash
# x86-64-v2 (SSE4.2, POPCNT)
bazel build //rust/hello_world:hello_world_amd64_v2

# x86-64-v3 (AVX2, FMA)
bazel build //rust/hello_world:hello_world_amd64_v3

# x86-64-v4 (AVX-512)
bazel build //rust/hello_world:hello_world_amd64_v4
```

AArch64 ISA variants (v8.1a through v9.6a) are defined but currently commented out as unstable. Uncomment them in `rust/hello_world/BUILD` as needed.

## Build Modes

- **Debug** (default): `bazel build //rust/hello_world:hello_world_amd64`
- **Release**: `bazel build -c opt //rust/hello_world:hello_world_amd64`

Release mode enables `-Clto=thin`, `-Copt-level=3`, `-Cpanic=abort`, and symbol stripping.

## Project Structure

```text
├── MODULE.bazel          # Bazel module deps (rules_rust, toolchains_musl)
├── rust.MODULE.bazel     # Rust toolchain, musl toolchain, crates
├── settings.bzl          # Build flags, platform mappings
├── BUILD.bazel           # Platform definitions (linux-x86_64, linux-aarch64)
├── Cargo.toml            # Workspace Cargo manifest
├── Cargo.lock            # Locked dependencies
├── constraints/
│   ├── amd64/            # x86-64 microarchitecture constraints (v2, v3, v4)
│   └── arm64/            # AArch64 ISA version constraints (v8.1a–v9.6a)
└── rust/
    └── hello_world/      # Example binary (Tokio + mimalloc)
```

## Dependencies

| Dependency      | Version               |
| --------------- | --------------------- |
| rules_rust      | 0.68.1                |
| toolchains_musl | 0.1.27                |
| platforms       | 1.0.0                 |
| Rust            | 1.93.1 (edition 2024) |
| musl            | 1.2.3                 |

## Why musl?

musl was designed for static linking from the start. Compared to glibc:

- **No IFUNC**: musl doesn't use indirect functions, avoiding the [lld IRELATIVE bug](https://github.com/amsokol/rust-bazel-cross-compile-example-llvm#static-linking-and-the-lld-irelative-bug) that causes segfaults in statically linked glibc binaries built with LLVM/lld >= 20.
- **Smaller binaries**: musl produces significantly smaller static executables.
- **Self-contained**: no dynamic linker needed, no `/lib/ld-linux-*.so` dependency.
- **Container-friendly**: static musl binaries run in minimal containers like `gcr.io/distroless/static` or `scratch`.

The tradeoff is that musl has minimal locale support and a simpler DNS resolver (no nsswitch.conf). For server workloads these are rarely an issue.

## musl Version Note

Rust 1.93.1 bundles musl 1.2.5 for its pre-compiled standard library, while `toolchains_musl` 0.1.27 ships musl 1.2.3 for C code compilation. This version mismatch is safe — musl is ABI-stable across these versions. If exact version alignment is needed, consider using [`toolchains_llvm`](https://github.com/bazel-contrib/toolchains_llvm) with a custom musl sysroot built from source.

## License

MIT License — see [LICENSE](LICENSE).
