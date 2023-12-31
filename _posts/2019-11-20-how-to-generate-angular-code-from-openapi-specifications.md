---
layout: post
title: "How to Generate Angular code from OpenAPI specifications"
subtitle: "This post is about how to generate Angular code from Open API specifications. Open API is a standard way to document REST APIs."
date: 2019-11-20 00:00:00
categories: [Angular,OpenAPI,Swagger]
tags: [Angular,OpenAPI,Swagger]
author: "Anuraj"
---
This post is about how to generate Angular code from Open API specifications. Open API is a standard way to document REST APIs. It is recommended practise document Web APIs using Open API or Swagger. You can find detailed tutorials on enabling Open API in ASP.NET Core. 

### Creating ASP.NET Core Web API 

First you need to create an Web API project, you can use `dotnet new webapi` command to create an API project. Next you need to add the `Swashbuckle.AspNetCore` to enable Open API. And modify your `Startup.cs` file to use Swagger or Open API. And I am enabling `CORS` as well.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors();
    services.AddControllers();
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo
        {
            Title = "WeatherForecast API",
            Version = "v1"
        });

        c.CustomOperationIds(apiDesc =>
        {
            return apiDesc.TryGetMethodInfo(out MethodInfo methodInfo) ? methodInfo.Name : null;
        });
    });
}
{% endhighlight %}

And 

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseSwagger(c =>
    {
        c.PreSerializeFilters.Add((swagger, httpReq) =>
        {
            swagger.Servers = new List<OpenApiServer> { new OpenApiServer { Url = $"{httpReq.Scheme}://{httpReq.Host.Value}" } };
        });
    });
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "WeatherForecast API V1");
        c.RoutePrefix = string.Empty;
    });

    app.UseCors(builder =>
    {
        builder.AllowAnyHeader().AllowAnyMethod().AllowAnyOrigin();
    });
    /* Code removed for brevity. */
}

{% endhighlight %}

I am attaching few links for your reference on enabling Open API in ASP.NET Core and enabling CORS in ASP.NET Core.

* [Develop ASP.NET Core apps using OpenAPI tools](https://docs.microsoft.com/en-us/aspnet/core/web-api/microsoft.dotnet-openapi?view=aspnetcore-3.0&WT.mc_id=DT-MVP-5002040)
* [ASP.NET Core web API help pages with Swagger / OpenAPI](https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-3.0&WT.mc_id=DT-MVP-5002040). 
* [Enable Cross-Origin Requests (CORS) in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-3.0&WT.mc_id=DT-MVP-5002040)

Now you can run the API using `dotnet run` command. Here is the Open API documentation page for the Web API.

![Open API Page]({{ site.url }}/assets/images/2019/11/swagger-page.png)

### Generating and building Angular Client

To generate the Angular code or any client code from Open API definition, first you need to install the openapi-generator, you can do it by running the following command. 

```
npm install @openapitools/openapi-generator-cli -g
```

It is an NPM package - CLI wrapper on top of the JAR's command line options. It support multiple generators, you can find more details about the generators from this [website](https://openapi-generator.tech/docs/generators). So we are using the Angular typescript generator. Once the installation is complete, run the following command, which will generate the Angular client code.

```
openapi-generator generate
>> -g typescript-angular
>> -i http://localhost:5000/swagger/v1/swagger.json
>> -o backend
>> --additional-properties npmName=@backend/api,snapshot=true,ngVersion=8.0.0
```

It will generate Angular typescript code in a folder `backend`. Navigate to the directory, open the `package.json` file and modify the version element to something like `1.0.0`, otherwise you may not be able to install the packages and build the library.

```
npm install

npm run build  

npm pack
```
This commands will generate a `tgz` file, which we can be used in our Angular project.

### Building Angular client application

Since you have generated the API client library, we will create the consumer application, you can create the application using `ng new` command. Once the application is created, reference the package using `npm install` command like this - `npm install ..\backend\dist\backend-api-1.0.0.tgz`, this command will show some warnings around missing dependencies - install them as well.

Here is my app.component.ts file.

{% highlight CSharp %}
import { Component } from '@angular/core';
import { WeatherForecast, WeatherForecastService } from "@backend/api"
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  private weatherForecasts: WeatherForecast[];
  constructor(private weatherforcastService: WeatherForecastService) {
    weatherforcastService.get().subscribe(data => {
      this.weatherForecasts = data;
    });
  }
  title = 'hello-world';
}
{% endhighlight %}

In this file, I am referencing the library and using the GET method to fetch the Weather forecast information from the Web API. And here is the HTML file.

{% highlight HTML %}
<div *ngFor="let forecast of weatherForecasts">
{% raw %}
  {{ forecast.date }} - {{ forecast.summary }}
{% endraw %}
</div>
{% endhighlight %}

Now you can run the application using `ng serve` command and verify you're able to get the weather information from the Web API. Now we're built a Angular client application with Open API generator code. You might get some warnings from Open API generator if you use the default API template and enabling Swagger on it. I have added few more steps which will help you to fix those issues. And you need to enable `CORS` as well. For this post I enabled for all the headers, methods and any origin. It is not recommended for Production.

Happy Programming :)