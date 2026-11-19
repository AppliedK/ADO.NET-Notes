# ASP.NET Chapter 06 Sql injection prevention
SqlCommand object has Parameters property. 
AddWithValue or Add method of Parameters is used to define stored procedure parameter.
```
var sqlCommand = new SqlCommand(cmdText, connection);
sqlCommand.Parameters.AddWithValue("@parameter1",value);
sqlCommand.Parameters.AddWithValue("@parameter2",value);
```
Example
```
CREATE Database DiwaliGift;
GO
Use DiwaliGift;
GO
CREATE TABLE dbo.Employees
(
	EmployeeID INT NOT NULL IDENTITY(1,1) CONSTRAINT PK_Employees PRIMARY KEY,
	Name VARCHAR(50),
	Phone VARCHAR(50),
	Email VARCHAR(50),
	Country  VARCHAR(50)
);
GO
CREATE PROCEDURE InsertTable_Employees
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
GO
```
Create following DatabaseOperations class in ASP.NET Core.
```
using System.Data;
using Microsoft.Data.SqlClient;

namespace JsonDataApp.Data
{
    public class DatabaseOperations
    {
        const string connString = "Server=DESKTOP-PUI1MDR\\SQLEXPRESS;Database=DiwaliGift;Integrated Security=True;TrustServerCertificate=True;";

        public int RunStoredProcedure(string name, string phone, string email, string country)
        {
            string commandText = "InsertTable_Employees";

            using (SqlConnection sqlConnection = new SqlConnection(connString))
            {
                using (SqlCommand sqlCommand = new SqlCommand(commandText, sqlConnection))
                {
                    sqlCommand.CommandType = CommandType.StoredProcedure;

                    // Add typed parameters with explicit sizes matching the database schema
                    sqlCommand.Parameters.Add("@Name", SqlDbType.VarChar, 50).Value = (object?)name ?? DBNull.Value;
                    sqlCommand.Parameters.Add("@Phone", SqlDbType.VarChar, 50).Value = (object?)phone ?? DBNull.Value;
                    sqlCommand.Parameters.Add("@Email", SqlDbType.VarChar, 50).Value = (object?)email ?? DBNull.Value;
                    sqlCommand.Parameters.Add("@Country", SqlDbType.VarChar, 50).Value = (object?)country ?? DBNull.Value;

                    sqlConnection.Open();
                    int rowsAffected = sqlCommand.ExecuteNonQuery();

                    return rowsAffected;
                }
            }
        }
    }
}
```
JsonEmployeeImporter
```
using JsonDataApp.Models;
using System.Text.Json;

namespace JsonDataApp.Data
{
    public class JsonEmployeeImporter
    {
        private readonly DatabaseOperations _dbOps = new DatabaseOperations();

        public int ImportPersonsFromJson(string webRootPath)
        {
            // Build the absolute path to wwwroot/record/persons.json
            string filePath = Path.Combine(webRootPath, "record", "persons.json");

            if (!File.Exists(filePath))
            {
                throw new FileNotFoundException($"File not found at: {filePath}");
            }

            // Read and deserialize the JSON
            string jsonContent = File.ReadAllText(filePath);
            var options = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
            List<Person>? persons = JsonSerializer.Deserialize<List<Person>>(jsonContent, options);

            if (persons == null || persons.Count == 0)
            {
                return 0;
            }

            int totalInserted = 0;

            // Iterate and insert each person
            foreach (var person in persons)
            {
                _dbOps.RunStoredProcedure(
                    person.Name,
                    person.Phone,
                    person.Email,
                    person.Country
                );

                totalInserted++;
            }

            return totalInserted;
        }
    }
}
```
HomeController
```
using JsonDataApp.Data;
using JsonDataApp.Models;
using Microsoft.AspNetCore.Mvc;
using System.Diagnostics;

namespace JsonDataApp.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
        [HttpPost]
        public IActionResult Index(Person person)
        {
            if (ModelState.IsValid)
            {
                string name = person.Name;
                string phone = person.Phone;
                string email = person.Email;
                string country = person.Country;
                DatabaseOperations operation = new DatabaseOperations();
                int result = operation.RunStoredProcedure(name, phone, email, country);
                ViewData["rowsAffected"] = result;
            }
            return View();
        }
    }
}
```

```
using JsonDataApp.Data;
using Microsoft.AspNetCore.Mvc;

namespace JsonDataApp.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ImportController : ControllerBase
    {
        private readonly IWebHostEnvironment _env;

        public ImportController(IWebHostEnvironment env)
        {
            _env = env;
        }

        [HttpPost("load-persons")]
        public IActionResult LoadPersons()
        {
            var importer = new JsonEmployeeImporter();
            int count = importer.ImportPersonsFromJson(_env.WebRootPath);

            return Ok(new { message = $"Successfully processed {count} records." });
        }
    }
}
