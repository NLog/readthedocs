Writes log messages to the database using an ADO.NET provider. The database operation is always executed outside of a transaction.

Supported in .NET, Compact Framework and Mono

.NET Core users: add the nuget-package System.Data.SqlClient for SQL server etc.

## Configuration Syntax
```xml
<targets>
  <target xsi:type="Database"
          name="String"
          dbProvider="String"
          connectionString="Layout"
          connectionStringName="String"
          keepConnection="Boolean"
          dbDatabase="Layout"
          dbUserName="Layout"
          dbPassword="Layout"
          dbHost="Layout"
          commandType="Enum"
          commandText="Layout"
          installConnectionString="Layout">
    <install-command commandType="Enum" connectionString="Layout" ignoreFailures="Boolean"
                 text="Layout"/><!-- repeated -->
    <uninstall-command commandType="Enum" connectionString="Layout" ignoreFailures="Boolean"
                   text="Layout"/><!-- repeated -->
    <parameter name="String" layout="Layout"
         precision="Byte" scale="Byte" size="Integer" dbType="DbType value" format="string" /><!-- repeated -->
  </target>
</targets>
```
Read more about using the [[Configuration File]].

## Parameters
### General Options
* **name** - Name of the target.

### Connection Options
* **dbProvider** - Name of the database provider. Required. Default: sqlserver
  Following values are recognized by default:
  * sqlserver, mssql, microsoft or msde - Resolves to System.Data.SqlClient Data Provider
  * odbc - ODBC Data Provider (Not recognized on NetCore)
  * oledb - OLEDB Data Provider (Not recognized on NetCore)

  Note for .NET Core one should install the Nuget-package for the DbProvider (Ex. System.Data.SqlClient), and instead use the fully qualified name of the provider connection type (class implementing IDbConnection). See also [DbProvider Examples](#dbprovider-examples)

  Note for .NET then one can also use the invariant name as registered in machine.config or app.config. Common values are (Not supported by NET Core):
   * System.Data.SqlClient -
   * System.Data.OracleClient - (deprecated in .NET Framework 4)
   * Oracle.DataAccess.Client -
   * System.Data.SQLite -
   * Npgsql -
   * MySql.Data.MySqlClient

  If you get the following error in [[Internal-Logging]] then you might have to use the fully qualified name. 

  ```Error during initialization of Database Target[Database_wrapped] Could not load type '<Name Of DbProvider>' from assembly 'NLog, Version=4.0.0.0, Culture=neutral, PublicKeyToken=5120e14c03d0593c'.```

* **connectionString** - Connection string. When provided, it overrides the values specified in DBHost, DBUserName, DBPassword, DBDatabase and DBProvider. [Layout](Layout)  

* **connectionStringName** - Name of the connection string to lookup in app.config. The ProviderName of the connectionstring will be used to determine the SQL type. Since NLog 4.3 this  ProviderName attribute isn't required anymore and the `dbProvider` will be used as fallback.
  > Not supported on NetCore as app.config has been replaced with appsettings.json

* **keepConnection** - Indicates whether to keep the database connection open between the log events. [Boolean](Data types) Default: `false`  

* **dbDatabase** - Database name. If the ConnectionString is not provided this value will be used to construct the "Database=" part of the connection string. [Layout](Data types)  

* **dbUserName** - Database user name. If the ConnectionString is not provided this value will be used to construct the "User ID=" part of the connection string. [Layout](Layout)  

* **dbPassword** - Database password. If the ConnectionString is not provided this value will be used to construct the "Password=" part of the connection string. [Layout](Data types)  

* **dbHost** - Database host name. If the ConnectionString is not provided this value will be used to construct the "Server=" part of the connection string. [Layout](Data types)

* **useTransactions** - This option was removed in NLog 4.0 because the logging code always runs outside of transaction. This ensures that the log gets written to the database if you rollback the main transaction because of an error and want to log the error.

### SQL Statement
* **commandType** - Type of the command. Required. Default: `text`  
  Possible values:
    * `StoredProcedure` - The _commandText_ is the stored procedure name.
    * `TableDirect` -
    * `Text` - regular query
* **commandText** - Text of the SQL command to be run on each log level. [Layout](Data types) Required.  
Typically this is a SQL INSERT statement or a stored procedure call. It should use the database-specific parameters (marked as @parameter for SQL server or :parameter for Oracle, other data providers have their own notation) and not the layout renderers, because the latter is prone to SQL injection attacks. The layout renderers should be specified as \<parameter /> elements instead.

* **parameters** - The collection of parameters. Each parameter contains a mapping between NLog layout and a database named or positional parameter. [Collection](Data types)  
Each collection item is represented by \<parameter /> element with the following attributes:
  * _layout_ - Layout that should be use to calcuate the value for the parameter. [Layout](Data types) Required.  
  * _name_ - Database parameter name. Required.
  * _precision_ - Database parameter precision. [Byte](Data types) Default: 0
  * _scale_ - Database parameter scale. [Byte](Data types) Default: 0
  * _size_ - Database parameter size. [Integer](Data types) Default: 0
  * _dbType_ - One of the values of [DbType](https://docs.microsoft.com/en-us/dotnet/api/system.data.dbtype) (e.g. "int32", "decimal", "datetime2"), 
  or a value of DBType like prefixed with the property name, e.g. "SqlDbType.NChar" will set the property "SqlDbType" on "NChar".
  Another example: `"NpgsqlDbType.Json"` with NpgsqlParameter. 
   Introduced in NLog 4.6
  * _useRawValue_ - nullable boolean. Use the rawValue (from the IRawValue interface) for the value. `null` means yes for non-string-type, this is the default. Introduced in NLog 4.6
  * _format_ - format for parsing the value when not using the rawValue, of if the rawValue conversion to has been failed. Introduced in NLog 4.6
  * _culture_ - culture for parsing the value, analogous as "format". Introduced in NLog 4.6

### Performance Options
* **OptimizeBufferReuse** - Reduce logging overhead, by allowing buffer reuse. Default: `True`
  > Introduced with NLog v4.4.2. Default became `True` with NLog v4.5

### Installation Options
See  [Installing targets](Installing-targets).

* **installConnectionString** - Connection string using for installation and uninstallation. If not provided, regular ConnectionString is being used. [Layout](Layout)  
* **InstallDdlCommands** - The installation DDL commands. [Collection](Data types)  . 
Each collection item is represented by \<install-command /> element with the following attributes:
  * **commandType** - Type of the command. Required. Default: `text`  
  Possible values:
    * `StoredProcedure` - The command-text is the stored procedure name.
    * `TableDirect` -
    * `Text` - regular query
  * **Text** - The command-text (Or stored procedure name)
  * **parameters** - The collection of parameters. Each parameter contains a mapping between NLog layout and a database named or positional parameter. [Collection](Data types)  
Each collection item is represented by <parameter /> element with the following attributes:
    * _layout_ - Layout that should be use to calcuate the value for the parameter. [Layout](Data types) Required.
    * _name_ - Database parameter name. Required.
    * _precision_ - Database parameter precision. [Byte](Data types) Default: 0
    * _scale_ - Database parameter scale. [Byte](Data types) Default: 0
    * _size_ - Database parameter size. [Integer](Data types) Default: 0
    * _text_ - Command text. [Layout](Data types) Required.  
  * **connectionString** - Connection string to run the command against. If not provided, connection string from the target is used. [Layout](Data types)  
  * **ignoreFailures** - Indicates whether to ignore failures. [Boolean](Data types)  
* **uninstallDdlCommands** - The uninstallation DDL commands. [Collection](Data types)  
Each collection item is represented by \<uninstall-command /> element with the following attributes:
  * **commandType** - Type of the command. Required. Default: `text`  
  Possible values:
    * `StoredProcedure` - The command-text is the stored procedure name.
    * `TableDirect` -
    * `Text` - regular query
  * **Text** - The command-text (Or stored procedure name)
  * **parameters** - The collection of parameters. Each parameter contains a mapping between NLog layout and a database named or positional parameter. [Collection](Layout)  
Each collection item is represented by \<parameter /> element with the following attributes:
    * _layout_ - Layout that should be use to calcuate the value for the parameter. [Layout](Layout) Required.
    * _name_ - Database parameter name. Required.
    * _precision_ - Database parameter precision. [Byte](Data types) Default: 0
    * _scale_ - Database parameter scale. [Byte](Data types) Default: 0
    * _size_ - Database parameter size. [Integer](Data types) Default: 0
    * _text_ - Command text. [Layout](Data types) Required.
  * **connectionString** - Connection string to run the command against. If not provided, connection string from the target is used. [Layout](Layout)  
  * **ignoreFailures** - Indicates whether to ignore failures. [Boolean](Layout)  


## About DbType and IRawValue

The DbType works as follows
- NLog tries first if the Layout Renderer support IRawValue to get the non-string type. Then it will use the `IPropertyTypeConverter ` for converting to the desired dbType.
- If that fails, NLog will render the layout render to string and then parse it (with usage of the optional format option).   Then it will also use the `IPropertyTypeConverter ` for converting to the desired dbType.


## Example Configurations

### NLog and SQL Server Example Configuration
```xml
<target name="database" xsi:type="Database">
  <connectionString>server=localhost;Database=*****;user id=****;password=*****</connectionString>

  <!--
  Script for creating the dbo.Log table.
      
  SET ANSI_NULLS ON
  SET QUOTED_IDENTIFIER ON
  CREATE TABLE [dbo].[Log] (
	  [Id] [int] IDENTITY(1,1) NOT NULL,
	  [MachineName] [nvarchar](50) NOT NULL,
	  [Logged] [datetime] NOT NULL,
	  [Level] [nvarchar](50) NOT NULL,
	  [Message] [nvarchar](max) NOT NULL,
	  [Logger] [nvarchar](250) NULL,
	  [Callsite] [nvarchar](max) NULL,
	  [Exception] [nvarchar](max) NULL,
    CONSTRAINT [PK_dbo.Log] PRIMARY KEY CLUSTERED ([Id] ASC)
      WITH (PAD_INDEX  = OFF, STATISTICS_NORECOMPUTE  = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS  = ON, ALLOW_PAGE_LOCKS  = ON) ON [PRIMARY]
  ) ON [PRIMARY]
  -->
      
  <commandText>
    insert into dbo.Log (
      MachineName, Logged, Level, Message,
      Logger, Callsite, Exception
    ) values (
      @MachineName, @Logged, @Level, @Message,
      @Logger, @Callsite, @Exception
    );
  </commandText>

  <parameter name="@MachineName" layout="${machinename}" />
  <parameter name="@Logged" layout="${date}" />
  <parameter name="@Level" layout="${level}" />
  <parameter name="@Message" layout="${message}" />
  <parameter name="@Logger" layout="${logger}" />
  <parameter name="@Callsite" layout="${callsite}" />
  <parameter name="@Exception" layout="${exception:tostring}" />
</target>
```

### NLog and SQL Server using a stored procedure

This approach keeps the NLog.config file simpler, and helps confine 
database logic to the database.

#### NLog target configuration

```xml
<target name="db"
        xsi:type="Database"
        connectionString="server=localhost;Database=*****;user id=****;password=*****"
        commandType="StoredProcedure"
        commandText="[dbo].[NLog_AddEntry_p]"
        >
  <parameter name="@machineName"    layout="${machinename}" />
  <parameter name="@logged"         layout="${date}" />
  <parameter name="@level"          layout="${level}" />
  <parameter name="@message"        layout="${message}" />
  <parameter name="@logger"         layout="${logger}" />
  <parameter name="@properties"     layout="${all-event-properties:separator=|}" />
  <parameter name="@callsite"       layout="${callsite}" />
  <parameter name="@exception"      layout="${exception:tostring}" />
</target>
```

#### SQL scripts to set up the database objects

Remember to grant permissions on the database objects so that the website can execute the stored procedure.

```sql
CREATE TABLE [dbo].[NLog] (
   [ID] [int] IDENTITY(1,1) NOT NULL,
   [MachineName] [nvarchar](200) NULL,
   [Logged] [datetime] NOT NULL,
   [Level] [varchar](5) NOT NULL,
   [Message] [nvarchar](max) NOT NULL,
   [Logger] [nvarchar](300) NULL,
   [Properties] [nvarchar](max) NULL,
   [Callsite] [nvarchar](300) NULL,
   [Exception] [nvarchar](max) NULL,
 CONSTRAINT [PK_dbo.Log] PRIMARY KEY CLUSTERED ([ID] ASC) 
   WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY];

GO

CREATE PROCEDURE [dbo].[NLog_AddEntry_p] (
  @machineName nvarchar(200),
  @logged datetime,
  @level varchar(5),
  @message nvarchar(max),
  @logger nvarchar(300),
  @properties nvarchar(max),
  @callsite nvarchar(300),
  @exception nvarchar(max)
) AS
BEGIN
  INSERT INTO [dbo].[NLog] (
    [MachineName],
    [Logged],
    [Level],
    [Message],
    [Logger],
    [Properties],
    [Callsite],
    [Exception]
  ) VALUES (
    @machineName,
    @logged,
    @level,
    @message,
    @logger,
    @properties,
    @callsite,
    @exception
  );
END
```

### DbProvider Examples

#### MySql and .NET Core
Install package: https://www.nuget.org/packages/MySql.Data/
```xml
dbProvider="MySql.Data.MySqlClient.MySqlConnection, MySql.Data"
```

#### System.Data.SQLite and .NET Core
Install package: https://www.nuget.org/packages/System.Data.SQLite
```xml
dbProvider="System.Data.SQLite.SQLiteConnection, System.Data.SQLite"
```

#### Microsoft.Data.Sqlite and .NET Core
Install package: https://www.nuget.org/packages/Microsoft.Data.SQLite/

```xml
dbProvider="Microsoft.Data.Sqlite.SqliteConnection, Microsoft.Data.Sqlite"
```

#### Oracle.ManagedDataAccess and .NET

```xml
dbProvider="Oracle.ManagedDataAccess.Client.OracleConnection, Oracle.ManagedDataAccess"
```

#### Mono.Data.Sqlite and .NET

```xml
dbProvider="Mono.Data.Sqlite.SqliteConnection, Mono.Data.Sqlite"
```