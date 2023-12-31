---
layout: post
title: "Attribute Routing in ASP.NET Core"
subtitle: "Routing feature helps to match a URI to an action. In Web API 2, attribute routing introduced. Attribute Routing helps to implement routing by providng the attribute to action methods. ASP.NET Core also supports the convention-based routing as well."
date: 2016-04-11 00:00
author: "Anuraj"
categories: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Web API, Attribute Routing, CodeProject]
tags: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Web API, Attribute Routing, CodeProject]
header-img: "img/post-bg-01.jpg"
---
Routing feature helps to match a URI to an action. In Web API 2, attribute routing introduced. Attribute Routing helps to implement routing by providng the attribute to action methods. ASP.NET Core also supports the convention-based routing as well. In convention based routing, all the routing information can be configured in a single place, which will be applied to all the controllers in the application. But it is hard to support certain URI patterns with convention based routing, like API versioning, Overloaded URI segments etc. Unlike old versions of Web API you don't need to configure anything to use attribute routing in ASP.NET Core.

Here is a sample route attribute

{% highlight CSharp %}
[Route("/Users/{userid}")]
public IActionResult GetUsers(int userId)
{
    return Json(GetAllUsers().First(x => x.Id == userId));
}
{% endhighlight %}

Now if you try to access it using "http://localhost:5000/Users/18" URL, it will display something like this.

![Attribute Routing JSON Response]({{ site.url }}/assets/images/2016/04/attribute_routing_json_response1.png)

The string "/Users/{userid}" is the URI template for the route. ASPNET Core tries to match the request URI to the template. In this example, "Users" is literal segment, and "{userid}" is a place holder or parameter. The user id parameter can be anything, like string or number. And if you try to access the URL with a string userid, it will return a user object with default values. You can restrict them using constraints. Here is a constraint, which validates user id is always integer.

{% highlight CSharp %}
[Route("/Users/{userid:int}")]
public IActionResult GetUsers(int userId)
{
    return Json(GetAllUsers().First(x => x.Id == userId));
}
{% endhighlight %}

Now if you try to access the URL with string user id, it will return an HTTP Status 404.

![Attribute Routing JSON Response - 404 response]({{ site.url }}/assets/images/2016/04/attribute_routing_json_response_404.png)

Here is few examples of Route constraints

* alpha - Matches uppercase or lowercase Latin alphabet characters (a-z, A-Z)
* bool - Matches a Boolean value.
* datetime - Matches a DateTime value.
* decimal - Matches a decimal value.
* double - Matches a 64-bit floating-point value.
* float - Matches a 32-bit floating-point value.
* guid - Matches a GUID value.
* int - Matches a 32-bit integer value.
* length - Matches a string with the specified length or within a specified range of lengths.
* long - Matches a 64-bit integer value.	
* max - Matches an integer with a maximum value.
* maxlength - Matches a string with a maximum length.
* min - Matches an integer with a minimum value.
* minlength - Matches a string with a minimum length.
* range - Matches an integer within a range of values.
* regex - Matches a regular expression.

You can use multiple constraints to a parameter, separated by a colon.
{% highlight CSharp %}
[Route("/Users/{userid:int:max(1000):min(10)}")]
{% endhighlight %}

Similar to convention based routing, attribute routing also supports Optional URI Parameters and Default Values. You can make a URI parameter optional by adding a question mark to the route parameter. If a route parameter is optional, you must define a default value for the method parameter.

{% highlight CSharp %}
[Route("/Users/{userid:int?}")]
public IActionResult GetUsers(int userId = 20)
{
}
{% endhighlight %}

Another feature is Route Names, which are useful for generating links, so that you can include a link in an HTTP response. To specify the route name, set the Name property on the attribute.

{% highlight CSharp %}
[Route("/Users/{userid:int}", Name="GetUserById")]
public IActionResult GetUsers(int userId)
{
    return Json(GetAllUsers().First(x => x.Id == userId));
}
[HttpPost("/Users/Create")]
public IActionResult Create(User user)
{
	//Create user implementation
    string uri = Url.Link("GetUserById", new { userid = user.Id });
    return Created(uri, user);
}
{% endhighlight %}

In the above code snippet, the Route name "GetUserById" is used in the Create method to generate the target link for the create response. Here is the response for Create method.

![Attribute Routing Create method Response]({{ site.url }}/assets/images/2016/04/attribute_routing_json_response_create.png)

Route Order is another feature which helps developers to specify which route to evaluate first, when a request coming for a resource. To specify the order, set the RouteOrder property on the route attribute. Lower values are evaluated first. The default order value is zero.

{% highlight CSharp %}
[Route("/Users/{userid:int}", Name = "GetUserById", Order = 1)]
{% endhighlight %}

You can create custom route constraints by implementing IRouteConstraint. 
{% highlight CSharp %}
public class CustomRouteConstraint : IRouteConstraint
{
    public bool Match(HttpContext httpContext, IRouter route, string routeKey, 
        IDictionary<string, object> values, RouteDirection routeDirection)
    {
        object value;
        if (values.TryGetValue(routeKey, out value) && value != null)
        {
            long longValue;
            if (value is long)
            {
                longValue = (long)value;
                return longValue != 10;
            }

            string valueString = Convert.ToString(value, CultureInfo.InvariantCulture);
            if (Int64.TryParse(valueString, NumberStyles.Integer, 
                CultureInfo.InvariantCulture, out longValue))
            {
                return longValue != 10;
            }
        }
        return false;
    }
}
{% endhighlight %}

This Constraint will returns a 404 response if the User Id is 10. You can register the constraint in the Startup.cs like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddRouting(options => 
        options.ConstraintMap.Add("Custom", typeof(CustomRouteConstraint)));
}
{% endhighlight %}

And you can use it like this in the route attribute.

{% highlight CSharp %}
[Route("/Users/{userid:int:custom}")]
{% endhighlight %}

You can find the ASP.NET Core Constraint implementations [here](https://github.com/aspnet/Routing/tree/dev/src/Microsoft.AspNetCore.Routing/Constraints)

Happy Programming :)
