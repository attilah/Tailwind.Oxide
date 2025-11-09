# Tailwind Oxide

[![CI](https://github.com/attilah/Tailwind.Oxide/actions/workflows/ci.yml/badge.svg)](https://github.com/attilah/Tailwind.Oxide/actions/workflows/ci.yml)
[![CD](https://github.com/attilah/Tailwind.Oxide/actions/workflows/cd.yml/badge.svg)](https://github.com/attilah/Tailwind.Oxide/actions/workflows/cd.yml)
[![NuGet](https://img.shields.io/nuget/v/Tailwind.Oxide.svg)](https://www.nuget.org/packages/Tailwind.Oxide)
[![Downloads](https://img.shields.io/nuget/dt/Tailwind.Oxide.svg)](https://www.nuget.org/packages/Tailwind.Oxide)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](#license)

Tailwind Oxide is a managed .NET wrapper over the Tailwind CSS Oxide compiler’s C Foreign Function Interface, exposing a friendly API surface while preserving Tailwind’s native performance on every supported runtime. Native binaries are distributed as per-RID NuGet packages so consuming apps automatically pick up the right artifact.

> The underlying compiler and native artifacts live in the [Tailwind Labs](https://github.com/tailwindlabs/tailwindcss) repository.

## Build

```bash
# Restore dependencies and build every managed/native package
 dotnet build Tailwind.Oxide.slnx

# Pack NuGet artifacts (already enabled via Directory.Build.props)
 dotnet pack Tailwind.Oxide.slnx
```

## Supported platforms

| Platform target | Runtime identifier (RID) | Native extension | Project directory |
|-----------------|--------------------------|------------------|-------------------|
| macOS arm64     | osx-arm64                | .dylib           | bindings/Tailwind.Oxide.Native.osx-arm64 |
| macOS x64       | osx-x64                  | .dylib           | bindings/Tailwind.Oxide.Native.osx-x64 |
| Linux arm gnu   | linux-arm                | .so              | bindings/Tailwind.Oxide.Native.linux-arm |
| Linux arm gnueabihf | linux-arm-gnueabihf  | .so              | bindings/Tailwind.Oxide.Native.linux-arm-gnueabihf |
| Linux arm64 gnu | linux-arm64              | .so              | bindings/Tailwind.Oxide.Native.linux-arm64 |
| Linux arm64 musl | linux-arm64-musl        | .so              | bindings/Tailwind.Oxide.Native.linux-arm64-musl |
| Linux x64 gnu   | linux-x64                | .so              | bindings/Tailwind.Oxide.Native.linux-x64 |
| Linux x64 musl  | linux-x64-musl           | .so              | bindings/Tailwind.Oxide.Native.linux-x64-musl |
| FreeBSD x64     | freebsd-x64              | .so              | bindings/Tailwind.Oxide.Native.freebsd-x64 |
| Windows arm64   | win-arm64                | .dll             | bindings/Tailwind.Oxide.Native.win-arm64 |
| Windows x64     | win-x64                  | .dll             | bindings/Tailwind.Oxide.Native.win-x64 |
| Browser WASM    | browser-wasm             | .wasm            | bindings/Tailwind.Oxide.Native.browser-wasm |

Each native project packs a runtime-specific binary located under `runtimes/<rid>` and exports a build-transitive target so managed consumers can use Tailwind Oxide with zero manual RID wiring.

## License

Tailwind Oxide is distributed under the matching MIT license of Tailwind CSS.
