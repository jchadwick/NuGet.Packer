# NuGet Packer
NuGet Packer helps you turn your .NET (.csproj/.vbproj) projects into NuGet packages as simply and effectively as possible.

## Basic Usage
Using NuGet Packer is as simple as installing a NuGet package and building your project:

1. Install the NuGet package `NuGet.Packer`
  * Using [the NuGet package management GUI](http://docs.nuget.org/consume/package-manager-dialog), or
  * Typing the following command in the Package Management Console:
  
    `Install-Package NuGet.Packer`

2. Compile your project using the 'Release' configuration
  * [Choose "Release" from the Visual Studio configuration drop-down](https://msdn.microsoft.com/en-us/library/wx0123s5.aspx), or
  * Specify the `/p:Configuration=Release` flag when calling MSBuild from the command line

That's it!  Your NuGet package will be in your `/bin` folder, right next to your project's output assembly (.dll).  It will be named `[Assembly Name].[Version].nupkg`

## Customizing Your NuGet Package
NuGet Packer is just a thin wrapper around the NuGet executable, so all of the [standard documentation around creating NuGet packages](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package] still applies, especially the part about [defining your project's metadata](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package#user-content-from-a-project) so that NuGet can use it in the package it generates.

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

  `..\packages\NuGet.CommandLine.3.3.0\tools\nuget.exe spec`

This will generate the NuSpec file for you with the proper name in the proper location.

__2. Reference and Customize the NuSpec file__
After you've created the NuSpec file, jump back into Visual Studio and add it to your project.  

Then, feel free to customize it however you like following all of the [documentation from the NuGet docs site](http://docs.nuget.org/create), especially the [NuSpec reference](http://docs.nuget.org/Create/NuSpec-Reference).

