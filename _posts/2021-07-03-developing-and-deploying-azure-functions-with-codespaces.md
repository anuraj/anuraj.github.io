---
layout: post
title: "Developing and Deploying Azure Functions with GitHub Codespaces"
subtitle: "This article will discuss about developing and deploying Azure Functions with GitHub Codespaces. Codespaces sets up a cloud-hosted, containerized, and customizable VS Code environment."
date: 2021-07-03 00:00:00
categories: [Azure,Serverless,GitHub,Codespaces]
tags: [Azure,Serverless,GitHub,Codespaces]
author: "Anuraj"
image: /assets/images/2021/07/debug_azure_function.png
---
This article will discuss about developing and deploying Azure Functions with GitHub Codespaces. Codespaces sets up a cloud-hosted, containerized, and customizable VS Code environment.

## Developing an Azure Function with Codespaces

To get started first you need to create a github repo with one file in it - you can select the `Add a README file` option.

![New Repository with Readme]({{ site.url }}/assets/images/2021/07/azure_function_demo_empty.png)

Once it got created, you can click on the Code &gt; Open with Codespaces. 

![Open with Codespaces]({{ site.url }}/assets/images/2021/07/github_open_with_codespaces.png)

This will show a dialog where you need to create a `New codespace`. This will take some time. And once it is done, the browser window will show a VS Code style editor.

![VS Code on Browser]({{ site.url }}/assets/images/2021/07/vscode_in_browser.png)

If you check the extensions tab, you will be able to find few extensions pre installed on VS Code including one to work with Azure Functions.

![Azure Function extension]({{ site.url }}/assets/images/2021/07/azure_function_extension.png)

Since the extension already installed, you can create an azure function similar to the way you're building with VS Code. You can click on `F1` or `Ctrl+Shift+P` - which will popup VS Code command pallette. Select the `Azure Functions: Create New Project...` option. In the next screen use the same location. Next you need to choose the language. For the demo I am choosing Javascript - I am building a NodeJs function. And in the next, you need to choose the `HTTP trigger`, and provide a name - I am using the default one. And authentication mode as anonymous. Once you choose the authentication level the function app will be created.

![Function app created]({{ site.url }}/assets/images/2021/07/function_app_created.png)

To debug the functions, you need to install Azure Function app core tools - you can get more details about installation from [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?WT.mc_id=AZ-MVP-5002040){:target="_blank"}. You need to follow the installation steps for Linux - Ubuntu. You need to run the following commands in the terminal.

{% highlight Shell %}
{% raw %}
wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install azure-functions-core-tools-3
{% endraw %}
{% endhighlight %}

Once installed, you can run `func --version` command in the terminal - which should display 3.x. Now you can debug the function using `F5` key or you can click on the debug button and click on the debug button. Now Codespaces will run your application in `http://localhost:7071/api/HttpTrigger1` and which is forwarded to a global unique address where you can browse and test the azure function. As I mentioned earlier in Codespaces you will get all the development / debugging experience with VS Code.

![Debug Function app]({{ site.url }}/assets/images/2021/07/debug_azure_function.png)

## Deploying the Function to Azure

To deploy the function Azure you can click on the Azure Icon on the left side toolbar. And click on the Signin with Azure Portal. Once you signed in you will be able to see existing function apps in your subscription and there is an option - `Deploy to function app`, you can click on that - which will prompt the azure function creation command wizard. In the wizard, you need to configure select the Create new function project as the first output, then you need to configure a globally unique name for the function app. Next you need to choose the NodeJs framework versions and finally the location you want to deploy the function app. Once you provide all the values, VSCode will create a function app and deploy the current project to the function app. This might take some time. Once it is completed you will be able to see a dialog like this. And you will be able to browse the function in the URL shown in the output window.

![Node JS function deployment completed]({{ site.url }}/assets/images/2021/07/nodejs_function_deployed.png)

This way you will be able to develop and deploy azure function app using GitHub codespaces. You can use Codespaces for developing dotnet core web app as well. By default Codespaces comes with dotnet SDKs preinstalled.

If you received the GitHub CoPilot beta invite, you will be able to install CoPilot on codespaces as well - so you can use the AI capabilities to write your code.

Happy Programming :)