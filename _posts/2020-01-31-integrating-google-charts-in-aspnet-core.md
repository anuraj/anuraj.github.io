---
layout: post
title: "Integrating Google Charts in ASP.NET Core"
subtitle: "This post is about integrating Google Charts in ASP.NET Core. We will learn about how to integrate Google Charts with Razor pages."
date: 2020-01-31 00:00:00
categories: [aspnetcore,google-charts]
tags: [aspnetcore,google-charts]
author: "Anuraj"
---
This post is about integrating Google Charts in ASP.NET Core. We will learn about how to integrate Google Charts with Razor pages. 

Google chart tools are powerful, simple to use, and free. Google Charts help to build interactive charts for browsers and mobile devices. You can get more details about Google Charts from [https://developers.google.com/chart](https://developers.google.com/chart) website.

To get started, create a web app project. And in the `index.cshtml` file, remove the existing code add the following code.

{% highlight HTML %}
{% raw %}
<div id="chart"></div>

@section Scripts
{
    <script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>

    <script type="text/javascript">

      // Load the Visualization API and the corechart package.
      google.charts.load('current', {'packages':['corechart']});

      // Set a callback to run when the Google Visualization API is loaded.
      google.charts.setOnLoadCallback(drawChart);

      // Callback that creates and populates a data table,
      // instantiates the pie chart, passes in the data and
      // draws it.
      function drawChart() {

        // Create the data table.
        var data = new google.visualization.DataTable();
        data.addColumn('string', 'Topping');
        data.addColumn('number', 'Slices');
        data.addRows([
          ['Mushrooms', 3],
          ['Onions', 1],
          ['Olives', 1],
          ['Zucchini', 1],
          ['Pepperoni', 2]
        ]);

        // Set chart options
        var options = {'title':'How Much Pizza I Ate Last Night',
                       'width':600,
                       'height':500};

        // Instantiate and draw our chart, passing in some options.
        var chart = new google.visualization.PieChart(document.getElementById('chart'));
        chart.draw(data, options);
      }
    </script>
}

{% endraw %}
{% endhighlight %}

So the above code is simple and straight forward. It will create a pie chart with the data and display it in the browser.

![Google Charts in ASP.NET Core]({{ site.url }}/assets/images/2020/01/google_charts_aspnet.png)

If you want convert the pie chart to bar chart, change the `var chart = new google.visualization.PieChart(document.getElementById('chart'));` to `var chart = new google.visualization.BarChart(document.getElementById('chart'));` it will become a bar chart.

Okay, but we are not using any C# code - how this is relevant in ASP.NET Core context. So in the above code we are creating a `datatable` object in the JavaScript code. If we can convert C# data structures into this `datatable` object, we will be able to display C# lists and data tables in Google Charts. There is code available in [Google Charts website](https://developers.google.com/chart/interactive/docs/php_example), where a PHP file used to serve data from JSON file.

So I converted that code to C# and which something like this.

{% highlight CSharp %}
public class Chart
{
    public object[] cols { get; set; }
    public object[] rows { get; set; }
}
{% endhighlight %}

And in the Razor pages code I wrote something like this.

{% highlight CSharp %}
public ActionResult OnGetChartData()
{
    var chart = new Chart
    {
        cols = new object[]
        {
            new { id = "topping", type = "string", label = "Topping" },
            new { id = "slices", type = "number", label = "Slices" }
        },
        rows = new object[]
        {
            new { c = new object[] { new { v = "Mushrooms" }, new { v = 3 } } },
            new { c = new object[] { new { v = "Onions" }, new { v = 1 } } },
            new { c = new object[] { new { v = "Olives" }, new { v = 1 } } },
            new { c = new object[] { new { v = "Zucchini" }, new { v = 1 } } },
            new { c = new object[] { new { v = "Pepperoni" }, new { v = 2 } } }
        }
    };

    return new JsonResult(chart);
}
{% endhighlight %}

And the above code is modified like this.

{% highlight Javascript %}
google.charts.load('current', {'packages':['corechart']});
google.charts.setOnLoadCallback(drawChart);
function drawChart() {
var jsonData = $.ajax({
    url: "/Index?handler=ChartData",
    dataType: "json",
    async: false
    }).responseText;
var data = new google.visualization.DataTable(jsonData);
var options = {'title':'How Much Pizza I Ate Last Night',
                'width':600,
                'height':500};

var chart = new google.visualization.PieChart(document.getElementById('chart'));
chart.draw(data, options);
}
{% endhighlight %}

Now if you run the app, you will be able to see the chart data is coming from Razor page code. But this solution is not scalable and maintainable. So lets use a C# library which converts the `List` and/or `DataTable` objects from C# to JavaScript `DataTable`, which is required for the Google Charts to render.

There is a `Google DataTable .NET Wrapper` available in nuget - we need to install the package. We can do this by running the command `dotnet add package Google.DataTable.Net.Wrapper --version 4.0.0`. Next lets modify the `OnGetChartData` method and instead of creating the JSON with code use the library.

{% highlight CSharp %}
public ActionResult OnGetChartData()
{
    var pizza = new[]
    {
        new {Name = "Mushrooms", Count = 3},
        new {Name = "Onions", Count = 1},
        new {Name = "Olives", Count = 1},
        new {Name = "Zucchini", Count = 1},
        new {Name = "Pepperoni", Count = 2}
    };

    var json = pizza.ToGoogleDataTable()
            .NewColumn(new Column(ColumnType.String, "Topping"), x => x.Name)
            .NewColumn(new Column(ColumnType.Number, "Slices"), x => x.Count)
            .Build()
            .GetJson();

    return Content(json);
}
{% endhighlight %}

Now you will be able to use the server side data to generate a Google charts compatible datatable. Please note in the code instead of JsonResult I am retuning `Content`.

You can use the same implementation in Web API and ASP.NET Core MVC - the change will be only in the client side consumer code. You can get more information about the .NET Wrapper in the [GitHub](https://github.com/zoranmax/GoogleDataTableLib) page

Happy Programming :)