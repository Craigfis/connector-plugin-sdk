---
title: Create a Tableau Dialect Definition File 
---

A Tableau Dialect Definition file (.tdd) maps Tableau's query language to a database’s SQL. This is an XML file with a .tdd filename extension, and is one of the main components of a Tableau connector. 

You should create a dialect definition file whenever you need to make changes to an existing Tableau dialect or define an entirely new dialect. If your connector uses the same SQL dialect as the connector it’s based on, such as PostgreSQL, then a new  TDD file isn't necessary. 

## Create a TDD file 

To get started quickly, you can copy the sample dialect.tdd file from the [postgres_odbc or postgres_jdbc folder](https://github.com/tableau/connector-plugin-sdk/tree/master/samples/plugins) and use the copy to make your modifications. 

For an example of the XML schema for the TDD file format, see the [sample XSD file](https://github.com/tableau/connector-plugin-sdk/blob/master/validation/tdd_latest.xsd).

Tableau searches for a TDD file in the location specified by the connector manifest XML file. The TDD format breaks down as follows: 

### Dialect tag

The <span style="font-family: courier new">dialect</span> tag serves as the root for the document. For example:

```
<dialect
   name='CustomDialect'
   class='postgres'
   version='20.1'>
```

The <span style="font-family: courier new">dialect</span> tag has several required attributes and a couple of optional ones. 

Attribute | Required | Description 
-|-|- 
name | Y | Dialect name. Used for dependency lookup. 
class | Y | Should match the plug-in's class, as defined in the manifest. 
base | N | Specifies a base dialect to build upon. If a certain property or function isn't defined in a dialect definition file, the connector will fall back to its base dialect's behavior (assuming a value for <span style="font-family: courier new">base</span> is defined), and SQL-92 default behavior (if a value for <span style="font-family: courier new">base</span> is not defined).  **Important:** This must be a valid, existing dialect. If the specified base does not exist, the connector will fail to load. For a list of bases, see [Dialect base classes]({{ site.baseurl }}/docs/design#dialect-base-classes). 
dialect-version | N | Indicates the minimum database version applicable to the TDD file. For example, if you're adding a function definition that wasn't implemented until FooDB 3.0, then <span style="font-family: courier new">set dialect-version='3.0'</span>. 
version | Y | Must match the current Tableau version in the format YY.Q; for example, "20.1". 


### Supported-aggregations element

The <span style="font-family: courier new">supported-aggregations</span> element contains a list of aggregations and date truncation levels supported by the database, represented by one or more <span style="font-family: courier new">aggregation</span> elements. 

- __aggregation element__  
An <span style="font-family: courier new">aggregation</span> element has one required attribute, <span style="font-family: courier new">value</span>, which specifies a single aggregation or truncation. For a list, see [supported aggregations](https://github.com/tableau/connector-plugin-sdk/blob/master/samples/components/dialects/Annotated.tdd#L1051). 

### Function-map element

The <span style="font-family: courier new">function-map</span> element has no attributes and contains any number of <span style="font-family: courier new">function</span>, <span style="font-family: courier new">date-function</span>, and <span style="font-family: courier new">remove-function</span> elements. 

- __function element__   
Each <span style="font-family: courier new">function</span> elements defines a single function. It has a few required attributes, contain a <span style="font-family: courier new">formula</span> element (and in some cases, an <span style="font-family: courier new">unagg-formula</span> element) and any number of <span style="font-family: courier new">argument</span> elements. For a list of supported functions, see the [full_dialect.tdd](https://github.com/tableau/connector-plugin-sdk/blob/master/samples/components/dialects/full_dialect.tdd) file sample. 

 

Attribute | Required | Description 
-|-|-
name | Y | Indicates the name of the function being added or overridden. Remember, just because a function is present in a dialect doesn't mean that our calculated field expression parser will recognize it. 
group | Y | Contains one or more (comma-separated) groups that this function belongs to. Allowable types: aggregate, cast, date, logical, numeric, operator, passthru, special, string, system. 
return-type | Y | Indicates the return type of the function. For a list of allowable types, see [argument-element](#Argument) below. 

- __date-function element__  
The <span style="font-family: courier new">date-function</span> element is a specialized variant of <span style="font-family: courier new">function</span>. In addition to the base formula, you can specify one or more datepart formulas, which are used instead of the generic formula when available.  
The function <span style="font-family: courier new">name</span> must be one of these: DATEADD, DATEDIFF, DATEFORMAT, DATENAME, DATEPARSE, DATEPART, DATETRUNC.  
  
  Like <span style="font-family: courier new">function</span>, <span style="font-family: courier new">date-function</span> requires name and return-type, but unlike <span style="font-family: courier new">function</span>, group is not required. 

- __remove-function element__  
The <span style="font-family: courier new">remove-function</span> is used to remove existing functions in a function map without overriding them. It requires only a <span style="font-family: courier new">name</span> attribute and doesn't require you to specify any <span style="font-family: courier new">formula</span>. 

Each of the <span style="font-family: courier new">function</span>, <span style="font-family: courier new">date-function</span>, and <span style="font-family: courier new">remove-function</span> elements can contain the following:

- __formula element__    
The <span style="font-family: courier new">formula</span> element is a required child of <span style="font-family: courier new">function</span> and <span style="font-family: courier new">date-function</span> elements and specifies the function's formula using standard Tableau string substitution syntax for arguments (%1, %2, etc.). You can use the optional <span style="font-family: courier new">part</span> attribute to specify a formula for a specific date part. 

- __unagg-formula element__  
The <span style="font-family: courier new">unagg-formula</span> (unaggregated formula) element is an optional child of <span style="font-family: courier new">function</span> elements. It should be specified *only* if the function is part of the aggregate group. Unaggregated formulas should represent a reasonable way of expressing the aggregate of a single value. For example, for average, it should be the value itself. For variance, it should be 0. 

- __argument element__  
The <span style="font-family: courier new">argument</span> element is an optional child of all three types of <span style="font-family: courier new">function</span> elements. It contains a single attribute, <span style="font-family: courier new">type</span>, which specifies the abbreviated argument type. Arguments must be listed in the correct order.   
Allowable types: none, bool, real, int, str, datetime, date, localstr, null, error, any, tuple, spatial, localreal, localint.  
Allowable date parts: year, quarter, month, dayofyear, day, weekday, week, hour, minute, second. 
