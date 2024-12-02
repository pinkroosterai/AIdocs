```plaintext
# Comprehensive Reference Guide for Function Calling with Betalgo.Ranul.OpenAI

This guide explains how to use function calling with the Betalgo.Ranul.OpenAI library, including defining functions, using them in chat completions, handling tool calls, and managing iterative interactions. It incorporates best practices and advanced scenarios such as handling multi-tool calls, stream completions, and JSON schema integration.

---

## 1. Defining Functions and Tools

### Function Definition
Functions are defined using `FunctionDefinitionBuilder` and describe the behavior and parameters for external interactions.

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

## 2. Using Tools in Chat Completions

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

## 3. Handling Tool Calls in Responses

### Extracting Tool Calls
After sending a chat completion request, check if a tool call is present in the response.

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

## 4. Executing Functions and Adding Results

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

## 5. Managing Iterative Completions

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

## 6. Stream-Based Completions

### Handling Stream Completions
Use `CreateCompletionAsStream` for handling large responses or real-time interactions.

```csharp
var completionResult = sdk.ChatCompletion.CreateCompletionAsStream(request);

await foreach (var completion in completionResult)
{
    if (completion.Successful)
    {
        Console.Write(completion.Choices.First().Message.Content);
    }
    else
    {
        Console.WriteLine($"{completion.Error?.Code}: {completion.Error?.Message}");
    }
}
```

---

## 7. Multi-Tool Calls

### Managing Concurrent Tools
When multiple tools are called in a single response, handle each tool's result independently.

```csharp
foreach (var toolCall in message.ToolCalls)
{
    var functionResult = ExecuteFunction(toolCall.FunctionCall.Name, toolCall.FunctionCall.ParseArguments());
    request.Messages.Add(ChatMessage.FromTool(functionResult, toolCall.Id!));
}
```

---

## 8. JSON Schema Integration

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

## 9. Best Practices

### Error Handling
- Detect and manage null or invalid responses to prevent runtime errors.
- Validate function arguments and results before using them.

### Usage Monitoring
- Track token usage for optimization.
```csharp
Console.WriteLine($"Total tokens used: {completionResult.Usage?.TotalTokens}");
```

### Multi-Turn Argument Aggregation
- Combine fragmented arguments across iterations to ensure completeness.

---

## Example Workflow

1. Define functions and tools.
2. Add tools to the chat request.
3. Send the chat completion request.
4. Handle tool calls by:
   - Extracting and parsing arguments.
   - Executing the function.
   - Adding results to the conversation.
5. Iterate until no tools are called.
6. Optionally, integrate stream completions or JSON schemas.

By following this guide, developers can leverage Betalgo.Ranul.OpenAI to build powerful, dynamic AI-driven applications.
```
