---
layout: post
title: "Creating a Sales Copilot - Working with PDF files"
subtitle: "In this blog post, we will learn about creating a Sales Copilot - in this part we will learn about the issue with the existing PDF MCP Reader and how to implement different PDF reader to work with PDF files."
date: 2026-03-10 00:00:00
categories: [dotnet,AI]
tags: [dotnet,AI]
author: "Anuraj"
image: /assets/images/2026/03/aspire_dashboard_log_ingest
---

In this blog post, we will learn about creating a Sales Copilot - in this part we will learn about the issue with the existing PDF MCP Reader and how to implement different PDF reader to work with PDF files. In the existing project, since it is powered by Aspire, dotnet team introduced an MCP Server to read the PDF files. Unfortunately when working with large PDF files, we will get an issue like this - `System.Net.Http.HttpRequestException: Response status code does not indicate success: 413 (Content Too Large)`.

![.NET Aspire Dashboard - PDF Ingestion failed]({{ site.url }}/assets/images/2026/03/aspire_dashboard_error_log2.png)

I didn't explored this issue much, I simply replaced the existing PDF reader which comes with same template - not Aspire version.

Here is the code. 

```csharp
internal sealed class PdfPigReader : IngestionDocumentReader
{
    public override Task<IngestionDocument> ReadAsync(Stream source, string identifier, 
        string mediaType, CancellationToken cancellationToken = default)
    {
        using var pdf = PdfDocument.Open(source);
        var document = new IngestionDocument(identifier);
        foreach (var page in pdf.GetPages())
        {
            document.Sections.Add(GetPageSection(page));
        }
        return Task.FromResult(document);
    }

    private static IngestionDocumentSection GetPageSection(Page pdfPage)
    {
        var section = new IngestionDocumentSection
        {
            PageNumber = pdfPage.Number,
        };

        var letters = pdfPage.Letters;
        var words = NearestNeighbourWordExtractor.Instance.GetWords(letters);

        foreach (var textBlock in DocstrumBoundingBoxes.Instance.GetBlocks(words))
        {
            section.Elements.Add(new IngestionDocumentParagraph(textBlock.Text)
            {
                Text = textBlock.Text
            });
        }

        return section;
    }
}

```

This code is similar to our Word document reader or Powerpoint document reader. It uses a nuget package `PdfPig`. We need to add the reference of this package to the `SalesCopilot.Web` project. Now if I need to modify the `DocumentReader` class and start using the `PdfPigReader` instead of the MCP server.

The updated `DocumentReader` class will look like this.

```csharp
internal sealed class DocumentReader(DirectoryInfo rootDirectory) : IngestionDocumentReader
{
    private readonly MarkdownReader _markdownReader = new();
    private readonly PdfPigReader _pdfReader = new();
    private readonly WordReader _wordReader = new();
    private readonly PowerPointReader _powerPointReader = new();
    public override Task<IngestionDocument> ReadAsync(Stream source, string identifier,
        string mediaType, CancellationToken cancellationToken = default)
    => mediaType switch
    {
        "application/pdf" => _pdfReader.ReadAsync(source, identifier, mediaType, cancellationToken),
        "text/markdown" => _markdownReader.ReadAsync(source, identifier, mediaType, cancellationToken),
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document" =>
            _wordReader.ReadAsync(source, identifier, mediaType, cancellationToken),
        "application/vnd.openxmlformats-officedocument.presentationml.presentation" =>
            _powerPointReader.ReadAsync(source, identifier, mediaType, cancellationToken),
        _ => throw new InvalidOperationException($"Unsupported media type '{mediaType}'"),
    };
    public override Task<IngestionDocument> ReadAsync(FileInfo source, string identifier, string? mediaType = null, CancellationToken cancellationToken = default)
    {
        if (Path.IsPathFullyQualified(identifier))
        {
            // Normalize the identifier to its relative path
            identifier = Path.GetRelativePath(rootDirectory.FullName, identifier);
        }

        mediaType = GetCustomMediaType(source) ?? mediaType;
        return base.ReadAsync(source, identifier, mediaType, cancellationToken);
    }

    private static string? GetCustomMediaType(FileInfo source)
        => source.Extension switch
        {
            ".md" => "text/markdown",
            _ => null
        };
}
```
Now if we run the app again, we will be able to see the PDF file ingesting without any issues. We can remove the reference of the MCP server from the AppHost file as well. 

Here is the screenshot of Aspire Logs - the PDF file Ingestion completed.

![.NET Aspire Dashboard - PDF Ingestion successful]({{ site.url }}/assets/images/2026/03/aspire_dashboard_pdf_ingest.png)

This way we can improve the application to support big PDF files.

Happy Programming