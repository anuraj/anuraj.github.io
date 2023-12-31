---
layout: post
title: "How to pass parameters to Work Flow"
subtitle: "How to pass parameters to Work Flow"
date: 2014-07-02 21:35
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, WF, Windows Forms]
tags: [.Net, C#, WF, Windows Forms, WorkFlow]
header-img: "img/post-bg-01.jpg"
---
You can create arguments to workflow and can pass the values from outside. For creating arguments, you can use the Arguments Tab in the WorkFlow designer.

![WF Arguments Tab]({{ site.url }}/assets/images/2014/07/workflow_arguments.png)

In the last [post](http://www.dotnetthoughts.net/how-to-load-wf-workflows-dynamically/) I mentioned about loading and invoking a WorkFlow, the WorkFlowInvoker class's Invoke method has an overload, which accepts inputs parameter, which is type of IDictionary<string, object>. You can create an Dictionary<string, object> and pass it as the parameter to the Invoke method. 

You can get the results from the WorkFlow in similar fashion, it is also returning IDictionary<string, object>. Here is the code snippet, which provides one input parameter and accepts the results.

{% highlight CSharp %}
var inputParameters = new Dictionary<string, object>();
inputParameters.Add("CurrentDate", DateTime.Now.AddDays(-20));
var result = WorkflowInvoker.Invoke(activity, inputParameters);
object timeAgo = string.Empty;
if (result.TryGetValue("Result", out timeAgo))
{
    Console.WriteLine(timeAgo);
}
{% endhighlight %}

You can use any type as the argument type.

Happy Coding :)

