# ASP.NET Chapter 02 SqlConnection

### Important points:

- Connection should be opened as soon as possible.
- Connection should be closed as soon as possible.
- Connection is a valuable resource that must be handled with care.
- Use try-catch block to handle resources like SqlConnection, SqlCommand.

If you are using "using block" then there is no need of finally block, dispsable object is disposed automatically. 
For example, if you are using "using block" then SqlConnection object is automatically disposed off.
