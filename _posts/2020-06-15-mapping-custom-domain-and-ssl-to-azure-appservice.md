---
layout: post
title: "How to map custom domain and SSL to Azure app service programmatically"
subtitle: "In this article, I am writing about how to map custom domain and SSL to Azure app service programmatically"
date: 2020-06-15 00:00:00
categories: [Azure,AppService,AspNetCore]
tags: [Azure,AppService,AspNetCore]
author: "Anuraj"
---
If you're building SAAS applications, it is a good practise to provide sub domains to tenants. If you're using Azure VMs it is simple and straight forward. You can configure the A NAME in DNS records and create a wild card CNAME record to the domain name of your Azure VM, so that all the subdomain requests comes to your VM - IIS. And application can decide which tenant based on the HOST header. 

But in case of Azure App Service, it is little bit tricky, because even if we point a wild card CNAME record to Azure App service - unless you create a domain name mapping in the custom domain name configuration in Azure App service, it won't work. I was facing this issue when we moved our app from VM to App Service. Later I found Azure provides APIs to manage this. Here is the code snippet.

For this to work properly, first you need to configure a CNAME entry in your DNS records - pointing to the app service. You can get the details from the custom domain configuration in App Service. Recently Azure introduced a TXT record also for configuring this. So you need to create one CNAME record with a wildcard entry and TXT record. You can find more details about how to configure this [here](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-domain#a?WT.mc_id=DT-MVP-5002040)

Next I have created a .NET Core API application and added reference of `Microsoft.Azure.Management.Fluent` NuGet package. This package is help us to work with Azure App Service. Using this package, we will create an instance of Azure Credentials and using this an instance of Azure object. You can find more details like how to create the Azure credentials from [here](https://github.com/Azure/azure-libraries-for-net/blob/master/AUTH.md).

Here is the code which will create the instance of the Azure Credentials and Azure management object instance.

{% highlight CSharp %}
var clientId = _configuration["ClientId"];
var clientSecret = _configuration["ClientSecret"];
var tenantId = _configuration["TenantId"];

var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(clientId, clientSecret, 
    tenantId, AzureEnvironment.AzureGlobalCloud);
var azure = Microsoft.Azure.Management.Fluent.Azure
    .Configure()
    .Authenticate(credentials)
    .WithDefaultSubscription();
{% endhighlight %}

Once you get the instance of Azure object, you can create map the domain name like this.

{% highlight CSharp %}
var webApp = azure.AppServices.WebApps.GetById(id);
webApp.Update().DefineHostnameBinding()
    .WithThirdPartyDomain(domain)
    .WithSubDomain(subdomain)
    .WithDnsRecordType(CustomHostNameDnsRecordType.CName)
    .Attach()
    .Apply();

{% endhighlight %}

In the above code snippet, the domain and subdomain variables are provided by the user. And here is the other code snippet which will map the SSL certificate to the domain name. The Id - Azure App Service unique identifier can be is like this - `/subscriptions/YOUR-SUBSCRIPTION-ID/resourceGroups/RESOURCE-GROUP-NAME/providers/Microsoft.Web/sites/APP-SERVICE-NAME`.

{% highlight CSharp %}

webApp.Update().DefineSslBinding()
    .ForHostname(subdomain)
    .WithExistingCertificate(certificate)
    .WithSniBasedSsl()
    .Attach()
    .Apply();

{% endhighlight %}

The certificate variable is the thumbprint information about the uploaded wildcard certificate. You can click on the TLS/SSL settings - select the certificate and copy the value of `Thumbprint` field from the Certificate Details blade.

If you're facing any issues while working with the Azure object, you can enable tracing / logging and understand what is the exact issue. Here is the updated code with tracing and logging enabled.

{% highlight CSharp %}
var clientId = _configuration["ClientId"];
var clientSecret = _configuration["ClientSecret"];
var tenantId = _configuration["TenantId"];
ServiceClientTracing.AddTracingInterceptor(new LoggingTracer());
ServiceClientTracing.IsEnabled = true;
var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(clientId, clientSecret,
    tenantId, AzureEnvironment.AzureGlobalCloud);
var azure = Microsoft.Azure.Management.Fluent.Azure
    .Configure()
    .WithLogLevel(HttpLoggingDelegatingHandler.Level.Basic)
    .Authenticate(credentials)
    .WithDefaultSubscription();
{% endhighlight %}

For implementing the LoggingTracer, you need to implement the `IServiceClientTracingInterceptor` interface. I was implemented it using simple `Console.WriteLine` statements. 

### Adding the domain entries to CORS section.

One of the other aspect of SAAS application security is CORS. Usually we used to put a wild card in the CORS section, it is not recommended so instead of that we need to provide the exact domain name. Here is the code snippet which will add the domain entry to the CORS section.

{% highlight CSharp %}
var siteConfig = await azure.WebApps.Inner.GetConfigurationAsync(webApp.ResourceGroupName, webApp.Name);
if (!appServiceDomain.Domain.StartsWith("https://"))
{
    appServiceDomain.Domain = $"https://{appServiceDomain.Domain}";
}
siteConfig.Cors.AllowedOrigins.Add(appServiceDomain.Domain);
await azure.WebApps.Inner.CreateOrUpdateConfigurationAsync(webApp.ResourceGroupName, webApp.Name, siteConfig);
{% endhighlight %}

Even if you add this configuration to your App Service, your VAPT team won't clear your app for security issues related to CORS. I will discuss that on a later post.

Using the above code snippets you can map custom domain,SSL and CORS to the App Service. It is recommended for App Service automation and provisioning instances. 

Happy Programming :)