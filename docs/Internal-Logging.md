If you have trouble getting NLog to work properly you may want to enable internal logging, which can help identify where the problem is. Internal log output can be sent to a file, console window or both.

## Enabling internal logging using configuration file
When you configure NLog using [Configuration File](Configuration-file), you can enable internal logging by setting the following attribute on the `<nlog>` element:

* `internalLogLevel="Trace|Debug|Info|Warn|Error|Fatal"` – determines internal log level. The higher the level, the less verbose the internal log output.
* `internalLogFile="file.txt"` - adding internalLogFile cause NLog to write its internal debugging messages to the specified file. This includes any exceptions that may be thrown during logging.
* `internalLogToConsole="false|true"` – sends internal logging messages to the console.
* `internalLogToConsoleError="false|true"` – sends internal logging messages to the console error output (stderr).
* `internalLogToTrace="false|true"` – sends internal logging messages to `System.Diagnostics.Trace` (introduced in NLog 4.3)
* `internalLogIncludeTimestamp="false|true"` - indicates whether timestamps should be included in the internal log output (NLog 4.3+)


Here is an example of a configuration file which enables internal logging to a file:
```xml
<nlog internalLogFile="c:\log.txt" internalLogLevel="Trace">
   <targets>
      <!-- target configuration here -->
   </targets>
   <rules>
      <!-- log routing rules -->
   </rules>
</nlog>
```

## Enabling internal logging using environment variables
There are environment variables which control internal logging. You can set those variables before running your program to enable internal logging:
* **NLOG_INTERNAL_LOG_LEVEL** - sets the internal logging level. The available values are Trace, Debug, Info, Warn, Error or Fatal - the default is Info which should be appropriate for most cases, to get more detailed logging - set it to Debug or Trace.
* **NLOG_INTERNAL_LOG_FILE** - if this variable is found in the environment NLog will save internal log to the specified file. The file must be writable by the current user or it will not be created.
* **NLOG_INTERNAL_LOG_TO_CONSOLE** - if this variable is found in the environment, will outputs internal diagnostic information to the console
* **NLOG_INTERNAL_LOG_TO_CONSOLE_ERROR** - sets internalLogToConsoleError
* **NLOG_INTERNAL_LOG_TO_TRACE** - write internal log to `System.Diagnostics.Trace` (introduced in NLog 4.3)
* **NLOG_INTERNAL_INCLUDE_TIMESTAMP** - sets internalLogIncludeTimestamp (introduced in NLog 4.3)

## Enabling internal logging programmatically
Internal logging can be configured through code by setting the following properties on InternalLogger class:
* **InternalLogger.LogLevel** - specifies internal logging level
* **InternalLogger.LogFile** - specifies name of the log file (null will disable logging to a file)
* **InternalLogger.LogToConsole** - enables or disables logging to the console
* **InternalLogger.LogToConsoleError** - enables or disables logging to the console error stream
* **InternalLogger.LogToTrace** - enables or disables logging to `System.Diagnostics.Trace` (introduced in NLog 4.3)
* **InternalLogger.LogWriter** - specifies a `TextWriter` object to use for logging
* **InternalLogger.IncludeTimestamp** - enables or disables whether timestamps should be included in the internal log output (NLog 4.3+)
* ** 

```csharp
using NLog;
using NLog.Common;
 
class Program
{
  static void Main()
  {
    // enable internal logging to the console
    InternalLogger.LogToConsole = true;
 
    // enable internal logging to a file
    InternalLogger.LogFile = "c:\\log.txt";

    // enable internal logging to a custom TextWriter
    InternalLogger.LogWriter = new StringWriter(); //e.g. TextWriter writer = File.CreateText("C:\\perl.txt")

 
    // set internal log level
    InternalLogger.LogLevel = LogLevel.Trace;
  }
}
```