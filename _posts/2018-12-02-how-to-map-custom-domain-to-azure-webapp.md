---
layout: post
title: "How To Map Custom Domain To Azure Web App"
subtitle: "This post is about how To Map Custom Domain To Azure Web App. By default azure web app comes with a subdomain.azurewebsites.net. In this post I am explaining how to map a custom domain to an azure web app."
date: 2018-12-02 00:00:00
categories: [Azure]
tags: [Azure]
author: "Anuraj"
---
This post is about how To Map Custom Domain To Azure Web App. By default azure web app comes with a subdomain.azurewebsites.net. In this post I am explaining how to map a custom domain to an azure web app.

You can map domain or sub domain to an azure web app. If you're mapping a domain, you need to modify your `A NAME` records and for subdomain's you need to configure `C NAME` records.

1. First you need to create azure web app. For configuring custom domain's you need to choose `D1 (Shared Infrastructure)` or more, which supports Custom domains. 

![Azure Web App]({{ site.url }}/assets/images/2018/12/azure_web_app.png)

2. I created this web app with shared pricing plan. Once you created the web app, select custom domain option from the overview blade. 

![Custom domain configuration in Azure Web app]({{ site.url }}/assets/images/2018/12/custom_domain_in_azure_webapp.png)

3. Click on the Add HostName option, enter a Hostname and click on `Validate` button. This will show configuration details for the hostname. If it is domain, then it will show A NAME configuration, and if you're entering a subdomain, it will show C NAME configuration details, since you didn't configured it, it is showing `Domain Ownership` as error.

![Custom domain configuration in Azure Web app]({{ site.url }}/assets/images/2018/12/custom_domain_configure.png)

5. You will get IP Address of your azure web app, you need to configure A NAME records and TXT records of your domain provider to azure web app. Here is the A NAME configuration.

![A NAME record]({{ site.url }}/assets/images/2018/12/aname_records.png)

6. And next you need to configure TXT records. Here is the screenshot for the same.

![TXT NAME record]({{ site.url }}/assets/images/2018/12/txt_records.png)

You can check whether the DNS records updated or not using websites like `https://www.whatsmydns.net`, this app will help you to verify the A NAME and TXT records got updated or not. Depends on your domain name provider, it might take sometime. Once it is done, you need to click on the `Validate` button again. Then you will be able to see the domain ownership also changed to green. 

![Domain Ownership - Completed]({{ site.url }}/assets/images/2018/12/domain_ownership.png)

7. Next click on the `Add Hostname` button. This will display the domain name under `Hostname Assigned to Site`.

![Custom domain - Updated]({{ site.url }}/assets/images/2018/12/custom_domain_updated.png)

Now onwards if you browse the site, it will bring your azure web app. In this way you can configure custom domain to azure web app. In the next blog post I will explain how to enable SSL for azure web app with custom domain.

Happy Programming :)