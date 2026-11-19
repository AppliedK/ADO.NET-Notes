# ASP.NET Chapter 01 What is ADO.NET
## What is ADO.NET?
ADO stands for ActiveX Data Object.
NET stands for Network Enabled Technology
ADO.NET is a Microsoft Framework to interact with different Data sources.
ADO.NET is a set of classes such as 
- SqlConnection
- SqlCommand
- SqlDataAdapter
- SqlDataReader
- OracleConnection
- OracleCommand
- OracleDataAdapter
- OracleDataReader
- DataSet

The following are tied with SQL Server data provider.
- SqlConnection
- SqlCommand
- SqlDataAdapter
- SqlDataReader
The following are tied with Oracle data provider.
- OracleConnection
- OracleCommand
- OracleDataAdapter
- OracleDataReader
The DataSet is not tied to any specific data provider.
ADO.NET is used in .NET applications - Web, Windows Desktop, Console applications.
## What is ADO.NET Data Provider?
ADO.NET Data Provider is used to communicate with a database.
A Data Provider  means a set of .NET classes that are used to communicate with a specific database.
Different data providers are available to communicate with different database.

**Examples:**

- Data Provider for Oracle - System.Data.OracleClient
- Data Provider for SQL Server - System.Data.SqlClient	
- Data Provider for OLEDB - System.Data.Oledb
- Data Provider for ODBC - System.Data.Odbc

- System.Data.OracleClient Data Provider is used to communicate with Oracle databases.
- System.Data.SqlClient Data Provider is used to communicate with SQL Server databases.

**How to run SQL command using ADO.NET System.Data.SqlClient for SQL Server?**
```
SqlConnection sqlConnection = new SqlConnection(connectionString);
SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
sqlConnection.Open();
SqlDataReader sdr = sqlCommand.ExecuteReader();
while(sdr.Read())
{
// read row by row
}
sqlConnection.Close();
Example.
<?xml version="1.0" encoding="utf-8"?>
<!--
  Note \\ in connection string will cause error.
  Example. Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database
  -->
<configuration>
	<connectionStrings>
		<add name="DefaultConnection"
			 connectionString="Server=DESKTOP-PUI1MDR\SQLEXPRESS;Database=DropDownDb;Integrated Security=True;TrustServerCertificate=True;"
			 providerName="System.Data.SqlClient"/>
	</connectionStrings>

</configuration>
  <system.web>
```
Add a WebForm1.aspx file.
```
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="WebForm1.aspx.cs" Inherits="WebApplication1.WebForm1" %>
Front end HTML
<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title></title>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <asp:GridView ID="GridView1" runat="server"></asp:GridView>
        </div>
    </form>
</body>
</html>
```
Add following code in WebForm1.aspx.cs file.
```
using System;
using System.Configuration;
using System.Data.SqlClient;

namespace WebApplication1
{
    public partial class WebForm1 : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            string connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;

            using (SqlConnection sqlConnection = new SqlConnection(connectionString))
            {
                string cmdText = "SELECT * FROM tblCity";
                SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
                try
                {
                    sqlConnection.Open();
                    SqlDataReader sdr = sqlCommand.ExecuteReader();
                    GridView1.DataSource = sdr;
                    GridView1.DataBind();
                }
                catch (Exception ex)
                {
                    throw new Exception(ex.Message);
                }
                finally
                {
                    sqlConnection.Close();
                }
            }
        }
    }
}
```

Remarks:
The Instance failure error happens because of the double backslash (\\) in Web.config connection string:
Server=DESKTOP-PUI1MDR\\SQLEXPRESS;

**Why this happens?**

Unlike C# string literals—where \\ is used to escape a single backslash—XML files like Web.config treat backslashes literally. Because of the double backslash, ADO.NET attempts to connect to a SQL Server named instance literally called \SQLEXPRESS instead of SQLEXPRESS, resulting in an Instance failure exception.
How to fix it?
1.	Change \\ to a single \ in Web.config:
2. Since sqlConnection is already wrapped in a using block, it closes and disposes automatically—you don't need the finally { sqlConnection.Close(); } block.
3. Wrap SqlDataReader in a using block as well so it disposes properly after binding.

Improved code:
```
protected void Page_Load(object sender, EventArgs e)
{
    if (!IsPostBack)
    {
        string connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;

        using (SqlConnection sqlConnection = new SqlConnection(connectionString))
        using (SqlCommand sqlCommand = new SqlCommand("SELECT * FROM tblCity", sqlConnection))
        {
            sqlConnection.Open();
            using (SqlDataReader sdr = sqlCommand.ExecuteReader())
            {
                GridView1.DataSource = sdr;
                GridView1.DataBind();
            }
        }
    }
}
```

