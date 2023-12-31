---
layout: post
title: "Consuming a GraphQL API with Angular"
subtitle: "This post is about consuming a GraphQL API with Angular."
date: 2022-03-10 00:00:00
categories: [Azure,AspNetCore,Angular,GraphQL]
tags: [Azure,AspNetCore,Angular,GraphQL]
author: "Anuraj"
image: /assets/images/2022/03/angular_running.png
---
This post is about consuming a GraphQL API with an Angular application. For this post I am using a Azure Function with GraphQL - I already wrote a blog post - [Create Azure Functions with GraphQL Support](https://dotnetthoughts.net/create-azure-functions-with-graphql-support/).

I created the Azure function using the HotChocolate Templates. You can install the template using `dotnet new -i HotChocolate.Templates` command. And then create the function using `dotnet new graphql-azf` command. In the function code, I added the Query class and here is the whole function code.

Here is the `Startup.cs` class.

{% highlight CSharp %}
{% raw %}
[assembly: FunctionsStartup(typeof(Startup))]

public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        builder
            .AddGraphQLFunction()
            .AddQueryType<Query>();
    }
}

{% endraw %}
{% endhighlight %}

And the `Query.cs` class file.

{% highlight CSharp %}
{% raw %}
public class Query
{
    public IQueryable<Link> Links => new List<Link>
    {
        new Link
        {
            Id = 1,
            Url = "https://example.com",
            Title = "Example",
            Description = "This is an example link",
            ImageUrl = "https://example.com/image.png",
            Tags = new List<Tag> { new Tag(){ Name = "Example" } },
            CreatedOn = DateTime.Now
        }
    }.AsQueryable();
}
{% endraw %}
{% endhighlight %}

And finally here is the `GraphQLFunction` class.

{% highlight CSharp %}
{% raw %}
public class GraphQLFunction
{
    [FunctionName("GraphQLHttpFunction")]
    public Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = "graphql/{**slug}")] 
        HttpRequest request,
        [GraphQL] 
        IGraphQLRequestExecutor executor)
        => executor.ExecuteAsync(request);
}
{% endraw %}
{% endhighlight %}

Now we are ready with the Azure Function, we can run it using `func start --cors "*" --csharp` command. The CORS flag is required otherwise Angular will not able to access the GraphQL endpoint.

Next let us create an Angular Application. For demo purposes I am creating it like this - `ng new App --minimal --skip-tests`. This will create an Angular application with the name `App`. Next we need to add different packages which required to consume GraphQL endpoint, we are using a client package `Apollo Angular`. We can run this command to install the packages - `npm install apollo-angular @apollo/client graphql`. In their documentation they mentioned to install this using `ng add apollo-angular`, but due to some reason it didn't worked for me.

Next we need to modify the `tsconfig.json`, we need to include `es2020` inside `compilerOptions` if it not available. It is required because the `@apollo/client` package requires `AsyncIterable`. And next we need to modify the `app.module.ts` like this.

{% highlight Javascript %}
{% raw %}
import {HttpClientModule} from '@angular/common/http';
import {ApolloModule, APOLLO_OPTIONS} from 'apollo-angular';
import {HttpLink} from 'apollo-angular/http';
import {InMemoryCache} from '@apollo/client/core';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    ApolloModule,
    HttpClientModule
  ],
  providers: [
    {
      provide: APOLLO_OPTIONS,
      useFactory: (httpLink: HttpLink) => {
        return {
          cache: new InMemoryCache(),
          link: httpLink.create({
            uri: 'http://localhost:7071/api/graphql/',
          }),
        };
      },
      deps: [HttpLink],
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endraw %}
{% endhighlight %}

In the `httpLink.create` method, we need to configure the GraphQL endpoint - our Azure Function endpoint. Now we can use the Apollo service in the angular components. In this post I am querying the graphql endpoint. In the `app.component.ts`, we can write code like this.

{% highlight Javascript %}
{% raw %}
export class AppComponent implements OnInit {
links: any[] = [];
isLoading = true;
isError: any;
constructor(private apollo: Apollo) {}
ngOnInit(): void {
  this.apollo
  .watchQuery({
    query: gql`
      {
        links {
          url
          title
        }
      }
    `,
  })
  .valueChanges.subscribe((result: any) => {
    this.links = result?.data?.links;
    this.isLoading = result.loading;
    this.isError = result.error;
  });
}
{% endraw %}
{% endhighlight %}

In this code, the apollo service is injected via constructor and consumed in the `ngOnInit`. We can use the `gql` object to create queries. Both `Apollo` and `gql` classes are part of `apollo-angular` module which should be included in the code. And the data can be displayed in the UI using the `ngFor`. Here is an example.

{% highlight Javascript %}
{% raw %}
@Component({
  selector: 'app-root',
  template: `
    <div *ngIf="isLoading">
      Loading...
    </div>
    <div *ngIf="isError">
      Error :( Something went wrong.
    </div>
    <div *ngIf="links">
      <table border="1">
        <thead>
          <tr>
            <th>Title</th>
            <th>URL</th>
        </thead>
        <tbody>
          <tr *ngFor="let link of links">
            <td>{{ link.title }}</td><td>{{ link.url }}</td>
          </tr>
      </table>
    </div>
    <router-outlet></router-outlet>
  `,
  styles: []
})
{% endraw %}
{% endhighlight %}

This way GraphQL can be consumed in Angular. Other GraphQL operations can be implemented similar way. Subscriptions maintain a persistent connection, they can't use the default HTTP transport that Apollo Client uses for queries and mutations. Instead, Apollo Client subscriptions most commonly communicate over WebSocket, via the community-maintained `subscriptions-transport-ws` library.

Happy Programming :)