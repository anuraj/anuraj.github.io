---
layout: post
title: "Running DeepSeek-R1 locally for free"
subtitle: "In this blog post, we'll learn how to run DeepSeek-R1 locally for free"
date: 2025-01-28 00:00:00
categories: [dotnet,AI,DeepSeek]
tags: [dotnet,AI,DeepSeek]
author: "Anuraj"
image: /assets/images/2025/01/deepseek_r1_page.png
---

In this blog post, we'll learn how to run DeepSeek-R1 locally for free. DeepSeek R1 is a new large language model. DeepSeek R1 is explicitly designed as a "reasoning model". It's remarkable to say that reasoning model is going to drive key advancement in large model progress in 2025.

To run DeepSeek-R1 locally, first we need to install Ollama. We can download it from [https://ollama.com](https://ollama.com) 

![Ollama Homepage]({{ site.url }}/assets/images/2025/01/ollama_homepage.png)

Once the installation file downloaded, execute it, it will show an installation dialog like this.

![Ollama Installation]({{ site.url }}/assets/images/2025/01/ollama_install.png)

It is simple and  straight forward installation. Once the installation is completed, we can run the ollama app by running `ollama` command in terminal.

![Ollama Running]({{ site.url }}/assets/images/2025/01/ollama_running.png)

Now we are ready to install the DeepSeek-R1. To install and configure, first check out [Ollama models](https://ollama.com/search) page, where we can find various models which we can use locally. In the page search for DeepSeek-R1, and once we decided which tag we want to use. I am using the 7b version.

![DeepSeek Page]({{ site.url }}/assets/images/2025/01/deepseek_r1_page.png)

And then we can run the command `ollama run deepseek-r1:7b` - which will download the model and show a chat interface like this.

![DeepSeek Page]({{ site.url }}/assets/images/2025/01/deepseek_r1_chat.png)

We can ask questions and interact with the LLM from this interface. Now we can run the DeepSeek model without worrying about the privacy and data sharing concerns. We can use the Ollama connector in Semantic Kernel and we can use DeepSeek in our C# and .NET applications.

Happy Programming