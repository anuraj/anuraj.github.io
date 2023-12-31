---
layout: post
title: "k web command and agile development environment"
subtitle: "k web command and agile development environment"
date: 2015-02-19 12:10
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
With the new version of ASP.NET, you can avail the flexibility of deploying your app via an agile app development methodology, where you can modify and save the C# file and refresh the browser, your changes will be updated automatically. You don't need to go for any explicit compilation in Visual Studio. The in-built .NET Compiler Platform ROSLYN available with ASP.NET vNext handles app deployment transparently in either the server or the development machine. This feature is only available with Visual Studio, it won't work with "k" command. The k command comes with --watch switch, documentation says "Watch file changes", but it is also not working. When there is a file change, it will stop the server. I found few Node plugins for this purpose. Today I thought of writing a C# implementation. 

Here is the implementation, I have a FileSystem watcher, which will monitor the files inside the directory, will raise events on file changes, if there is a change, it will restart the server. For some files, like project.json, if there is change, it will restore the packages and restart the server.

Implementation - File System watcher - Listening on file system change events

{% highlight CSharp %}
static void Main(string[] args)
{
    var currentDirectory = AppDomain.CurrentDomain.BaseDirectory;
    var fileSystemWatcher = new FileSystemWatcher(currentDirectory);
    fileSystemWatcher.IncludeSubdirectories = true;
    fileSystemWatcher.NotifyFilter = NotifyFilters.LastAccess
        | NotifyFilters.LastWrite
        | NotifyFilters.FileName
        | NotifyFilters.DirectoryName;
    fileSystemWatcher.Changed += new FileSystemEventHandler(OnChanged);
    fileSystemWatcher.Created += new FileSystemEventHandler(OnChanged);
    fileSystemWatcher.Deleted += new FileSystemEventHandler(OnChanged);
    fileSystemWatcher.EnableRaisingEvents = true;

    Console.WriteLine("Starting server");
    StartProcess("k --watch web");
    Console.WriteLine("Press \'q\' to quit the web monitor.");
    while (Console.Read() != 'q') ;
}
{% endhighlight %}

Here is the implementation - Starting / Restarting server and restoring packages.

{% highlight CSharp %}
private static Process StartProcess(string arguments)
{
    var filename = "cmd.exe";
    var processStartInfo = new ProcessStartInfo(filename, " /C " + arguments);
    processStartInfo.RedirectStandardError = true;
    processStartInfo.RedirectStandardInput = true;
    processStartInfo.RedirectStandardOutput = true;
    processStartInfo.CreateNoWindow = true;
    processStartInfo.WindowStyle = ProcessWindowStyle.Hidden;
    processStartInfo.UseShellExecute = false;
    var process = Process.Start(processStartInfo);

    process.EnableRaisingEvents = true;
    process.OutputDataReceived += (o, dre) =>
    {
        Console.WriteLine(dre.Data);
    };

    process.BeginOutputReadLine();
    return process;
}
{% endhighlight %}

And here is the core part - OnChanged, which will listen the file changes and based on the file type do the process.

{% highlight CSharp %}
private static void OnChanged(object source, FileSystemEventArgs e)
{
    DateTime lastWriteTime = File.GetLastWriteTime(e.FullPath);
    if (lastWriteTime != _lastRead)
    {
        var extension = Path.GetExtension(e.Name);
        if (extension.Equals(".json"))
        {
            Console.WriteLine("{0} file modified. Package restore starting", e.Name);
            var process = StartProcess("kpm restore");
            process.WaitForExit();
            if (process.ExitCode == 0)
            {
                Console.WriteLine("Package restore completed successfully. Restarting server.");
                process = StartProcess("k --watch web");
                process.WaitForExit();
            }
            else
            {
                Console.WriteLine("Package restore failed. Please make sure your {0} file is valid.", e.Name);
            }
        }
        else
        {
            Console.WriteLine("{0} file modified. Restarting server", e.Name);
            var process = StartProcess("k --watch web");
            process.WaitForExit();
            if (process.ExitCode != 0)
            {
                Console.WriteLine("Server restart failed. Please make sure you can compile {0} file.", e.Name);
            }
        }

        _lastRead = lastWriteTime;
    }
}
{% endhighlight %}

There is a problem with FileSystemWatcher Changed event, when ever there is change, FileSystem watcher will raise the OnChanged event twice. To overcome this issue, the _lastRead and lastWriteTime variables used. Here is the screenshot of kmonitor.exe running on my system.

![kmonitor - running]({{ site.url }}/assets/images/2015/02/kmonitor.png)

Here is the [gist for kmonitor](https://gist.github.com/anuraj/bd39d8c6f46a0af4380b)

Happy Programming :)
