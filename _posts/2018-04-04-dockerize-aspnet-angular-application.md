---
layout: post
title: "Dockerize an ASP.NET MVC 5 Angular application with Docker for Windows"
subtitle: "This post is about how to setup a docker container for an ASP.NET MVC - Angular application."
date: 2018-04-04 00:00:00
categories: [ASP.NET,Docker,Angular4]
tags: [ASP.NET,Docker,Angular4]
author: "Anuraj"
---
Few days back I wrote a [post](https://dotnetthoughts.net/how-to-use-angular4-wth-aspnet-mvc/) about working with Angular 4 in ASP.NET MVC. I received multiple queries on deployment aspects - how to setup the development environment or how to deploy it in IIS, or in Azure etc. In this post I am explaining how to deploy a ASP.NET MVC - Angular application to Docker environment.

In the first section, I deploying the published version of the application to Docker. I already posted the source code for this already available in [GitHub](https://github.com/anuraj/MVCAngularCRUDExample). 

Firstly, you need to modify the project file to copy the output folder of Angular (In my scenario, I am using bundles folder) to the publish folder, so I added a MSBuild steps to do this.

{% highlight XML %}
<PropertyGroup>
  <CopyAllFilesToSingleFolderForPackageDependsOn>
    CustomCollectFiles;
    $(CopyAllFilesToSingleFolderForPackageDependsOn);
  </CopyAllFilesToSingleFolderForPackageDependsOn>
</PropertyGroup>
<Target Name="CustomCollectFiles">
  <ItemGroup>
    <_CustomFiles Include=".\Bundles\**\*" />
    <FilesForPackagingFromProject Include="%(_CustomFiles.Identity)">
      <DestinationRelativePath>Bundles\%(RecursiveDir)%(Filename)%(Extension)</DestinationRelativePath>
    </FilesForPackagingFromProject>
  </ItemGroup>
</Target>
{% endhighlight %}

In this build step, I am copying all the files from `bundles` folder to the publish folder. You may also need one more build step to compile the angular project, you can add one more build step to do this.

{% highlight XML %}
<Target Name="NgDebug" BeforeTargets="Build" Condition="'$(Configuration)' == 'Debug'">
  <Exec WorkingDirectory="$(ProjectDir)ClientApp" Command="ng build -ec" />
</Target>
<Target Name="NgRelease" BeforeTargets="Build" Condition="'$(Configuration)' == 'Release'">
  <Exec WorkingDirectory="$(ProjectDir)ClientApp" Command="ng build --prod" />
</Target>
{% endhighlight %}

Next you need to create the Docker file to deploy the output folder to `C:\Inetpub\wwwroot` in the container. You can do it like this.

{% highlight YML %}
FROM microsoft/aspnet
COPY . /inetpub/wwwroot
{% endhighlight %}

For database, now I am using SQL Server in my host machine. And to build SQL Database, I am using `Database.SetInitializer()` method in DbContext class. Here is the code for DbContext class.

{% highlight CSharp %}
public class WebAppContext : DbContext
{
    public WebAppContext() : base("name=WebAppContext")
    {
        Database.SetInitializer(new DropCreateDatabaseIfModelChanges<WebAppContext>());
    }

    public DbSet<Book> Books { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
    }
}
{% endhighlight %}

Next you need to build the container and run it. Once it is running, you can do a `docker inspect` command and identify the IP Address of the container. 

![Docker Build]({{ site.url }}/assets/images/2018/04/docker_build.png)

Here is the app running in my machine.

![Running ASP.NET MVC Angular App in Docker]({{ site.url }}/assets/images/2018/04/angular_app_running.png)

Next, I am building a `Dockerfile`, in which we are building the project - both ASP.NET MVC and Angular. 

{% highlight YML %}
FROM microsoft/aspnet

SHELL ["powershell"]

ADD https://aka.ms/vs/15/release/vs_buildtools.exe C:\\Downloads\\vs_buildtools.exe
ADD https://dist.nuget.org/win-x86-commandline/v4.3.0/nuget.exe C:\\Nuget\\nuget.exe
ADD https://nodejs.org/dist/v8.10.0/node-v8.10.0-win-x64.zip C:\\Downloads\\node-v8.10.0-win-x64.zip

RUN "Expand-Archive" "-LiteralPath C:\\Downloads\\node-v8.10.0-win-x64.zip" "-DestinationPath C:\\NodeJs\\"

RUN C:\\Downloads\\vs_buildtools.exe --add Microsoft.VisualStudio.Workload.MSBuildTools --add Microsoft.VisualStudio.Workload.WebBuildTools --quiet --wait

RUN "[Environment]::SetEnvironmentVariable('Path', $env:Path + ';C:\NodeJs\node-v8.10.0-win-x64;C:\Nuget;C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\MSBuild\15.0\Bin', [EnvironmentVariableTarget]::Machine)"

COPY WebApp C:/Source/WebApp/
COPY WebApp.sln C:/Source/

WORKDIR C:/Source/WebApp/ClientApp

RUN node C:\Source\WebApp\ClientApp\node_modules\@angular\cli\bin\ng build --prod

WORKDIR C:/Source

RUN Nuget.exe restore WebApp.sln

RUN MSBuild.exe WebApp.sln /p:Configuration=Release /p:PublishProfile=FolderProfile /p:DeployOnBuild=true

{% endhighlight %}

In this `Dockerfile` first I am downloading MSBuild, nuget and Node JS. Once downloaded, I am extracting nodejs, to C:\NodeJs folder. Then I am installing MS Build with MSBuildTools and WebBuildTools. Once both installations is done, I am setting the tools to the PATH. 

You can run `npm install` in the `dockerfile`, but it will take sometime. So I ran `npm install` in my developer machine and I am copying the files to `C:\Source` folder in the container. Also I removed the `ng build` step from the project file, and I am running it outside. After building the Angular project, I am restoring the nuget packages with nuget.exe. And finally, I am building and deploying the solution with MSBuild.exe. For deploying I am using Visual Studio profile file. It is a simple pubxml file with FileSystem and deploying the output to `C:\Inetpub\wwwroot` directory. Here is the FolderProfile.pubxml file.

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<!--
This file is used by the publish/package process of your Web project. You can customize the behavior of this process
by editing this MSBuild file. In order to learn more about this please visit https://go.microsoft.com/fwlink/?LinkID=208121. 
-->
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <WebPublishMethod>FileSystem</WebPublishMethod>
    <PublishProvider>FileSystem</PublishProvider>
    <LastUsedBuildConfiguration>Release</LastUsedBuildConfiguration>
    <LastUsedPlatform>Any CPU</LastUsedPlatform>
    <SiteUrlToLaunchAfterPublish />
    <LaunchSiteAfterPublish>True</LaunchSiteAfterPublish>
    <ExcludeApp_Data>False</ExcludeApp_Data>
    <publishUrl>C:\Inetpub\wwwroot</publishUrl>
    <DeleteExistingFiles>False</DeleteExistingFiles>
  </PropertyGroup>
</Project>
{% endhighlight %}

Please let me know if you're facing any issues in running the application in Docker.

Happy Programming :)