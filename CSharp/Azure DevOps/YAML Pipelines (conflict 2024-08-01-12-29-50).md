## Pipeline files
### Indentation
YAML enforces indentation for steps that you are describing.

### Templates
Templates allow you to reuse steps. These files consist of a `parameters` and a `steps` section.
```yml
parameters:
- name: 'stringParam'
  type: string
  default: ''
- name: 'objectParam'
  type: object
  default: {}

steps:
- bash: |
    echo "Hello World!"
  name: helloWorldAnnouncement
  displayName: 'Hello World'
```
#### Parameters
String parameters can be defined in template files as such:
``` yml
parameters:
  stringParam: ''
```
Or they can have more complex forms:
```yml
parameters:
- name: 'stringParam'
  type: string
  default: ''
```
### Referencing templates
Referencing templates within pipeline files is done like so (within the file `pipeline.yml`):
```yml
steps:
  - template: ci-build-template.yml
    parameters:
      stringParam: ''
      objectParam:
      - 'prop1'
      - 'prop2'
```

Note that `ci-build-template.yml` is in the same directory as the pipeline file above.
Also note that the indentation at the start of the set of `steps` does not matter as long as you are consistent. Either the above or the below indentations works:
```yml
steps:
- template: ci-build-template.yml
  parameters:
    stringParam: ''
    objectParam:
    - 'prop1'
    - 'prop2'
```
## Jobs
### Strategy
Run a set of build steps with different configurations as one run.
Jobs can run in parallel, so you can have for example Debug, Release, CodeAnalysis in one pipeline.
```yml
strategy:
  matrix:
    Release:

steps:
- template: ci-build-template.yml
  parameters:
    stringParam: ''
    objectParam:
    - 'prop1'
    - 'prop2'
```

### Steps
#### Tasks
Only parameters are allowed to be referenced in the `displayName` property. ${{}}
However variables are allowed to be used in the `name` property for pipeline files.

#### Referencing variables from output steps
#### Referencing templates within templates
Referencing of sub-templates is executed based on the current directory that a template is sat within.

A template file called `ci-build-template.yml` is in the same directory as the pipeline file that uses it. A further template file `post-build-steps.yml` is in a directory beneath called `steps`, and within `steps` is further directory called `custom`, containing a template file called `announce-helloworld-template.yml`.
The folder structure looks like the below:

```bash
├── steps
│   ├── custom
│   │   ├── announce-helloworld-template.yml
│   ├── post-build-steps.yml
├── ci-build-template.yml
└── pipeline.yml
```

First of all `pipeline.yml` references `ci-build-template.yml` in the same directory:
```yml
steps:
  - template: ci-build-template.yml
    parameters:
      stringParam: ''
      objectParam:
      - 'prop1'
      - 'prop2'
```
Then  `ci-build-template.yml` references `post-build-steps.yml`:
```yml
steps:
  - template: steps/post-build-steps.yml
    parameters:
      stringParam: ''
      objectParam:
      - 'prop1'
      - 'prop2'
```
In turn, `post-build-steps.yml` then references `announce-helloworld-template.yml`:
```yml
steps:
  - template: custom/announce-helloworld-template.yml
    parameters:
      stringParam: ''
      objectParam:
      - 'prop1'
      - 'prop2'
```
#### If syntax
```yml
- ${{ if eq(variables['System.TeamProject'], 'public') }}:
```


#### Iteration syntax
A template file called `announce-helloworld-template.yml` could contain the following:

```yaml
parameters:
- name: 'stringParam'
  type: string
  default: ''
- name: 'objectParam'
  type: object
  default: {}

steps:
- ${{ each prop in parameters.objectParam }}
  - bash: |
      echo "Hello World!"
    name: helloWorldAnnouncement
    displayName: 'Hello World ${{ prop }}'
```
This iterates the values provided to the object and outputs them on the display name only.
### Async

### Conditions
In the below pipeline file, `JobTwo` will run if `JobOne` succeeds, and `JobThree` will run is `JobOne` fails. If the pipeline is cancelled before `JobOne` is complete, then `JobTwo` and `JobThree` will not run.r

```yml
jobs:
- job: JobOne
  steps:

- job: JobTwo
  dependsOn: JobOne
  steps:
  condition: and(succeeded('JobOne'), not(canceled()))

- job: JobThree
  dependsOn: JobOne
  steps:
  condition: and(failed('JobOne'), not(canceled()))
```

#### Conditionally check if running within a PR
Use the predefined variable 
[`Build.Reason`](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops#:~:text=Yes-,Build.Reason,-The%20event%20that)
```yml
stages:
  - stage:
    condition: eq(variables['Build.Reason'], 'PullRequest')
```

## DevOps Tasks
### Dependabot
Unofficial Dependabot task.
Usage: 
```yml
- task: dependabot@1
```
Add an `.azuredevops/dependabot.yml` file to configure this.
```yml
version: 2
updates:
- package-ecosystem: nuget
  directory: "/src"
  schedule:
    interval: daily
  open-pull-requests-limit: 25
registries:
  nuget-example:
    type: nuget-feed
    url:
    username:
    password: ${{}}
  docker-example:
    type: docker-registry
    url:
    username:
    password: ${{}}
    replaces-base: true # Replaces the Base URL with the URL passed
```
### Mergify
Automatic bot for merging Dependabot pull requests. (Unavailable in Azure DevOps as of 23/07/2024)

### ATP.ATP-GitTag.GitTag.GitTag@5
Requires to add a checkout step too:
```yml
- checkout: self
  persistCredentials: true
```

### UseDotNet
```yml
- task: UseDotNet@2
  displayName: 'Install a particular version of the .NET SDK'
  inputs:
    packageType: 'sdk'
    useGlobalJson: true
    workingDirectory: '$(Build.SourcesDirectory)\src\'
```

### DotNetCore
#### Restore
Arguments:
- Argument '--no-cache' forces the restore to use the machine-wide global cache instead of the NuGet http cache.
- Argument '--locked-mode' forces the restore to use the `packages.lock.json` file present against the project to get the exact versions of packages or else fail the restore.

#### Test
##### Running specific tests
Tests can be filtered to run only under a particular namespace (tilde here is a Contains operator).
```yml
- task: DotNetCoreCLI@2
    displayName: 'Run tests'
    continueOnError: true
    inputs:
      command: 'test'
      projects: '$(solution)'
      arguments: '--no-build --no-restore --filter "FullyQualifiedName~Namespace.Of.Tests.To.Run"'
```

Tests can also be filtered based on other Properties, for example `TestCategory`
`--filter "TestCategory=IntegrationTests"`.

## Gotchas
### DotNetCore
#### Chaining
`dotnet` CLI sub-commands like `dotnet pack`, `dotnet publish`, `dotnet test` are chained, they will call one another automatically by default. For example a `dotnet test` command will call `dotnet build`, which in turn will call `dotnet restore`.
A very common error when setting a pipeline up is to do something like the following:
```yml
stages:
- stage: Build
  jobs:
  - job:
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Restore project'
- task: DotNetCoreCLI@2
  displayName: 'Restore project'
  inputs:
    command: 'restore'
    projects: '$(Solution)'
    feedsToUse: 'config'
    nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
    restoreArguments: '--no-cache --locked-mode'

// build task - without arguments: '--no-restore'

// test task - without arguments: '--no-build'
      
```
This is erroneous because you are restoring 3 times, building 2 times and testing once. This is not the pipeline that was seemingly configured and will slow down builds unnecessarily!

#### Variable name clashes with Environment variables
In dotnet core cli tasks, variables become environment variables (even when not explicitly set with an `env:` declaration).
`dotnet build` [for example has `OutputPath` env variable](https://GitHub.com/Microsoft/azure-pipelines-tasks/issues/18584), but any MSBuild property will be a taken if it is also the name of an environment variable. Ensure variable names do not conflict with MSBuild environment variables.
Further examples are `versionPrefix`, `versionSuffix`, and `packageVersion`.

Where possible, avoid using variables and instead use parameters, or use variables that are scoped to a particular step and that have unique names.

Note: This does not work with `test` command when running within the test runner. You will still need to pass in variables e.g. `-e email=$(email)`. This could be because only MSBuild is affected, not the test runner.

#### `dotnet restore`
The `dotnet restore` task `DotNetCoreCLI@2` has a couple of issues.
An open problem with this task is that [Pipeline Variables with 'Version' in the name cause errors](https://github.com/dotnet/core/issues/4491#issuecomment-1798535652).
Another problem is that the `arguments:` are ignored, instead you must use `restoreArguments:`. This is now reflected in official documentation and it is the only `dotnet` sub-command that has this problem.
Example usage would be:
```yml
- task: DotNetCoreCLI@2
  displayName: 'Restore project'
  inputs:
    command: 'restore'
    projects: '$(Solution)'
    feedsToUse: 'config'
    nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
    restoreArguments: '--no-cache --locked-mode'
```

## Questions:
do the NuGetCommand@2 and the dotnet restore commands handle paths differently for the 'nugetConfigPath'?

In the ifSyntax, how are typed variables coerced?