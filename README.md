# VariableJson (vjson)

[![GitHub](https://img.shields.io/github/license/noahdavis319/vjson)](https://github.com/noahdavis319/vjson/blob/main/LICENSE) [![.NET](https://github.com/noahdavis319/vjson/actions/workflows/dotnet.yml/badge.svg?branch=main)](https://github.com/noahdavis319/vjson/actions/workflows/dotnet.yml) ![.NET 6](https://img.shields.io/badge/-.NET%206.0-blueviolet) [![Nuget](https://img.shields.io/nuget/dt/VariableJson) ![Nuget](https://img.shields.io/nuget/v/VariableJson) ![Nuget (with prereleases)](https://img.shields.io/nuget/vpre/VariableJson?label=nuget-latest)](https://www.nuget.org/packages/VariableJson/)

vjson is a JSON parser that adds support for variables.

Supported languages
 - [C#](https://github.com/noahdavis319/vjson)
 - [JavaScript](https://github.com/noahdavis319/vjson-js)
 - [Python](https://github.com/noahdavis319/vjson-python)

### Examples

The simplest example is

```json
{
  "$vars": {
    "name": "John Doe"
  },
  "johndoe": "$(name)"
}
```

which gets converted to

```json
{
  "johndoe": "John Doe"
}
```

Variables can also reference other variables

```json
{
  "$vars": {
    "name": "John Doe",
    "greeting": "Hello $(name)"
  },
  "johndoe": "$(greeting)"
}
```

which generates

```json
{
  "johndoe": "Hello John Doe"
}
```

You can nest objects and arrays with each other and all other non-complex data types. Here is a complex sample

```json
{
  "$vars": {
    "name": "John Doe",
    "greeting": "Hello $(name)",
    "age": 42,
    "address": {
      "street": "123 Main St",
      "city": "Anytown",
      "state": "CA",
      "zip": "12345"
    },
    "phone": ["000-123-4567", "000-123-4568"]
  },
  "johndoe": {
    "name": "$(name)",
    "greeting": "$(greeting)",
    "age": "$(age)",
    "address": "$(address)",
    "phone": "$(phone)"
  }
}
```

which would give you

```json
{
  "johndoe": {
    "name": "John Doe",
    "greeting": "Hello John Doe",
    "age": 42,
    "address": {
      "street": "123 Main St",
      "city": "Anytown",
      "state": "CA",
      "zip": "12345"
    },
    "phone": ["000-123-4567", "000-123-4568"]
  }
}
```

You can reference objects and values that are in an array by specifying the index

```json
{
  "$vars": {
    "array": [1, false, true, "hello, world!", null]
  },
  "first": "$(array.0)"
}
```

which would produce

```json
{
  "first": 1
}
```

You cannot reference objects that are not stored in the variable container. For example, this will not work

```json
{
  "$vars": {
    "hello": "world!"
  },
  "fizz": "$(buzz)",
  "buzz": "$(hello)"
}
```

This will throw an exception because `fizz` references a variable that does not exist in the variable container.

Lastly, you can also reference environment variables using `$[EnvironmentVariableName]`

```json
{
  "PATH": "$[PATH]"
}
```

When referencing **only** environment variables you do not need to specify a variable container, it won't be used. You may use both environment variables and variables in the same JSON file.

```json
{
  "$vars": {
    "name": "John Doe"
  },
  "johndoe": "$(name)",
  "PATH": "$[PATH]"
}
```

which produces

```json
{
  "johndoe": "John Doe",
  "PATH": "..." // varies by system
}
```

### Usage

> **Note**
> This library uses the `System.Text.Json` library for JSON parsing and serialization and does not handle any exceptions that may be thrown by that library. You should handle any thrown exception yourself.

VariableJson only parses and produces JSON, it does not provide a mechanism for deserializing the JSON into an object. You can use `System.Text.Json`, `Newtonsoft.Json`, or any other JSON library to deserialize the JSON into an object.

```csharp
string originalJson = File.ReadAllText("path/to/file.json");
string convertedJson = VariableJson.Json.Parse(originalJson);
```

### VariableJsonOptions

You can specify some options when parsing JSON using the `VariableJsonOptions` class.

```csharp
string originalJson = File.ReadAllText("path/to/file.json");
string convertedJson = VariableJson.Json.Parse(originalJson, new VariableJsonOptions { VariableKey = "myVars" });
```

The following options are available:

`VariableKey` - The name of the variable container. Defaults to `$vars`.

`Delimiter` - The delimiter to use when parsing variables. Defaults to `.` (period). This string should not appear in any of your JSON key names.

`MaxRecurse` - The maximum number of times to recurse when resolving variables. Defaults to 1024.

`KeepVars` - Whether or not to keep the variable container in the output. Defaults to `false`. The variable container will be identical to the one in the input. It's value will not be resolved.

`EmittedName` - The name of the variable container in the output. Defaults to `$vars`. Only used if `KeepVars` is `true`.

### JSON Schema

While vjson itself is valid JSON, it uses special markers to denote variables. This means that you can't use these same markers in string-type values. To identify that you want to use the variable value instead, you should wrap the variable name in `$(variableName)` and set it as a string value.

```json
{
  "$vars": {
    "name": "John Doe"
  },
  "johndoe": "$(name)"
}
```

If you don't want to use `$vars` as the variable container, you can use the `VariableJsonOptions` class to specify a different variable container name.

```csharp
string originalJson = File.ReadAllText("path/to/file.json");
string convertedJson = VariableJson.Json.Parse(originalJson, new VariableJsonOptions { VariableKey = "myVars" });
```

In the above example, this will cause variable lookups to be performed in the `myVars` object instead of the `$vars` object.

```json
{
  "myVars": {
    "name": "John Doe"
  },
  "johndoe": "$(name)"
}
```

### Performance

vjson deserializes the JSON document and then resolves variable references recursively. Once the document has been parsed, it then serializes the resultant object back to JSON to generated the final output. You can then use this output as you would with any JSON parsing library, such as `System.Text.Json` or `Newtonsoft.Json`, for example.

There's currently no variable lookup caching, but this is a planned feature that you'll be able to opt into using the `VariableJsonOptions` class.
