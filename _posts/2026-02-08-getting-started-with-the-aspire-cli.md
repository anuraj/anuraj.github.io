---
layout: post
title: "Getting Started with the Aspire CLI"
subtitle: "In this blog post, we will learn about Aspire CLI. The Aspire CLI is a powerful, cross-platform tool designed to streamline the development, management, and deployment of application systems"
date: 2026-02-08 00:00:00
categories: [Aspire]
tags: [Aspire]
author: "Anuraj"
image: /assets/images/2026/02/aspire_dashboard.png
---

In this blog post, we will learn about Aspire CLI. The Aspire CLI is a powerful, cross-platform tool designed to streamline the development, management, and deployment of application systems.

First we can install the Aspire CLI using `dotnet tool install --global Aspire.Cli` command. And then we can run `aspire --version` command to verify the installation.

Once installation is successful, we can start using Aspire CLI using `aspire new` command.

![Aspire CLI new command]({{ site.url }}/assets/images/2026/02/aspire_new_command.png)

For this blog post, I am creating the React app with Redis Cache. And we can run the app using `aspire run` command. When we run the `aspire run` command, it will run the application and we can open the dashboard using the displayed URL.

![Aspire CLI run command]({{ site.url }}/assets/images/2026/02/aspire_run_command.png)

And we can add integration and packages using the `aspire add` command. This command will list various available components we can add.

![Aspire CLI add command]({{ site.url }}/assets/images/2026/02/aspire_add_command.png)

And we can publish the Aspire application using `aspire publish` command. This command serializes the resources for deployment tools.

There a lot of other command options available as well. Explore the CLI and let me know which is the favorite feature.

Happy Programming.