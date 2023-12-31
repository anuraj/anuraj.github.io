---
layout: post
title: "Preventing Cross-Site Request Forgery (CSRF) Attacks in WebAPI"
subtitle: "Preventing Cross-Site Request Forgery (CSRF) Attacks in WebAPI"
date: 2013-11-11 18:09
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, Javascript, Web API]
tags: [.Net, ASP.Net MVC, CSRF, Javascript, JQuery, WebAPI]
header-img: "img/post-bg-01.jpg"
---
CSRF is an attack which forces an end user to execute unwanted actions on a web application in which he/she is currently authenticated. With a little help of social engineering (like sending a link via email/chat), an attacker may trick the users of a web application into executing actions of the attacker's choosing. A successful CSRF exploit can compromise end user data and operation in case of normal user. If the targeted end user is the administrator account, this can compromise the entire web application.

Here is an example of CSRF attack.


1.  User requesting a page - with Forms authentication enabled and Anonymous authentication disabled. Server is checking for Forms authentication cookie. And responding with a 302 status code, as the request doesn't contains the cookie.
![Requesting a web page with Forms authentication]({{ site.url }}/assets/images/2013/11/loadpage1.png)
2.  Browser is redirecting to the login page, and gets the login page.
![Browser redirecting to login page]({{ site.url }}/assets/images/2013/11/loginpage.png)
3.  User authenticating with the credentials, sending a POST request.
![credentials send a POST request for authentication]({{ site.url }}/assets/images/2013/11/loginwithcredentials.png)
4.  Server authenticates the user and Forms authentication cookie is set.
![Server Authenticated the user and cookie is set]({{ site.url }}/assets/images/2013/11/cookieset.png)
5.  User sending a Ajax POST request to WebAPI. And resource created, server responded with status 201.
Here is the controller action.

{% highlight Javascript %}
[Authorize]
public HttpResponseMessage Post(Employee employee)
{
    var employeeId = _employeeRepository.Create(employee);
    var response = Request.CreateResponse<Employee>
        (HttpStatusCode.Created, employee);
    response.Headers.Location =
        new Uri(Url.Link("DefaultApi", new { id = employeeId }));

    return response;
}
{% endhighlight %}

Here is the request 

{% highlight Javascript %}
function createEmployee() {
    var emp = { Name: "anuraj", Email: "anuraj@server.com", Phone: "938944" };
    $.ajax("/api/Employee", {
        type: "POST",
        contentType: "application/json",
        data: JSON.stringify(emp),
        dataType: "json"
    });
}
{% endhighlight %}

![POST request to Web API]({{ site.url }}/assets/images/2013/11/post1.png)
The controller action contains an Authorize attribute, which means only Authorized request will be accepted.

6.  From another web page, user sending another POST request, without logged in.

Here is the request. Please not the URL, in previous request it was /api/Employee, but in this request, it is absolute url.

{% highlight Javascript %}
function createEmployee() {
    var emp = { Name: "attack", Email: "attack@fake.com", Phone: "attack" };
    $.ajax("http://localhost:56103/api/Employee", {
        type: "POST",
        contentType: "application/json",
        data: JSON.stringify(emp),
        dataType: "json"
    });
}
{% endhighlight %}

And here is the server response.

![POST request to WebAPI without authentication]({{ site.url }}/assets/images/2013/11/post2.png)

It is also created the resource, and server responded with status 201. As the request comes to an authenticated web page, the browser will automatically send the cookie with the request.


Typically, CSRF attacks are possible against web sites that use cookies for authentication, because browsers send all relevant cookies to the destination web site. However, CSRF attacks are not limited to exploiting cookies. For example, Basic and Digest authentication are also vulnerable. After a user logs in with Basic or Digest authentication. the browser automatically sends the credentials until the session ends. 

Anti-Forgery Tokens - To prevent CSRF attacks ASP.NET MVC uses Anti-Forgery Tokens or request verification tokens. If you enable this, server will includes two tokens with the response. One token is sent as a cookie. The other is placed in a hidden form field. When data is posted, the Cookie and the Hidden Field are both sent back and if they are missing or they donâ€™t match, the POST is rejected. In MVC this happens automatically when you request for an AntiForgeryToken. In Web API, we have to do this check manually. 

Here is the Anti-Forgery Token implementation for Ajax requests, this function will generate the cookie token and form token, and returns as string.

{% highlight CSharp %}
public string TokenHeaderValue()
{
    string cookieToken, formToken;
    AntiForgery.GetTokens(null, out cookieToken, out formToken);
    return cookieToken + ":" + formToken;
}
{% endhighlight %}

And you can use this in Ajax requests like this.

{% highlight Javascript %}
function createEmployee() {
    var emp = { Name: "anuraj", Email: "anuraj@server.com", Phone: "938944" };
    $.ajax("/api/Employee", {
        type: "POST",
        contentType: "application/json",
        data: JSON.stringify(emp),
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
}
{% endhighlight %}

And in server side, I created an Action Filter, which will validate the request.

{% highlight Javascript %}
public override void OnActionExecuting(HttpActionContext actionContext)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (actionContext.Request.Headers.
        TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
{% endhighlight %}

[AntiForgery.Validate](http://msdn.microsoft.com/en-us/library/system.web.helpers.antiforgery.validate%28v=vs.111%29.aspx) method to validate the tokens. The Validate method throws an exception if the tokens are not valid.

And here is the server response with and without AntiForgery implementation.

POST request with RequestVerification header

![POST request with RequestVerification header]({{ site.url }}/assets/images/2013/11/post_with_anti.png)

And POST request without RequestVerification header - Server response is 500, Internal server error, ActionFilter attribute is throwing exception.

![POST request without RequestVerification header]({{ site.url }}/assets/images/2013/11/post_without_anti.png)

You can also prevent CSRF attacks by verifying the request referrer too, to a certain extend. 

Happy Programming
