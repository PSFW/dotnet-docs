## Pipelines
Automatically add the access token for the build user for use when making requests to the `az` command line, for example in bash.
```yaml
steps:
- bash: |
    az boards work-item update ${args[@]}
  env:
    AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
  displayName: 'Update Work Item'
```
