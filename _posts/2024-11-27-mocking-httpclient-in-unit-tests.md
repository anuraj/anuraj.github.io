---
layout: post
title: "Mocking HttpClient in unit tests"
subtitle: "In this blog post, we'll learn mock HttpClient in unit tests"
date: 2024-11-27 00:00:00
categories: [dotnet,unittest]
tags: [dotnet,unittest]
author: "Anuraj"
---

In this blog post, we'll learn mock HttpClient in unit tests. Recently I started working on a C# wrapper on BlueSky API. To write the unit tests for the library, I had to mock HttpClient. 

Since the HttpClient does not inherit from any interface so we will have to write our own. We can implement a custom `IHttpHandler` interface like this.

```
public interface IHttpHandler
{
    Task<HttpResponseMessage> PostAsJsonAsync<TValue>(string? requestUri, TValue value, CancellationToken cancellationToken);
    //Ignoring the other method implementations.
}
```

And we can implement it like this.

```
public class HttpHandler : IHttpHandler
{
    private readonly HttpClient _httpClient = new();
    public async Task<HttpResponseMessage> PostAsJsonAsync<TValue>(string? requestUri, 
        TValue value, CancellationToken cancellationToken)
    {
        return await _httpClient.PostAsJsonAsync(requestUri, value, cancellationToken);
    }
}
```

And we can use this in the library - instead of using HttpClient directly.

In my case, I am using Moq so I used different approach - where I am able to Mock the httpClient, here is an example.

```
var mockHandler = new Mock<HttpMessageHandler>(MockBehavior.Strict);

mockHandler.Protected().Setup<Task<HttpResponseMessage>>("SendAsync", ItExpr.IsAny<HttpRequestMessage>(), ItExpr.IsAny<CancellationToken>())
.ReturnsAsync(new HttpResponseMessage()
{
    StatusCode = HttpStatusCode.OK,
    Content = new StringContent(JsonSerializer.Serialize(createSessionResponse), Encoding.UTF8, "application/json")
});

var httpClient = new HttpClient(mockHandler.Object);
var client = new BlueSkyClient(httpClient);
await client.CreateSession("Handle", "Password");

mockHandler.Protected().Verify("SendAsync",
    Times.Exactly(1),
    ItExpr.Is<HttpRequestMessage>(m => m.Method == HttpMethod.Post),
    ItExpr.IsAny<CancellationToken>());
```

This way we can mock httpClient in unit tests.

Happy Programming