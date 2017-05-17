# NuGet Packer
NuGet Packer helps you turn your .NET (.csproj/.vbproj) projects into NuGet packages as simply and effectively as possible.

## Basic Usage
Using NuGet Packer is as simple as installing a NuGet package and building your project:

1. Install the NuGet package [NuGet.Packer](https://www.nuget.org/packages/NuGet.Packer/)
  * Using [the NuGet package management GUI](http://docs.nuget.org/consume/package-manager-dialog), or
  * Typing the following command in the Package Management Console:
  
    `Install-Package NuGet.Packer`

2. Compile your project using the 'Release' configuration
  * [Choose "Release" from the Visual Studio configuration drop-down](https://msdn.microsoft.com/en-us/library/wx0123s5.aspx), or
  * Specify the `/p:Configuration=Release` flag when calling MSBuild from the command line

That's it!  Your NuGet package will be in your `/bin` folder, right next to your project's output assembly (.dll).  It will be named `[Assembly Name].[Version].nupkg`

## Customizing Your NuGet Package
NuGet Packer is just a thin wrapper around the NuGet executable, so all of the [standard documentation around creating NuGet packages](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package) still applies, especially the part about [defining your project's metadata](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package#user-content-from-a-project) so that NuGet can use it in the package it generates.

In other words, if you update the attributes in your project's `AssemblyInfo.cs` file, NuGet will use those to create your package.  So you want to make it look something like this:

_AssemblyInfo.cs_
```
using System.Reflection;

[assembly: AssemblyTitle("My Awesome NuGet Package")]
[assembly: AssemblyCompany("Jess Chadwick")]
[assembly: AssemblyDescription("This is going to be the best NuGet Package in the repository!")]
```

## Really Customizing Your NuGet Package

If you want to really control what goes into your NuGet package or take advtange of some of NuGet's cool features, then you'll want to create your own NuSpec file.  NuGet Packer still makes this ridiculously easy:  just add your NuSpec file to the root of your project folder that matches the same name of your assembly and NuGet Packer will use this instead of generating one for you.

__1. Generate NuSpec file__

The easiest way to do this is to just into the command line, go to your project's folder (_project's_ folder, not _solution_ folder!) and then type:

  `nuget spec`

If that doesn't work, you can use the version of NuGet that NuGet Packer installed, in which case it'd be:

  `..\packages\NuGet.CommandLine.3.5.0\tools\nuget.exe spec`

This will generate the NuSpec file for you with the proper name in the proper location.

__2. Reference and Customize the NuSpec file__

After you've created the NuSpec file, jump back into Visual Studio and add it to your project.  

Then, feel free to customize it however you like following all of the [documentation from the NuGet docs site](http://docs.nuget.org/create), especially the [NuSpec reference](http://docs.nuget.org/Create/NuSpec-Reference).

## Publishing Your Package

NuGet Packer also supports publishing your package right from your build.  Remember, NuGet Packer is just a wrapper around `nuget.exe` so you're going to want to familiarize yourself with the [instructions on how to publish your package](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package#user-content-publishing-in-nuget-gallery) in the NuGet docs.  As you're reading just keep in mind that NuGet Packer is going to be taking care of everything any time it tells you to use `nuget push` from the command line.

### Setting your API Key
The most important thing that you'll need to do in order to publish your package is to set your API key.  You have two options:

1. Set the value in your local user configuration (as it shows you in the documentation)
2. Set the MSBuild property `NuGetApiKey`

I prefer option #2, especially since you should only be pushing NuGet packages from a build server and not your local machine!  See the next section for an example.

### Setting your target package source _(optional)_

By default, NuGet Packer will upload your package to the public NuGet repository ([nuget.org](http://nuget.org)).  If you'd like to publish to another repository, you'll need to set the MSBuild property `NuGetPublishSource`.

### Enable Publishing with NuGet Packer
Even though NuGet Packer generates a NuGet package by default, automatic publishing is not enabled by default.  In order to enable automatic publishing, you'll nee to set the MSBuild property `PublishNuGetPackage` to `true`.


## NuGet Packer Options

Here is the full list of options that NuGet Packer supports (see the [NuGet.Packer.props](NuGet.Packer.props) file):

| MSBuild Property   | Default       | Description                          |
|--------------------|:-------------:|--------------------------------------|
| BuildNuGetPackage  | true          | Build the .nupkg during compilation  |
| BuildSymbolsPackage| true          | Build the corresponding symbols package during compilation                                     |
| MajorVersion       | 1             | Package version (major)              |
| MinorVersion       | 0             | Package version (minor)              |
| NuGetApiKey        | true          | NuGet API Key (or set via `nuget.exe setApiKey`) |
| NuGetPublishSource | true          | The NuGet package repository that packages will be published to (defaults to nuget.org)                                     |
| PackageVersion     | [MajorVersion].[MinorVersion].[PatchVersion] | The full version number.  Override this property if you are managing version numbers outside of this system. |
| PatchVersion       | TFS Build Number or Current UTC time (yyMMddHH) | The patch (third part) of the version number |
| PreReleaseVersion  | N/A           | Specify a value (e.g. "alpha", "beta", "rc", etc.) to create a pre-release package.  If empty, a regular (non-pre-release) package will be created.  |
| PublishNuGetPackage| false         | Enables publishing a NuGet package   |


### Setting MSBuild Property Values
NuGet Packer is an MSBuild-based solution which means that all of these settings are MSBuild properties.
Generally speaking, you can set an MSBuild property in one of two ways:

1. Add it to the `.csproj` or `.vbproj` file:

```
<PropertyGroup>
 <PublishNuGetPackage>true</PublishNuGetPackage>
 <NuGetApiKey>12312312312</NuGetApiKey>
 <NuGetPublishSource>http://nuget.mycompany.org</NuGetPublishSource>
</PropertyGroup>
```

2. Pass it as a command line parameter:
`msbuild.exe MyPackage.sln /p:PublishNuGetPackage=true /p:NuGetPublishSource="http://nuget.mycompany.org" /p:NuGetApiKey=12312312312`

    **NOTE:** If building your solution with TFS's build system, 
    you can pass these parameters in the `MSBuild Arguments` property, like this:
    `/p:PublishNuGetPackage=true /p:NuGetPublishSource="http://nuget.mycompany.org" /p:NuGetApiKey=12312312312`
