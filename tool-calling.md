# Comprehensive Reference Guide for Function Calling with Betalgo.Ranul.OpenAI

This enhanced guide explains how to use function calling with the Betalgo.Ranul.OpenAI library, integrating advanced knowledge about `PropertyDefinition` types. It includes defining functions, using them in chat completions, handling tool calls, and managing iterative interactions.

---

## 1. Defining Functions and Tools

### Function Definition
Functions are defined using `FunctionDefinitionBuilder` and describe the behavior and parameters for external interactions. Use `PropertyDefinition` for parameter specification.

```csharp
var function = new FunctionDefinitionBuilder("get_current_weather", "Get the current weather")
    .AddParameter("location", PropertyDefinition.DefineString("The city and state, e.g., San Francisco, CA"))
    .AddParameter("format", PropertyDefinition.DefineEnum(new[] { "celsius", "fahrenheit" }, "The temperature unit to use."))
    .Validate()
    .Build();
```

### Tool Definition
Convert a `FunctionDefinition` into a `ToolDefinition` to use it as a callable tool in the `ChatCompletionCreateRequest`.

```csharp
var tool = ToolDefinition.DefineFunction(function);
```

---

## 2. Property Types in `PropertyDefinition`

The `PropertyDefinition` class allows defining function parameters using JSON Schema properties.

### Available Types
- **String**: Represents a string value.
  ```csharp
  PropertyDefinition.DefineString("A string parameter description");
  ```

- **Integer**: Represents an integer value.
  ```csharp
  PropertyDefinition.DefineInteger("An integer parameter description");
  ```

- **Number**: Represents a numerical value (integer or floating-point).
  ```csharp
  PropertyDefinition.DefineNumber("A numeric parameter description");
  ```

- **Boolean**: Represents a boolean value (true/false).
  ```csharp
  PropertyDefinition.DefineBoolean("A boolean parameter description");
  ```

- **Array**: Represents an array with a specified item type.
  ```csharp
  PropertyDefinition.DefineArray(PropertyDefinition.DefineString("Array item description"));
  ```

- **Object**: Represents a complex object with nested properties.
  ```csharp
  PropertyDefinition.DefineObject(
      new Dictionary<string, PropertyDefinition>
      {
          { "key1", PropertyDefinition.DefineString("String property description") },
          { "key2", PropertyDefinition.DefineInteger("Integer property description") }
      },
      new List<string> { "key1" }, // Required properties
      true, // Additional properties allowed
      "An object parameter description",
      null // No enum restriction
  );
  ```

- **Null**: Represents a null value.
  ```csharp
  PropertyDefinition.DefineNull("A null parameter description");
  ```

- **Enum**: Represents a string with predefined values.
  ```csharp
  PropertyDefinition.DefineEnum(new List<string> { "value1", "value2" }, "Enum parameter description");
  ```

### Additional Properties
- **Properties**: Nested property definitions for objects.
- **Required**: Specifies required fields in an object.
- **AdditionalProperties**: Determines if additional fields are allowed in an object (default: true).
- **Enum**: Specifies allowed values for the parameter.
- **MinProperties/MaxProperties**: Restricts the number of properties for objects.
- **Items**: Specifies the item type for arrays.

---

## 3. Using Tools in Chat Completions

### Adding Tools to a Chat Completion
Include tools in the `Tools` property of the chat completion request. Optionally, use `ToolChoice` to enforce a specific function or allow automatic selection.

```csharp
var request = new ChatCompletionCreateRequest
{
    Messages = new List<ChatMessage>
    {
        ChatMessage.FromSystem("You are a helpful assistant."),
        ChatMessage.FromUser("What's the weather like in New York?")
    },
    Tools = new List<ToolDefinition> { tool },
    ToolChoice = ToolChoice.Auto,
    Model = Models.Gpt_4
};
```

---

## 4. Handling Tool Calls in Responses

### Extracting Tool Calls
Check if a tool call is present in the response.

```csharp
var response = await sdk.ChatCompletion.CreateCompletion(request);
var message = response.Choices.First().Message;

if (message.FunctionCall != null)
{
    var functionName = message.FunctionCall.Name;
    var functionArguments = message.FunctionCall.Arguments;
    // Execute the function with the provided arguments
}
```

### Parsing Function Arguments
Use `FunctionCall.ParseArguments()` to extract and validate arguments.

```csharp
var parsedArguments = message.FunctionCall.ParseArguments();
foreach (var arg in parsedArguments)
{
    Console.WriteLine($"{arg.Key}: {arg.Value}");
}
```

---

## 5. Executing Functions and Adding Results

### Executing a Function
Invoke the function based on the extracted arguments.

```csharp
var weatherInfo = GetWeatherInfo(parsedArguments);
```

### Adding Results to the Conversation
Include the function's result in the `Messages` list as a tool message.

```csharp
request.Messages.Add(new ChatMessage
{
    Role = "function",
    Name = message.FunctionCall.Name,
    Content = weatherInfo
});
```

---

## 6. Managing Iterative Completions

### Iterating Until No Tools Are Called
Send successive completions, adding results to the `Messages` list, until no more tool calls are present.

```csharp
bool toolCalled;
do
{
    var response = await sdk.ChatCompletion.CreateCompletion(request);
    var message = response.Choices.First().Message;

    toolCalled = false;

    if (message.FunctionCall != null)
    {
        toolCalled = true;
        var functionResult = ExecuteFunction(message.FunctionCall.Name, message.FunctionCall.ParseArguments());

        request.Messages.Add(new ChatMessage
        {
            Role = "function",
            Name = message.FunctionCall.Name,
            Content = functionResult
        });
    }
    else
    {
        request.Messages.Add(message);
    }
} while (toolCalled);
```

---

## 7. JSON Schema Integration

### Defining and Using JSON Schema
Define a schema for structured outputs to enforce predictable response formats.

```csharp
var schema = PropertyDefinition.DefineObject(new Dictionary<string, PropertyDefinition>
{
    { "steps", PropertyDefinition.DefineArray(PropertyDefinition.DefineObject(new Dictionary<string, PropertyDefinition>
        {
            { "explanation", PropertyDefinition.DefineString("The explanation of the step") },
            { "output", PropertyDefinition.DefineString("The output of the step") }
        }, new List<string> { "explanation", "output" })) },
    { "final_answer", PropertyDefinition.DefineString("The final answer") }
}, new List<string> { "steps", "final_answer" });

request.ResponseFormat = new()
{
    Type = StaticValues.CompletionStatics.ResponseFormat.JsonSchema,
    JsonSchema = schema
};
```

---

## Best Practices
- **Error Handling:** Validate function arguments and handle null responses gracefully.
- **State Management:** Maintain `Messages` state between iterations.
- **Monitoring Usage:** Track token usage for optimization.
```csharp
Console.WriteLine($"Total tokens used: {completionResult.Usage?.TotalTokens}");
```

By incorporating these principles and `PropertyDefinition` knowledge, developers can build powerful and dynamic AI-driven applications using Betalgo.Ranul.OpenAI.
```
