---
layout: post
title: "GitHub Copilot CLI - Bring Your Own Key (BYOK)"
subtitle: "In this blog post, we will explore how to use your own LLMs instead of GitHub hosted models."
date: 2026-05-24 00:00:00
categories: [AI,GitHub,Copilot]
tags: [AI,GitHub,Copilot]
author: "Anuraj"
image: /assets/images/2026/05/copilot_with_deepseek_pro.png
---

In this blog post, we will explore how to use your own LLMs instead of GitHub hosted models. This is called BYOK - Bring Your Own Key. BYOK helps us to 

* **Keep your data private:** Your prompts and code stay within your own systems.
* **Choose what works best:** Use the AI model that fits your team’s needs and way of working.
* **Save on costs:** If you already have systems running for AI workloads, adding a Copilot endpoint can cost very little extra.

In this blog I will be using Azure Models. To use Azure, we need to set the following environment variables.

```
export COPILOT_PROVIDER_BASE_URL=https://<YOUR_PROJECT_NAME>.services.ai.azure.com/openai/v1
export COPILOT_PROVIDER_TYPE=azure
export COPILOT_PROVIDER_API_KEY=YOUR_AZURE_FOUNDRY_API_KEY
export COPILOT_MODEL=DeepSeek-V4-Pro
```

Now we can run the Copilot CLI using `copilot` command - which will use the `DeepSeek-V4-Pro` model.

![Copilot is running with DeepSeek model]({{ site.url }}/assets/images/2026/05/copilot_with_deepseek_pro.png)

And if we want to use Ollama models or any other local LLMs, we need to set the following environment variables.

```
export COPILOT_PROVIDER_BASE_URL=https://<your-endpoint>/v1
export COPILOT_MODEL=<your-model-name>
```

This way we can use Azure Foundry models in GitHub Copilot CLI. We can use local models when we are working with Privacy-sensitive code or Offline workflows or learning & experimentation with LLM behavior. When it is not recommended for High-accuracy, large-context, or production-critical tasks - cloud models remain superior in speed and reliability.

Happy Programming