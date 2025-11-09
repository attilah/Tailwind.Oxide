# Tailwind Oxide Solution — Execution Plan

This runbook captures every command/file needed to scaffold the `Tailwind Oxide` .NET solution with per-platform native NuGet packages under a `bindings/` folder (mirroring the structure shown in the reference screenshot). Execute the steps sequentially from the repo root.

## Useful references
- [Creating Native NuGet Packages (Microsoft Learn)](https://learn.microsoft.com/en-us/nuget/guides/native-packages) — overview of how NuGet treats `native` packages, including required tags, supported folder layout (`build`, `content`, `tools`), and guidance for shipping platform-specific props/targets files that native consumers import automatically.
- Native projects are named `<Product>.Native.<rid>` (e.g., `Tailwind.Oxide.Native.osx-arm64`) and sit directly inside a `bindings/` directory. Runtime identifiers use the lowercase `.NET RID` strings such as `osx-arm64`, `linux-x64`, `win-arm64`, etc.

## 0. Verify prerequisites
- [.NET SDK 9.0+](https://dotnet.microsoft.com/download) available on PATH
- Git installed and configured with your user.name/user.email

```bash
# Confirm toolchain
 dotnet --info
 git --version
```

## 1. Initialize git repository + folders
```bash
mkdir -p bindings src
rm -rf .git              # only if an old repo exists
rm -f .gitignore global.json Directory.Build.props Tailwind.Oxide.slnx

git init
```

## 2. Create `.gitignore` via dotnet template
```bash
dotnet new gitignore
```

## 3. Pin SDK via `dotnet new globaljson`
```bash
dotnet new globaljson --sdk-version 9.0.0 --force
```
> The template already targets .NET 9.0; no manual editing required.

## 4. Configure root-level `Directory.Build.props`
```bash
cat <<'DBP' > Directory.Build.props
<Project>
  <PropertyGroup>
    <Version>0.1.0-preview</Version>
    <OxideVersion>4.1.17</OxideVersion>
  </PropertyGroup>

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>

  <PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>

  <PropertyGroup>
    <ArtifactsDir>$(MSBuildThisFileDirectory)/artifacts</ArtifactsDir>
    <PackageOutputPath>$(ArtifactsDir)/nuget</PackageOutputPath>
  </PropertyGroup>

  <PropertyGroup>
    <Company>Tailwind Oxide</Company>
    <Authors>Tailwind Oxide</Authors>
    <Copyright>Copyright (c) 2025 Tailwind Oxide</Copyright>
    <PackageIcon>logo.png</PackageIcon>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageTags>tailwind;tailwindcss;oxide</PackageTags>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageRequireLicenseAcceptance>false</PackageRequireLicenseAcceptance>
    <RepositoryUrl>https://github.com/attilah/Tailwind.Oxide</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <PackageProjectUrl>https://github.com/attilah/Tailwind.Oxide</PackageProjectUrl>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  </PropertyGroup>

  <ItemGroup>
    <None Include="$(MSBuildThisFileDirectory)assets/logo.png" Pack="true" PackagePath="/" />
    <None Include="$(MSBuildThisFileDirectory)assets/README.md" Pack="true" PackagePath="/" />
  </ItemGroup>
</Project>
DBP
```

## 4a. Document the assets directory
```bash
mkdir -p assets
cat <<'ASSETS_README' > assets/README.md
# Tailwind Oxide Assets

Tailwind Oxide is a set of native and a managed packages on top of Tailwind CSS Oxide compiler's CFFI interface.
ASSETS_README
```

## 5. Add `bindings/Directory.Build.props`
```bash
cat <<'BINDINGS_DBP' > bindings/Directory.Build.props
<Project>
  <Import Project="$([MSBuild]::GetPathOfFileAbove('Directory.Build.props', '$(MSBuildThisFileDirectory)../'))" Condition="'' != $([MSBuild]::GetPathOfFileAbove('Directory.Build.props', '$(MSBuildThisFileDirectory)../'))" />

  <PropertyGroup>
    <Version>$(OxideVersion)</Version>
  </PropertyGroup>

  <PropertyGroup>
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <PackageTags>$(PackageTags);native</PackageTags>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <NativePackageIdPrefix>Tailwind.Oxide.Native</NativePackageIdPrefix>
    <NoWarn>$(NoWarn);NU5128</NoWarn>
    <IncludeSymbols>false</IncludeSymbols>
  </PropertyGroup>

   <ItemGroup>
    <Content Include="$(MSBuildThisFileDirectory)../runtimes/$(RuntimeIdentifier)/$(TailwindOxideNativeLibraryName)">
      <PackagePath>runtimes/$(RuntimeIdentifier)/native/</PackagePath>
      <Pack>true</Pack>
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      <Link>runtimes/$(RuntimeIdentifier)/native/$(TailwindOxideNativeLibraryName)</Link>
    </Content>
  </ItemGroup>

  <ItemGroup>
    <Content Include="buildTransitive/$(NativePackageIdPrefix).$(RuntimeIdentifier).targets">
      <PackagePath>buildTransitive/</PackagePath>
      <Pack>true</Pack>
    </Content>
  </ItemGroup>
</Project>
BINDINGS_DBP
```

## 6. Create the Tailwind Oxide solution shell
```bash
dotnet new sln -n Tailwind.Oxide --format slnx
```

## 7. Bootstrap a single native project first
> Per requirement, scaffold **only one** native project until it is verified end-to-end. Once it builds and packs correctly, duplicate the pattern for the remaining RIDs.

1. Pick an initial runtime (example below uses `osx-arm64`, matching the screenshot naming pattern).
2. Scaffold and wire it up:
```bash
dotnet new classlib -f net9.0 -n Tailwind.Oxide.Native.osx-arm64 -o bindings/Tailwind.Oxide.Native.osx-arm64
rm bindings/Tailwind.Oxide.Native.osx-arm64/Class1.cs
```
3. Add the project to the solution:
```bash
dotnet sln Tailwind.Oxide.slnx add bindings/Tailwind.Oxide.Native.osx-arm64/Tailwind.Oxide.Native.osx-arm64.csproj
```

## 8. Add build-transitive targets + enrich the project file
1. Create `buildTransitive/Tailwind.Oxide.Native.targets` so restored packages automatically expose the right targets:
```bash
mkdir -p bindings/Tailwind.Oxide.Native.osx-arm64/buildTransitive
cat <<'TARGETS' > bindings/Tailwind.Oxide.Native.osx-arm64/buildTransitive/Tailwind.Oxide.Native.osx-arm64.targets
<Project>
  <!-- This targets file is automatically imported for transitive PackageReference dependencies -->
  <!-- NuGet will automatically resolve and copy the correct native library based on RuntimeIdentifier -->
</Project>
TARGETS
```
2. Update `bindings/Tailwind.Oxide.Native.osx-arm64/Tailwind.Oxide.Native.osx-arm64.csproj` so each native project includes RID metadata, package identity, descriptive text, and the platform-specific library name:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <RuntimeIdentifier>osx-arm64</RuntimeIdentifier>

    <PackageId>$(NativePackageIdPrefix).$(RuntimeIdentifier)</PackageId>
    <Description>macOS arm64 native binaries for Tailwind.Oxide</Description>

    <Tailwind.OxideNativeLibraryName>libtailwind_cffi.dylib</Tailwind.OxideNativeLibraryName>
  </PropertyGroup>
</Project>
```
> Adjust the `RuntimeIdentifier`, `Description`, and `Tailwind.OxideNativeLibraryName` whenever you copy this pattern to new runtime directories. Always select the correct native extension: use `.dylib` for macOS RIDs, `.dll` for Windows RIDs, `.so` for Linux/FreeBSD RIDs, and the `tailwindcss-oxide.wasm32-wasi.wasm` payload for the browser target so consuming apps load the proper binary.

## 9. Create the managed `Tailwind.Oxide` facade project
1. Scaffold the root library in `src/`:
```bash
dotnet new classlib -f net9.0 -n Tailwind.Oxide -o src/Tailwind.Oxide
rm src/Tailwind.Oxide/Class1.cs
```
2. Update `src/Tailwind.Oxide/Tailwind.Oxide.csproj` to allow unsafe blocks and remove the default `<ImplicitUsings>` / `<Nullable>` entries so the property group stays minimal:
```xml
<PropertyGroup>
  <TargetFramework>net9.0</TargetFramework>
  <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```
3. Create a `buildTransitive/Tailwind.Oxide.targets` file similar to the native packages so the managed package brings in the binding references automatically:
```bash
mkdir -p src/Tailwind.Oxide/buildTransitive
cat <<'TWOTARGETS' > src/Tailwind.Oxide/buildTransitive/Tailwind.Oxide.targets
<Project>
  <!-- Note: This file is included in the Tailwind.Oxide NuGet package and will auto-reference native packages -->
  <!-- The version is embedded during pack time, so consuming projects don't need to specify it -->
</Project>
TWOTARGETS
```
4. Add the project to the solution and reference every native binding project (for now, add the single prototype; repeat commands as new bindings come online):
```bash
dotnet sln Tailwind.Oxide.slnx add src/Tailwind.Oxide/Tailwind.Oxide.csproj
for proj in bindings/*/*.csproj; do
  dotnet add src/Tailwind.Oxide/Tailwind.Oxide.csproj reference "$proj"
done
```
> The managed `Tailwind.Oxide` project should always reference *all* binding projects so consumers pull in platform-specific native assets transitively. Re-run the `dotnet add reference` loop each time a new binding is created.

## 10. Capture the remaining runtimes (after approval)
When the initial platform passes validation, replicate the same native project structure, `buildTransitive/Tailwind.Oxide.Native.targets` file, and folder layout for the rest of the matrix. Reference list for later rollout (all inside `bindings/`):

| Platform target | Project directory | Runtime identifier (RID) | Native artifact |
|-----------------|-------------------|--------------------------|-----------------|
| macOS arm64     | bindings/Tailwind.Oxide.Native.osx-arm64 | osx-arm64 | `.dylib` |
| macOS x64       | bindings/Tailwind.Oxide.Native.osx-x64   | osx-x64   | `.dylib` |
| Linux arm gnu   | bindings/Tailwind.Oxide.Native.linux-arm | linux-arm | `.so` |
| Linux arm gnueabihf | bindings/Tailwind.Oxide.Native.linux-arm-gnueabihf | linux-arm-gnueabihf | `.so` |
| Linux arm64 gnu | bindings/Tailwind.Oxide.Native.linux-arm64 | linux-arm64 | `.so` |
| Linux arm64 musl | bindings/Tailwind.Oxide.Native.linux-arm64-musl | linux-arm64-musl | `.so` |
| Linux x64 gnu   | bindings/Tailwind.Oxide.Native.linux-x64 | linux-x64 | `.so` |
| Linux x64 musl  | bindings/Tailwind.Oxide.Native.linux-x64-musl | linux-x64-musl | `.so` |
| FreeBSD x64     | bindings/Tailwind.Oxide.Native.freebsd-x64 | freebsd-x64 | `.so` |
| Windows arm64   | bindings/Tailwind.Oxide.Native.win-arm64 | win-arm64 | `.dll` |
| Windows x64     | bindings/Tailwind.Oxide.Native.win-x64   | win-x64   | `.dll` |
| Browser WASM    | bindings/Tailwind.Oxide.Native.browser-wasm | browser-wasm | `tailwindcss-oxide.wasm32-wasi.wasm` |

## 11. Add future projects to the solution in bulk
> Skip this bulk add until the remaining platform projects exist. For the initial prototype, the explicit `dotnet sln add ...` calls in steps 7 and 9 are sufficient.

```bash
for proj in bindings/*/*.csproj; do
  dotnet sln Tailwind.Oxide.slnx add "$proj"
done
```

## 12. Prime git state
```bash
git add .
git status
# Review outputs, then commit when satisfied
# git commit -m "chore: initial scaffold Tailwind.Oxide solution"
```

## 13. Next steps
- Fill in native interop source files per platform (likely under `buildTransitive/` with RID-specific targets, mirroring the reference layout).
- Add CI workflows and README badges as needed.
