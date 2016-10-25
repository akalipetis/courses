## ADO.NET

or how to access data the *old* way


## ADO.NET
ADO.NET is an object-oriented set of libraries that allows you to interact with data sources. Commonly, the data source is a database, but it could also be a text file, an Excel spreadsheet, or an XML file.


## Data Providers
ADO.NET provides a relatively common way to interact with data sources, but comes in different sets of libraries for each way you can talk to a data source. These libraries are called Data Providers and are usually named for the protocol or data source type they allow you to interact with.


## SQL Data Provider
Since, we are interested in the SQL Server, we will use the .NET Framework Data Provider for SQL Server which resides in the `System.Data.SqlCient` namespace.
```csharp
using System.Data.SqlClient;
```


## Overview
Before jumping into the code, we will have to understand some of the important objects of ADO.NET. In a typical scenario requiring data access, we need to perform four major tasks:
1. Connecting to the database
2. Passing the request to the database, i.e., a command like select, insert, or update.
3. Getting back the results, i.e., rows and/or the number of rows effected.
4. Storing the result and displaying it to the user.


## Overview
This can be visualized as:

<img src="media/adonet-flow.jpg" width="1000">


## SqlConnection
The `SqlConnection` class is used to establish a connection to the database. The `SqlConnection` uses a `ConnectionString` to identify the database server location, authentication parameters, and other information to connect to the database.

http://www.connectionstrings.com/ is the website where you can easily find the connection string for your database. They provide the strings, for almost all of the database services and their types.


## Example ConnectionString - 1/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* `DataSource`: For SQL Server Express, LocalDB, SQL Server, and SQL Database, this setting specifies the name of the server and the SQL Server instance on the server. For example, you can specify ServerName\Instancename. You can use ".", "(local)", or "localhost" in place of the server name to specify the local computer, and you can use an IP address instead of the server name.
* `AttachDbFileName` specifies the path and name of the database file for SQL Server Express or LocalDB databases that are not defined in the local SQL Server Express instance.


## Example ConnectionString - 2/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* `InitialCatalog` specifies the name of the database in the SQL Server instance catalog. If omitted, ADO.NET connects to the default database for the SQL Server instance.
*  `Integrated Security` specifies whether the connection should use the user ID and password in the connection string to log on to the SQL Server instance (=false), or the current Windows account credentials should be used for authentication (=true).


## Example ConnectionString - 3/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* The `MultipleActiveResultSets` (MARS) option makes it possible to execute multiple queries simultaneously. This is a common scenario when you use the Entity Framework.


## SqlConnection Lifecycle
The purpose of creating a `SqlConnection` object is so you can enable other ADO.NET code to work with a database. Other ADO.NET objects, such as a `SqlCommand` and a `SqlDataAdapter` take a connection object as a parameter. The sequence of operations occurring in the lifetime of a `SqlConnection` are as follows:
1. Instantiate
2. Open the connection
3. Pass the connection to other ADO.NET objects
4. Perform database operations with the other ADO.NET objects
5. Close the connection


## Demo Code
```csharp
// 1. Instantiate the connection
SqlConnection conn = new SqlConnection(
  "Data Source=(local);Initial Catalog=Northwind;Integrated Security=SSPI");
SqlDataReader rdr = null;
try {
  // 2. Open the connection
  conn.Open();
  // 3. Pass the connection to a command object
  SqlCommand cmd = new SqlCommand("select * from Customers", conn);
  // 4. Use the connection to get query results
  rdr = cmd.ExecuteReader();
} finally {
  // close the reader
  if (rdr != null)
    rdr.Close();
  // 5. Close the connection
  if (conn != null)
    conn.Close();
}
```


## SqlCommand
A `SqlCommand` object allows you to specify what type of interaction you want to perform with a database. For example, you can do select, insert, modify, and delete commands on rows of data in a database table.


## SqlCommand
Similar to other C# objects, you instantiate a `SqlCommand` object via the new instance declaration, as follows:
```csharp
SqlCommand cmd = new SqlCommand("select CategoryName from Categories", conn);
```


## Quering data
When using a SQL `SELECT` command, you retrieve a data set for viewing. To accomplish this with a `SqlCommand` object, you would use the `ExecuteReader` method, which returns a `SqlDataReader` object. The example below shows how to use the `SqlCommand` object to obtain a `SqlDataReader` object:
```csharp
// 1. Instantiate a new command with a query and connection
 SqlCommand cmd = new SqlCommand("select CategoryName from Categories", conn);

// 2. Call Execute reader to get query results
 SqlDataReader rdr = cmd.ExecuteReader();
```


## Getting single values
Sometimes all we need from a database is a single value, which could be a count, sum, average, or other aggregated value from a data set. Performing an `ExecuteReader` and calculating the result in the code is not the most efficient way to do this. The best choice is to let the database perform the work and return just the single value we need. The following example shows how to do this with the `ExecuteScalar` method:
```csharp
// 1. Instantiate a new command
 SqlCommand cmd = new SqlCommand("select count(*) from Categories", conn);

 // 2. Call ExecuteNonQuery to send command
 int count = (int)cmd.ExecuteScalar();
 ```


## Inserting data
To insert data into a database, use the `ExecuteNonQuery` method of the `SqlCommand` object. The following code shows how to insert data into a database table:
```csharp
// prepare command string
string insertString = "insert into Categories (CategoryName, Description) values ('Miscellaneous', 'Whatever doesn''t fit elsewhere')";

// 1. Instantiate a new command with a query and connection
SqlCommand cmd = new SqlCommand(insertString, conn);

// 2. Call ExecuteNonQuery to send command
cmd.ExecuteNonQuery();
```


## Updating data
The ExecuteNonQuery method is also used for updating data. The following code shows how to update data:
```csharp
// prepare command string
 string updateString = @"
 update Categories
 set CategoryName = 'Other'
 where CategoryName = 'Miscellaneous'";

 // 1. Instantiate a new command with command text only
 SqlCommand cmd = new SqlCommand(updateString);

 // 2. Set the Connection property
 cmd.Connection = conn;

 // 3. Call ExecuteNonQuery to send command
 cmd.ExecuteNonQuery();
 ```


## Deleting data
You can also delete data using the `ExecuteNonQuery` method. The following example shows how to delete a record from a database:
```csharp
// prepare command string
 string deleteString = @"
 delete from Categories
 where CategoryName = 'Other'";

 // 1. Instantiate a new command
 SqlCommand cmd = new SqlCommand();

 // 2. Set the CommandText property
 cmd.CommandText = deleteString;

 // 3. Set the Connection property
 cmd.Connection = conn;

 // 4. Call ExecuteNonQuery to send command
 cmd.ExecuteNonQuery();
```


## Parameterizing the query