## Directory.Build.props
### `Choose` Element
The `Choose` Element exists to allow 'switching' on multiple conditions.
```xml
  <Choose>
    <When Condition="'$(OS)' == 'Windows_NT'">
      <PropertyGroup>
        <IsWindows>true</IsWindows>
      </PropertyGroup>
    </When>
  </Choose>
```
[Reference to Sonarr projects Directory.Build.props file]()


## Directory.Build.targets
```xml
<Project>
    <PropertyGroup>
        <CodeAnalysisRuleSet>$(MSBuildThisFileDirectory)Company.ruleset</CodeAnalysisRuleSet>
        <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
        <IsLibrary Condition="'$(IsLibrary)' == ''">true</IsLibrary>
        <NoWarn Condition="'$(IsLibrary)' == 'false'">VSTHRD111</NoWarn>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.333">
            <PrivateAssets>true</PrivateAssets>
        </PackageReference>
        <PackageReference Include="Microsoft.VisualStudio.Threading.Analyzers" Version="16.10.56">
            <PrivateAssets>true</PrivateAssets>
        </PackageReference>
    </ItemGroup>

    <!-- The below adds automatic suppression of StyleCop rules for the generated Entity Framework Migration. -->
    <PropertyGroup>
        <GeneratedMigrationSuppressionText>
            <![CDATA[
using System.Diagnostics.CodeAnalysis%3B

[assembly: SuppressMessage("Style", "SA1200: Using directive should appear within a namespace declaration", Justification = "This is not to be applied to generated classes", Scope = "namespaceanddescendants", Target = "Namespace.Of.Project")]
            ]]>
        </GeneratedMigrationSuppressionText>
    </PropertyGroup>

    <Target Name="AddGeneratedMigrationSuppressionFile" BeforeTargets="BeforeCompile;CoreCompile"
      Inputs="$(MSBuildAllProjects)" Outputs="$(IntermediateOutputPath)GlobalMigrationSuppressions.cs">
        <PropertyGroup>
            <GeneratedMigrationSuppressionFilePath>$(IntermediateOutputPath)GlobalMigrationSuppressions.cs</GeneratedMigrationSuppressionFilePath>
        </PropertyGroup>
        <ItemGroup>
            <Compile Include="$(GeneratedMigrationSuppressionFilePath)" />
            <WriteLines Include="$(GeneratedMigrationSuppressionFilePath)" />
        </ItemGroup>
        <WriteLinesToFile Lines="$(GeneratedMigrationSuppressionText)" File="$(GeneratedMigrationSuppressionFilePath)" WriteOnlyWhenDifferent="true" Overwrite="true"/>
    </Target>
</Project>
```


You could also add a property for when testing a project to expose internal classes (starting from .NET 5):
```xml
<ItemGroup>
<InternalsVisibleTo Include="$(AssemblyName).Tests"/>
</ItemGroup>
```

### Directory.Packages.props
This is different to a `Directory.Build.props` file in that it allows you to opt in to using a package in a csproj file, but it is still centrally managed and locked to a version. This is therefore more obvious that a package reference has been applied. It is similar in that the closest `Directory.Packages.props` file is taken to be the file to use to override any other.

`ManagePackageVersionsCentrally`true
`CentralPackageTransitivePinningEnabled`true
`CentralPackageVersionOverrideEnabled`false

Note: Multiple package sources result in a restore warning, you will need to add packageSourceMappings for each source. (nuget.org can have *, other sources can have a pattern that matches more explicitly)

This warning caused problems, this is not simple to setup, potentially is has a bug.

#### Gotchas
The element `<PackageVersion>` is in use within this file, not `<PackageReference>`.

### dcproj file

#### Gotchas
Dotnet run only works on the project 

### SDK-Style Project properties
#### `BannedSymbols.txt` solution-wide
Add a `Directory.Build.props` file with the below:
```xml
<ItemGroup>
    <AdditionalFiles Include="$(MSBuildThisFileDirectory)BannedSymbols.txt"/>
</ItemGroup>
```

#### `<InternalsVisibleTo>`
