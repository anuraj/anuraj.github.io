---
layout: post
title: "Unit testing ASP.NET Web API Controller "
subtitle: "Unit testing ASP.NET Web API Controller "
date: 2013-09-03 10:50
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Entity Framework, Unit Testing, Web API]
tags: [.Net, ASP.Net, C#, Entity Framewrok, Unit Testing, WebAPI]
header-img: "img/post-bg-01.jpg"
---
This post is about unit testing Web API controller. As we are using Entity Framework and some Web API related classes, we need to implement dependency injection to unit test Web API controllers. 

To manage Entity Framework dependency, I am using a simple [Repository pattern](http://martinfowler.com/eaaCatalog/repository.html). Here is my repository interface.

{% highlight CSharp %}
public interface IRepository
{
    IEnumerable<Employee> ReadAll();
    Employee ReadById(int id);
    int Create(Employee employee);
    int Update(int id, Employee employee);
    int Delete(int id);
}
{% endhighlight %}

As this post is more focusing on unit testing, I am not including the implementation of the IRepository interface, it is same as we used in the controller. And I am modified the controller like this, one more constructor added, which accepts IRepository as one parameter.

{% highlight CSharp %}
private readonly IRepository _employeeRepository;
public EmployeeController()
    : this(new EmployeeRepository())
{
//Empty constructor.
}
//For Testability
public EmployeeController(IRepository repository)
{
    _employeeRepository = repository;
}
{% endhighlight %}

And the Get methods in the controller is modified like this.

{% highlight CSharp %}
public HttpResponseMessage Get()
{
    return Request.CreateResponse<IEnumerable<Employee>>
                    (HttpStatusCode.OK, _employeeRepository.ReadAll());
}

public HttpResponseMessage Get(int id)
{
    var selectedEmployee = _employeeRepository.ReadById(id);
    if (null != selectedEmployee)
    {
        return Request.CreateResponse<Employee>
            (HttpStatusCode.OK, selectedEmployee);
    }
    else
    {
        return Request.CreateErrorResponse
            (HttpStatusCode.NotFound, "Employee not found");
    }
}
{% endhighlight %}

Instead of using Entity Framework directly, interacting to DB via Repository. You can write the unit test like this to verify the Get method.

{% highlight CSharp %}
//ARRANGE
var expected = new List<Employee>(1) { 
    new Employee() { Id = 1, Email = "email", Name = "name", Phone = "123" }
};
Mock<IRepository> mockRepository = new Mock<IRepository>();
mockRepository.Setup(x => x.ReadAll()).Returns(() => expected);
var controller = new EmployeeController(mockRepository.Object);
controller.Request = new HttpRequestMessage()
{
    Properties = { { HttpPropertyKeys.HttpConfigurationKey, new HttpConfiguration() } }
};
//ACT
var response = controller.Get();

//ASSERT
Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);
var actual = response.Content.ReadAsAsync<List<Employee>>().Result;
Assert.AreEqual(expected.Count, actual.Count);
{% endhighlight %}

For mocking purposes I am using [Moq](http://code.google.com/p/moq/) library. As we are using the Request property to create the HttpResponseMessage, you need to set a default value to the request property, otherwise you will get some null reference exception from the unit test.

Similar to Request property, in the post method we are using the Url property from APIController class, which is an instance of the UrlHelper class. Unlike the Request property, Url property is quite complex to create, so I am using a wrapper class, which will wraps the Url.Link() method. And I created an interface like this.

{% highlight CSharp %}
public interface IUrlHelper
{
    string GetLink(string routeName, object routeValues);
    UrlHelper Url { get; set; }
}
{% endhighlight %}

And the implementation is like this.

{% highlight CSharp %}
public class CustomUrlHelper : IUrlHelper
{
    public UrlHelper Url { get; set; }

    public string GetLink(string routeName, object routeValues)
    {
        return Url.Link(routeName, routeValues);
    }
}
{% endhighlight %}

To use the IUrlHelper interface, I modified the controller class again like this.

{% highlight CSharp %}
private readonly IRepository _employeeRepository;
private readonly IUrlHelper _urlHelper;

public EmployeeController()
    : this(new EmployeeRepository(), new CustomUrlHelper())
{
    _urlHelper.Url = Url; //In controller empty constructor, you need to set this property.
}
//For Testability
public EmployeeController(IRepository repository, IUrlHelper urlHelper)
{
    _employeeRepository = repository;
    _urlHelper = urlHelper;
}
{% endhighlight %}

Both Post and Put method modified to use _urlHelper, instead of the Url property.
{% highlight CSharp %}
public HttpResponseMessage Post(Employee employee)
{
    _employeeRepository.Create(employee);
    var response = Request.CreateResponse<Employee>
        (HttpStatusCode.Created, employee);
    response.Headers.Location = 
        new Uri(_urlHelper.GetLink("DefaultApi", new { id = employee.Id }));
    return response;
}
{% endhighlight %}

Here is the test method for verifying the Post method in the controller.

{% highlight CSharp %}
//ARRANGE
var isCreatedInvokedInRepository = false;
var employee = new Employee()
{
    Id = 1,
    Email = "email",
    Name = "name",
    Phone = "phone"
};

var mockRepository = new Mock<IRepository>();
var mockUrlHelper = new Mock<IUrlHelper>();
mockUrlHelper.Setup(x => x.GetLink(It.IsAny<string>(), It.IsAny<object>()))
    .Returns("http://localhost/webapi/employee/");
mockRepository.Setup(x => x.Create(It.IsAny<Employee>())).
    Callback(() => isCreatedInvokedInRepository = true);
var controller = new EmployeeController(
    mockRepository.Object, mockUrlHelper.Object);
controller.Request = new HttpRequestMessage()
{
    Properties = { { HttpPropertyKeys.HttpConfigurationKey, new HttpConfiguration() } }
};

//ACT
var response = controller.Post(employee);

//ASSERT
Assert.IsTrue(isCreatedInvokedInRepository,
    "Create method in Repository not invoked");
Assert.AreEqual(HttpStatusCode.Created, response.StatusCode);
Assert.IsNotNull(response.Headers.Location);
{% endhighlight %}

You can write similar tests for all the other methods.

Happy Unit testing.
