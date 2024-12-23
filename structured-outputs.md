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

## Use Cases for Structured Outputs Focusing on Workflows

Structured Outputs in OpenAI's API provide a robust mechanism for ensuring that AI-generated responses adhere to a predefined JSON schema. This capability is particularly beneficial for workflows that require precise data formatting and integration. Here are some key use cases for Structured Outputs in workflow contexts:

### 1. Displaying Structured Data in User Interfaces

- **Use Case**: When building applications that require displaying structured data, such as dashboards or detailed reports, Structured Outputs ensure that the data is consistently formatted.
- **Example**: A math tutoring application that displays each step of a solution separately, allowing users to progress through the solution at their own pace [2].

### 2. Populating Databases with Extracted Content

- **Use Case**: Structured Outputs can be used to extract specific information from documents or user inputs and populate databases with this structured data.
- **Example**: Extracting key information from articles or documents and storing it in a database for later retrieval or analysis [2].

### 3. Entity Extraction for Tool Integration

- **Use Case**: In applications that involve calling external tools or APIs, Structured Outputs can be used to extract entities from user input and map them to tool parameters.
- **Example**: A recommendation system that extracts user preferences and searches a product database using structured parameters [2].

### 4. Multi-Step Workflow Automation

- **Use Case**: Complex workflows that involve multiple steps can benefit from Structured Outputs by ensuring each step's output is correctly formatted for the next step.
- **Example**: Automating a workflow that involves data extraction, transformation, and loading (ETL) processes, where each step requires specific data formats [3].

### 5. Function Calling with Strict Schema Adherence

- **Use Case**: When using function calling in applications, Structured Outputs ensure that the function parameters adhere to a strict schema, reducing errors and improving reliability.
- **Example**: A calendar application that extracts event details from user input and schedules events using a predefined schema [3].

### 6. Text Summarization with Consistent Output

- **Use Case**: Applications that summarize text or transform content into structured formats can use Structured Outputs to ensure consistency.
- **Example**: Summarizing articles with a specific schema that includes fields like invented year, summary, inventors, and key concepts [2].

### 7. Safety and Compliance in User Interactions

- **Use Case**: Structured Outputs can help manage safety and compliance by ensuring that responses adhere to predefined formats, especially in sensitive applications.
- **Example**: Handling user-generated input that may require refusal for safety reasons, with the model indicating refusals distinctly [2].

## FAQ
Answers to the Most Important Questions Regarding Structured Outputs in Betalgo.Ranul.OpenAI

1. Schema Construction

How does DefineObject handle optional vs. required properties in complex schemas?

DefineObject allows developers to define both required and optional properties explicitly:

Required properties are listed in the required parameter of the schema definition, which is a list of property names that must be included in the object.

Optional properties are not listed in the required array, but their schema definitions must still be included in the properties dictionary.


Optional fields can also be emulated by defining a union type that includes null for the field. This allows the schema to accept the property as absent or explicitly set to null.

Example:

PropertyDefinition.DefineObject(
    new Dictionary<string, PropertyDefinition>
    {
        { "requiredField", PropertyDefinition.DefineString("Required field description") },
        { "optionalField", PropertyDefinition.DefineNull("Optional field description") }
    },
    new List<string> { "requiredField" }, // Only "requiredField" is mandatory
    true, // Additional properties allowed
    "Object with required and optional fields"
);



---

2. Error Handling

What are common errors when the model fails to adhere to a strict schema?

Schema Limitations:

Maximum 5 levels of nested objects.

Maximum 100 properties in a schema.

Violations of these limits result in errors like 413 Request Entity Too Large.


Validation Failures:

The model might fail to strictly follow the schema if:

The prompt does not clearly align with the schema.

The model generates unexpected or invalid responses (e.g., a string instead of an integer).



Error Mitigation:

Ensure schema complexity is minimized to stay within the limits.

Use clear and concise prompts to help the model generate valid outputs.

Implement fallback mechanisms for invalid or incomplete responses.




---

3. Prompt Design

How detailed should the system prompt be to ensure the model outputs strictly adhere to the schema?

Prompt Guidance:

The prompt should briefly describe the purpose of the schema and the type of information expected.

Explicitly mention the importance of adhering to the schema in the system message.


Example Prompt:

ChatMessage.FromSystem("You are an assistant that outputs JSON adhering to the provided schema. Follow the schema strictly without adding or omitting fields.")

Prompt Simplification:

The schema itself enforces adherence, so complex prompts are often unnecessary. However, reinforcing adherence helps minimize schema violations.




---

4. Model Behavior

Does enabling strict: true increase model latency or reduce token limits for responses?

Impact on Latency:

Enabling strict: true introduces additional validation steps that slightly increase latency, as the system checks the output against the schema before returning the response.


Impact on Token Limits:

The schema itself does not directly reduce the token limit, but responses may become more concise when adhering to a strict format.


Caching and Validation:

The system caches schema artifacts at the organization level, improving validation efficiency over time.




---

5. Testing Outputs

How can Structured Outputs be tested without making live API calls?

Schema Testing Tools:

Use the PropertyDefinitionGenerator.GenerateFromType() method to generate JSON schemas from C# types, enabling offline validation.

Validate schemas with mock data using libraries like Newtonsoft.Json.Schema or System.Text.Json.


Mocking Responses:

Simulate API responses using test frameworks such as Moq to mock outputs adhering to the schema.


Example Offline Testing Code:

var schema = PropertyDefinitionGenerator.GenerateFromType<MyResponseType>();
var mockResponse = "{ \"key\": \"value\" }"; // Mock JSON output
var isValid = JsonSchemaValidator.Validate(mockResponse, schema);
Console.WriteLine(isValid ? "Valid Response" : "Invalid Response");



---

Best Practices Summary

Schema Design: Use DefineNull for optional fields and limit nested structures to stay within OpenAI's restrictions.

Prompt Optimization: Keep prompts concise and schema-focused.

Testing: Leverage schema generation tools and offline validation to reduce API testing costs.

Error Handling: Implement fallback mechanisms and validate outputs rigorously.

Performance Optimization: Use batch processing and respect API rate limits for better throughput.




## Conclusion

Structured Outputs provide a powerful tool for managing workflows that require precise and reliable data formats. By enforcing schema adherence, they enable developers to build robust applications that integrate seamlessly with various systems and tools. These use cases illustrate the versatility and importance of Structured Outputs in modern application development.

