Example usage (Installs Git, Visual Studio 22, and uses the `.vsconfig` file to download workloads and extensions):
```yml
# yaml-language-server: $schema=https://aka.ms/configuration-dsc-schema/0.2
# Reference: https://github.com/microsoft/vscode/wiki/How-to-Contribute
properties:
  resources:
    - resource: Microsoft.WinGet.DSC/WinGetPackage
      directives:
        description: Install Git
        allowPrerelease: true
      settings:
        id: Git.Git
        source: winget
    - resource: Microsoft.WinGet.DSC/WinGetPackage
      id: vsPackage
      directives:
        description: Install Visual Studio 2022 (any edition is OK)
        allowPrerelease: true
      settings:
        id: Microsoft.VisualStudio.2022.Community
        source: winget
    - resource: Microsoft.VisualStudio.DSC/VSComponents
      dependsOn:
        - vsPackage
      directives:
        description: Install required VS workloads from project .vsconfig file
        allowPrerelease: true
      settings:
        productId: Microsoft.VisualStudio.Product.Community
        channelId: VisualStudio.17.Release
        vsConfigFile: '${WinGetConfigRoot}\..\src\.vsconfig'
  configurationVersion: 0.2.0
```




winget install --id=Microsoft.DotNet.SDK.6  -e