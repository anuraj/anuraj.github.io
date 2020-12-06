---
layout: post
title: "Read Excel as DataTable using OpenXML and C#"
subtitle: "Read Excel as DataTable using OpenXML and C#"
date: 2014-07-11 08:11
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.Net MVC, C#, Import /Export Excel, Open XML, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In the current project we were using [OpenXML](http://msdn.microsoft.com/en-us/library/office/bb448854(v=office.15).aspx) extensively for reading Excel files. Here is the code snippet, which will help you to read / convert Excel files to DataTable. 

{% highlight CSharp %}
using System;
using System.Data;
using System.Linq;
using DocumentFormat.OpenXml.Packaging;
using DocumentFormat.OpenXml.Spreadsheet;

public static DataTable ReadAsDataTable(string fileName)
{
    DataTable dataTable = new DataTable();
    using (SpreadsheetDocument spreadSheetDocument = SpreadsheetDocument.Open(fileName, false))
    {
        WorkbookPart workbookPart = spreadSheetDocument.WorkbookPart;
        IEnumerable<Sheet> sheets = spreadSheetDocument.WorkbookPart.Workbook.GetFirstChild<Sheets>().Elements<Sheet>();
        string relationshipId = sheets.First().Id.Value;
        WorksheetPart worksheetPart = (WorksheetPart)spreadSheetDocument.WorkbookPart.GetPartById(relationshipId);
        Worksheet workSheet = worksheetPart.Worksheet;
        SheetData sheetData = workSheet.GetFirstChild<SheetData>();
        IEnumerable<Row> rows = sheetData.Descendants<Row>();

        foreach (Cell cell in rows.ElementAt(0))
        {
            dataTable.Columns.Add(GetCellValue(spreadSheetDocument, cell));
        }

        foreach (Row row in rows)
        {
            DataRow dataRow = dataTable.NewRow();
            for (int i = 0; i < row.Descendants<Cell>().Count(); i++)
            {
                dataRow[i] = GetCellValue(spreadSheetDocument, row.Descendants<Cell>().ElementAt(i));
            }

            dataTable.Rows.Add(dataRow);
        }

    }
    dataTable.Rows.RemoveAt(0);

    return dataTable;
}

private static string GetCellValue(SpreadsheetDocument document, Cell cell)
{
    SharedStringTablePart stringTablePart = document.WorkbookPart.SharedStringTablePart;
    string value = cell.CellValue.InnerXml;

    if (cell.DataType != null && cell.DataType.Value == CellValues.SharedString)
    {
        return stringTablePart.SharedStringTable.ChildElements[Int32.Parse(value)].InnerText;
    }
    else
    {
        return value;
    }
}
{% endhighlight %}

Happy Programming :)

**Update : Previous code snippet had some problem, now I fixed it.**
