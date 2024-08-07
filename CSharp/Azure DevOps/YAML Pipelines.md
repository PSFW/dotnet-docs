## Pipeline files
### Indentation
`YAML` enforces indentation for the objects that you are describing.
The indentation at the start of the set of steps does not matter as long as you are consistent.
Objects with properties must not have their properties indented.

Both below forms are valid:
```yml
object:
  propertyOne: ''
  propertyTwo: ''
```

```yml
  object:
    propertyOne: ''
    propertyTwo: ''
```
### Spaces
`YAML` forces use of spaces after an objects property definition.

For example, this is invalid:
```yml
object:
  propertyOne:''
  propertyTwo:''
```
Whereas this is valid:
```yml
object:
  propertyOne: ''
  propertyTwo: ''
```

A valid definition of a multi-line string:
```yml
 object:
  propertyOne: ''
  multiLineProperty: |
    line one
    line two
  propertyTwo: ''
```
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
Also, the indentation at the start of the set of `steps` does not matter as long as you are consistent. Either the above or the below indentation works:
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
This is required when a variable in one job is updated, and a dependent job needs the updated value.

Example usage:
```yml
variables:
  varOne: ''
  varTwo: ''

jobs:
- job: JobA
  steps:
  - script: "echo '##vso[task.setvariable variable=varTwo;isOutput=true]hey'"
    name: StepOne
    displayName: 'Step One'
  - script: "echo '##vso[task.setvariable variable=varThree;isOutput=true]hello'"
    name: StepTwo
    displayName: 'Step Two'

- job: JobB
  condition: and(succeeded())
  dependsOn: JobA
  variables:
    varTwo: $[ dependencies.JobA.outputs['StepOne.varTwo'] ]
    varThree: $[ dependencies.JobA.outputs['StepTwo.varThree'] ]    
  steps:
  - script: echo '$(varThree) from JobB'
```
_Note: The "isOutput=true" is required to set the variable for the output scope (otherwise this would be set for the scope of the job, previously undefined variables can be added in this way)._

The `name` property is required, it must be set in order to reference the output variable in another job.
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
To conditionally run a `task`/`step`, you can use an if statement in an `expression`:
```yml
- ${{ if eq(variables['System.TeamProject'], 'public') }}:
```
_Note: You can also use a `condition` property, but this is evaluated at the beginning of a run, therefore any changes to variables or parameters at runtime will not be evaluated._

[All variables are treated as strings](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/conditions?view=azure-devops#:~:text=Since%20all%20variables%20are%20treated%20as%20strings%20in%20Azure%20Pipelines), so you must use the string syntax when checking for equality.
However, this is not the same for parameters, these can be any one of [these data types](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runtime-parameters?view=azure-devops&tabs=script#parameter-data-types).

Other functions are available besides equality (`eq`). Be aware of performing these functions on variables, as they are strings they may not produce the expected result for greater than (`gt`) for example.
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
By default jobs will run in parallel.
### Conditions
In the below pipeline file, `JobTwo` will run if `JobOne` succeeds, and `JobThree` will run is `JobOne` fails. If the pipeline is cancelled before `JobOne` is complete, then `JobTwo` and `JobThree` will not run.

These jobs will also run in an order dictated by the `dependsOn` property, `JobOne` will always be run before `JobTwo` or `JobThree`. *By default jobs will run in parallel.*

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
The`condition` property is available on `stage`, `job`, and `step`/`task` objects.

Conditions are evaluated at the beginning of a job run, therefore any changes to variables or parameters at runtime will not be evaluated.
However, [variables passed from a previous job as output variables will be evaluated](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml#:~:text=You%20can%20specify%20that%20a%20job%20run%20based%20on%20the%20value%20of%20an%20output%20variable%20set%20in%20a%20previous%20job.%20In%20this%20case%2C%20you%20can%20only%20use%20variables%20set%20in%20directly%20dependent%20jobs%3A).
#### Conditionally check if running within a PR
Use the predefined variable 
[`Build.Reason`](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops#:~:text=Yes-,Build.Reason,-The%20event%20that)
```yml
stages:
  - stage:
    condition: eq(variables['Build.Reason'], 'PullRequest')
```
## DevOps Tasks
Tasks are `YAML` objects that represent commands to run. 
These objects can take a direct form, or an aliased form, as shown below:
Direct use of the `Bash@3` task:
```yml
    - task: Bash@3
      displayName: 'Hello World Direct'
      inputs:
        targetType: 'inline'
        script: | 
          echo "Hello World"
```
Aliased use of bash, at an unknown version:
```yml
    - bash: | 
        echo "Hello World"
      displayName: 'Hello World Alaised'
```
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
Requires to add a checkout step too (as the first task of the build):
```yml
- checkout: self
  persistCredentials: true
```

### Checkout
The checkout task retrieves the git repository to built, this is implicit and so does not usually need to be added.
```yml
    - checkout: self
```
[Documentation on this step is here.](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/steps-checkout?view=azure-pipelines)

To make a git commit you would set `persistCredentials` to true, this allows further steps to use the credentials (they do not need to be passed).
```yml
    - checkout: self
      persistCredentials: true
    
	- bash: |
	    fullName="$(Build.SourceBranch)"
	    bName=$(echo "$fullName" | sed 's/refs\/heads\///g')
	    echo "##vso[task.setvariable variable=branchName;]${bName}"
	  displayName: "Set branch name variable"
      
    - task: PowerShell@2
      displayName: 'Commit and Push'
      inputs:
        targetType: 'inline'
        script: |
            git add -A
            git commit -m "committing nothing"
            git push origin HEAD:"$(branchName)"
```

_Note: When performing actions on the repository beyond the current tip i.e. changing branches, getting other commits etc. you must set `fetchDepth: 0`._
```yml
    - checkout: self
      persistCredentials: true
      fetchDepth: 0
```
#### Order
This should be the first step in an ordinary pipeline.
### SonarQubePrepare
Prepares the server to begin analysis, without starting the analysis.
```yml
    - task: SonarQubePrepare@5
      displayName: 'Prepare analysis on SonarQube'
      inputs:
        SonarQube: '<server-dns-name-here> SonarQube'
        projectKey: '<project-fully-qualified-name-here>'
        projectVersion: '$(Build.BuildNumber)'
        extraproperties: sonar.cs.vstest.reportsPaths=$(Agent.TempDirectory)\**\*.trx
```
[Documentation on this step is here.](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/sonar-qube-prepare-v5?view=azure-pipelines)
#### Order
It is ideal to put this step in as early as possible, as it integrates with a third-party server so it is one of the steps most likely to fail.
Preparing early before the build steps means users are alerted to problems faster.
### SonarQubeAnalyze
The purpose of this task is to analyse the code using MSBuild to determine if it is of a good quality.
```yml
    - task: SonarQubeAnalyze@5
      displayName: 'Run code quality analysis'
```
[Documentation on this step is here.](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/sonar-qube-analyze-v5?view=azure-pipelines)
#### Order
Place this after the build and test steps.
### Sonar-buildbreaker
This task will break the build if the code quality analysis returns a negative result for sufficient code quality.
```yml
    - task: sonar-buildbreaker@8
      displayName: 'Break build on code quality failure'
      inputs:
        SonarQube: '<server-dns-name-here> SonarQube'
```
*Note: This must be added to a build in order to force a failure, `SonarQubeAnalyze` on it's own will fail a check silently.*
[Documentation on this step is here.](https://marketplace.visualstudio.com/items?itemName=SimondeLang.sonar-buildbreaker)
#### Order
Place this after the `SonarQubeAnalyze` step.
### SnykSecurityScan
This task adds a check for the use of vulnerable packages.
Note: This can be used for solutions, Dockerfiles, or container images *(Static code analysis is also available).*
```yml
    - task: SnykSecurityScan@1
      displayName: 'Snyk Security Scan'
      inputs:
        serviceConnectionEndpoint: 'Snyk'
        targetFile: '$(Solution)'
        testType: 'app'
        monitorWhen: 'always'
        failOnIssues: true
        severityThreshold: 'high'
```
[Documentation on this step is here.](https://docs.snyk.io/scm-ide-and-ci-cd-integrations/snyk-ci-cd-integrations/azure-pipelines-integration/snyk-security-scan-task-parameters-and-values)
#### Order
Place this after the build steps. This can come before or after `SonarQube` analysis steps.
### UseDotNet
This task allows for the installation of a particular version of the `.NET SDK`, using the standard `global.json` file.
```yml
    - task: UseDotNet@2
      displayName: 'Install a particular version of the .NET SDK'
      inputs:
        packageType: 'sdk'
        useGlobalJson: true
        workingDirectory: '$(Build.SourcesDirectory)\src\'
```
[Documentation on this step is here.](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/use-dotnet-v2?view=azure-pipelines)
#### Order
Place this before the build steps (including restore).
### DotNetCore
This task is a generic task that allows for the running of `dotnet CLI` commands.
For information on this task, [visit the documentation here](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/dotnet-core-cli-v2?view=azure-pipelines).
#### Restore
This sub-command task exposes a method to call `dotnet restore`.
```yml
    - task: DotNetCoreCLI@2
      displayName: 'Restore projects'
      inputs:
        command: 'restore'
        projects: '$(Solution)'
        feedsToUse: 'config'
        nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
        restoreArguments: '--no-cache --locked-mode'
```
_Note: `$(Build.SourcesDirectory)` is the root directory of the repository. `restoreArguments` must be used to pass arguments, `arguments` will silently fail to pass arguments._

Arguments:
- Argument '--no-cache' forces the restore to use the machine-wide global cache instead of the NuGet http cache.
- Argument '--locked-mode' forces the restore to use the `packages.lock.json` file present against the project to get the exact versions of packages or else fail the restore.
##### Order
Place this step before the `dotnet build` step.
#### Build
This sub-command task will build our projects via `dotnet build`.
```yml
    - task: DotNetCoreCLI@2
      displayName: 'Build projects'
      inputs:
        command: 'build'
        projects: '$(Solution)'
        arguments: '--no-restore'
```
#### Test
This sub-command task allows for running tests via `dotnet test`.
```yml
    - task: DotNetCoreCLI@2
      displayName: 'Run tests'
      continueOnError: false
      inputs:
        command: 'test'
        projects: '$(Solution)'
        publishTestResults: true
        nobuild: true
        arguments: '/p:CollectCoverage=true p:/CoverletOutputFormat=opencover --collect "Code Coverage"'
```
##### Order
Place this step after the `dotnet build` step.
##### Running specific tests
Tests can be filtered to run only under a particular namespace (tilde here is a Contains operator).
```yml
    - task: DotNetCoreCLI@2
        displayName: 'Run tests'
        continueOnError: true
        inputs:
          command: 'test'
          projects: '$(solution)'
          arguments: '--no-build --filter "FullyQualifiedName~Namespace.Of.Tests.To.Run"'
```

Arguments:
- Argument '--no-build' tells the `dotnet CLI` not to chain to a `dotnet build`, and by extension to not chain to the `dotnet restore` command either.
- Argument '--filter' tells the command to run tests that match certain conditions.

Tests can also be filtered based on other Properties, for example `TestCategory`
`--filter "TestCategory=IntegrationTests"`.
Further [popular test framework examples can be found here](https://learn.microsoft.com/en-us/dotnet/core/testing/selective-unit-tests?pivots=mstest#mstest-examples).

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
      inputs:
        command: 'restore'
        projects: '$(Solution)'
        feedsToUse: 'config'
        nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
        restoreArguments: '--no-cache --locked-mode'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build project'
      inputs:
        command: 'build'
        projects: '$(Solution)'
        feedsToUse: 'config'
        nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
    
    - task: DotNetCoreCLI@2
      displayName: 'Test project'
      inputs:
        command: 'test'
        projects: '$(Solution)'
        publishTestResults: true
        arguments: '/p:CollectCoverage=true p:/CoverletOutputFormat=opencover --collect "Code Coverage"'
        feedsToUse: 'config'
        nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
```
This is erroneous because you are restoring 3 times, building 2 times and testing once. This is not the pipeline that was seemingly configured and will slow down builds unnecessarily!

You may also find that a step that is chaining another `dotnet` sub-command is erroring in one of it's chained tasks and masking a problem.

The preferred usage is:
```yml
stages:
- stage: Build
  jobs:
  - job:
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Restore project'
      inputs:
        command: 'restore'
        projects: '$(Solution)'
        feedsToUse: 'config'
        nugetConfigPath: '$(Build.SourcesDirectory)\nuget.config'
        restoreArguments: '--no-cache --locked-mode'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build project'
      inputs:
        command: 'build'
        projects: '$(Solution)'
        arguments: '--no-restore'
    
    - task: DotNetCoreCLI@2
      displayName: 'Test project'
      inputs:
        command: 'test'
        projects: '$(Solution)'
        publishTestResults: true
        arguments: '/p:CollectCoverage=true p:/CoverletOutputFormat=opencover --collect "Code Coverage"'
        nobuild: true
```
#### Variable name clashes with Environment variables
In `dotnet CLI` tasks, variables become environment variables even when not explicitly set with an `env:` declaration (`dotnet test` is unaffected by this).
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

In the ifSyntax, how are typed variables coerced? "All variables are treated as strings"
 Will variables passed from a previous job be evaluated? Yes