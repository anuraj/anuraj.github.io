---
layout: post
title: "Deploy a website with Azure virtual machines - Creating SSL certificates"
subtitle: "This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines"
date: 2023-06-23 00:00:00
categories: [Azure,IAAS,Virtual Machine,AspNetCore]
tags: [Azure,IAAS,Virtual Machine,AspNetCore]
author: "Anuraj"
image: /assets/images/2023/06/create_ssl_certificate_step9.png
---

This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines. This is part of series where we will be discussing about setting up web server, configuring custom domains, creating and installing SSL certificate in web server and finally configuring CI / CD pipelines to deploy our application to Azure VM from Azure DevOps. In this blog post we will be exploring about creating SSL certificate for the web server.

We will be using Azure CLI and Powershell for the implementation. For this post I am using LetsEncrypt as the certificate authority. 

First we need to install `win-acme` - which offers a dotnet tool. We can install it by running the command `dotnet tool install win-acme --global`. To create the certificate, I am using the DNS verification method - where I will be updating the DNS records of the my domain provider. This method is easy for me. There are other methods available as well. We can execute the following steps and create a free SSL certificate.

1. Open terminal, execute the following command - `wacs`

![Create free SSL certificate - Step 1]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step1.png)

This will prompt a console wizard like this - as I am running as normal user, some of the IIS features will not be enabled for me. We need to enter the first option (N).

2. In the second step, we can set the 2 option - Manual input.

![Create free SSL certificate - Step 2]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step3.png)

3. Next step, we need to enter the domain name, we are using `vm.dotnetthoughts.net`.

![Create free SSL certificate - Domain name]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step3.png)

4. In the 4th step, we need to validate the domain name authorization, as mentioned earlier I am choosing the option 6 - `Create verification records manually (auto-renew not possible)`.

![Create free SSL certificate - Domain validation]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step4.png)

5. Next we need to select where we want to store the certificate, once it is generated, I am selecting the default PFX archive option.

![Create free SSL certificate - Certificate storage]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step5.png)

6. Once we select the option, it will prompt for a location. We need to enter one location.

![Create free SSL certificate - Certificate location]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step6.png)

7. And once the location is provided, we need to give the password for the certificate, I am entering it manually, so I am choosing the 2 option. And I am entering the PFX password.s

![Create free SSL certificate - Password configuration]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step7.png)

8. In the next step, it will prompt for `Save to vault for future reuse`, I am entering No for the option. And in the next step I am selecting 3 option -  No (additional) installation steps - since I am manually updating the certificate and bindings.

9. In the next step, the tool will prompt and show the DNS settings we need to update in the domain control panel. 

![Create free SSL certificate - Domain verification]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step7.png)

And once it is done, we can press enter and verify the changes.

10. And in the domain control panel, add the TXT records like this.

![Create free SSL certificate - DNS changes]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step8.png)

As I am using Google Domains, it will propagate the changes very fast and we will be able to verify quickly. Once validation is successful, we can delete the DNS record. The tool will prompt to delete it and it will validate the deletion as well.

11. Once it is completed, the tool will generate PFX file and display the details in the screen.

![Create free SSL certificate - PFX file created]({{ site.url }}/assets/images/2023/06/create_ssl_certificate_step9.png)

And we can quit from the tool, by entering `Q` as the option.

This way we can use the win-acme tool and LetsEncrypt to generate free PFX file. Please note - this certificate will not renew automatically, we need to renew it before expiry. In the next blog post, we will explore how we can install it in the Azure VM and configure IIS bindings.

Happy Programming.