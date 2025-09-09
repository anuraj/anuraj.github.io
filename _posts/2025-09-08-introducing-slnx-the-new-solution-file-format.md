---
layout: post
title: "Introducing SLNX - The New Solution File Format for .NET"
subtitle: "In this blog post, we'll explore the new solution file format SLNX and the advantages of using it."
date: 2025-09-08 00:00:00
categories: [dotnet,csharp]
tags: [dotnet,csharp]
author: "Anuraj"
---

With .NET 9 Microsoft introduced `slnx` file format - which is a replacement for the existing `sln` file. In this blogpost we will explore why we want to use it and the advantages of using it.

Here is an example of an existing .NET solution file with 3 projects, MVC frontend, API backend and common class library.

```
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio Version 17
VisualStudioVersion = 17.0.31903.59
MinimumVisualStudioVersion = 10.0.40219.1
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "api", "api\api.csproj", "{5BBA9EC2-DCD7-4AD2-813E-FD343A0461DA}"
EndProject
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "web", "web\web.csproj", "{400570D1-28BD-41DC-BCED-E0A567822E04}"
EndProject
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "shared", "shared\shared.csproj", "{3EC0A808-7C5C-45C1-A895-D300FF41F571}"
EndProject
Global
	GlobalSection(SolutionConfigurationPlatforms) = preSolution
		Debug|Any CPU = Debug|Any CPU
		Release|Any CPU = Release|Any CPU
	EndGlobalSection
	GlobalSection(SolutionProperties) = preSolution
		HideSolutionNode = FALSE
	EndGlobalSection
	GlobalSection(ProjectConfigurationPlatforms) = postSolution
		{5BBA9EC2-DCD7-4AD2-813E-FD343A0461DA}.Debug|Any CPU.ActiveCfg = Debug|Any CPU
		{5BBA9EC2-DCD7-4AD2-813E-FD343A0461DA}.Debug|Any CPU.Build.0 = Debug|Any CPU
		{5BBA9EC2-DCD7-4AD2-813E-FD343A0461DA}.Release|Any CPU.ActiveCfg = Release|Any CPU
		{5BBA9EC2-DCD7-4AD2-813E-FD343A0461DA}.Release|Any CPU.Build.0 = Release|Any CPU
		{400570D1-28BD-41DC-BCED-E0A567822E04}.Debug|Any CPU.ActiveCfg = Debug|Any CPU
		{400570D1-28BD-41DC-BCED-E0A567822E04}.Debug|Any CPU.Build.0 = Debug|Any CPU
		{400570D1-28BD-41DC-BCED-E0A567822E04}.Release|Any CPU.ActiveCfg = Release|Any CPU
		{400570D1-28BD-41DC-BCED-E0A567822E04}.Release|Any CPU.Build.0 = Release|Any CPU
		{3EC0A808-7C5C-45C1-A895-D300FF41F571}.Debug|Any CPU.ActiveCfg = Debug|Any CPU
		{3EC0A808-7C5C-45C1-A895-D300FF41F571}.Debug|Any CPU.Build.0 = Debug|Any CPU
		{3EC0A808-7C5C-45C1-A895-D300FF41F571}.Release|Any CPU.ActiveCfg = Release|Any CPU
		{3EC0A808-7C5C-45C1-A895-D300FF41F571}.Release|Any CPU.Build.0 = Release|Any CPU
	EndGlobalSection
EndGlobal
```

For the same project, the new slnx file format will look something like this.

```xml
<Solution>
  <Project Path="api/api.csproj" />
  <Project Path="web/web.csproj" />
  <Project Path="shared/shared.csproj" />
</Solution>
```

We can migrate our existing solution file to the new format using `dotnet sln migrate` command - this will create a new solution file with `.slnx` extenstion in same directory. We can review the slnx file and then delete the solution file. When running commands like `dotnet build` if both solution files exist, we need to specify which solution file we want to build. Otherwise it will show an error - asking to specify the solution or project file to build.

To build the slnx file, we can run the command like this `dotnet build helloworld.slnx` - this applies to other commands like adding projects to the solution as well. We need to like `dotnet sln helloworld.slnx /Frontend`. To list the projects, we can run `dotnet sln helloworld.slnx list`

C# DevKit can support SLNX files, but in order to do so we must set the `dotnet.defaultSolution` property inside `.vscode\settings.json` to the path to your slnx file like this.

```json
{
  "dotnet.defaultSolution": "helloworld.slnx"
}
```

Here are few advantages of using the new `slnx` format.

1. Performance improvement: Faster loading times for large solutions.
2. Simplified Editing: Since the format is XML-based, it is easy for us to edit with other tools.
3. Improved Support for Large Projects: Reduces conflicts, improves dependency management and reviewing.

And in .NET 10, dotnet new sln generates an SLNX-format solution file instead of an SLN-formatted solution file - [dotnet new sln defaults to SLNX file format](https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/10.0/dotnet-new-sln-slnx-default)

The slnx files are an exciting new change to the solution file format that I think will make it easier for teams to collaborate and understand their projects. 

Happy Programming