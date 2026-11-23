# ADO.NET-Notes
Learn ADO.NET from scratch. The following code is taken from ASP.NET Core SDK version 8.0.

ADO.NET Framework can be used for CRUD Operation with ASP.NET/ ASP.NET Core applications.
Notes are attached as separate chapters.

**Create Database and Table in SQL Server**
```
CREATE DATABASE DiwaliGift
GO
USE DiwaliGift
GO

CREATE TABLE Employees (
	EmployeeID int IDENTITY(1,1) NOT NULL  CONSTRAINT PK_Employees PRIMARY KEY,
	Name varchar(30) NULL,
	Phone varchar(20) NULL,
	Email varchar(50) NULL,
	Country varchar(20) NULL,
	Age int NULL
)
GO
```
**Stored Procedures**
```
USE [DiwaliGift]
GO
CREATE OR ALTER PROCEDURE [dbo].[InsertTable_Employees]
    @Name VARCHAR(50),
    @Phone VARCHAR(50),
    @Email VARCHAR(50),
    @Country VARCHAR(50)
AS
BEGIN
    SET NOCOUNT ON;

    IF EXISTS (
        SELECT 1
        FROM sys.tables
        WHERE name = 'Employees'
    )
    BEGIN
        INSERT INTO Employees (Name, Phone, Email, Country) 
        VALUES (@Name, @Phone, @Email, @Country);
    END
END;
```
**Other Stored Procedures**
```
USE [DiwaliGift]
GO
ALTER PROC [dbo].[RandomAges]
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @minAge INT = 20;
    DECLARE @maxAge INT = 65;

    UPDATE Employees
    SET Age = @minAge + ABS(CHECKSUM(NEWID())) % (@maxAge - @minAge + 1);
END;
GO
ALTER PROC [dbo].[usp_EmployeesAverageAge] 
AS
BEGIN
	SELECT AVG(Age) FROM Employees
END
GO
ALTER PROC [dbo].[usp_EmployeesAverageAgeByCountry] @country VARCHAR(50)
AS
BEGIN
	SELECT AVG(Age) 'Average Age' FROM Employees GROUP BY Country HAVING Country = @country 
END
GO
ALTER PROC [dbo].[usp_GetAverageAge] @averageAge DECIMAL(10,2) OUTPUT
AS
SELECT @averageAge = AVG(CAST(Age AS DECIMAL(10,2))) FROM Employees;
GO
```
**Create application using ASP.NET Core Empty template in Visual Studio 2022.**

- Solution name: AdoNetDemo
- Project name: AdoNetDemo

Program.cs
```
var builder = WebApplication.CreateBuilder();
builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run();
```

Install Packages.
- dotnet add package Microsoft.Data.SqlClient -v 7.1.1

Create Employee class.

```
namespace AdoNetDemo.Models;

public class Employee
{
    public int EmployeeId { get; set; }
    public string Name { get; set; }
    public string Phone { get; set; }
    public string Email { get; set; }
    public string Country { get; set; }
}
```
Create EmployeesController class.
```
using AdoNetDemo.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Data.SqlClient;
using System.Data;

namespace AdoNetDemo.Controllers;

[Route("api/[controller]")]
[ApiController]
public class EmployeesController : ControllerBase
{
    private readonly IWebHostEnvironment _env;

    public EmployeesController(IWebHostEnvironment env)
    {
        _env = env;
    }

    [HttpGet]
    [Route("all-employees")]
    public IActionResult GetAll()
    {
        List<Employee> employees = new List<Employee>();
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        string cmdText = "SELECT * FROM Employees;";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        SqlDataReader sdr = sqlCommand.ExecuteReader();
        while (sdr.Read())
        {
            var emp = new Employee
            {
                EmployeeId = sdr.GetFieldValue<int>(0),
                Name = sdr.GetFieldValue<string>(1),
                Phone = sdr.GetFieldValue<string>(2),
                Email = sdr.GetFieldValue<string>(3),
                Country = sdr.GetFieldValue<string>(4)
            };
            employees.Add(emp);
        }

        // serialize employees object
        string jsonData = System.Text.Json.JsonSerializer.Serialize(employees);
        return Ok(jsonData);
    }

    [HttpPost]
    [Route("create-employee")]
    public IActionResult Create(Employee model)
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        string cmdText = "INSERT INTO Employees(Name, Phone, Email, Country) VALUES('"
            + model.Name + "','"
            + model.Phone + "','"
            + model.Email + "','"
            + model.Country + "');";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        int rowsAffected = sqlCommand.ExecuteNonQuery();
        return CreatedAtAction("Create", model);
    }

    [HttpPut]
    [Route("update-employee")]
    public IActionResult Update(Employee model, int id)
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        // All columns are updated.
        string cmdText = "UPDATE Employees SET Name='"
            + model.Name + "',Phone='"
            + model.Phone + "',Email='"
            + model.Email + "',Country='"
            + model.Country + "' WHERE EmployeeId=" + id + ";";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        int rowsAffected = sqlCommand.ExecuteNonQuery();
        return Ok(model);
    }

    [HttpDelete]
    [Route("delete-employee")]
    public IActionResult Delete(int id)
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        string cmdText = "DELETE Employees WHERE EmployeeId=" + id + ";";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        int rowsAffected = sqlCommand.ExecuteNonQuery();
        return Ok(id);
    }

    [HttpGet]
    [Route("average-age")]
    public IActionResult GetAverage()
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        string cmdText = "SELECT AVG(Age) FROM Employees;";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        var result = sqlCommand.ExecuteScalar();
        return Ok(new { msg = "Average age is " + result });
    }

    [HttpGet]
    [Route("count-countrywise")]
    public IActionResult EmployeesCount(string country)
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        // issue if country is null
        string cmdText = "SELECT COUNT(*) FROM Employees WHERE Country='" + country + "';";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlConnection.Open();
        var result = sqlCommand.ExecuteScalar();
        return Ok(new { msg = "Total " + result + " employees are from " + country });
    }
    [HttpGet]
    [Route("average-age-proc")]
    public IActionResult GetAverageByProcedure()
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        string cmdText = "usp_EmployeesAverageAge";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlCommand.CommandType = CommandType.StoredProcedure;
        sqlConnection.Open();
        var result = sqlCommand.ExecuteScalar();
        return Ok(new { msg = "Average age of employees is " + result });
    }

    [HttpGet]
    [Route("average-age-by-country-proc")]
    public IActionResult GetEmployeesAverageAgeByCountry(string country)
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        string cmdText = "usp_EmployeesAverageAgeByCountry";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlCommand.CommandType = CommandType.StoredProcedure;
        SqlParameter sqlParameter1 = new SqlParameter("@country", SqlDbType.Text, 50);
        sqlCommand.Parameters.Add(sqlParameter1).Value = country;
        sqlConnection.Open();
        var result = sqlCommand.ExecuteScalar();
        return Ok(new { msg = $"Average age of employees from {country} is {result}." });
    }

    [HttpGet]
    [Route("outputparameter")]
    public IActionResult GetAverageAge()
    {
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        // Data validation overlooked here for simplicity
        string cmdText = "usp_GetAverageAge";
        using SqlConnection sqlConnection = new SqlConnection(connectionString);
        using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
        sqlCommand.CommandType = CommandType.StoredProcedure;
        SqlParameter sqlParameter1 = new SqlParameter()
        {
            ParameterName = "@averageAge",
            SqlDbType = SqlDbType.Decimal,
            Direction = ParameterDirection.Output,
            Precision = 10,
            Scale = 2,
        };
        sqlCommand.Parameters.Add(sqlParameter1);
        sqlConnection.Open();
        sqlCommand.ExecuteReader();
        var result = sqlCommand.Parameters[sqlParameter1.ParameterName].Value;
        return Ok(new { msg = $"Average age of employees is {result}." });
    }

    [HttpGet]
    [Route("by-adapter")]
    public IActionResult GetDataBySqlAdapter()
    {
        List<Employee> employees = new List<Employee>();
        string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
        string cmdText = "SELECT * FROM Employees;";
        DataSet dataSet = new DataSet();
        using (SqlConnection sqlConnection = new SqlConnection(connectionString))
        using (SqlDataAdapter adapter = new SqlDataAdapter(cmdText, sqlConnection))
        {
            int rowsFilled = adapter.Fill(dataSet);
        }
        Console.WriteLine(dataSet.Tables.Count);
        Console.WriteLine(dataSet.Tables[0].Rows.Count);
        Console.WriteLine(dataSet.Tables[0].Columns.Count);
        DataRowCollection dataRows = dataSet.Tables[0].Rows;
        foreach (DataRow row in dataRows)
        {
            var emp = new Employee
            {
                EmployeeId = row.Field<int>(0),
                Name = row.Field<string>(1),
                Phone = row.Field<string>(2),
                Email = row.Field<string>(3),
                Country = row.Field<string>(4)
            };
            employees.Add(emp);
        }
        // serialize employees object
        string jsonData = System.Text.Json.JsonSerializer.Serialize(employees);
        return Ok(jsonData);
    }
}
```
