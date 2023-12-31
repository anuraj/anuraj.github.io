---
layout: post
title: "Working with Azure Logic Apps"
subtitle: "This post is about working with Azure Logic Apps. Logic App provide a way to solve, simplify and implement scalable integration and workflow in the cloud. It provides coding and visual designer windows to automate the process. In this post I am creating a azure logic app, which helps to monitor website."
date: 2016-12-18 00:00:00
categories: [Azure, Logic App, App service]
tags: [Azure, Logic App, App service]
author: "Anuraj"
---
This post is about working with Azure Logic Apps. Logic App provide a way to solve, simplify and implement scalable integration and workflow in the cloud. It provides coding and visual designer windows to automate the process. In this post I am creating a azure logic app, which helps to monitor website. Logic Apps is a fully managed iPaaS (integration Platform as a Service) allowing developers not to have to worry about building hosting, scalability, availability and management. Logic Apps will scale up automatically to meet demand.

![Azure Logic App]({{ site.url }}/assets/images/2016/12/azure_logic_app.png)

Once you clicked on Create button, Create Logic App blade will open up. You can provide the name and other details.

![Create Azure Logic App]({{ site.url }}/assets/images/2016/12/create_logic_app.png)

Provide the name and other details will create the logic app. Once Logic app created successfully, clicking on Logic app name will redirect to the Logic App Designer. 

![Logic App Designer]({{ site.url }}/assets/images/2016/12/logic_app_designer.png)

You can either choose blank template or any other predefined templates. For this post I am using blank template. 

The Logic App, I am creating will monitor my blog on specific intervals and if it is down, it will send an email to me. For implementing this, first I am adding an HTTP API, in the method I am setting GET and in the URI I am setting my blog url. 

![Logic App Designer - HTTP API]({{ site.url }}/assets/images/2016/12/http_step1.png)

I am configuring the Frequency of HTTP GET to 3 hours, so every 3 hours Logic App will execute a GET request to my blog and returns the status code. Now click on the New Step button, and select add a condition option, this is to check wheather I am getting a HTTP Ok status code (200) or something else.

![Condition option]({{ site.url }}/assets/images/2016/12/codition_option.png)

And if I am not getting a 200, I need to send an email. For that click on the Add an action option and need to configure email API. Here I am using Outlook API to send email. For that you need to authorize Azure Logic app to send email and once it is done, you will get an interface like this, where you can provide the e-mail details.

![Send email action]({{ site.url }}/assets/images/2016/12/send_email_action.png)

Now you can click on the save button in logic app designer, which will save the changes you did so far. You can test the implementation using Run option. Clicking on Code View option will display the Javascript code of the app.

Here is the email I recevied from Logic App

![Email from Logic App]({{ site.url }}/assets/images/2016/12/email_from_logic_app.png)

Logic Apps brings speed and scalability into the enterprise integration space. The ease of use of the designer, variety of available triggers and actions, and powerful management tools make centralizing your APIs simpler than ever. As businesses move towards digitalization, Logic Apps allows you to connect legacy and cutting-edge systems together. To turn off the app, click Disable in the command bar. View run and trigger histories to monitor when your logic app is running. You can click Refresh to see the latest data.

Happy Programming :)