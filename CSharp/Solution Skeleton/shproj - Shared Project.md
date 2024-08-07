Shared projects differ from class library `.csproj` files in that they do not become assemblies themselves when compiled.

## Gotchas
### `.projitems` file
A `.projitems` file must be created as well in the same directory to define what will be added to compilation.

### Old-style Format
shproj files can be referenced by SDK-style csproj files, but they are in an old format themselves.

References for them are made instead to the `.projitems` file.