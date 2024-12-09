
# Comprehensive Reference Guide for Tool Calling with Betalgo.Ranul.OpenAI

Tool calling is a powerful feature available in OpenAI API-compatible libraries, allowing models like GPT-4o and Qwen2.5-Coder to interact with user-defined functions through structured outputs. This capability enables developers to define specific tools that these models can intelligently invoke when appropriate during conversations. The feature is particularly valuable for tasks requiring structured data extraction or integration with external systems, eliminating the need for complex prompt engineering techniques.

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

---

## 3. Automating Tool Creation with Attributes

### Custom Attributes
`Betalgo.OpenAI.Utilities` provides attributes for automating function definition and parameter specification.

- **`FunctionDescriptionAttribute`**: Applied to methods, this attribute specifies the function's name and description.
  ```csharp
  [FunctionDescription(Name = "get_current_weather", Description = "Retrieves the current weather for a specified location.")]
  public WeatherInfo GetCurrentWeather(string location, string unit)
  {
      // Implementation
  }
  ```

- **`ParameterDescriptionAttribute`**: Applied to method parameters, this attribute specifies each parameter's name, description, type, and other characteristics.
  ```csharp
  public WeatherInfo GetCurrentWeather(
      [ParameterDescription(Name = "location", Description = "The city and state, e.g., San Francisco, CA")] string location,
      [ParameterDescription(Name = "unit", Description = "The temperature unit.", Enum = "celsius,fahrenheit")] string unit)
  {
      // Implementation
  }
  ```

### FunctionCallingHelper Class
The `FunctionCallingHelper` provides methods to work with attributes and automate tool creation:

- **`GetFunctionDefinition(MethodInfo methodInfo)`**: Generates a `FunctionDefinition` from a method's metadata.
  ```csharp
  MethodInfo methodInfo = typeof(WeatherService).GetMethod("GetCurrentWeather");
  FunctionDefinition functionDefinition = FunctionCallingHelper.GetFunctionDefinition(methodInfo);
  ```

- **`GetToolDefinition(MethodInfo methodInfo)`**: Converts a method into a `ToolDefinition` for use in API requests.
  ```csharp
  ToolDefinition toolDefinition = FunctionCallingHelper.GetToolDefinition(methodInfo);
  ```

- **`GetToolDefinitions(object obj)`**: Enumerates methods with `FunctionDescriptionAttribute` in an object and generates a list of `ToolDefinition` instances.
  ```csharp
  List<ToolDefinition> tools = FunctionCallingHelper.GetToolDefinitions(new WeatherService());
  ```

- **`CallFunction<T>(FunctionCall functionCall, object obj)`**: Executes a function on the given object using the parameters from a `FunctionCall`.
  ```csharp
  var result = FunctionCallingHelper.CallFunction<WeatherInfo>(functionCall, new WeatherService());
  ```

---

## 4. Using Tools in Chat Completions

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
    Tools = FunctionCallingHelper.GetToolDefinitions(new WeatherService()),
    ToolChoice = ToolChoice.Auto,
    Model = Models.Gpt_4
};
```

---

## 5. Handling Tool Calls in Responses

### Extracting Tool Calls
Check if a tool call is present in the response. Use the `ToolCalls` field to extract and execute tool calls.

```csharp
var response = await sdk.ChatCompletion.CreateCompletion(request);
var message = response.Choices.First().Message;

if (message.ToolCalls != null)
{
    foreach (var toolCall in message.ToolCalls)
    {
        var functionResult = FunctionCallingHelper.CallFunction<string>(toolCall.FunctionCall, new WeatherService());
        request.Messages.Add(ChatMessage.FromTool(functionResult, toolCall.Id));
    }
}
```

---

## Best Practices
- **Automation**: Use attributes to simplify tool definition and reduce boilerplate code.
- **Error Handling**: Validate function arguments and handle null responses gracefully.
- **State Management**: Maintain the `Messages` state between iterations.
- **Monitoring Usage**: Track token usage for optimization.
```csharp
Console.WriteLine($"Total tokens used: {completionResult.Usage?.TotalTokens}");
```

By integrating these advanced features of `Betalgo.Ranul.OpenAI` and `Betalgo.OpenAI.Utilities`, developers can build dynamic and efficient AI-driven applications with minimal manual configuration.
