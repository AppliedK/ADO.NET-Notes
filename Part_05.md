ASP.NET Chapter 05 SQL Injection
Example.
```
string query = "SELECT Name, Age, Salary FROM Employees WHERE Salary > 50000;";
```
You can use variables also:
```
int value = 50000;
string query = "SELECT Name, Age, Salary FROM Employees WHERE " + Salary > value + " ;";
```
Look at the following case.
```
string pattern ="";
string query = "SELECT Name, Age, Salary FROM Employees WHERE Name LIKE " + pattern + " ;";
```
What if user inputs like:
```
pattern;DELETE * FROM tblInventory;
```
Dynamic queries using string concatenation is prone to SQL injection attacks.
	
