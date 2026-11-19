# ASP.NET Chapter 08 SqlDataReader
SqlDataReader object is a forward only reader that reads record one by one.
The data can not be modified.
It works as long as connection exists. It is connection oriented.
Open the connection before using SqlDataReader.

Step1. Create SqlConnection object.
```
SqlConnection sqlConnection = new SqlConnection(connectionString);
```
Step2. Create SqlCommand object from SqlConnection object.
```
SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
```
Step3. Create SqlDataReader object from SqlCommand object, use command.ExecuteReader() to get SqlDataReader object.
```
SqlDataReader dr = sqlCommand.ExecuteReader();
dr.HasRows returns true if number of rows returns is more than 0.
dr.Read() returns true as long as record is found in memory in forward reading.
while(dr.Read())
{
// row by row processing
}
```
Create DataTable to fill the records row by row.
```
DataTable dt = new DataTable();
dt.Columns.Add("Id");
dt.Columns.Add("Name");
dt.Columns.Add("Price");

dt.NewRow() returns a new DataRow.
DataRow row = dt.NewRow();
```
Example.
```
while(sqlDataReader.Read())
{
// row by row processing
DataRow row = dt.NewRow();
}
```

Three Steps:
1. SqlConnection sqlConnection = new SqlConnection(connectionString);
2. SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
3. SqlDataReader dr = sqlCommand.ExecuteReader();
