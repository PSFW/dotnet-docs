## `<summary>`
The `<summary>` XML tag 
Note: `<para/>` can be used to split onto a new line.

## `<see cref="">`

## `<seealso cref="">`

## `<exception cref="">`

## `<typeparam name="">`

## `<list type="bullet">`
```csharp
/// <list type="bullet">
    /// <item><description>Item 1</description></item>
    /// <item><description>Item 2</description></item>
    /// <item><description>Item 3</description></item>
    /// </list>
```
[Reference](https://stackoverflow.com/a/27293267)

## `<remarks>`
The `<remarks>` XML tag can be used for adding a more lengthy description than the `<summary>`.
To provide an example use the `<example>` and `<code>` tags.
```
///<remarks>
```

# Tips
## Reuse XML comments
Apply `<inheritdoc cref="" path="" />` to a member and use that text in another XML comment.
[Reference here](https://stackoverflow.com/a/74600911)

Use `<include file="" path="">`