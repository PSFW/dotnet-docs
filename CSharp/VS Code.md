## Recommend Workspace Extensions
VS Code can recommend extension installations to users of your code.
Create a file called `.vscode/extensions.json`, within this file input (Examples from previous jobs installed extensions):
```json
{
  "recommendations":[
    "Thinker.data-size-count",
    "waderyan.gitblame",
    "joshuapoehls.json-escaper",
    "shd101wyy.markdown-preview-enhanced",
    "esbenp.prettier-vscode",
    "ms-vscode-remote.remote-wsl",
    "redhat.vscode-xml",
    "fabianlauer.vs-code-xml-format",
    "eamodio.gitlens",
    "streetsidesoftware.code-spell-checker",
    "VisualStudioExptTeam.vscodeintellicode"
  ]
}
```

Add "ms-dotnettools.dotnet-interactive-vscode" for Jupyter Notebooks with C#

## `launchSettings.json` file
This file can be used by dotnet run, VS Code and Visual Studio. You can use it to configure how your .NET Core application should start. 

### `.dcproj` Vs. `devcontainer.json`
It is particularly useful for it's integration with `.dcproj` files. This does make the `devcontainer` specification unnecessary, however a `devcontainer.json` file is still helpful for projects that do not use `launchSettings.json` and `.dcproj` files.

