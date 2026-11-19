# ASP.NET Chapter 03 ConnectionStrings in configuration files
- The web.config or app.config configuration file is used to store configuration values which might be changed at runtime.
- web.config file is used in web applications.
- app.config file is used in windows desktop applications.
- There is no need of redeploying the application just for just in configuration settings change. The configuration files come to help in these situations.
- Hard coding the configuration settings is bad practice. Always use configuration file for such things.

### System.Configuration
You need System.Configuration namespace/assembly for ConfigurationManager class that helps to read configuration file.
- In case of desktop applications, get the reference of System.Configuration DLL because it is not available by default.
- In case of web applications, the System.Configuration is available by default.
```
var CS = "Server=DESKTOP-PUI1MDR\SQLEXPRESS;Database=DropDownDb;Integrated Security=True;TrustServerCertificate=True;"
var connectionString = ConfigurationManager.ConnectionStrings["CS"].ConnectionString;
```
