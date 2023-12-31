---
layout: post
title: "Add Unit Tests To Your Azure Functions"
subtitle: "This post is about how to add unit tests your azure functions."
date: 2022-10-05 00:00:00
categories: [Azure,Functions,Serverless,UnitTesting]
tags: [Azure,Functions,Serverless,UnitTesting]
author: "Anuraj"
image: /assets/images/2022/10/test_explorer.png
---

This post is about how to add unit tests your azure functions. Like ASP.NET Core apps and Web APIs, we can add unit tests for Azure Functions as well. I am using Visual Studio and C#. I am creating the function in .NET as well. First I am creating an Azure Function using Visual Studio. I am modifying the existing function a little and the updated function looks like this.

{% highlight csharp %}
{% raw %}
public static class EchoFunction
{
    [FunctionName("EchoFunction")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        string message = req.Query["message"];

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        message ??= data?.message;

        return string.IsNullOrEmpty(message) ? new BadRequestResult() : new OkObjectResult(message);
    }
}
{% endraw %}
{% endhighlight %}

In the function I am expecting a query string or element with key `message` and if it exists function returns it, if not, function return a `BadRequest` result. Next I am adding a XUnit Project to the solution. Then I am adding the a nuget package reference - `Microsoft.AspNetCore.Mvc` this is required for the unit test to interact with the azure function. And then add reference of the Azure Function as well to the unit test project. Now the unit test project looks like this.

{% highlight xml %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk">
	<PropertyGroup>
		<TargetFramework>net6.0</TargetFramework>
		<ImplicitUsings>enable</ImplicitUsings>
		<Nullable>enable</Nullable>
		<IsPackable>false</IsPackable>
	</PropertyGroup>
	<ItemGroup>
		<PackageReference Include="Microsoft.AspNetCore.Mvc" Version="2.2.0" />
		<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.1.0" />
		<PackageReference Include="xunit" Version="2.4.1" />
		<PackageReference Include="xunit.runner.visualstudio" Version="2.4.3">
			<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
			<PrivateAssets>all</PrivateAssets>
		</PackageReference>
		<PackageReference Include="coverlet.collector" Version="3.1.2">
			<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
			<PrivateAssets>all</PrivateAssets>
		</PackageReference>
	</ItemGroup>
	<ItemGroup>
	  <ProjectReference Include="..\HelloWorld\HelloWorld.csproj" />
	</ItemGroup>
</Project>
{% endraw %}
{% endhighlight %}

Now let us write the unit tests. I am writing two unit tests for this function, one with input parameter and one without input parameter. Here are the unit tests. 

> Update : Based on the inputs from [Sean Killeen](https://disqus.com/by/seankilleen/){:target="_blank"}, changed the unit test methods to async.

{% highlight CSharp %}
{% raw %}
[Fact]
public async Task TestFunctionSuccess()
{
    var queryStringValue = "abc";
    var request = new DefaultHttpRequest(new DefaultHttpContext())
    {
        Query = new QueryCollection
        (
            new Dictionary<string, StringValues>()
            {
        { "message", queryStringValue }
            }
        )
    };

    var logger = NullLoggerFactory.Instance.CreateLogger("Null Logger");
    var response = await EchoFunction.Run(request, logger);

    Assert.IsAssignableFrom<OkObjectResult>(response);
    var result = (OkObjectResult)response;

    Assert.Equal(queryStringValue, result.Value);
}

[Fact]
public async Task TestFunctionFailure()
{
    var request = new DefaultHttpRequest(new DefaultHttpContext());

    var logger = NullLoggerFactory.Instance.CreateLogger("Null Logger");

    var response = await EchoFunction.Run(request, logger);

    Assert.IsAssignableFrom<BadRequestResult>(response);
}
{% endraw %}
{% endhighlight %}

In the first test, I am creating `HttpRequest` with a query parameter collection. Then I am creating a logger instance - this is required since the Azure function is taking the logger as a parameter. Next I am invoking the `Run` static method of the function with the parameters. Since it is a async function, I am waiting for the execution to complete and then asserting whether it is `OkayObjectResult` class or not. Then validates the input and output as well. In the second one, I am not passing any parameter and verify the result. Now we can open the test explorer and execute the tests and view the results.

Here is the screenshot.

![Test Explorer]({{ site.url }}/assets/images/2022/10/test_explorer.png)

We can use the same technique for non static functions as well, where you configured `Startup` class and injecting dependencies through constructor. I wrote a blog post on this long back - [How to add a Startup class to Azure Functions](https://dotnetthoughts.net/azure-functions-startup-class/)

Happy Programming.