---
layout: post
title: "Converting DataTable to Generic List<T>"
subtitle: "Converting DataTable to Generic List<T>"
date: 2015-02-09 00:14
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, Windows Forms]
tags: [.Net, .Net 4.0, ASP.Net, C#, C#.Net, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Code snippet for converting a DataTable to Generic List<T>.

{% highlight CSharp %}
public static List<TEntity> ToList<TEntity>(this DataTable dataTable,
    Dictionary<string, string> mapping) where TEntity : new()
{
    var list = new List<TEntity>();
    foreach (DataRow dataRow in dataTable.Rows)
    {
        var entity = new TEntity();
        foreach (var keyvaluePair in mapping)
        {
            var property = entity.GetType().GetProperty(keyvaluePair.Key,
                            BindingFlags.IgnoreCase | BindingFlags.Public | BindingFlags.Instance);
            if (dataRow[keyvaluePair.Value] != DBNull.Value)
            {
                var propertyType = 
                    Nullable.GetUnderlyingType(property.PropertyType) ?? property.PropertyType;
                var value = 
                    Convert.ChangeType(dataRow[keyvaluePair.Value], propertyType);
                property.SetValue(entity, value, null);
            }
        }

        list.Add(entity);
    }

    return list;
}
{% endhighlight %}

You can use this as an extension method to the Data Table class. By providing the mapping dictionary, the consumer class can decide which all columns / properties need to participate in the conversion. 

Happy Programming :)
