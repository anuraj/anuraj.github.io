---
layout: post
title: "Mapping a wildcard domain name to an Azure App Services"
subtitle: "This post is about how we can map wild card domain to Azure App Service."
date: 2022-05-10 00:00:00
categories: [Azure,AppService]
tags: [Azure,AppService]
author: "Anuraj"
image: /assets/images/2022/05/add_custom_domain.png
---
This post is about how we can map wild card domain to Azure App Service. Why we need to map wild card domains? When we are building SAAS applications it is a good practice to provision the tenants with your application sub domain. For example in case JIRA, when you create an instance your instance will be https://instance-name.atlassian.net/. And you need wild card SSL as well for running the instance of HTTPS.

For mapping a custom domain, open your app service, select the custom domain option &gt; and then select the Add custom domain option.

![Add Custom domain]({{ site.url }}/assets/images/2022/05/add_custom_domain.png)

For a subdomain you can provide the name in the textbox and click on validate. This will show two DNS records one CNAME and one TXT record. We need to update these records in the domain name provider DNS records settings. I am using a domain which I bought from Google Domains - I recommend Google Domains because it is pretty fast on update DNS records. For wild card domain mapping in the input field we need to set the url as `*.domain.name`. I am setting up a domain `*.anuraj.dev`. When we click on Validate button, Azure will display a TXT record and CNAME record. 

![Add Custom Domain]({{ site.url }}/assets/images/2022/05/add_custom_domain_dns.png)

Next we need to open Google Domains &gt; DNS. Then click on the `Manage Custom Records`. And add the CNAME and TXT record values from Azure. Once it completed, it will be something like this.

![Google Domain - Manage custom records]({{ site.url }}/assets/images/2022/05/google_domains_dns.png)

Once it is update the DNS records, we can click on the Validate button - if DNS records updated, the Add Domain button will be enabled and we can click on the button. Now we are ready to add any domain under this domain, like `hello.anuraj.dev` or `support.anuraj.dev` etc - without configuring anything differently. 

Configuring wild card domains will be useful when you're provisioning custom domain name programmatically.

Happy Programming :)