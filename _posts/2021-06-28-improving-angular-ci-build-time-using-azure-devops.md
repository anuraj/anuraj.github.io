---
layout: post
title: "Improving Angular CI Build Time Using Azure DevOps Cache task"
subtitle: "This article will discuss about improving Angular CI build time using Azure DevOps Cache task."
date: 2021-06-28 00:00:00
categories: [AzureDevOps,Angular,DevOps]
tags: [AzureDevOps,Angular,DevOps]
author: "Anuraj"
image: /assets/images/2021/06/npm_install_step_skip.png
---
This article will discuss about improving Angular CI build time using Azure DevOps Cache task. If you're building Angular applications with Azure DevOps, you will be running `npm install` command every time even if you're not changed anything in the `package.json`. It is a time consuming process. Azure Devops offers a Cache task, which will help you to cache downloaded libraries or tools your build pipeline. In this example, I am using a minimal angular application, which is added to Azure Repo - you can use GitHub or any other source control supported by Azure DevOps. And then I created a build pipeline for the Angular application.

![Azure Angular Build Pipeline]({{ site.url }}/assets/images/2021/06/angular_azure_pipeline.png)

And lets run the build pipeline. You will be able to see - `npm install` is taking 23 seconds - for a minimal Angular application.

![Running the Build Pipeline]({{ site.url }}/assets/images/2021/06/angular_azure_build.png)

Let's add the cache step. You can click on the Add Step button, and search for Cache. In there Cache options you need to configure `Key` and `Path` values. The `Key` value should be set to the identifier for the cache you want to restore or save. 

![Add Cache Step]({{ site.url }}/assets/images/2021/06/add_cache_step.png)

In this example the key will be a hash of the package-lock.json file which lives in the root level - `$(System.DefaultWorkingDirectory)/package-lock.json`, and Path is the `node_modules` in the `$(System.DefaultWorkingDirectory)`. And to optimise the caching and restore, you need to configure `Cache hit variable` - this variable to set to 'true' when the cache is restored (i.e. a cache hit), otherwise set to 'false'. I am setting the `Cache hit variable` to `CacheRestored`.

![Add Cache Step]({{ site.url }}/assets/images/2021/06/add_cache_step.png)

And in the `npm install` step, modify the `Run this task` and select `Custom condition`. And in the `Custom condition` textbox set the value like this - `eq(variables['CacheRestored'],False)`.

![NPM Install Step]({{ site.url }}/assets/images/2021/06/npm_install_step.png)

Next let's the build again. Since it is first time running with Cache it will run the npm install step again. But next run the pipeline, it will be cached. It will be like this.

![NPM Install Step - Skipped]({{ site.url }}/assets/images/2021/06/npm_install_step_skip.png)

Sometimes you will get a warning like this - `##[warning]The given cache key has changed in its resolved value between restore and save steps;`. And because of this you might miss the cache. This is because the node version of Azure DevOps and your source code is different. You can fix this by adding one more node installer step - with the same version as your code.

![Node Install Step]({{ site.url }}/assets/images/2021/06/node_install_step.png)

This way you can cache the node modules in your build pipeline. If you apply caching into your real world project it can save lot of time and lot of API calls. You can find more details about the [Pipeline caching](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/caching?view=azure-devops&WT.mc_id=AZ-MVP-5002040){:target="_blank"} in official Microsoft documentation. And you will be able to see how to cache other tools as well.

Happy Programming :)