# ASP.NET Chapter 07 Calling a stored procedure with output parameters
Use sqlCommand.Parameters.Add() method, not sqlCommand.Parameters.AddWithValue() method in case of stored procedure with output parameters.
Define SqlParameter object and set its properties. For example,
```
SqlParameter sqlParameter1 = new SqlParameter()
{
	ParameterName = "@averageAge",
	SqlDbType = SqlDbType.Decimal,
	Direction = ParameterDirection.Output,
	Precision = 10,
	Scale = 2,
};
```
Example.
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
```
