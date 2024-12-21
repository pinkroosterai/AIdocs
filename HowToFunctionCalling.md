To effectively utilize the `FunctionDescriptionAttribute` and `ParameterDescriptionAttribute` in conjunction with the `FunctionCallingHelper` from the `Betalgo.OpenAI.Utilities` library, follow these steps:

### 1. **Define Your Methods with Attributes**

Decorate your methods with the `FunctionDescriptionAttribute` to provide metadata about the function, and use the `ParameterDescriptionAttribute` for each parameter to specify details such as name, description, type, and whether it's required.

```csharp
using Betalgo.OpenAI.Utilities.FunctionCalling;

public class Calculator
{
    [FunctionDescription("Adds two numbers.")]
    public int Add(
        [ParameterDescription("First number to add.")] int a,
        [ParameterDescription("Second number to add.")] int b)
    {
        return a + b;
    }

    [FunctionDescription("Multiplies two numbers.")]
    public int Multiply(
        [ParameterDescription("First number to multiply.")] int a,
        [ParameterDescription("Second number to multiply.")] int b)
    {
        return a * b;
    }
}
```

In this example, each method is annotated to describe its purpose and the parameters it accepts.

### 2. **Generate Function Definitions**

Utilize the `FunctionCallingHelper` to extract function definitions from your class. This process involves reflection to read the attributes and create corresponding definitions.

```csharp
using Betalgo.OpenAI.Utilities.FunctionCalling;

var calculator = new Calculator();
var functionDefinitions = FunctionCallingHelper.GetToolDefinitions<Calculator>();
```

Here, `functionDefinitions` will contain the metadata for all methods in the `Calculator` class that are decorated with the `FunctionDescriptionAttribute`.

### 3. **Integrate with OpenAI Service**

When creating a chat completion request, include the generated function definitions to inform the model about the available functions it can invoke.

```csharp
using Betalgo.Ranul.OpenAI;
using Betalgo.Ranul.OpenAI.Models;
using Betalgo.Ranul.OpenAI.ObjectModels.RequestModels;

var openAIService = new OpenAIService(new OpenAIOptions
{
    ApiKey = "YOUR_API_KEY"
});

var chatRequest = new ChatCompletionCreateRequest
{
    Messages = new List<ChatMessage>
    {
        ChatMessage.FromSystem("You are a helpful assistant."),
        ChatMessage.FromUser("What is 2 + 2?")
    },
    Tools = functionDefinitions,
    Model = Models.Gpt_4o
};
```

This setup allows the model to understand which functions it can call during the conversation.

### 4. **Handle Function Calls**

Process the model's responses to detect and execute function calls iteratively until no further function calls are requested.

```csharp
do
{
    var reply = await openAIService.ChatCompletion.CreateCompletion(chatRequest);

    if (!reply.Successful)
    {
        Console.WriteLine(reply.Error?.Message);
        break;
    }

    var responseMessage = reply.Choices.First().Message;
    chatRequest.Messages.Add(responseMessage);

    if (responseMessage.ToolCalls != null)
    {
        var functionCall = responseMessage.ToolCalls.First().FunctionCall;
        var result = FunctionCallingHelper.CallFunction<int>(functionCall!, calculator);
        chatRequest.Messages.Add(ChatMessage.FromFunction(functionCall.Name, result.ToString()));
    }
    else
    {
        Console.WriteLine(responseMessage.Content);
    }
} while (chatRequest.Messages.Last().FunctionCall != null);
```

This loop continues processing until the model's response does not include a function call, ensuring all necessary function executions are handled appropriately.

### References

- [FunctionCallingHelper.cs](https://github.com/betalgo/openai/blob/master/OpenAI.Utilities/FunctionCalling/FunctionCallingHelper.cs)
- [ParameterDescriptionAttribute.cs](https://github.com/betalgo/openai/blob/master/OpenAI.Utilities/FunctionCalling/ParameterDescriptionAttribute.cs)
- [FunctionDescriptionAttribute.cs](https://github.com/betalgo/openai/blob/master/OpenAI.Utilities/FunctionCalling/FunctionDescriptionAttribute.cs)
- [Function Calling Discussion](https://github.com/betalgo/openai/discussions/344)
- [Function Calling Wiki](https://github.com/betalgo/openai/wiki/Function-Calling)

By following these steps, you can effectively define functions with descriptive attributes and integrate them into your application using the `FunctionCallingHelper` from the `Betalgo.OpenAI.Utilities` library. 
