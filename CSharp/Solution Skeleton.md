### root directory
.dockerignore
.gitignore


### `.devcontainer` folder
devcontainer.json

files.exclude - exclude files from a workspace
workspace.code-workspace - open specific folders for a workspace

### src folder
Directory.Build.props
Directory.Build.targets
Directory.Packages.props
Directory.Build.rsp - automatic MSBuild command line options
global.json
NuGet.config

Company.ruleset
.http file
config.json file - related to `.http` file (same directory)
vars.json file - related to `.http` file (same directory)

docker-compose.dcproj
docker-compose.yml

`launchSettings.json` - `VS Code`, `Visual Studio` and `dotnet run` launch file

`launch.json` - `VS Code` specific configuration (for tasks related to debugging).
`tasks.json` - `VS Code` specific configuration (about tasks outside of direct debugging).
`settings.json` - `VS Code` specific configuration file to configure settings specific to the Workspace i.e JSON Schemas.

packages.lock.json

esproj - project file for react, angular, vite and other JavaScript-based project types
slnf - Solution filter file (for selecting projects within a solution without adding all of them)
.vsconfig - Holds Visual Studio extensions and workflow information. The Visual Studio Installer is able to emit this file. Adding this file to the same directory as the solution will prompt the user to install the extensions and components.

BannedSymbols.txt - Ban specific members using this file (`Microsoft.CodeAnalysis.BannedApuAnalyzers`)

#### global.json file
To install a particular version of the SDK, [download this powershell script](https://dot.net/v1/dotnet-install.ps1), and run it with the following option:
```powershell
dotnet-install.ps1 -Version 6.0.308
```

#### /src/.vscode folder


#### /src/.template.config
`template.json` - 
