# Comprehensive Reference Guide for Tool Calling with Betalgo.Ranul.OpenAI

Tool calling is a powerful feature available in OpenAI API compatible APIs, allowing models like GPT-4o and Qwen2.5-Coder to interact with user-defined functions through structured outputs. 

This capability enables developers to define specific tools that these models can intelligently invoke when appropriate during conversations, with the model generating the necessary parameters while leaving the actual function execution to the developers code. 

The feature is particularly valuable for tasks requiring structured data extraction or integration with external systems, eliminating the need for complex prompt engineering techniques.

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
Check if a tool call is present in the response. The `ToolCalls` field in the response contains a list of all tools called by the model. Use `ChatMessage.FromTool(result, toolCall.Id)` to add the result of each tool invocation back into the conversation.

```csharp
var response = await sdk.ChatCompletion.CreateCompletion(request);
var message = response.Choices.First().Message;

if (message.ToolCalls != null)
{
    foreach (var toolCall in message.ToolCalls)
    {
        // Log the function call
        var functionName = toolCall.FunctionCall.Name;
        Console.WriteLine($"Function call: {functionName}");

        // Process the function arguments
        foreach (var entry in toolCall.FunctionCall.ParseArguments())
        {
            Console.WriteLine($"{entry.Key}: {entry.Value}");
        }

        // Add the tool result to the message history
        var toolResult = ExecuteFunction(toolCall.FunctionCall.Name, toolCall.FunctionCall.ParseArguments());
        request.Messages.Add(ChatMessage.FromTool(toolResult, toolCall.Id));
    }
}
```

### Tool Calls in Response:
- The response object’s `ToolCalls` field will contain tool calls that need to be executed.
- Use `toolCall.FunctionCall.Name` to identify the function being called and `toolCall.FunctionCall.Arguments` to retrieve the required arguments.

---

## 5. Executing Functions and Adding Results

### Executing a Function
Invoke the function based on the extracted arguments from the `ToolCall`.

```csharp
var weatherInfo = GetWeatherInfo(parsedArguments);
```

### Adding Results to the Conversation
Include the function's result in the `Messages` list as a tool message using `ChatMessage.FromTool(result, toolCall.Id)`.

```csharp
request.Messages.Add(ChatMessage.FromTool(weatherInfo, toolCall.Id));
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

    if (message.ToolCalls != null)
    {
        toolCalled = true;
        foreach (var toolCall in message.ToolCalls)
        {
            var functionResult = ExecuteFunction(toolCall.FunctionCall.Name, toolCall.FunctionCall.ParseArguments());
            request.Messages.Add(ChatMessage.FromTool(functionResult, toolCall.Id));
        }
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
