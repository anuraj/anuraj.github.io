---
layout: post
title: "Build a .NET AI vector search app"
subtitle: "In this blog post, we'll build a .NET console application that performs semantic search using SQL Server 2025 as a vector store. We'll also learn how to generate embeddings from user queries and leverage those embeddings to retrieve the most relevant results directly from SQL Server 2025."
date: 2025-12-12 00:00:00
categories: [dotnet,aspnetcore,sqlserver,rag,efcore]
tags: [dotnet,aspnetcore,sqlserver,rag,efcore]
author: "Anuraj"
image: /assets/images/2025/12/simple_chatbot.png
---

In this blog post, we'll build a .NET console application that performs semantic search using SQL Server 2025 as a vector store. We'll also learn how to generate embeddings from user queries and leverage those embeddings to retrieve the most relevant results directly from SQL Server 2025.

First we will create a Console application - I am using the following command `dotnet new console -o VectorDataAI`.

As I am using dependency injection and EF Core, I am adding following nuget packages.

```
dotnet add package Microsoft.Extensions.Hosting
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Then I am creating the Database context and Model classes. Here is the model class I am using.

```csharp
public class CloudService
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public string? Description { get; set; }
    [Column(TypeName = "vector(384)")]
    public SqlVector<float> Vector { get; set; }
}
```

And here is the DbContext.

```csharp
public class VectorDataAIDbContext : DbContext
{
    public VectorDataAIDbContext(DbContextOptions options) 
        : base(options)
    {
    }

    protected VectorDataAIDbContext()
    {
    }

    public DbSet<CloudService> CloudServices { get; set; }
}
```

Next we will wire up the dependency injection and configuration services, like this.

```csharp
var services = new ServiceCollection();

var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();

services.AddDbContext<VectorDataAIDbContext>(options =>
{
    var connectionString = configuration.GetConnectionString("VectorDataAIDbConnection");
    options.UseSqlServer(connectionString, sqlOptions =>
    {
        sqlOptions.EnableRetryOnFailure();
    });
});
```

Now we can seed the database. For seeding the database we need any embedding supported LLM. For this blog post, I am using Ollama. So I need to add `OllamaSharp` package. This data I took from [Build a .NET AI vector search app](https://learn.microsoft.com/en-us/dotnet/ai/quickstarts/build-vector-search-app?pivots=azure-openai&WT.mc_id=DT-MVP-5002040)

```csharp

var ollamaApiClient = new OllamaApiClient("http://localhost:11434", "all-minilm");
services.AddSingleton(ollamaApiClient);

var provider = services.BuildServiceProvider();

List<CloudService> cloudServices =
[
    new() {
            Name = "Azure App Service",
            Description = "Host .NET, Java, Node.js, and Python web applications and APIs in a fully managed Azure service. You only need to deploy your code to Azure. Azure takes care of all the infrastructure management like high availability, load balancing, and autoscaling."
    },
    new() {
            Name = "Azure Service Bus",
            Description = "A fully managed enterprise message broker supporting both point to point and publish-subscribe integrations. It's ideal for building decoupled applications, queue-based load leveling, or facilitating communication between microservices."
    },
    new() {
            Name = "Azure Blob Storage",
            Description = "Azure Blob Storage allows your applications to store and retrieve files in the cloud. Azure Storage is highly scalable to store massive amounts of data and data is stored redundantly to ensure high availability."
    },
    new() {
            Name = "Microsoft Entra ID",
            Description = "Manage user identities and control access to your apps, data, and resources."
    },
    new() {
            Name = "Azure Key Vault",
            Description = "Store and access application secrets like connection strings and API keys in an encrypted vault with restricted access to make sure your secrets and your application aren't compromised."
    },
    new() {
            Name = "Azure AI Search",
            Description = "Information retrieval at scale for traditional and conversational search applications, with security and options for AI enrichment and vectorization."
    }
];

var dbContext = provider.GetRequiredService<VectorDataAIDbContext>();
if (!dbContext.CloudServices.Any())
{
    var embeddingGenerator = provider.GetRequiredService<OllamaApiClient>();
    foreach (var service in cloudServices)
    {
        var embedding = await embeddingGenerator.EmbedAsync(service.Description!);
        service.Vector = new SqlVector<float>(embedding.Embeddings[0].ToArray());
        dbContext.CloudServices.Add(service);
    }
    await dbContext.SaveChangesAsync();
}

```

In the above code snippet, I am looping through the Cloud Services collection, generating the embeddings and insert them to the database - if no records exists in the database. And then we can  implement the search like this - like a general LINQ query.

```csharp

Console.WriteLine("Enter your question about Azure cloud services:");
var question =  Console.ReadLine();

var questionEmbedding = await ollamaApiClient.EmbedAsync(question!);
var questionVector = new SqlVector<float>(questionEmbedding.Embeddings[0].ToArray());
var topSimilarServices = dbContext.CloudServices
    .OrderBy(b => EF.Functions.VectorDistance("cosine", b.Vector, questionVector))
    .Take(2)
    .ToListAsync();

if (topSimilarServices.Result.Count == 0)
{
    Console.WriteLine("No similar cloud services found.");
}
else
{
    Console.WriteLine("Top similar cloud services:");
    foreach (var service in topSimilarServices.Result)
    {
        Console.WriteLine($"- {service.Name}: {service.Description}");
    }
}
```

Now we can enhance the application with Chat client. I am adding one more chat client for interacting with LLM. I am using Phi4 LLM.

```csharp
var ollamaApiEmbedClient = new OllamaApiClient("http://localhost:11434", "all-minilm");
services.AddKeyedSingleton("EmbeddingService", ollamaApiEmbedClient);
var ollamaApiChatClient = new OllamaApiClient("http://localhost:11434", "phi4-mini:latest");
services.AddKeyedSingleton("ChatService", ollamaApiChatClient);
```

Then I am using the following code to interact with LLM.

```csharp
Console.WriteLine("Enter your question about Azure cloud services:");
var question = Console.ReadLine();

var questionEmbedding = await ollamaApiEmbedClient.EmbedAsync(question!);
var questionVector = new SqlVector<float>(questionEmbedding.Embeddings[0].ToArray());
var topSimilarServices = dbContext.CloudServices
    .OrderBy(b => EF.Functions.VectorDistance("cosine", b.Vector, questionVector))
    .Take(2)
    .ToListAsync();

var chatPrompt = "You are an expert Azure cloud solutions architect. Based on the following Azure cloud services, answer the question asked by the user.";
if (topSimilarServices.Result.Count != 0)
{
    chatPrompt += "\nHere are some Azure cloud services that might be relevant:\n";
    foreach (var service in topSimilarServices.Result)
    {
        chatPrompt += $"Service Name: {service.Name}\nDescription: {service.Description}\n\n";
    }
}

chatPrompt += $"\nUser Question: {question}\nProvide a detailed answer with recommendations.";

var chatResponse = await ollamaApiChatClient.GetResponseAsync(chatPrompt);
Console.WriteLine("\nAnswer:\n");
Console.WriteLine(chatResponse.Text);
```

Here is the screenshot of the application running.

![Console chat application]({{ site.url }}/assets/images/2025/12/simple_chatbot.png)

This way we can implement a simple RAG application using Ollama and SQL Server 2025. In this application we will be using dependency injection in console application. We can use the similar code in ASP.NET Core API apps as well.

Happy Programming.