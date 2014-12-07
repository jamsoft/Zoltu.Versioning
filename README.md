# Zoltu.Version

NuGet package for automatically versioning a project with git.

[![Build status](http://img.shields.io/appveyor/ci/Zoltu/zoltu-versioning.svg)](https://ci.appveyor.com/project/Zoltu/zoltu-versioning)
[![NuGet Status](http://img.shields.io/nuget/v/Zoltu.Versioning.svg)](https://www.nuget.org/packages/Zoltu.Versioning/)


## Usage

 * Install the NuGet package (https://www.nuget.org/packages/Zoltu.Versioning/).
 * Remove `AssemblyVersion`, `AssemblyFileVersion` and `AssemblyInformationalVersion` attributes from your AssemblyInfo.cs (if you have them).
 * When you want to increase the major or minor version numbers, tag the commit that should bump the version with v#.#
  * Example Tag: v3.5
  * Note: If you are using a build server that automatically builds on commit, it is recommended that you tag *before* pushing to the remote master.  This is because your build server will kick off without that tag and therefore the build that is generated will not have the newly tagged version.
 * You can also add a tag in the form v#.#-string to generate a special informational version attribute on the assembly.  This is useful for NuGet prerelease packages.


## Versioning

The versioning system is simple, and designed to follow semantic versioning for CI/CD projects more or less.  When a build is kicked off, a VersionAssemblyInfo.cs file is generated (no need to add it to your project, it is compiled in automatically).  The VersionAssemblyInfo.cs file contains `AssemblyVersion`, `AssemblyFileVersion` and `AssemblyInformationalVersion` assembly attributes.

The first step to figuring out the version is getting the git repository.  This is done by traversing the directory tree until a valid .git folder is found.  If none is found then the version is 0.0.0.0.

The second step is to walk the commit history from HEAD to the first commit, looking for a tag with a name that matches v#.# (the actual regex is `^v(\d+).(\d+)$`). If such a tag is found, then the number of commits traversed are counted to generate the patch version and the major and minor versions are parsed from the tag.

## Example

action | version | info version
------------ | ------------- | -------------
tag `v1.2` | `1.2.0.0` | `1.2.0.0`
commit 3 times | `1.2.3.0` | `1.2.3.0`
tag `v1.3-RC` | `1.2.3.0` | `1.3.0.0-RC`
commit 3 times | `1.2.6.0` | `1.3.0.0-RC-003`
tag `v1.3` | `1.3.0.0` | `1.3.0.0`

Note: The build version never changes because in the .NET world it is ignored.

### Skipping revision and build parts

It is possible to instruct the versioning system to output only a version consiting of only major and minor. This can be done with each of the three version attributes independently by setting relevant variables in the .csproj file.

```
<GitVersionOnlyMajorAndMinorInAssemblyVersion />
<GitVersionOnlyMajorAndMinorInAssemblyFileVersion />
<GitVersionOnlyMajorAndMinorInAssemblyInformationalVersion />
```

When the first is set to `true` in the above example the output attributes will be as shown below.

``` xml
<PropertyGroup>
    <GitVersionOnlyMajorAndMinorInAssemblyVersion>
        true
    </GitVersionOnlyMajorAndMinorInAssemblyVersion>
</PropertyGroup>
```

``` c#
[assembly: AssemblyVersion("3.5.0.0")]
[assembly: AssemblyFileVersion("3.5.10.0")]
[assembly: AssemblyInformationalVersion("3.5.10.0")]
```

### NCrunch Integration

NCrunch builds will start failing after adding this project to your solution due to being unable to find LibGit2Sharp libraries `System.IO.FileNotFoundException: Could not load file or assembly 'LibGit2Sharp, Version=0.18.1.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified.`
To work around this, add the following to your .ncrunchproject file:

``` xml
<AdditionalFilesToInclude>..\packages\Zoltu.Versioning.*\build\**.*</AdditionalFilesToInclude>
```
