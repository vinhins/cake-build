# Cake build

Demonstrates a basic build of a `.NET Core` `NuGet` package using [Cake][cake].

I tried to create a somewhat realistic scenario without writing too much code:

- The solution contains two projects which will be packed as `NuGet` packages.
  - The `SuperLogic` project depends from `Logic` and when packing this project reference will be turned into a `NuGet` package reference (handled out of the box by `dotnet pack`).
  - The `Logic` project references a `NuGet` package from [nuget.org][nuget-org] via a `PackageReference`, `dotnet pack` will turn this into a package reference.
- The projects target both `nestandard2.0` and `net461` so they can be used with the `.NET Framework`.
- The solution contains a test project.
- Use [`SemVer`][semver] to version the `DLLs` and the `NuGet package`
  - *Note*: `SemVer` is implemented via [`GitVersion`][git-version]

## Benefits over a nuspec file

- A single file describing the package and the project instead of two (`*.csproj` and `*.nuspec`)
- References (projects or `Nuget` packages) are resolved automatically. There is no need to tweak a file manually anymore!

## Referencing a project without turning it into a package reference

The `SuperLogic` project depends on the `ExtraLogic` project but we don't want to ship `ExtraLogic` as a package. Instead we want to include `Contoso.Hello.ExtraLogic.dll` in the `SuperLogic` package directly. Currently this is not supported out of the box but the team is [tracking it][pack-issues].

Luckily [this issue][project-reference-dll-issue] provides a workaround. All the modifications will take place in `SuperLogic.csproj`.

- In the `<PropertyGroup>` section add the following line:

```xml
<TargetsForTfmSpecificBuildOutput>$(TargetsForTfmSpecificBuildOutput);IncludeReferencedProjectInPackage</TargetsForTfmSpecificBuildOutput>
```

- Prevent the project to be added as a package reference by making [all assets private][private-assets].

```xml
<ProjectReference Include="..\ExtraLogic\ExtraLogic.csproj">
  <PrivateAssets>all</PrivateAssets>
</ProjectReference>
```

- Finally add the target responsible of copying the `DLL`:

```xml
<Target Name="IncludeReferencedProjectInPackage">
  <ItemGroup>
    <BuildOutputInPackage Include="$(OutputPath)Contoso.Hello.ExtraLogic.dll" />
  </ItemGroup>
</Target>
```

## Pinning the version of Cake

To pin the version of `Cake`, add the following lines to your `.gitignore` file:

```bash
tools/*
!tools/packages.config
```

Pinning the version of `Cake` guarantees you'll be using the same version of `Cake` on your machine and in the build server.

## Reusing this to bootstrap your project

- Get the latest [`build.ps1`][build-ps1]
- Copy [`build.cake`][build-cake] into the root of your directory
- [Pin](#Pinning-the-version-of-Cake) the version of `Cake`

[cake]: https://cakebuild.net/
[build-ps1]: https://raw.githubusercontent.com/cake-build/example/master/build.ps1
[nuget-org]: https://www.nuget.org/
[build-cake]: build.cake
[semver]: https://semver.org/
[git-version]: https://gitversion.readthedocs.io/en/latest/
[pack-issues]: https://github.com/NuGet/Home/issues/6285
[project-reference-dll-issue]: https://github.com/NuGet/Home/issues/3891
[private-assets]: https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#includeassets-excludeassets-and-privateassets
