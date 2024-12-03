# Comprehensive Guide to Structured Outputs with Betalgo.Ranul.OpenAI 9.0.0

In the context of OpenAI-compatible APIs, **Structured Outputs** are essential tools for managing how AI models generate and return data. These features allow developers to produce predictable, machine-readable responses that can easily integrate with applications. Structured Outputs offer a precise solution with guaranteed schema adherence, making them ideal for applications requiring strict data formatting.

## What are Structured Outputs?

Structured Outputs in OpenAI’s API allow developers to define a strict JSON schema that the model must adhere to when generating responses. This feature is ideal for applications where precise data formatting is required, such as function calling, data extraction, or multi-step workflows. When `strict: true` is set, the model ensures its response adheres strictly to the provided schema, reducing the likelihood of errors and improving integration reliability.

## Steps to Use Structured Outputs

### 1. Define the JSON Schema

Use the `PropertyDefinition` class to construct a JSON schema. This schema specifies the structure, types, and constraints of the response.

#### Property Types in `PropertyDefinition`

The `PropertyDefinition` class allows defining function parameters using JSON Schema properties.

**Available Types:**
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

##### Example: Defining a Mathematical Response Schema
```csharp
var mathResponseSchema = PropertyDefinition.DefineObject(
    new Dictionary<string, PropertyDefinition>
    {
        {
            "steps", PropertyDefinition.DefineArray(
                PropertyDefinition.DefineObject(
                    new Dictionary<string, PropertyDefinition>
                    {
                        { "explanation", PropertyDefinition.DefineString("The explanation of the step") },
                        { "output", PropertyDefinition.DefineString("The output of the step") }
                    },
                    new List<string> { "explanation", "output" },
                    false,
                    "A step in the mathematical process",
                    null
                )
            )
        },
        { "final_answer", PropertyDefinition.DefineString("The final answer of the mathematical process") }
    },
    new List<string> { "steps", "final_answer" },
    false,
    "Response containing steps and final answer of a mathematical process",
    null
);
```

### 2. Configure the Chat Completion Request

Set the `ResponseFormat` to `JsonSchema` and provide the defined schema. Make sure to set the `Strict` property to `true` to ensure the model adheres to the schema.

##### Example: Setting Up Structured Output
```csharp
var completionRequest = new ChatCompletionCreateRequest
{
    Messages = new List<ChatMessage>
    {
        ChatMessage.FromSystem("You are a helpful math tutor. Guide the user through the solution step by step."),
        ChatMessage.FromUser("How can I solve 8x + 7 = -23?")
    },
    Model = "gpt-4o-2024-08-06",
    ResponseFormat = new ResponseFormat
    {
        Type = StaticValues.CompletionStatics.ResponseFormat.JsonSchema,
        JsonSchema = new JsonSchema
        {
            Name = "math_response",
            Strict = true,
            Schema = mathResponseSchema
        }
    }
};
```

### 3. Process the Model's Response

Once the model returns the response, deserialize the JSON content into a corresponding C# object for easy manipulation.

##### Example: Handling the Structured Response
```csharp
var completionResult = await sdk.ChatCompletion.CreateCompletion(completionRequest);

if (completionResult.Successful)
{
    var response = JsonSerializer.Deserialize<MathResponse>(completionResult.Choices.First().Message.Content);
    foreach (var step in response.Steps)
    {
        Console.WriteLine($"{step.Explanation}: {step.Output}");
    }
    Console.WriteLine($"Final Answer: {response.FinalAnswer}");
}
else
{
    Console.WriteLine($"Error: {completionResult.Error?.Message ?? "Unknown Error"}");
}
```

## Best Practices for Structured Outputs

1. **Ensure Model Compatibility:** Check that the model you are using supports the desired output format (Structured Outputs).
2. **Prompt Clarity:** Always be clear in your prompt about the expected output format.
3. **Error Handling:** Implement robust error handling for invalid JSON or schema mismatches.
4. **Use Structured Outputs for Precision:** If strict adherence to a schema is required, use Structured Outputs for greater reliability.
5. **Test with Real Data:** Test your implementation in real-world scenarios to ensure robustness, especially when using Structured Outputs.

## Conclusion

Structured Outputs are powerful tools for obtaining structured, machine-readable responses from models in Betalgo.Ranul.OpenAI 9.0.0. They guarantee precise schema adherence, making them suitable for applications that require predictable and reliable data formats. By following the steps and recommendations outlined in this guide, developers can efficiently leverage these features for their AI-driven solutions.

Citations:
[1] https://platform.openai.com/docs/guides/structured-outputs
[2] https://medium.com/@alexanderekb/openai-api-responses-in-json-format-quickstart-guide-75342e50cbd6
[3] https://community.openai.com/t/structured-outputs-deep-dive/930169
[4] https://openai.com/index/introducing-structured-outputs-in-the-api/
[5] https://community.openai.com/t/structured-outputs-with-assistants/900658
[6] https://community.openai.com/t/dynamic-json-schema-output-format-for-assistant/975365
[7] https://community.openai.com/t/how-can-i-use-function-calling-with-response-format-structured-output-feature-for-final-response/965784
[8] https://community.make.com/t/how-to-use-openai-s-new-structured-outputs/50330
[9] https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/structured-outputs
