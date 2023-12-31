---
layout: post
title: "App Service managed SSL certificates for Azure Web Apps"
subtitle: "This post is about creating App service managed SSL certificates for Azure Web Apps. SSL certificates play key role in authenticity of a web application. SSL helps to encrypt the traffic between browser and server and for verifying the server identity."
date: 2020-03-04 00:00:00
categories: [Azure,SSL,AppServices]
tags: [Azure,SSL,AppServices]
author: "Anuraj"
---
This post is about creating App service managed SSL certificates for Azure Web Apps. SSL certificates play key role in authenticity of a web application. SSL helps to encrypt the traffic between browser and server and for verifying the server identity. Usually for a web application, we need to buy SSL certificates from the providers like GoDaddy or Digicert. There are some free SSL providers also available like Lets Encrypt - but they are offering the SSL expiry for limited number of days, after that you need to renew it.

If you're hosting your application in Azure Web Apps, Microsoft provides an option to generate SSL certificate for your domain without any cost. To generate the app service managed SSL, first you need to configure your DNS records and point the CNAME records to your azure app service. Here is my DNS records from GoDaddy DNS settings.

![DNS Records]({{ site.url }}/assets/images/2020/03/dns_records.png)

Next you need to configure the custom domain name in Azure Web Apps. You can do this from Custom Domains blade in Azure. And click on the `Add custom domain` option. In the custom domain textbox provide the URL and click on validate. 

![Custom domain configuration]({{ site.url }}/assets/images/2020/03/custom_domain_configuration.png)

Since the DNS records are already updated, Azure will validate the domain name records and allow you to add the domain to Azure web app. Once the domain is added Azure portal will display a SSL certificate missing warning message. To fix this, select the `TLS/SSL settings` blade. In the `TLS/SSL settings`, click on the `Create App Service Managed Certificates` option. It is a preview feature. By default it will show the domain name without SSL.

![Create App Service Managed Certificates]({{ site.url }}/assets/images/2020/03/create_managed_certificate.png)

Click on the `Create` button, it will take some time and will show notification.

![List of associated certificates]({{ site.url }}/assets/images/2020/03/certificate_list.png)

Once it is done, you need to come back to the `Custom Domains` blade and click on `Add Binding`.

![Add SSL binding]({{ site.url }}/assets/images/2020/03/add_ssl_binding.png)

From the domain list choose the domain, select the Private Certificate Thumbprint and `TLS / SSL Type` choose `SNI SSL`. And click on `Add Binding` button.

Now browse the URL, it will show as secure.

![SSL Secure]({{ site.url }}/assets/images/2020/03/ssl_certificate.png)

Happy Programming :)