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

Make a POST request to the local administrator endpoint `http://localhost:<function-port>/admin/functions/<name-of-function>` (the Port can be found via `netstat -aof | findstr <Process-Id-Here>`).