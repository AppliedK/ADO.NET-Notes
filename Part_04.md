# ASP.NET Chapter 04 SqlCommand

SqlCommand represents a Transact-SQL statement or stored procedure to execute against a SQL Server database. This class cannot be inherited. In following code, cmdText is a Transact-SQL statement or stored procedure name.
```
public SqlCommand()
    {
        GC.SuppressFinalize(this);
    }

public SqlCommand(string cmdText): this()
    {
        CommandText = cmdText;
    }

public SqlCommand(string cmdText, SqlConnection connection) : this()
    {
        CommandText = cmdText;
        Connection = connection;
    }

public SqlCommand(string cmdText, SqlConnection connection, SqlTransaction transaction): this()
    {
        CommandText = cmdText;
        Connection = connection;
        Transaction = transaction;
    }
```

#### Three methods of SqlCommand:
- ExecuteReader > When query returns more than one single value. E.g. SELECT query.
- ExecuteNonQuery => Execute Insert, Update and Delete queries. It returns the total rows affected.
- ExecuteScalar => E.g when query returns a single value e.g. sum of salary, total number of rows.

**Practice the following:**
- Use ADO.NET to run a SQL SELECT statement.
- Use ADO.NET to run a SQL INSERT statement.
- Use ADO.NET to run a SQL UPDATE statement.
- Use ADO.NET to run a SQL DELETE statement.
- Use ADO.NET to run a SQL statement that returns average age of employees.
- Use ADO.NET to run a SQL statement that returns total number of employees which city is France
- Use ADO.NET to run a SQL Non Parameterized stored procedure.
- Use ADO.NET to run a SQL Parameterized stored procedure.
- Use ADO.NET to run a SQL Parameterized stored procedure with Output parameter.

### Example1. Use ADO.NET to run a SQL SELECT statement.
SELECT * FROM Employees;

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();

app.MapControllers();

app.Run();

namespace AdoNetDemo.Models
{
    public class EmployeeModel
    {
        public int EmployeeId { get; set; }
        public string Name { get; set; }
        public string Phone { get; set; }
        public string Email { get; set; }
        public string Country { get; set; }
    }
}
```
EmployeesController class using SqlCommand
```
using AdoNetDemo.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Data.SqlClient;

namespace AdoNetDemo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class EmployeesController : ControllerBase
    {
        [HttpGet]
        [Route("all-employees")]
        public IActionResult GetAll()
        {
            List<EmployeeModel> employees = new List<EmployeeModel>();
            string connectionString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";
            string cmdText = "SELECT * FROM Employees;";
            using SqlConnection sqlConnection = new SqlConnection(connectionString);
            using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
            sqlConnection.Open();
            SqlDataReader sdr = sqlCommand.ExecuteReader();
            while (sdr.Read())
            {
                var emp = new EmployeeModel
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
    }
}
```
Run the application and send GET request at http://localhost:5266/api/Employees/all-employees

### Example2. Use ADO.NET to run a SQL INSERT statement.
The previous code will be same except a new action method.
```
[HttpPost]
[Route("create-employee")]
public IActionResult Create(EmployeeModel model)
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
```
### Example3. Use ADO.NET to run a SQL UPDATE statement.
```
[HttpPut]
[Route("update-employee")]
public IActionResult Update(EmployeeModel model, int id)
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
```
### Example4. Use ADO.NET to run a SQL DELETE statement.
```
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
```		
### Example5. Use ADO.NET to run a SQL statement that returns average age of employees.
```
ALTER TABLE Employees
Add Age INT;
GO

CREATE PROC RandomAges
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @minAge INT = 20;
    DECLARE @maxAge INT = 65;

    UPDATE Employees
    SET Age = @minAge + ABS(CHECKSUM(NEWID())) % (@maxAge - @minAge + 1);
END;

Execute RandomAges
GO
```
Create following action method in EmployeesController.
```
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
```
### Example6. Use ADO.NET to run a SQL statement that returns total number of employees which city is France

Create following action method in EmployeesController.
```
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

```
### Example7. Use ADO.NET to run a SQL Non Parameterized stored procedure.

Create following action method in EmployeesController.
```
CREATE PROC usp_EmployeesAverageAge 
AS
BEGIN
	SELECT AVG(Age) FROM Employees
END
GO
```
Create following action method in EmployeesController.
```
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
```		
		
### Example8. Use ADO.NET to run a SQL Parameterized stored procedure.
```
CREATE PROC usp_EmployeesAverageAgeByCountry @country VARCHAR(50)
AS
BEGIN
	SELECT AVG(Age) 'Average Age' 
	FROM Employees 
	GROUP BY Country 
	HAVING Country = @country 
END
```

http://localhost:5266/api/Employees/average-age-by-country-proc

Create following action method in EmployeesController.
```
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
```

### Example9. Use ADO.NET to run a SQL Parameterized stored procedure with Output parameter.

Create following action method in EmployeesController.
```
CREATE PROC usp_GetAverageAge @averageAge DECIMAL(10,2) OUTPUT
AS
SELECT @averageAge = AVG(CAST(Age AS DECIMAL(10,2))) FROM Employees;
GO
DECLARE @result DECIMAL(10,2);
EXEC usp_GetAverageAge @result OUTPUT;
PRINT @result;
```
Create following action method in EmployeesController.
```
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
	SqlParameter sqlParameter1 = new SqlParameter("@averageAge", SqlDbType.Decimal)
	{
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
```
