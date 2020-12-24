BizAPP Structured Query Language
=========================

# Introduction

BizAPP Structured Query Language provides users with more options to simplify the practice of writing BizAPP specific sql statements using the familiar sql syntax. It provides options to use complex queries for business applications while improving the productivity.

## Query Object

It provided a richer API set to manage and create queries on the fly. With a simple xml representation, even the novice users that do not have knowledge on the sql specifics could create and use it with ease. For advanced users, it lacked configurability to generate more complex queries.

## Raw SQL

There was another option already available for the advanced users. But maintenance and its use in variety of scenarios such as paginated output and security features were left to the domain of authors of the queries. Hence if an application were to be multi-tenant enabled, it required significant effort from the developers/authors to ensure it complied with the tenancy requirements.

BSQL was created with an intention to address and alleviate the above scenarios while retaining the flavor of ubiquitous sql syntax.

# BSQL Queries

Any BSQL query would typically have a simple select statement followed by tables and optional joins, where conditions, group by clause and order by clauses in addition to the options applicable as defined later in the document.

SELECT 
	
	Select\_List
    FROM Table\_List
	[WHERE Conditions]
	(PIVOT|UNPIVOT Condition) | ([GROUP BY Column\_List])
	[UNION Clause]
	[HAVING Conditions]
	[ORDER BY Column\_List]
	[Additional Options]

## Select Clause

Select clause specifies the fields, constants, other sub queries and functions to be used in the select column list. It also supports specifying a set qualifier such as ALL or DISTINCT.

SELECT [ALL | DISTINCT] Select\_List\_Item [AS Column\_Name] [, ...]

The following examples shows the use of select list with some columns defined with aliases.

Select firstname as firstname, lastname as &quot;last name&quot; from SystemUnit.SystemModule.User

Select [u].firstname, [u].lastname from SystemUnit.SystemModule.User [u]

Table alias is enclosed within &#39;[&#39; and &#39;]&#39; characters to make sure any column with multiple dotted fields is isolated by the table alias by using the required format.

### Column Aliases

- Column name and aliases can optionally be enclosed within double quotes or reverse quotes. This would be useful when the column or alias names contain one or more spaces or other reserved keywords.
- Use of optional AS clause is allowed as a prefix for alias names.

### Field Options/Constraints

There are 2 types of options or constraints available for every column.

- If a field uses dotted notation, any part of the field could be marked as nullable. The keyword &quot;nullable&quot; follows the sign colon (:) For e.g. [u].PreferredLocale : nullable .DisplayName
- If a field&#39;s type either is abstract or a global type, it can be forced to use a user defined type. This is represented with a # followed by the type id of the object. For e.g. [u].PreferredLocale : #UID\_Locale ; nullable .Displayname
- If the field has allowed values, it can indicate to return a value, displayvalue or formatted value as shown below. If the constraint is omitted, it defaults to display value.

  [u].orgtype : value, [u].orgtype : displayvalue

> If both the options were to be used on a column, it can be separated using a semi colon character.

### Functions

BizAPP provides built-in functions which is compatible with the other databases supported. It also supports using database specific functions in the query.

The built-in functions are categorized as follows.

#### String functions

Deterministic functions return the same value for the same input values.

CHR

CHAR

CONCAT

LOWER

UPPER

LTRIM

RTRIM

TRIM

INSTR

SUBSTR

REPLACE

NVL

ASCII

#### Datetime functions

All datetime functions, except the ones that return the system datetime, are automatically converted to session timezone with the timezone conversion required. The same is enforced when these functions are used in the filters. The functions used in the filters are automatically converted to UTC timezone using the appropriate conversions.

Functions that return the system specific datetime values.

CURRENT\_TIMESTAMP

CURRENT\_DATE

Functions that operate on the datetime values and transform it.

DAY

WEEK

MONTH

YEAR

GETDATE

WEEKDAY

DAYOFYEAR

SECONDS\_BETWEEN

MINUTES\_BETWEEN

HOURS\_BETWEEN

DATEPART

DAYS\_BETWEEN

WEEKS\_BETWEEN

MONTHS\_BETWEEN

YEARS\_BETWEEN

#### Mathematical functions

The following mathematical functions are supported by the query.

ABS

EXP

ACOS

ASIN

ATAN

CEIL

COS

TAN

LOG

FLOOR

MOD

POWER

ROUND

SQRT

#### Conversion functions

Conversion functions such as CAST that works across the databases, but the cast target type is usually different. BizAPP provides a generic implementation of CAST function as follows using the BizAPP specific field data types. It is automatically converted to the target database upon execution.

Cast( cast\_operand AS cast\_target )

The supported cast target data types are

STRING ([length | max]), INT, SMALLINT, BIGINT, DATETIME, REAL, DECIMAL ([precision], [scale])

Max is the keyword and should be specified only for string data type.

## From Clause

From clause defines one or more tables that participate in the query. The table can either be a fully qualified name, inline view or a derived table.

FROM

	 Table\_List\_Item [, ...]
	 [[JoinType] JOIN] Table [[AS] Local\_Alias]
	 [ON JoinCondition [AND | OR [JoinCondition | FilterCondition] ...]

### Table Expression

#### Table Name

One or more tables can be specified to form a part of a table expression. If more than one table is used, it must use valid aliases to avoid any collision in the field conversions. Following are the valid table names

- Type Id of an object. For e.g. UID\_User as u.
- Fully qualified name of the object. For e.g. SystemUnit.SystemModule.User u

Table aliases can be specified with or without &quot;as&quot; statement.

#### Joins

Join can be introduced followed by one or more table name(s). Any user introduced joins are retained as part of the query conversion and additional joins are included next to the user defined join statements.

A join table could be another table expression such as a joined table or a derived table. Another select query could be used as a derived table and joined together with other tables.

#### Filters

An optional filter/where clause can be specified followed by the table expression. Even when there is no filter specified, the converted query will have additional filters depending on the options such as showing deleted objects, showing application bound objects and so on.

The following example demonstrates the use of from clause using either table name or inline view.

Select firstname, &quot;lastname&quot; from systemunit.systemmodule.user

Select \* from ( select firstname, lastname from systemunit.systemmodule.user ) [t] where [t].lastname is not null

## Where Clause

The WHERE clause specifies join and filter conditions that determine the rows that the query returns. Join operations in the WHERE clause function the same as JOIN operations in the FROM clause.

The syntax for where clause is as follows.

[WHERE JoinCondition | FilterCondition [AND | OR JoinCondition | FilterCondition] ...]

### Like Expression

Users have total control over how a like expression should be used. The converted query does not append &#39;%&#39; character neither as prefix nor as suffix. If the value is expected to evaluated, then it can be written using string concatenation operators as &#39;%&#39; || %DisplayName% || &#39;%&#39;

If the value is a simple string literal, &#39;%&#39; character can either be prefixed or suffixed. In addition, if any characters are escaped in the string literal, it must be specified using ESCAPE option with an escape character.

Firstname like &#39;%namewith\%percentage%&#39; escape &#39;\&#39;

### Boolean Values

Supported Boolean values are

- TRUE
- FALSE

All Boolean expressions can in either select list or conditional expressions need to use the either of the two states of Boolean values.

## Pivot/UnPivot Clause

Pivot and UnPivot operators can be used to turn a table expression into another table. Pivot rotates the unique values from one column into rows while unpivot turns unique column values into row based column values.

When Pivot/UnPivot is used, group by expression is ignored even when specified. These operators internally performs group by based on the unique column values that is being pivoted or unpivoted.

The syntax for Pivot/UnPivot is as follows.

Select [Non pivoted column], [first pivoted column],...

From (SELECT query table expression) as [alias]

PIVOT/UNPIVOT

(

[aggregate function] FOR [column that will become headers]

IN ([first pivoted column],[second pivoted column], ...)

) as [alias]

Aggregate functions such as SUM, COUNT, AVG, MAX and MIN can be used in Pivot clause. The pivoted columns is a static list of columns that will become headers. **It does not support a dynamic column headers using another select query.** Aggregate functions can only be a simple function without using window function primitives.

The select list can either include columns explicitly or \*(asterisk) to let the database transform the non-pivoted and pivoted columns into select list automatically.

The column that will become headers can use names with or without table aliases. By default Oracle allows only simple named columns and hence BSQL parser will ignore those table aliases for aggregate function columns and header columns when transformed to database specific query.

## Group by Clause

The group by clause specifies one or more columns used to group rows returned by the query. Columns referenced in the select list excluding aggregate functions must be included in the group by clause.

Having clause specifies the conditions that determines the groups included in the query. Any aggregate functions and other conditions can be included in the having clause. If the having clause is omitted, it will act like a where clause.

*[GROUP BY Column\_List\_Item [, ...] ] HAVING FilterCondition [AND | OR ...]*

## Order by Clause

The ORDER BY clause specifies one or more items used to sort the final query result set and the order for sorting the results.

The syntax for order by clause is as follows.

*[ORDER BY Order\_Item [ASC | DESC] [, ...]]*

Order by expressions can be used with numeric values to indicate the position of the column in the select list as shown below.

Order by 1 asc, 2 desc

## Query Set Expressions

Multiple queries can be used together with a set operator. The valid set operators are

- Union
- Union All
- Intersect

Any nested level of queries can be used with a set operator.

## BizAPP Expressions

BizAPP supports values to be evaluated, computed or both on the expressions. The following evaluated constructs are supported. Evaluated values SHOULD NOT be enclosed within single quotes as the query parser will identify them as literal strings instead of expressions. BSQL Parser understands the expressions and converts them into correct query parameters with required data types based on the returned values of the expressions.

- %[fieldname[,contextobject index][,valuetype]]%. Supported value type literals are value, displayvalue and formattedvalue.
- \&lt;[rulename]:[parametername]\&gt;
- @[session variable]@
- @@[global session variable]@@
- $[expression]$[&quot;fieldname&quot;]

The above supported expressions can be included in the computed expressions using **compute([evaluated expression])** as shown below.

**Compute(Parse(&quot;$today$&quot;))**

### Return Value Data Types

Evaluated and computed expressions are always returned as string values. If the returned value is required to be of a specific .NET type, it can be enforced by including the target data type for an evaluated or computed value. The following are the supported data types recognized by the query parser.

- STRING
- INT | INTEGER
- FLOAT
- DOUBLE
- DECIMAL
- DATETIME
- BOOLEAN | BOOL

Return type for an expression is indicated by a colon (&#39;:&#39;) operator or double colon (&#39;::&#39;) followed by the expression. The below examples show the use of enforcing return data type for a computed and evaluated values.

Compute(Parse(&quot;%createdon%&quot;)) : datetime

%createdon% :: datetime

## Element Types for Select Fields

At times, some of the fields included in the select list need not be displayed in the User Interface. But these field values need to be returned as part of the query results in order for the other hidden markup to use those fields. To enable this facility and to provide backward compatibility with the legacy query model, the following keywords are supported by BSQL.

- Hide
- Invisible

A select field can be marked with one of the keywords followed by the field as shown below.

Firstname, uniqueid as uid -\&gt; hide, enterpriseid -\&gt; invisible, …

This makes the uniqueid field with alias uid hidden from the user interface. The results will have the element type for the field set as hidden.

## Options

Some of the query options can either be specified as part of the BSQL object or in line on the sql query. It is specified by the keyword &quot;options&quot; followed by one or more of the below values separated by a semi colon. The valid query options are

- IncludeDeletedObjects - Returns the results with deleted objects included. The default is false.

- ReturnAppBoundObjectsOnly - Returns the current application specific records only. This is enforced only EnableApplicationBinding is set to true.

- EnableApplicationBinding
- IncludeDefaultFields - The query results includes default fields such as uniqueid, objecttype, version and displayname. Default value is true.

- Nullable - Shortcut to mark all relationship fields used in the query as nullable. This makes the query to use left outer joins instead of default inner joins.

- CultureSensitive - Uses the resource items for the fields.

- ReturnAllowedValues - Instructs that the query should use allowed values for both select fields and fields used in the filters.

- ReturnFormattedNumbers - Instructs the query to return formatted numbers for the user culture. The formatted numbers are accepted in the filters too.

- InsertVersionRecord - Instructs that a version record should be inserted when the object type is enabled for versioning.

- ConnectionType - Any string identifier that matches with a predefined connection in the enterprise configuration. For e.g if there is a section named &quot;audit&quot; in the config.xml file that points to a different db, 
	bsql can be run on that specific db connection by using this option. 
	For e.g.Connectiontype=&#39;audit&#39;

> Multiple options can either be separated by a semi colon or by a comma.

*[OPTIONS nullable = true ; includedefaultfields = false ; includedeletedobjects = false [;…]]*

## Nested Queries

Queries can be nested to any level as limited by the db provider. This provides an opportunity to have several inline views that work together to bring out the desired result. For e.g

Select [t].loginid, [t].firstname || [t].lastname as fullname from (

Select [u].loginid, [u].firstname, [u].lastname from systemunit.systemmodule.user as [u] where [u].active= true ) [t]

## External Queries

BSQL can also be marked as external query to indicate that the query should be executed on the external data source after the query is transformed by the parser. The following options are disabled or not honored by the query parser when it encounters external queries.

- Include default fields.
- Reference and additional queries.

## Allowed Values

Query results use allowed values from the corresponding the field to transform the value into an allowed display value. But the same is not applied to the filter fields. Hence users are expected to use the value instead of display value in the filters.

## Datetime Formats

Formatting of datetime values is though a trivial task, but complicated by multi database support for queries. BSQL provides a wrapper function FORMATDATE that supports pre-defined list of formats that can be used in the query. Any other unsupported format can be accomplished by using a database specific functions.

FORMATDATE( \&lt;datetime\&gt;,\&lt;format\&gt; )

The following are the supported formats.

&#39;MM/DD/YY&#39;

&#39;MM/DD/YYYY&#39;

&#39;MM-DD-YY&#39;

&#39;MM-DD-YYYY&#39;

&#39;DD/MM/YY&#39;

&#39;DD/MM/YYYY&#39;

&#39;DD-MM-YY&#39;

&#39;DD-MM-YYYY&#39;

&#39;YY/MM/DD&#39;

&#39;YYYY/MM/DD&#39;

&#39;YY-MM-DD&#39;

&#39;YYYY-MM-DD&#39;

&#39;YYYY-MM-DD HH: MI: SS&#39;

&#39;DD Mon YY&#39;

&#39;DD Mon YYYY&#39;

&#39;Mon DD, YY&#39;

&#39;Mon DD, YYYY&#39;

&#39;DD Mon YYYY HH: MI: SS&#39;

## Pagination and Count Queries

Query results are paginated either when the result set is large or to be shown by the applications for users to interact. Databases such as SQL SERVER and Oracle have custom rules that need to be enforced while generating paginated queries. BizAPP takes care of these requirements and relieve the users of using complex queries to paginate the results.

BSQL query is wrapped by another object model that can be represented in an XML format. The object model allows the users to set the page number and page size required for pagination. Similarly, the query can be marked to return count instead of normal query results.

Both the above options do not require any changes to existing queries. Instead, BizAPP BSQL query post processor alters the behavior of the queries in such a way that it conforms to the requirements of the underlying database.

The following rules are applied when generating pagination and count queries.

- If the outer most query has an Order by clause, it is preserved. Any Order by clause in other nested inline views are ignored.
- Adds Top Condition for Sql Server specific queries.
- Order by clause is removed for all count queries.
- When Union or Intersect queries don&#39;t have an outer most wrapper query, it is automatically added for pagination.

## Using BSQL to query against Raw Tables and Columns

BSQL abstracts the table and columns names to hide the complexity of names and other joins from users, it also allows users to have unhindered access to the underlying data sources to query where it is required either for external integrations or to query the system tables that otherwise cannot be accessed through metadata.

Dbtable is the keyword that follows the table or the view using the colon operator. For e.g.

ent\_objectproperties : dbtable

Dbcolumn is another keyword that is used to indicate a db column from a metadata type. If a column is not part of the metadata, but needs to be accessed either in the select list or conditional expressions, this keyword can be used. For e.g.

Select hash : dbcolumn, firstname, lastname from uid\_user

## Unsupported Features

The following features are unsupported considering the fact that users have more options to use nested queries and it can be accomplished without it.

- Format String
- Enable Expressions on filters