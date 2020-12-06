---
layout: post
title: "Generate your database entities using T4 templates"
subtitle: "Generate your database entities using T4 templates"
date: 2014-06-15 00:25
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, EF Code First, Entity Framework, SQL Server, WPF]
tags: [Code Generation, Database Entities, SQL Server, T4 Templates]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote some post about [code generation using T4 templates](http://www.dotnetthoughts.net/code-generation-using-t4-templates/). This post is about generating database entities using T4 templates. This code is pretty straight forward, you are reading the app.config to get the connection string, connecting to db server using Sql Server SMO objects, enumerating tables and columns.

Here is the code, which will read the App.Config file and get the connection string. (I am using a connection string with name, Default).

{% highlight CSharp %}
string appConfig = this.Host.ResolvePath("App.config");
if(!File.Exists(appConfig))
{
	Error("App.config not exists");
}

ExeConfigurationFileMap configFile = new ExeConfigurationFileMap();
configFile.ExeConfigFilename = appConfig;
var configuration = ConfigurationManager.OpenMappedExeConfiguration(configFile, ConfigurationUserLevel.None);
var connectionString = configuration.ConnectionStrings.ConnectionStrings["Default"].ConnectionString;
{% endhighlight %}

Here is the code which will connects to the database, enumerate tables and columns and generates classes and properties.

{% highlight CSharp %}
SqlConnection sqlConnection = new SqlConnection(connectionString);
ServerConnection serverConnection = new ServerConnection(sqlConnection);
Server server = new Server(serverConnection);
var tables = server.Databases[server.ConnectionContext.DatabaseName].Tables;
#>
namespace <#= server.ConnectionContext.DatabaseName #>.Entities
{
<#
foreach(Table table in tables)
{
#>
public sealed partial class <#= CleanName(table.Name) #>
{
<#
foreach(Column column in table.Columns)
{
#>
	public <#= ToClrType(column.DataType, column.Nullable) #> <#= CleanName(column.Name) #> { get; set; }
<#
}
#>
}
<#
}
#>
}
{% endhighlight %}

And here is the two functions which will fix any issue with table name and column names. And returns C# type by using a SQL Server type.

{% highlight CSharp %}
private string FixTableName(string tableName)
{
	var result = tableName.Replace(" ","_");
	return System.Globalization.CultureInfo.CurrentCulture.TextInfo.ToTitleCase(result);
}

private string ToClrType(DataType dataType, bool isNullable)
{
	string returnType = string.Empty;
	switch (dataType.SqlDataType)
    {
        case SqlDataType.BigInt:
            returnType = string.Format("{0}{1}","long", isNullable ? "?":"");
			break;
        case SqlDataType.Binary:
        case SqlDataType.Image:
        case SqlDataType.Timestamp:
        case SqlDataType.VarBinary:
            returnType = "byte[]";
			break;
        case SqlDataType.Bit:
			returnType = string.Format("{0}{1}","bool", isNullable ? "?":"");
			break;
        case SqlDataType.Char:
        case SqlDataType.NChar:
        case SqlDataType.NText:
        case SqlDataType.NVarChar:
        case SqlDataType.Text:
        case SqlDataType.VarChar:
        case SqlDataType.Xml:
            returnType = string.Format("{0}{1}","string", "");
			break;
        case SqlDataType.DateTime:
        case SqlDataType.SmallDateTime:
        case SqlDataType.Date:
        case SqlDataType.Time:
        case SqlDataType.DateTime2:
			returnType = string.Format("{0}{1}","System.DateTime", isNullable ? "?":"");
			break;
        case SqlDataType.Decimal:
        case SqlDataType.Money:
        case SqlDataType.SmallMoney:
			returnType = string.Format("{0}{1}","decimal", isNullable ? "?":"");
			break;
        case SqlDataType.Float:
			returnType = string.Format("{0}{1}","double", isNullable ? "?":"");
			break;
        case SqlDataType.Int:
			returnType = string.Format("{0}{1}","int", isNullable ? "?":"");
			break;
        case SqlDataType.Real:
			returnType = string.Format("{0}{1}","float", isNullable ? "?":"");
			break;
        case SqlDataType.UniqueIdentifier:
			returnType = string.Format("{0}{1}","Guid", isNullable ? "?":"");
			break;
        case SqlDataType.SmallInt:
            returnType = string.Format("{0}{1}","short", isNullable ? "?":"");
			break;
        case SqlDataType.TinyInt:
            returnType = string.Format("{0}{1}","byte", isNullable ? "?":"");
			break;
        case SqlDataType.Variant:
            returnType = string.Format("{0}{1}","object", "");
			break;
        case SqlDataType.DateTimeOffset:
            returnType = string.Format("{0}{1}","DateTimeOffset", isNullable ? "?":"");
			break;
		}

		return returnType;
}
{% endhighlight %}

Happy Coding :)

You can find the Gist [here](https://gist.github.com/anuraj/83ec6902304b749fb6eb)
