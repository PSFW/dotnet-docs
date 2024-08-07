## Description
`SourceLink` allows a package to be created with information that links back to source code, allowing a developer to automatically step-into and debug code from the originating repository.

The below example configures SourceLink for Azure DevOps.
```xml
<Project>

  <PropertyGroup Label="NuGet Package Output - SourceLink Configuration">
    <DebugType>embedded</DebugType>
    <EmbedAllSources>true</EmbedAllSources>
    <ContinuousIntegrationBuild Condition=" '$(TF_BUILD)' == 'True' ">true</ContinuousIntegrationBuild>
    <DeterministicSourcePath Condition=" '$(TF_BUILD)' != 'True' ">true</DeterministicSourcePath>
    <PackageProjectUrl Condition=" '$(TF_BUILD)' == 'True' ">https://example.com</PackageProjectUrl>
    <PublishRepositoryUrl Condition=" '$(TF_BUILD)' == 'True' ">true</PublishRepositoryUrl>
  </PropertyGroup>

  <ItemGroup Label="NuGet Package Output - SourceLink Dependencies">
    <PackageReference Condition=" '$(IsPackable)' == 'True' " Include="Microsoft.SourceLink.AzureRepos.Git" Version="1.0.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentFiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```