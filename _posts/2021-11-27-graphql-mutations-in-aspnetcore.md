---
layout: post
title: "GraphQL Mutations in ASP.NET Core"
subtitle: "This post is about GraphQL in ASP.NET Core with EF Core. In this post we will explore how to implement mutations using ASP.NET Core."
date: 2021-11-27 00:00:00
categories: [AspNetCore,GraphQL,DotNet6,EFCore]
tags: [AspNetCore,GraphQL,DotNet6,EFCore]
author: "Anuraj"
image: /assets/images/2021/11/graphql_mutation.png
---
This post is about GraphQL in ASP.NET Core with EF Core. In this post we will explore how to implement mutations using ASP.NET Core. Mutations help you to update the data in the database. Here is the Mutation class. In this class, we are adding the `link` object to database using Entity Framework Core.

{% highlight CSharp %}
{% raw %}
public class Mutation
{
    [UseDbContext(typeof(BookmarkDbContext))]
    public async Task<LinkOutput> AddLink(LinkInput linkInput,
        [ScopedService] BookmarkDbContext bookmarkDbContext)
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
        return new LinkOutput(true, link.Id, link.Url,
            link.Title, link.Description, link.ImageUrl, link.CreatedOn);
    }
}
{% endraw %}
{% endhighlight %}

Next, we need to configure the Mutation class to the GraphQL pipeline like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddGraphQLServer()
    .AddQueryType<Query>()
    .AddProjections()
    .AddFiltering()
    .AddSorting()
    .AddMutationType<Mutation>();
{% endraw %}
{% endhighlight %}

You can use the following code to implement update operation as well.

{% highlight CSharp %}
{% raw %}
[UseDbContext(typeof(BookmarkDbContext))]
public async Task<LinkOutput> UpdateLink(int id, LinkInput linkInput, 
  [ScopedService] BookmarkDbContext bookmarkDbContext)
{
    var link = bookmarkDbContext.Links.Find(id);
    if (link != null)
    {
        if (link.Title != linkInput.Title)
        {
            link.Title = linkInput.Title;
        }
        if (link.Description != linkInput.Description)
        {
            link.Description = linkInput.Description;
        }
        if (link.ImageUrl != linkInput.ImageUrl)
        {
            link.ImageUrl = linkInput.ImageUrl;
        }
        if (link.Url != linkInput.Url)
        {
            link.Url = linkInput.Url;
        }

        await bookmarkDbContext.SaveChangesAsync();
        return new LinkOutput(true, link.Id, link.Url, link.Title, 
          link.Description, link.ImageUrl, link.CreatedOn);
    }

    return new LinkOutput(false, null, null, null, null, null, null);
}
{% endraw %}
{% endhighlight %}

And you can get the schema and know about the Mutation operations in GraphQL from Schema Reference.

![Graph QL endpoint - Mutation Schema]({{ site.url }}/assets/images/2021/11/graphql_mutation.png)

You can execute the mutation operation like this.

{% highlight Javascript %}
{% raw %}
mutation {
  addLink(
    linkInput: {
      url: "https://microsoft.com"
      title: "Microsoft – Cloud, Computers, Apps & Gaming"
      description: "Explore Microsoft products and services for your home or business."
    }
  ) {
    url
    title
    description
  }
}

{% endraw %}
{% endhighlight %}

Once it execute the operation and then it return the Url, title and description, like this.

![Graph QL endpoint - Mutation]({{ site.url }}/assets/images/2021/11/graphql_mutation_executed.png)

If you're not specifying the return object, GraphQL will show an error like this.

![Graph QL endpoint - Mutation Error]({{ site.url }}/assets/images/2021/11/graphql_mutation_error.png)

Mutation will help you to alters data, like inserting data into a database or altering data already in a database.

You can find the source code in [GitHub](https://github.com/anuraj/GraphQLDemo){:target="_blank"}

Happy Programming :)