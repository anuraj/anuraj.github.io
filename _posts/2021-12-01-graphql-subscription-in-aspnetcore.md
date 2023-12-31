---
layout: post
title: "GraphQL Subscriptions in ASP.NET Core"
subtitle: "This post is about implementing Subscriptions in GraphQL. Subscriptions are a GraphQL feature that allows a server to send data to its clients when a specific event happens. Subscriptions are usually implemented with WebSockets."
date: 2021-12-01 00:00:00
categories: [AspNetCore,GraphQL,DotNet6]
tags: [AspNetCore,GraphQL,DotNet6]
author: "Anuraj"
image: /assets/images/2021/12/subscription_schema.png
---
This post is about implementing Subscriptions in GraphQL. Subscriptions are a GraphQL feature that allows a server to send data to its clients when a specific event happens. Subscriptions are usually implemented with WebSockets. The subscription type is almost implemented like a query. Most cases subscriptions are raised through mutations, but subscriptions could also be raised through other backend systems.

First we need to implement a Subscription class, like this.

{% highlight CSharp %}
{% raw %}
public class Subscription
{
    [Subscribe]
    public Link OnCreateLink([EventMessage] Link link) => link;
}
{% endraw %}
{% endhighlight %}

The `Subscribe` attribute will make the method support subscription. And Subscription works using WebSockets, so we need to add WebSocket also to the pipeline. Also we need to modify the Subscription type to the GraphQL server with the subscription storage. In this example, I am using in memory storage, you can use Redis as well. Here is the updated code.

{% highlight CSharp %}
{% raw %}
builder.Services.AddGraphQLServer()
    .AddQueryType<Query>()
    .AddProjections()
    .AddFiltering()
    .AddSorting()
    .AddMutationType<Mutation>()
    .AddSubscriptionType<Subscription>()
    .AddInMemorySubscriptions();

var app = builder.Build();
app.UseWebSockets();
{% endraw %}
{% endhighlight %}

And you can trigger the event using the `ITopicEventSender` instance which will be injected to the pipeline. We can modify the Mutation class - `AddLink` method like this.

{% highlight CSharp %}
{% raw %}
public async Task<LinkOutput> AddLink(LinkInput linkInput,
    [ScopedService] BookmarkDbContext bookmarkDbContext, [Service] ITopicEventSender sender)
{
    if (string.IsNullOrEmpty(linkInput.Url))
    {
        throw new ArgumentNullException(nameof(linkInput.Url));
    }

    var link = new Link
    {
        Url = linkInput.Url,
        Title = linkInput.Title,
        Description = linkInput.Description,
        ImageUrl = linkInput.ImageUrl,
        CreatedOn = DateTime.UtcNow
    };
    bookmarkDbContext.Links.Add(link);
    await bookmarkDbContext.SaveChangesAsync();
    await sender.SendAsync(nameof(Subscription.OnCreateLink), link);
    return new LinkOutput(true, "Link created successfully", link.Id, link.Url,
        link.Title, link.Description, link.ImageUrl, link.CreatedOn);
}
{% endraw %}
{% endhighlight %}

Now we are ready to run the application. Since we are not discussed the client applications, we can use the Banana Cake Pop client app to test the subscription. Here is the Subscription schema details.

![Graph QL endpoint - Subscription Schema]({{ site.url }}/assets/images/2021/12/subscription_schema.png)

You can write following GraphQL subscription code in the operations tab.

{% highlight Javascript %}
{% raw %}
subscription {
  onCreateLink {
    id
    title
    description
  }
}
{% endraw %}
{% endhighlight %}

And then click on the Run button. It will wait for the event, button will change to cancel - showing a progress circle, like this.

![Graph QL endpoint - Subscription]({{ site.url }}/assets/images/2021/12/create_subscription.png)

You can test it using a new browser window - one for subscription - right side - Chrome and another for mutation - left side Edge. Execute the Subscription code by clicking on the Run button and then execute the mutation code. This will create a link and which will send the event using `ITopicEventSender` instance `SendAsync` method. Here is the screen capture.

![Graph QL endpoint - Subscription in Action]({{ site.url }}/assets/images/2021/12/subscription_in_action.gif)

The subscription type in GraphQL is used to add real-time capabilities to our applications. Clients can subscribe to events and receive the event data in real-time, as soon as the server publishes it.

You can find more details about Subscriptions in HotChocolate [here](https://chillicream.com/docs/hotchocolate/defining-a-schema/subscriptions){:target="_blank"}

You can find the source code in [GitHub](https://github.com/anuraj/GraphQLDemo){:target="_blank"}

Happy Programming :)