Wraps the result of another layout output at specified line length.

Introduced in NLog 4.3.4

Supported in .NET, Silverlight, Compact Framework and Mono.

## Configuration Syntax
```
${wrapline=Integer:inner=Layout}
```

or by using ambient property to modify output of other layout renderer:

```
${other:wrapline=Integer}
```

## Parameters
### Transformation Options
* **inner** - Wrapped layout. Layout.  Default attribute.
* **wrapline** - Indicates wrapping position. Integer greater than 0. Default: 80

## Examples

```
${wrapline:${message}}
${wrapline:Inner=${message}:WrapLine=80}
${message:wrapline=80}

```
