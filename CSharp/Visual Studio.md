## .http files
Http files allow you to create and send pre-determined HTTP requests to endpoints. They can have variable substitution via a `http-client.env.json` file.

`.http` files can include secrets, using the AspnetUserSecrets provider in the `http-client.env.json` file.

(UserSecretsId MSBuild property)

VS Code has an extension for editing these files "humao.rest-client".

## `.esproj` file


## Enable Source Link
Go to Tools > Options > Debugging > Symbols and select Nuget and Microsoft Symbol Servers
Go to Tools > Options > Debugging > General and Disable Just My Code and Enable Source Link Support

You can also enable support for this in VSCode via the `launch.json` file.

## Enable IntelliCode
IntelliCode offers simple but intelligent code completion suggestions.
