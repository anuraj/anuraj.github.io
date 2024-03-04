---
layout: post
title: "Supercharge Your ASP.NET Core Web API: A Beginner's Guide to Bombardier Benchmarking"
subtitle: "In this blog post, we will learn about Bombardier and how we can benchmark ASP.NET Core Web API applications using Bombardier."
date: 2024-02-29 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2024/02/bombardier_running.png
---

Making sure your ASP.NET Core Web API runs like a well-oiled machine is crucial for a smooth user experience. Enter Bombardier, a user-friendly tool designed specifically to test and optimize the performance of your web API. In this guide, we'll explore how Bombardier can be your ally in ensuring your web API stands up to the demands of real-world usage.

Bombardier is like a helpful tool that checks if your ASP.NET Core Web API is doing its job well. It's easy to use, and the best part is, it is Free.

Using Bombardier is easy. Here's a quick guide:

1. First we need to install Go language in the machine, we can download Go language from [here](https://go.dev/doc/install)
2. Once Go installed, we can run the following command - `go install github.com/codesenberg/bombardier@latest`

This command will install Bombardier in the machine. By running `bombardier --help` will help us to verify the installation is correct.

Next we can create a simple Web API application using `dotnet new webapi --name WeatherForecastApi --output WeatherForecastApi\Src`, next move to the Src directory and run the `dotnet run` command, which will start the API application. Next we can run the following command `bombardier -c 8 -n 100000 -m GET http://localhost:5087/weatherforecast` - this command will to send 100,000 requests with 8 connections to your API to see how it handles them.

![Bombardier running]({{ site.url }}/assets/images/2024/01/bombardier_running.png)

The numbers Bombardier gives you might look confusing, but they're like clues about how your API is doing:

* Requests per second (Reqs/sec): On average, your API handled 13366.17 requests per second.
* Latency: On average, it took 604.45 microseconds to respond, with a maximum of 169.84 milliseconds.
* HTTP codes: All responses were in the 2xx category, meaning everything went well.
* Throughput: Your API processed data at a rate of 7.84MB/s.

These numbers help you know where your API is doing well and where it can be faster.

Here is an example of Post request with Body - `bombardier -c 120 -n 50000 -H "Content-Type: application/json" -m POST -b '{"OriginalUrl": "http://example.com/helloworld"}' http://localhost:5306/create`

To get the most out of Bombardier:

1. Test Like Real People Use It: Try to test your API like how real people would use it.
2. Try Different Settings: Play around with different settings to see what makes your API work the best.
3. Test Regularly: Keep using Bombardier regularly to see if your API is getting better or if it needs some improvement.

Think of Bombardier as your friendly helper, making sure your ASP.NET Core Web API is running at its best. By checking your API regularly, you can make sure it's always quick and ready for action. So, give Bombardier a try and make your web API even better!

Happy Programming.