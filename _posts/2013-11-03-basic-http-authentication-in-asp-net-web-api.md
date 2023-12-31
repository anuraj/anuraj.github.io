---
layout: post
title: "Basic HTTP authentication in ASP.Net Web API"
subtitle: "Basic HTTP authentication in ASP.Net Web API"
date: 2013-11-03 09:22
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Web API]
tags: [.Net, ASP.Net, Basic Authentication, C#, JQuery, WebAPI]
header-img: "img/post-bg-01.jpg"
---
In this post I am going to show how to implement Basic HTTP authentication in a Web API project by customizing [AuthotrizeAttribute](http://msdn.microsoft.com/en-us/library/system.web.http.authorizeattribute.aspx). HTTP authentication is a standard protocol and can be easily handled by most popular client and mobile platforms. 

Basic authentication works as follows: - If a request requires authentication, the server returns 401 (Unauthorized). The response includes a WWW-Authenticate header, indicating the server supports Basic authentication. The client sends another request, with the client credentials in the Authorization header. The credentials are formatted as the string â€œname:passwordâ€, base64-encoded. The credentials are not encrypted. Basic authentication is performed within the context of a â€œrealm.â€ The server includes the name of the realm in the WWW-Authenticate header. The userâ€™s credentials are valid within that realm. The exact scope of a realm is defined by the server. 

![Basic authentication flow]({{ site.url }}/assets/images/2013/11/basic_auth.png)

Because the credentials are sent unencrypted, Basic authentication is only secure over HTTPS. Basic authentication is also vulnerable to CSRF attacks. After the user enters credentials, the browser automatically sends them on subsequent requests to the same domain, for the duration of the session. This includes AJAX requests. 

Basic Authentication built into IIS uses Windows credentials, which means you need to create accounts for your users on the hosting server. But for an internet application, it may not be feasible, normally user accounts are typically stored in an external database.

Here is the implementation.

{% highlight CSharp %}
public override void OnAuthorization(HttpActionContext actionContext)
{
    var authHeader = actionContext.Request.Headers.Authorization;
    if (authHeader != null
        && !string.IsNullOrEmpty(authHeader.Parameter)
        && authHeader.Scheme.Equals("basic", StringComparison.OrdinalIgnoreCase))
    {
        var encoding = Encoding.GetEncoding("iso-8859-1");
        var credentialstring = encoding.GetString(Convert.FromBase64String(authHeader.Parameter));
        var credentials = credentialstring.Split(':');
        var isValid = Validate(credentials[0], credentials[1]);
        if (isValid)
        {
            var identity = new GenericIdentity(credentials[0]);
            var principal = new GenericPrincipal(identity, null);
            Thread.CurrentPrincipal = principal;
            if (HttpContext.Current != null)
            {
                HttpContext.Current.User = principal;
            }

            return;
        }
    }

    HandleUnauthorizedRequest(actionContext);
}

protected override void HandleUnauthorizedRequest(HttpActionContext actionContext)
{
    //Modify if required.
    var realm = "dotnetthoughts";
    var result = new HttpResponseMessage()
    {
        StatusCode = HttpStatusCode.Unauthorized,
        RequestMessage = actionContext.ControllerContext.Request
    };

    result.Headers.Add("WWW-Authenticate",
        string.Format("Basic realm=\"{0}\"", realm));

    actionContext.Response = result;
}

private bool Validate(string username, string password)
{
    //Validate username and password against the Database
    return true;
}
{% endhighlight %}

And you can decorate the API controller / actions using this attribute.

{% highlight CSharp %}
[BasicAuthentication]
public IEnumerable<string> Get()
{
    return new string[] { "value1", "value2" };
}
{% endhighlight %}

or 

{% highlight CSharp %}
[BasicAuthentication]
public class ValuesController : ApiController
{
{% endhighlight %}

And you can access the controller method(s) using JQuery like this.

{% highlight Javascript %}
$.ajax({
    url: '/api/values',
    type: 'GET',
    contenttype: 'application/json',
    success: function (data) {
        alert(data);
    },
    beforeSend: function (xhr) {
        xhr.setRequestHeader("Authorization",
            "Basic " + btoa("username" + ":" + "password"));
    }
});
{% endhighlight %}

The beforeSend function will help to set the Authorization header for the request.

Happy Programming.
