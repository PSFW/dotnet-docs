## Gotchas
### Serilog
Serilog integration needs to have a `<FunctionsPreservedDependencies>` statement below to prevent runtime errors:
```xml
<PropertyGroup>
  <TargetFramework>net6.0</TargetFramework>
  <AzureFunctionsVersion>v4</AzureFunctionsVersion>
</PropertyGroup>

<ItemGroup>
  <FunctionsPreservedDependencies Include="Microsoft.Extensions.DependencyModel.dll" />
</ItemGroup>
```

## Host file
### Setting Logging Levels per Assembly
You can set a logging level for assemblies (in the `host.json` file):
```json
{
	"version": "2.0",
	"logging": {
		"logLevel": {
			"Namespace.Of.Project": "Information"
		}
	}
}
```


## Debugging
### HTTP Request
You can make http requests to activate the ServiceBus trigger on a Function (or another non-http triggered function).

Make a POST request to the local administrator endpoint `http://localhost:<function-port>/admin/functions/<name-of-function>` (the port can be found in the `launchSettings.json` file or via `netstat -aof | findstr <Process-Id-Here>`).

## Configuration
Configuring an Azure Function does not always follow the same pattern as other .NET technologies. As well as the standard `appSettings.json` file, locally you have the `locala.settings.json` file.

A `local.settings.json` file is required to hold information that the trigger framework needs, such as a connection string app setting. This translates to Environment Variables in the Azure Portal.

As a result of this flat structure for environment variables, configuration must be translated.
A `JSON` object is represented below:
```json
{
  "ShoppingBag": {
    "Name": "Reusable Bag 1",
    "ListOfItems": [
      {
        "Name": "Orange"
      },
      {
        "Name": "Banana"
      },      
    ]
  }
}
```

The `local.settings.json` equivalent is below:
```json
{
  "IsEncrypted": false,
  "Values": {
    "ShoppingBag:Name": "Reusable Bag 1",
    "ShoppingBag:ListOfItems:0:Name": "Orange",
    "ShoppingBag:ListOfItems:1:Name": "Banana",    
  }
}
```

### Gotchas
When deserialising using `IOptions` from this Environment Variables pattern, you must not use an interface, as the configuration deserialiser cannot use an interface such as `IEnumerable`.
