To utilize the `PropertyDefinitionGenerator` from the `Betalgo.OpenAI.Utilities` library in conjunction with a response format JSON schema in a chat completion, follow these steps:

### 1. **Define the Response Class**

Begin by creating a C# class that represents the structured format you expect in the response. For instance, if you're expecting a mathematical solution with steps and a final answer, define a class like this:

```csharp
public class MathResponse
{
    public List<Step> Steps { get; set; }
    public string FinalAnswer { get; set; }
}

public class Step
{
    public string Explanation { get; set; }
    public string Output { get; set; }
}
```

### 2. **Generate the JSON Schema Using `PropertyDefinitionGenerator`**

Utilize the `PropertyDefinitionGenerator` to create a JSON schema from the `MathResponse` class. This schema will inform the model about the expected structure of the response.

```csharp
using Betalgo.OpenAI.Utilities.FunctionCalling;

var schema = PropertyDefinitionGenerator.GenerateFromType(typeof(MathResponse));
```

### 3. **Set Up the Chat Completion Request**

Configure the chat completion request to include the generated JSON schema. This setup guides the model to produce responses that conform to the specified structure.

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
        ChatMessage.FromUser("Solve the equation 8x + 7 = -23.")
    },
    Model = Models.Gpt_4o,
    ResponseFormat = new ResponseFormat
    {
        Type = StaticValues.CompletionStatics.ResponseFormat.JsonSchema,
        JsonSchema = new JsonSchema
        {
            Name = "math_response",
            Strict = true,
            Schema = schema
        }
    }
};
```

### 4. **Process the Model's Response**

After receiving the response, deserialize the JSON content into the `MathResponse` object to access the structured data.

```csharp
using System.Text.Json;

var completionResult = await openAIService.ChatCompletion.CreateCompletion(chatRequest);

if (completionResult.Successful)
{
    var responseContent = completionResult.Choices.First().Message.Content;
    var mathResponse = JsonSerializer.Deserialize<MathResponse>(responseContent);

    foreach (var step in mathResponse.Steps)
    {
        Console.WriteLine($"Explanation: {step.Explanation}");
        Console.WriteLine($"Output: {step.Output}");
    }
    Console.WriteLine($"Final Answer: {mathResponse.FinalAnswer}");
}
else
{
    Console.WriteLine($"Error: {completionResult.Error?.Message ?? "Unknown error"}");
}
```

### References

- [Structured Outputs · betalgo/openai Wiki](https://github.com/betalgo/openai/wiki/Structured-Outputs)
- [PropertyDefinitionGenerator.cs](https://github.com/betalgo/openai/blob/master/OpenAI.Utilities/FunctionCalling/PropertyDefinitionGenerator.cs)

By following these steps, you can effectively define a response format using a JSON schema and process structured outputs in chat completions with the `Betalgo.OpenAI.Utilities` library. 
