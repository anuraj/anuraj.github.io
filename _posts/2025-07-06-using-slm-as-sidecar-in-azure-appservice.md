---
layout: post
title: "Leveraging Small Language Model as a Sidecar for Linux App Service"
subtitle: "In this blog post, we'll learn how to use a small language model aka SLM as a sidecar for Linux App Service"
date: 2025-07-06 00:00:00
categories: [dotnet,azure,ai]
tags: [dotnet,azure,ai]
author: "Anuraj"
image: /assets/images/2025/07/chat_with_slm.png
---

In this blog post, we'll learn how to use a small language model aka SLM as a sidecar for Linux App Service. A small language model (SLM) is a Gen AI technology that functions like a large language model (LLM), but with a much smaller footprint. Small language models (SLMs) are increasingly fine-tuned on domain-specific datasets, enabling them to excel in targeted applications like specialized chat bots, document summarization, and industry-specific information retrieval. Their compact size not only boosts efficiency in these focused tasks but also makes them ideal for deployment on devices with limited computational resources—such as mobile phones, IoT devices, or edge computing environments—bringing the benefits of generative AI to more accessible and resource-constrained settings.

In this example, I am using a very small LLM - `smollm2:135m` - We can use any other LLMs like Phi family or anything. And for interacting with language model, I am using `OllamSharp` and `Microsoft.Extensions.AI` nuget package.

First I will be creating a Docker image for the SLM. Here is the Dockerfile and shell script which I am using to build the SLM. Here is the `Dockerfile` code

```yaml
FROM alpine/ollama:latest

ENV OLLAMA_HOST=0.0.0.0
ENV OLLAMA_PORT=11434
ENV OLLAMA_KEEP_ALIVE=24h

EXPOSE 11434

COPY start.sh /start.sh
RUN chmod +x /start.sh

ENTRYPOINT ["/start.sh"]
```

In the Docker file, I am using Ollam based on Alpine which reduces the size - my size of the image is around 300 MB only. Here is the `start.sh` file, which pulls the image.

```shell
#!/bin/sh
ollama serve &
sleep 10
ollama pull smollm2:135m
wait
```

Next we need to push the image to Docker Registry. For demo purposes I am using Docker Hub. We can use the `docker build` command to create the image, and `docker push` command to publish the image to Docker hub. Next we can create the .NET application which interact with the SLM. In the app, I am using two nuget packages, as mentioned earlier. Here is the project file.

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.AI" Version="9.6.0" />
  <PackageReference Include="OllamaSharp" Version="5.2.3" />
</ItemGroup>
```
Next in the `Program.cs` file, I will be adding the following code.

```csharp
var innerChatClient = new OllamaApiClient("http://localhost:11434/", "smollm2:135m");
builder.Services.AddChatClient(innerChatClient);
```

And in the controller we can use `IChatClient` interface, like this.

```csharp
[HttpPost]
public async Task<IActionResult> SendMessage([FromServices] IChatClient chatClient,
    [FromBody] ChatMessage request)
{
    try
    {
        if (request == null || string.IsNullOrWhiteSpace(request.Message))
        {
            return BadRequest(new { role = "system", message = "Message cannot be empty." });
        }

        var response = await chatClient.GetResponseAsync(request.Message);
        return Json(new { role = "system", message = response.ToString() });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error processing chat message: {Message}", request?.Message);
        return Json(new
        {
            role = "system",
            message = "Sorry, I encountered an error processing your message. Please try again."
        });
    }
}
```

And the `ChatMessage` class looks like this.

```csharp
public class ChatMessage
{
    public string Role { get; set; } = string.Empty;
    public string Message { get; set; } = string.Empty;
}
```

Now I need to build the Docker image for the application as well. I am using `dotnet publish` command to create Docker image. And publishing the image to Docker hub again.

Next I will be creating an Azure app service. Here is the screenshot of the first screen.

![Create Azure App Service]({{ site.url }}/assets/images/2025/07/create_web_app.png)

We need to select the `Container` in Publish configuration, then choose `Linux` as Operating System. Next, select the `Container` option. In the screen, select the `Sidecar support` option. Next for the Image source, select `Other container registries`. Next under `Docker hub options`, select the `Access Type` - it can be public or private. For this demo I am using `Public`. For the `Registry server URL`, we can keep the URL as it is `https://index.docker.io`. And for `Image and tag`, use the web app docker image and tag. As we are using .NET 9, we need to set the port as 8080.

![Container Options]({{ site.url }}/assets/images/2025/07/container_option.png)

we can keep other settings as default, and click on `Review + Create` button to create the App service and deploy the application. Once the application is running, select the `Deployment Center` option of the Azure App Service. Then click on the `Add +`, and select `Custom Container`.

![Add Container]({{ site.url }}/assets/images/2025/07/add_custom_container.png)

In the screen, similar to web app container, we need to configure various settings - select `Image source` as `Other container registries`, next `Image type` as `Public`. For the `Registry server URL`, set the URL as `https://index.docker.io`. Set the `Image and Tag` and port - for Ollama it is 11434. Now we can run the application and try interacting with that.

![Chat Application]({{ site.url }}/assets/images/2025/07/chat_with_slm.png)

In this blog post, we demonstrated how to deploy smollm2, a Small Language Model (SLM), as a sidecar on Linux App Service to seamlessly infuse AI capabilities into your web applications. We highlighted the advantages of SLMs—including their lightweight footprint, enhanced accessibility, and improved security—making them ideal for domain-specific customization and environmentally conscious development. To bring these concepts to life, we also walked through the setup of a simple .NET chat application that interacts with the SLM, offering a hands-on example of how to integrate SLMs into real-world projects.

Happy Programming