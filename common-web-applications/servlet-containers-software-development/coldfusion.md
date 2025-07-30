# ColdFusion

Coldfusion is a Java-based Programming Language and Web App development platform used to build dyanmic and interactive web applications that can be integrated with various API's and databases.

CMFL 9ColdFusion Markup Language) is the proprietary language used by Coldfusion with a similar syntax to HTML. It uses tags and functions within the CMFL reducing the amount of code needing to be written. For example cfquery can execute SQL satements:

```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfquery>
```

cfloop can loop through data that was retrieved via cfquery:

```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```
