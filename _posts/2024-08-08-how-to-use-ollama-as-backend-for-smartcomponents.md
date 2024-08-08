---
layout: post
title: "How to use Ollama as backend for Smart Components"
subtitle: "In this blog post, we'll learn how to use Ollama as backend for Smart Components"
date: 2024-08-08 00:00:00
categories: [Docker,AI,dotnet]
tags: [Docker,AI,dotnet]
author: "Anuraj"
image: /assets/images/2024/08/downloading_the_model.png
---

Few weeks back I wrote a blog post on Smart Components - a few components which helps .NET developers to enable infuse AI into their applications with zero or less code. By default we need an Open AI endpoint and API Key. But we can do it with Ollama. To use Ollama, make sure you're running Ollama - I am using Ollama in Docker with the command - `docker run -d -v D:\OllamaModels:/root/.ollama -p 11434:11434 --name ollama ollama/ollama`. Next we need to run one model, I am using `mistral`. To download and run this, we first need to open shell in the running container with the command - `docker exec -it ollama bash` - which will open bash shell in the ollama container. Next we can run the command `ollama pull mistral:7b` - this command will download `mistral:7b` model to the local machine. This may take some time. Here is the screenshot.

![Download the mistral:7b model]({{ site.url }}/assets/images/2024/08/downloading_the_model.png)

Next in the application configuration instead of Open AI configuration, use the following configuration.

{% highlight JSON %}
{% raw %}

"SmartComponents": {
  "SelfHosted": true,
  "DeploymentName": "mistral:7b",
  "Endpoint": "http://localhost:11434/"
}

{% endraw %}
{% endhighlight %}

There is no code change require other than this. It will work properly.

Here are few recommendations on self hosted the models - Quality and speed varies dramatically from one model to another. For best results, use GPT 3.5 Turbo or better - this will respond quickly and at good quality.

Based on experimentation:

* GPT 3.5 Turbo (hosted service) produces good quality results and is fast.
* Mistral 7B produces good output for Smart TextArea, but is inconsistent for Smart Paste (filling some forms, but leaving others blank or introducing strange characters). Output speed is good for a local model, but on most workstations is still way slower than a hosted service.
* Llama2 7B output quality is insufficient (with Smart Paste, it puts strange or hallucinated text into form fields, and with Smart TextArea, it writes generic text that doesn't well account for the configured phrases).
* Mixtral 47B produced good output for Smart TextArea, but wouldn't follow the instructions properly for Smart Paste and hence left forms blank. Additionally it's too big to run on most workstation GPUs so can be impractically slow.

From [GitHub](https://github.com/dotnet-smartcomponents/smartcomponents/blob/main/docs/configure-openai-backend.md)

This way we can use Ollama as the backend for Smart Components.

Happy Programming