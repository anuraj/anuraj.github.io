---
layout: post
title: "Performing CURD operations using Web API â€“ Part 3"
subtitle: "Performing CURD operations using Web API â€“ Part 3"
date: 2013-09-01 23:57
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Entity Framework, Web API]
tags: [.Net, ASP.Net, C#, Entity Framewrok, WebAPI]
header-img: "img/post-bg-01.jpg"
---
I couldn't complete this series without mentioning the model validations part. Like normal services you can do custom validations, if validation fails, can create error responses and return. Similar to ASP.Net MVC, WebAPI also supports Model validations using DataAnnotations. And in the code you can use ModelState.IsValid property to validate Model is valid or not. For the validation purposes I modified the Employee model class like this.

{% highlight CSharp %}
public class Employee
{
    public int Id { get; set; }
    [Required(ErrorMessage = "Name cannot be empty")]
    public string Name { get; set; }
    [Required(ErrorMessage = "Email cannot be empty")]
    [EmailAddress(ErrorMessage = "Valid email address required")]
    public string Email { get; set; }
    [RegularExpression(@"[0-9]*$",
        ErrorMessage = "Valid phone number required.")]
    public string Phone { get; set; }
}
{% endhighlight %}

And the Post method code like this.

{% highlight CSharp %}
public HttpResponseMessage Post(Employee employee)
{
    using (DataContext dataContext = new DataContext())
    {
        if (ModelState.IsValid)
        {
            dataContext.Employees.Add(employee);
            dataContext.SaveChanges();
            var response = Request.CreateResponse<Employee>
                (HttpStatusCode.Created, employee);
            response.Headers.Location =
                new Uri(Url.Link("DefaultApi", new { id = employee.Id }));
            return response;
        }
        else
        {
            return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
        }
    }
}
{% endhighlight %}

In Web API, Request.CreateErrorResponse() method supports an overload which helps to return ModelState property.

Here is the Fiddler request, with invalid email address in POST body.

![Fiddler - POST request with Invalid Email address]({{ site.url }}/assets/images/2013/09/fiddler_compose.png)

And here is the response from Web API Service, in Fiddler.

![Service response in Fiddler]({{ site.url }}/assets/images/2013/09/fiddler_response.png)

Happy Programming

