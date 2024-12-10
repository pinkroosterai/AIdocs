# Betalgo.Ranul.OpenAI Tool Calling and structured outputs

The Betalgo.Ranul.OpenAI library offers a comprehensive framework for enabling tool calling with OpenAI-compatible models, allowing developers to define and integrate external functions seamlessly. This powerful feature supports structured data extraction, real-time information retrieval, workflow automation, and dynamic multi-turn conversations, making it an essential tool for building sophisticated AI-powered applications.

## Package Installation Guide
The Betalgo.Ranul.OpenAI library has undergone significant changes to improve compatibility and align with new project directions. The core library now uses the package ID and namespace `Betalgo.Ranul.OpenAI`, replacing the previous `Betalgo.OpenAI` [1](https://www.nuget.org/packages/Betalgo.OpenAI). This change enhances compatibility with Microsoft libraries and aligns with the Ranul Tinga project [1](https://www.nuget.org/packages/Betalgo.OpenAI).

To install the core library, use the following NuGet command:

```shell
Install-Package Betalgo.Ranul.OpenAI
```

For developers working with experimental features, the utilities library remains available:

```shell
Install-Package Betalgo.OpenAI.Utilities
```

The library offers flexible integration options. For applications not using dependency injection, you can initialize the service directly:

```csharp
var openAIService = new OpenAIService(new OpenAIOptions() {
    ApiKey = Environment.GetEnvironmentVariable("MY_OPEN_AI_API_KEY")
});
```

For projects leveraging dependency injection, the library provides extension methods for seamless integration:

```csharp
serviceCollection.AddOpenAIService();
```

or with custom configuration:

```csharp
serviceCollection.AddOpenAIService(settings => {
    settings.ApiKey = Environment.GetEnvironmentVariable("MY_OPEN_AI_API_KEY");
});
```

Security is paramount when working with API keys. The library supports the use of user secrets for secure key management [2](https://github.com/betalgo/openai/wiki). Configure your `secrets.json` file as follows:

```json
"OpenAIServiceOptions": {
    "ApiKey": "Your api key goes here",
    "Organization": "Your Organization Id goes here (optional)",
    "UseBeta": "true/false (optional)"
}
```

After configuration, you can retrieve the service from the dependency injection container:

```csharp
var openAiService = serviceProvider.GetRequiredService();
```

The library allows setting a default model for convenience:

```csharp
openAiService.SetDefaultModelId(Models.Gpt_4o);
```

A key feature of the Betalgo.Ranul.OpenAI library is its support for the latest OpenAI models and APIs. For example, you can easily create chat completions using GPT-4o:

```csharp
var completionResult = await openAiService.ChatCompletion.CreateCompletion(new ChatCompletionCreateRequest {
    Messages = new List
    {
        ChatMessage.FromSystem("You are a helpful assistant."),
        ChatMessage.FromUser("Who won the world series in 2020?"),
        ChatMessage.FromAssistant("The Los Angeles Dodgers won the World Series in 2020."),
        ChatMessage.FromUser("Where was it played?")
    },
    Model = Models.Gpt_4o,
});

if (completionResult.Successful)
{
    Console.WriteLine(completionResult.Choices.First().Message.Content);
}
```

This example demonstrates the library's intuitive API for creating multi-turn conversations with AI models [3](https://github.com/betalgo/openai).

For developers transitioning from the previous version, a migration guide is available to assist with breaking changes [1](https://www.nuget.org/packages/Betalgo.OpenAI). The library's documentation and community support channels provide resources for troubleshooting and best practices in implementing OpenAI's capabilities in .NET applications [2](https://github.com/betalgo/openai/wiki) [3](https://github.com/betalgo/openai).


---
**Sources:**
- [(1) Betalgo.OpenAI 8.7.2 - NuGet](https://www.nuget.org/packages/Betalgo.OpenAI)
- [(2) Home · betalgo/openai Wiki - GitHub](https://github.com/betalgo/openai/wiki)
- [(3) NET library for the OpenAI service API by Betalgo Ranul - GitHub](https://github.com/betalgo/openai)


## Tool Calling Fundamentals
Tool calling is a powerful feature that enables Large Language Models (LLMs) to interact with external functions and APIs, significantly expanding their capabilities beyond text generation. This mechanism allows models to delegate specific tasks to predefined functions, enhancing their ability to provide accurate, up-to-date information and perform complex operations [1](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling) [2](https://ai.google.dev/gemini-api/docs/function-calling). By defining a set of tools with specific parameters, developers can create a bridge between the AI model and external systems, enabling structured data extraction, real-time information retrieval, and execution of complex workflows [3](https://arize.com/blog-course/structured-data-extraction-openai-function-calling/).

The process involves three key steps: (1) defining functions with clear descriptions and parameters, (2) allowing the model to determine when and how to use these functions based on user input, and (3) executing the function calls and incorporating the results back into the conversation [4](https://platform.openai.com/docs/guides/function-calling). This approach not only improves the model's ability to handle specialized tasks but also maintains a natural conversational flow, as the model can seamlessly integrate external data and actions into its responses. Tool calling is particularly valuable in scenarios requiring access to dynamic data, performing calculations, or interacting with external services, making it an essential feature for building sophisticated AI-powered applications [1](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling) [3](https://arize.com/blog-course/structured-data-extraction-openai-function-calling/).


---
**Sources:**
- [(1) Introduction to function calling | Generative AI on Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling)
- [(2) Intro to function calling with the Gemini API | Google AI for Developers](https://ai.google.dev/gemini-api/docs/function-calling)
- [(3) Structured Data Extraction with LLMs: What You Need To Know](https://arize.com/blog-course/structured-data-extraction-openai-function-calling/)
- [(4) Function calling - OpenAI API](https://platform.openai.com/docs/guides/function-calling)


## Betalgo.Ranul.OpenAI Namespace Imports
The Betalgo.Ranul.OpenAI library provides a comprehensive set of namespaces and classes for integrating OpenAI's capabilities into .NET applications. Here's an overview of the key namespaces and their purposes:

*   `Betalgo.OpenAI.Utilities.FunctionCalling`: Contains utility classes for implementing function calling features, such as the `FunctionCallingHelper` class for automating tool generation [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.Builders`: Offers builder classes for constructing complex objects, like `FunctionDefinitionBuilder` for creating function definitions [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.Extensions`: Provides extension methods to enhance the functionality of core classes, including methods for adding OpenAI services to dependency injection containers [2](https://www.nuget.org/packages/Betalgo.OpenAI).
*   `Betalgo.Ranul.OpenAI.Interfaces`: Defines interfaces for the library's core components, such as `IOpenAIService` for interacting with OpenAI's API [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.Managers`: Contains manager classes for handling specific OpenAI features, like `ChatCompletionManager` for chat-based interactions [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.ObjectModels.RequestModels`: Includes classes representing request models for various API endpoints, such as `ChatCompletionCreateRequest` [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.ObjectModels.SharedModels`: Contains shared model classes used across different API requests and responses [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI.ObjectModels`: Encompasses the broader set of object models used throughout the library [1](https://github.com/betalgo/openai).
*   `Betalgo.Ranul.OpenAI`: The root namespace containing core classes and configurations for the library [1](https://github.com/betalgo/openai).

To use these namespaces effectively:

1.  Import them at the top of your C# files using the `using` directive.
2.  Utilize the `OpenAIService` class as the main entry point for interacting with OpenAI's API [2](https://www.nuget.org/packages/Betalgo.OpenAI).
3.  Leverage the builder classes and extension methods to construct requests and configure services [1](https://github.com/betalgo/openai).
4.  Implement function calling using the utility classes and attributes provided [1](https://github.com/betalgo/openai).

By organizing functionality into these namespaces, the library promotes a modular and extensible approach to integrating OpenAI's capabilities into .NET applications [1](https://github.com/betalgo/openai) [2](https://www.nuget.org/packages/Betalgo.OpenAI).


---
**Sources:**
- [(1) NET library for the OpenAI service API by Betalgo Ranul - GitHub](https://github.com/betalgo/openai)
- [(2) Betalgo.OpenAI 8.7.2 - NuGet](https://www.nuget.org/packages/Betalgo.OpenAI)


## Function and Tool Definition
The `FunctionDefinitionBuilder` class in Betalgo.Ranul.OpenAI provides a fluent interface for defining functions that can be used as tools by AI models. This approach allows for precise specification of function parameters and their constraints, ensuring that the AI model can correctly interpret and use the function.

When defining a function, the builder pattern is employed to construct the function definition step by step:

```csharp
var function = new FunctionDefinitionBuilder("get_current_weather", "Get the current weather")
    .AddParameter("location", PropertyDefinition.DefineString("The city and state, e.g., San Francisco, CA"))
    .AddParameter("format", PropertyDefinition.DefineEnum(new[] { "celsius", "fahrenheit" }, "The temperature unit to use."))
    .Validate()
    .Build();
```

Each parameter is added using the `AddParameter` method, which takes two arguments: the parameter name and a `PropertyDefinition`. The `PropertyDefinition` class offers various static methods to define different types of parameters:

*   `DefineString`: For text input
*   `DefineInteger`: For whole numbers
*   `DefineNumber`: For floating-point numbers
*   `DefineBoolean`: For true/false values
*   `DefineEnum`: For a predefined set of options
*   `DefineArray`: For lists of items
*   `DefineObject`: For complex nested structures

These definitions correspond to JSON Schema types, allowing for rich parameter specifications [1](https://json-schema.org/understanding-json-schema/reference/object). For example, you can define required properties for object parameters or set minimum and maximum values for numeric parameters:

```csharp
.AddParameter("coordinates", PropertyDefinition.DefineObject(
    new Dictionary
    {
        { "latitude", PropertyDefinition.DefineNumber("Latitude coordinate") },
        { "longitude", PropertyDefinition.DefineNumber("Longitude coordinate") }
    },
    new List { "latitude", "longitude" },
    false,
    "Geographic coordinates"
))
```

The `Validate` method ensures that the function definition is well-formed before building. This step is crucial for catching configuration errors early in the development process.

Once a function is defined, it can be converted into a tool using the `ToolDefinition` class:

```csharp
var tool = ToolDefinition.DefineFunction(function);
```

This conversion creates a callable tool that can be passed to the AI model. The `ToolDefinition` encapsulates the function's metadata and provides a standardized interface for the model to interact with external functions.

It's important to note that while the function definition describes the interface, the actual implementation of the function logic remains separate. This separation allows for flexibility in how the function is executed, whether it's a local method call, an API request, or any other form of computation [2](https://firebase.google.com/docs/genkit/tool-calling).

By leveraging these building blocks, developers can create a rich set of tools that enable AI models to perform complex tasks, access external data sources, and integrate with existing systems seamlessly. The structured approach to function definition ensures that the AI model has clear guidelines on how to use each tool, improving the accuracy and reliability of tool calls in conversational AI applications.

## PropertyDefinition Parameter Specification
The `PropertyDefinition` class in Betalgo.Ranul.OpenAI provides a powerful mechanism for specifying parameters in tool definitions, closely aligning with JSON Schema specifications. This alignment ensures compatibility with OpenAI's function calling feature and allows for precise control over parameter validation and structure.

For string parameters, `PropertyDefinition.DefineString()` offers additional options to refine the input:

```csharp
PropertyDefinition.DefineString(
    "Description",
    minLength: 1,
    maxLength: 100,
    pattern: @"^[a-zA-Z0-9]+$"
)
```

This example defines a string parameter with a minimum length of 1, maximum length of 100, and a regex pattern allowing only alphanumeric characters [1](https://json-schema.org/understanding-json-schema/reference/object).

Numeric parameters can be defined with `DefineInteger()` or `DefineNumber()`, allowing for range specifications:

```csharp
PropertyDefinition.DefineNumber(
    "Temperature in Celsius",
    minimum: -273.15,
    maximum: 100,
    multipleOf: 0.1
)
```

This definition ensures the temperature value is between absolute zero and 100°C, with a precision of 0.1°C [1](https://json-schema.org/understanding-json-schema/reference/object).

For array parameters, `DefineArray()` provides control over the array's structure:

```csharp
PropertyDefinition.DefineArray(
    PropertyDefinition.DefineString("Item description"),
    minItems: 1,
    maxItems: 5,
    uniqueItems: true
)
```

This creates an array of strings with 1 to 5 unique items [1](https://json-schema.org/understanding-json-schema/reference/object).

The `DefineEnum()` method is particularly useful for restricting input to a predefined set of options:

```csharp
PropertyDefinition.DefineEnum(
    new[] { "low", "medium", "high" },
    "Priority level"
)
```

For complex nested structures, `DefineObject()` allows for hierarchical parameter definitions:

```csharp
PropertyDefinition.DefineObject(
    new Dictionary
    {
        { "name", PropertyDefinition.DefineString("Full name") },
        { "age", PropertyDefinition.DefineInteger("Age in years", minimum: 0) },
        { "address", PropertyDefinition.DefineObject(
            new Dictionary
            {
                { "street", PropertyDefinition.DefineString("Street address") },
                { "city", PropertyDefinition.DefineString("City name") },
                { "zipCode", PropertyDefinition.DefineString("Zip code", pattern: @"^\d{5}$") }
            },
            new List { "street", "city" }
        )}
    },
    new List { "name", "age" },
    false
)
```

This example demonstrates a nested object structure for a person's details, with required properties and a nested address object [2](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty).

By leveraging these detailed parameter specifications, developers can create robust tool definitions that guide the AI model in generating appropriate function calls. This level of detail not only improves the accuracy of tool usage but also enhances the overall reliability of the AI-powered application by ensuring that input data conforms to expected formats and ranges.


---
**Sources:**
- [(1) object - JSON Schema](https://json-schema.org/understanding-json-schema/reference/object)
- [(2) Object.defineProperty() - JavaScript - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)


## Attribute-Driven Tool Generation
The `Betalgo.OpenAI.Utilities` namespace provides a set of attributes and helper classes that significantly streamline the process of creating tool definitions for AI function calling. This approach leverages C# attributes to declaratively define function metadata, reducing boilerplate code and improving maintainability.

The `FunctionDescriptionAttribute` is used to annotate methods that should be exposed as callable functions to the AI model. It encapsulates essential metadata:

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class FunctionDescriptionAttribute : Attribute
{
    public string Name { get; set; }
    public string Description { get; set; }
}
```

Similarly, the `ParameterDescriptionAttribute` is applied to method parameters to provide detailed specifications:

```csharp
[AttributeUsage(AttributeTargets.Parameter)]
public class ParameterDescriptionAttribute : Attribute
{
    public string Name { get; set; }
    public string Description { get; set; }
    public string Type { get; set; }
    public string Enum { get; set; }
    public bool Required { get; set; } = true;
}
```

These attributes can be used to create rich, self-documenting function definitions:

```csharp
public class WeatherService
{
    [FunctionDescription(Name = "get_current_weather", Description = "Retrieves the current weather for a given location.")]
    public WeatherInfo GetCurrentWeather(
        [ParameterDescription(Name = "location", Description = "City and state, e.g., 'San Francisco, CA'")]
        string location,
        [ParameterDescription(Name = "unit", Description = "Temperature unit", Enum = "celsius,fahrenheit", Required = false)]
        string unit = "celsius")
    {
        // Implementation
    }
}
```

The `FunctionCallingHelper` class provides methods to automatically generate `FunctionDefinition` and `ToolDefinition` objects from these annotated methods:

```csharp
public static class FunctionCallingHelper
{
    public static FunctionDefinition GetFunctionDefinition(MethodInfo methodInfo)
    {
        // Reflection-based logic to extract function metadata from attributes
    }

   public static List GetToolDefinitions(object obj)
    {
        // Reflection-based logic to generate tool definitions for all annotated methods in an object
    }
}
```

This reflection-based approach allows for dynamic discovery of tool definitions, which is particularly useful in scenarios where the set of available tools may change at runtime [1](https://github.com/betalgo/openai/wiki/Structured-Outputs).

To leverage this automation in your application, you can simply create an instance of your service class and generate tool definitions:

```csharp
var weatherService = new WeatherService();
var tools = FunctionCallingHelper.GetToolDefinitions(weatherService);
```

These generated tool definitions can then be passed directly to the chat completion request:

```csharp
var request = new ChatCompletionCreateRequest
{
    Model = Models.Gpt_4,
    Messages = chatMessages,
    Tools = tools,
    ToolChoice = "auto"
};
```

By using attributes and reflection, this approach significantly reduces the amount of manual code required to define and maintain tool definitions. It also ensures that the function definitions remain synchronized with the actual method implementations, reducing the risk of inconsistencies between the AI model's understanding of the functions and their actual behavior [2](https://www.nuget.org/packages/Betalgo.OpenAI).

Moreover, this attribute-based system allows for easy extension and customization. Developers can create additional attributes to capture more complex parameter constraints or function behaviors, such as rate limiting or authentication requirements. These can then be processed by custom logic in the `FunctionCallingHelper` to generate more sophisticated tool definitions [3](https://swagger.io/docs/specification/v3_0/describing-parameters/).

This automated approach to tool creation not only simplifies the development process but also promotes a more maintainable and scalable architecture for AI-powered applications using the Betalgo.OpenAI library.


---
**Sources:**
- [(1) Structured Outputs · betalgo/openai Wiki - GitHub](https://github.com/betalgo/openai/wiki/Structured-Outputs)
- [(2) Betalgo.OpenAI 8.7.2 - NuGet](https://www.nuget.org/packages/Betalgo.OpenAI)
- [(3) Describing Parameters | Swagger Docs](https://swagger.io/docs/specification/v3_0/describing-parameters/)


## Multi-Turn Tool Calling Implementation
Implementing a multi-turn conversation chatbot with tool calling using the Betalgo.Ranul.OpenAI library involves several key steps that leverage the library's features for seamless integration of external functions into AI-driven conversations. This approach allows for dynamic and context-aware interactions that can access real-time data or perform complex operations.

The process begins with defining tools for specific tasks. In this example, we use a `WeatherService` class that encapsulates weather-related functionality:

```csharp
public class WeatherService
{
    [FunctionDescription(Name = "get_current_weather", Description = "Retrieves the current weather for a location.")]
    public string GetCurrentWeather(
        [ParameterDescription(Name = "location", Description = "City name")] string location)
    {
        // Simulated weather data retrieval
        return $"The current weather in {location} is sunny with a temperature of 25°C.";
    }
}
```

To initiate the conversation, we create a `ChatCompletionCreateRequest` object, which includes the initial system and user messages, along with the defined tools:

```csharp
var request = new ChatCompletionCreateRequest
{
    Messages = new List
    {
        ChatMessage.FromSystem("You are a helpful assistant."),
        ChatMessage.FromUser("What's the weather in New York?")
    },
    Tools = FunctionCallingHelper.GetToolDefinitions(new WeatherService()),
    ToolChoice = ToolChoice.Auto,
    Model = Models.Gpt_4o
};
```

The `ToolChoice.Auto` setting allows the model to decide when to use tools based on the conversation context [1](https://docs.empower.dev/inference/tool-use/multi-turn).

The core of the multi-turn functionality lies in the loop that handles tool calls and maintains the conversation context:

```csharp
var response = await sdk.ChatCompletion.CreateCompletion(request);
var message = response.Choices.First().Message;
request.Messages.Add(ChatMessage.FromAssistant(message));

while (message.ToolCalls != null)
{
    foreach (var toolCall in message.ToolCalls)
    {
        var result = FunctionCallingHelper.CallFunction(toolCall.FunctionCall, new WeatherService());
        request.Messages.Add(ChatMessage.FromTool(result, toolCall.Id));
    }
    response = await sdk.ChatCompletion.CreateCompletion(request);
    message = response.Choices.First().Message;
    request.Messages.Add(ChatMessage.FromAssistant(message));
}
```

This loop continues as long as the model generates tool calls. For each tool call, the corresponding function is executed, and its result is added to the conversation history. The `FunctionCallingHelper.CallFunction` method dynamically invokes the appropriate method based on the tool call [2](https://sdk.vercel.ai/docs/ai-sdk-ui/chatbot-with-tool-calling).

By appending each message (assistant, tool, and user) to the `request.Messages` list, we maintain the full conversation context. This ensures that the model has access to all previous interactions when generating responses, enabling coherent multi-turn conversations [3](https://docs.lm-kit.com/lm-kit-net/guides/samples-overview/multi-turn-chat-demo.html).

The loop terminates when the model no longer generates tool calls, indicating that it has sufficient information to provide a final response. This response is then displayed to the user:

```csharp
Console.WriteLine("Assistant: " + message.Content);
```

This implementation demonstrates how the Betalgo.Ranul.OpenAI library facilitates the creation of sophisticated chatbots that can seamlessly integrate external data and functions into natural language conversations. By handling tool calls dynamically and maintaining conversation context, developers can create AI-powered applications that provide rich, interactive experiences while leveraging the power of large language models.


---
**Sources:**
- [(1) Multi-Turn Tool Calling - Serverless Fine-tuned LLM Hosting Platform](https://docs.empower.dev/inference/tool-use/multi-turn)
- [(2) Chatbot with Tools - AI SDK UI](https://sdk.vercel.ai/docs/ai-sdk-ui/chatbot-with-tool-calling)
- [(3) Create Multi Turn Chatbot in .NET Applications - LM-Kit Docs](https://docs.lm-kit.com/lm-kit-net/guides/samples-overview/multi-turn-chat-demo.html)


## Advanced Tool Calling Techniques
The Betalgo.Ranul.OpenAI library enables sophisticated implementations of tool calling, allowing developers to create complex AI-driven applications. Here are some advanced use cases and implementation strategies:

1.  Multi-step workflows with decision trees:  
    Implement decision trees where each node represents a tool call using a recursive approach. This allows for complex, branching logic based on previous tool call results:

```csharp
async Task ExecuteWorkflow(Node currentNode, ChatCompletionCreateRequest request)
{
    var toolResult = await ExecuteToolCall(currentNode.ToolDefinition, request);
    request.Messages.Add(ChatMessage.FromTool(toolResult, currentNode.ToolDefinition.Name));
    
    if (currentNode.Children.Any())
    {
        var nextNode = await DetermineNextNode(currentNode.Children, request);
        return await ExecuteWorkflow(nextNode, request);
    }
    
    return toolResult;
}
```

This approach enables the creation of sophisticated workflows where each step depends on the outcome of previous steps [1](https://platform.openai.com/docs/guides/function-calling).

2.  Dynamic tool selection:  
    Implement a meta-tool that selects appropriate tools based on the conversation context:

```csharp
[FunctionDescription(Name = "select_tools", Description = "Selects appropriate tools for the current context")]
public List SelectTools(
    [ParameterDescription(Name = "context", Description = "Current conversation context")]
    string context)
{
    var allTools = GetAllToolDefinitions();
    var selectToolsRequest = new ChatCompletionCreateRequest
    {
        Messages = new List
        {
            ChatMessage.FromSystem("Select the most appropriate tools for the given context."),
            ChatMessage.FromUser($"Context: {context}")
        },
        Tools = allTools
    };
    
    var response = openAiService.ChatCompletion.CreateCompletion(selectToolsRequest);
    return ParseSelectedTools(response);
}
```

This meta-tool can be called at the beginning of each conversation turn to dynamically adjust the available tools [2](https://www.floweq.com/product/decision-tree-software).

3.  Recursive tool calling:  
    Enable tools to call other tools by passing the OpenAIService instance to the tool implementation:

```csharp
public class RecursiveToolService
{
    private readonly IOpenAIService _openAiService;

   public RecursiveToolService(IOpenAIService openAiService)
    {
        _openAiService = openAiService;
    }

   [FunctionDescription(Name = "complex_operation", Description = "Performs a complex operation that may require multiple tool calls")]
    public async Task ComplexOperation(
        [ParameterDescription(Name = "input", Description = "Input data for the operation")]
        string input)
    {
        var subToolResult = await ExecuteSubTool(input);
        return await FinalizeOperation(subToolResult);
    }

   private async Task ExecuteSubTool(string input)
    {
        var subToolRequest = new ChatCompletionCreateRequest
        {
            Messages = new List { ChatMessage.FromUser(input) },
            Tools = GetSubToolDefinitions()
        };
        var response = await _openAiService.ChatCompletion.CreateCompletion(subToolRequest);
        return ExtractSubToolResult(response);
    }
}
```

This approach allows for the creation of hierarchical tool structures, enabling complex operations to be broken down into smaller, manageable steps [3](https://github.com/langchain-ai/langchain/discussions/14962).

4.  Error handling and retries:  
    Implement a robust error handling system with automatic retries using exponential backoff:

```csharp
async Task ExecuteToolWithRetry(ToolDefinition tool, ChatCompletionCreateRequest request)
{
    int maxRetries = 3;
    int retryDelay = 1000; // milliseconds

   for (int attempt = 1; attempt  ExecuteCachedTool(ToolDefinition tool, string input)
    {
        string cacheKey = $"{tool.Name}:{input.GetHashCode()}";

       if (!_cache.TryGetValue(cacheKey, out string cachedResult))
        {
            cachedResult = await ExecuteToolCall(tool, input);
            _cache.Set(cacheKey, cachedResult, TimeSpan.FromMinutes(10));
        }

       return cachedResult;
    }
}
```

This caching mechanism can significantly improve performance for frequently used tools or operations with deterministic outputs [4](https://gorilla.cs.berkeley.edu/blogs/13_bfcl_v3_multi_turn.html).

By implementing these advanced techniques, developers can create AI applications that not only leverage the power of large language models but also integrate seamlessly with complex backend systems and business logic. These implementations enable the creation of highly responsive, context-aware, and efficient AI-driven applications that can handle sophisticated multi-step processes while maintaining a natural conversational flow.


---
**Sources:**
- [(1) Function calling - OpenAI API](https://platform.openai.com/docs/guides/function-calling)
- [(2) Decision Tree Software - FlowEQ](https://www.floweq.com/product/decision-tree-software)
- [(3) Tool lookup during conversation #14962 - GitHub](https://github.com/langchain-ai/langchain/discussions/14962)
- [(4) BFCL V3 • Multi-Turn & Multi-Step Function Calling - Gorilla LLM](https://gorilla.cs.berkeley.edu/blogs/13_bfcl_v3_multi_turn.html)


## Structured Output Implementation
Structured Outputs in Betalgo.Ranul.OpenAI 9.0.0 provide a powerful mechanism for ensuring AI model responses adhere to predefined JSON schemas. This feature is particularly useful for applications requiring precise data formatting and integration with existing systems. Here's an in-depth look at implementing Structured Outputs using the library:

Implementation Details
----------------------

1.  Schema Definition:  
    The `PropertyDefinition` class is central to defining schemas. It offers a fluent API for constructing complex JSON schemas:```csharp
    var schema = PropertyDefinition.DefineObject(
        new Dictionary
        {
            { "name", PropertyDefinition.DefineString("User's name") },
            { "age", PropertyDefinition.DefineInteger("User's age") },
            { "interests", PropertyDefinition.DefineArray(
                PropertyDefinition.DefineString("An interest"),
                minItems: 1,
                maxItems: 5
            )}
        },
        new List { "name", "age" },
        false,
        "User profile information"
    );
    ```
    This example defines a schema for a user profile with required name and age fields, and an optional array of interests [1](https://github.com/betalgo/openai/wiki/Structured-Outputs).
2.  Request Configuration:  
    To use Structured Outputs, configure the `ChatCompletionCreateRequest` with a `ResponseFormat`:```csharp
    var request = new ChatCompletionCreateRequest
    {
        Model = Models.Gpt_4o_2024_08_06,
        Messages = new List
        {
            ChatMessage.FromSystem("Generate a user profile based on the given information."),
            ChatMessage.FromUser("Create a profile for John, a 30-year-old who likes hiking and photography.")
        },
        ResponseFormat = new ResponseFormat
        {
            Type = StaticValues.CompletionStatics.ResponseFormat.JsonSchema,
            JsonSchema = new JsonSchema
            {
                Name = "user_profile",
                Strict = true,
                Schema = schema
            }
        }
    };
    ```
    Setting `Strict = true` ensures the model adheres strictly to the provided schema [2](https://platform.openai.com/docs/guides/structured-outputs).
3.  Handling Responses:  
    Process the structured response using `System.Text.Json`:```csharp
    var result = await openAiService.ChatCompletion.CreateCompletion(request);
    if (result.Successful)
    {
        var profile = JsonSerializer.Deserialize(result.Choices[0].Message.Content);
        Console.WriteLine($"Name: {profile.Name}, Age: {profile.Age}");
        Console.WriteLine($"Interests: {string.Join(", ", profile.Interests)}");
    }
    ```
    

Advanced Techniques
-------------------

1.  Nested Schemas:  
    Betalgo.Ranul.OpenAI supports nested object definitions, allowing for complex data structures:```csharp
    var addressSchema = PropertyDefinition.DefineObject(
        new Dictionary
        {
            { "street", PropertyDefinition.DefineString("Street address") },
            { "city", PropertyDefinition.DefineString("City name") },
            { "zipCode", PropertyDefinition.DefineString("Zip code") }
        },
        new List { "street", "city", "zipCode" },
        false,
        "Address information"
    );
    
    var userSchemaWithAddress = PropertyDefinition.DefineObject(
        new Dictionary
        {
            { "name", PropertyDefinition.DefineString("User's name") },
            { "address", addressSchema }
        },
        new List { "name", "address" },
        false,
        "User information with address"
    );
    ```
    This approach allows for hierarchical data representation, useful in complex applications [3](https://json-schema.org/learn/getting-started-step-by-step).
2.  Enum Constraints:  
    Use `DefineEnum` to restrict string values to a predefined set:```csharp
    var userTypeSchema = PropertyDefinition.DefineEnum(
        new List { "regular", "premium", "admin" },
        "User account type"
    );
    ```
    This ensures the model only generates valid user types [4](https://docs.dndocs.com/n/Betalgo.OpenAI/8.6.1/api/OpenAI.ObjectModels.SharedModels.PropertyDefinition.html).
3.  Numeric Constraints:  
    Apply constraints to numeric fields for data validation:```csharp
    var ageSchema = PropertyDefinition.DefineInteger(
        "User's age",
        minimum: 0,
        maximum: 120
    );
    ```
    This prevents the model from generating invalid age values [4](https://docs.dndocs.com/n/Betalgo.OpenAI/8.6.1/api/OpenAI.ObjectModels.SharedModels.PropertyDefinition.html).

Best Practices
--------------

1.  Schema Complexity:  
    Keep schemas as simple as possible while meeting your requirements. Complex schemas with deep nesting can increase processing time and the likelihood of errors [5](https://platform.openai.com/docs/guides/structured-outputs/introduction).
2.  Error Handling:  
    Implement robust error handling to manage cases where the model fails to adhere to the schema:```csharp
    if (!result.Successful)
    {
        Console.WriteLine($"Error: {result.Error?.Message ?? "Unknown error"}");
        // Implement fallback logic or retry mechanism
    }
    ```
    
3.  Testing:  
    Utilize the `PropertyDefinitionGenerator` for offline schema testing:```csharp
    var generatedSchema = PropertyDefinitionGenerator.GenerateFromType();
    // Use generatedSchema for offline validation
    ```
    This approach allows for thorough testing without making API calls [6].
4.  Performance Optimization:  
    When dealing with large volumes of requests, consider batch processing and respect API rate limits to optimize performance [5](https://platform.openai.com/docs/guides/structured-outputs/introduction).

By leveraging these advanced features and best practices, developers can create robust, schema-compliant AI integrations using Betalgo.Ranul.OpenAI 9.0.0. Structured Outputs provide a powerful tool for ensuring data consistency and reliability in AI-driven applications, particularly in scenarios requiring precise data formatting and seamless system integration.


---
**Sources:**
- [(1) Structured Outputs · betalgo/openai Wiki - GitHub](https://github.com/betalgo/openai/wiki/Structured-Outputs)
- [(2) Structured Outputs - OpenAI API](https://platform.openai.com/docs/guides/structured-outputs)
- [(3) Creating your first schema - JSON Schema](https://json-schema.org/learn/getting-started-step-by-step)
- [(4) Class PropertyDefinition | Betalgo.OpenAI 8.6.1 - DNDocs](https://docs.dndocs.com/n/Betalgo.OpenAI/8.6.1/api/OpenAI.ObjectModels.SharedModels.PropertyDefinition.html)
- [(5) Structured Outputs - OpenAI Platform](https://platform.openai.com/docs/guides/structured-outputs/introduction)
- [(6) Betalgo.OpenAI Structured Outputs - YouTube](https://www.youtube.com/embed/KoztI8qZass?autoplay=1&color=white&playsinline=true&enablejsapi=1&origin=https%3A%2F%2Fwww.perplexity.ai&widgetid=1)


## Workflow-Driven Task Execution
Workflow-driven structured outputs in Betalgo.Ranul.OpenAI enable precise task planning and execution by leveraging predefined schemas to guide AI responses. This approach ensures that each step of a workflow is clearly defined, executed, and validated against a structured format, enhancing reliability and integration with external systems.

Workflow Planning with Structured Outputs
-----------------------------------------

1.  **Defining the Workflow Schema**:  
    A workflow schema outlines the structure of tasks and their sequential or parallel dependencies. Using the `PropertyDefinition` class, developers can define workflows as nested objects or arrays representing steps, sub-steps, and expected outputs:```csharp
    var workflowSchema = PropertyDefinition.DefineObject(
        new Dictionary
        {
            { "task_name", PropertyDefinition.DefineString("Name of the task") },
            { "steps", PropertyDefinition.DefineArray(
                PropertyDefinition.DefineObject(
                    new Dictionary
                    {
                        { "step_name", PropertyDefinition.DefineString("Name of the step") },
                        { "status", PropertyDefinition.DefineEnum(new[] { "pending", "in_progress", "completed" }, "Status of the step") },
                        { "output", PropertyDefinition.DefineString("Output of the step") }
                    },
                    new List { "step_name", "status" },
                    false,
                    "Details of a workflow step"
                )
            )}
        },
        new List { "task_name", "steps" },
        false,
        "Workflow structure with tasks and steps"
    );
    ```
    This schema ensures that each task contains a list of steps with a defined name, status, and optional output.
2.  **Configuring the Task Execution Request**:  
    To execute a workflow, configure a `ChatCompletionCreateRequest` with the defined schema for structured outputs:```csharp
    var request = new ChatCompletionCreateRequest
    {
        Messages = new List
        {
            ChatMessage.FromSystem("You are an assistant that plans and executes tasks step by step."),
            ChatMessage.FromUser("Plan a workflow for organizing a conference.")
        },
        Model = Models.Gpt_4o,
        ResponseFormat = new ResponseFormat
        {
            Type = StaticValues.CompletionStatics.ResponseFormat.JsonSchema,
            JsonSchema = new JsonSchema
            {
                Name = "conference_workflow",
                Strict = true,
                Schema = workflowSchema
            }
        }
    };
    ```
    
3.  **Executing Steps Dynamically**:  
    As the model generates structured responses adhering to the workflow schema, developers can dynamically execute each step and update its status:```csharp
    var result = await openAiService.ChatCompletion.CreateCompletion(request);
    if (result.Successful)
    {
        var workflow = JsonSerializer.Deserialize(result.Choices.First().Message.Content);
        foreach (var step in workflow.Steps)
        {
            Console.WriteLine($"Executing: {step.StepName}");
            // Simulate execution
            step.Status = "completed";
            step.Output = $"Result of {step.StepName}";
        }
    }
    ```
    
4.  **Iterative Updates**:  
    Update the AI model with progress after executing each step to maintain context and ensure coherent task management:```csharp
    request.Messages.Add(ChatMessage.FromAssistant(JsonSerializer.Serialize(workflow)));
    var updatedResult = await openAiService.ChatCompletion.CreateCompletion(request);
    ```
    

Advanced Use Cases
------------------

1.  **Parallel Task Execution**:  
    Define workflows with parallel steps using additional metadata to indicate concurrency:```csharp
    { 
        "step_name": "Book Venue",
        "parallel": true,
        "dependencies": ["Research Venues"]
    }
    ```
    
2.  **Error Handling in Workflows**:  
    Incorporate error states into the schema to manage failures gracefully:```csharp
    { 
        "status": ["pending", "in_progress", "completed", "error"],
        "error_message": PropertyDefinition.DefineString("Error details if any")
    }
    ```
    
3.  **Dynamic Workflow Adjustments**:  
    Allow real-time modifications to workflows by enabling users to add or reorder steps during execution. The schema can include an `editable` flag for flexibility.

Best Practices
--------------

*   **Strict Schema Adherence**: Use `strict: true` to ensure all responses conform to the defined workflow structure, reducing errors during execution.
*   **Incremental Updates**: Continuously update the model with task progress to maintain alignment between planned and executed workflows.
*   **Testing**: Validate workflows offline using mock data and schema validation tools before deployment.

By leveraging structured outputs for workflow-driven tasks, Betalgo.Ranul.OpenAI enables seamless planning, execution, and monitoring of complex processes while ensuring data consistency and adherence to predefined structures [1](https://tettra.com/article/workflows-vs-processes/) [2](https://js.langchain.com/docs/concepts/structured_outputs/) [3](https://www.leewayhertz.com/structured-outputs-in-llms/).


---
**Guidelines for Choosing Between Structured Outputs and Function Calling in Workflow Automation:**

1. Structured Outputs

Use structured outputs when:

Predefined Automation Steps:

Scenario: You have tasks that follow a clear, linear sequence.

Example: Breaking down a data processing task into steps like data extraction, transformation, and loading (ETL). Each step outputs data in a consistent format for the next step to process.


Workflow Standardization:

Scenario: The workflow requires standardized data formats for seamless integration.

Example: Generating JSON or XML schemas for API responses to ensure compatibility across different systems.


Predictable Input and Output Patterns:

Scenario: Inputs are consistent, and outputs need to adhere to a strict structure.

Example: Creating standardized reports where each section (e.g., introduction, data analysis, conclusion) follows a specific format for automated reporting systems.



Benefits:

Consistency: Ensures that outputs are uniform, facilitating reliable downstream processing.

Repeatability: Standardized formats allow workflows to be easily replicated and scaled.

Ease of Integration: Structured data aligns well with existing pipelines and data exchange protocols.



---

2. Function Calling

Use function calling when:

Ad Hoc Decisions:

Scenario: Tasks require on-the-spot reasoning and context-sensitive actions.

Example: Determining the appropriate marketing strategy based on real-time sales data and customer feedback.


Dynamic Workflow Execution:

Scenario: Workflows need to adapt to varying inputs without a fixed sequence.

Example: An automated support system that selects different troubleshooting steps based on the user's specific issue reported in real-time.


On-the-Fly Adjustments:

Scenario: The workflow must respond to live data and interact with external systems dynamically.

Example: An investment bot that analyzes live market data and executes trades by invoking external financial APIs as conditions change.



Benefits:

Flexibility: Capable of handling diverse and unexpected inputs by leveraging the model’s reasoning capabilities.

Adaptability: Easily adjusts to changes in real-time, ensuring the workflow remains effective under varying conditions.

Integration with External Systems: Can dynamically interact with APIs and other external services to perform necessary actions based on current data.



---

3. Key Principles

Structured Outputs:

Best For: Workflows with predictable paths and well-defined steps.

Advantages: Promotes consistency, repeatability, and seamless integration within automated pipelines.


Function Calling:

Best For: Dynamic, decision-driven tasks that benefit from the model's ability to reason and trigger appropriate actions.

Advantages: Offers flexibility and rapid adaptation to changing inputs and contexts.




---

Actionable Examples

Structured Outputs Example:

Automated Report Generation

Process:

1. Data Collection: Extract data from databases.


2. Data Processing: Transform data into the required format.


3. Report Assembly: Generate a report in JSON format.



Why Structured Outputs: Each step produces a predictable output, ensuring the next step can process the data without issues.


Function Calling Example:

Real-Time Customer Support Bot

Process:

1. User Query: Receive a customer question.


2. Decision Making: Analyze the query to determine the appropriate response or action.


3. Function Invocation: Call external APIs to fetch information or perform actions like creating a support ticket.



Why Function Calling: The bot needs to adapt responses based on diverse and unpredictable user inputs, requiring dynamic decision-making.



---

Additional Notes

Consistency and Repeatability with Structured Outputs:

Leveraging structured outputs ensures that automated pipelines remain reliable and maintainable. This is crucial for large-scale operations where consistency reduces errors and simplifies troubleshooting.


Flexibility and Adaptation with Function Calling:

Function calling excels in environments where inputs can vary widely and decisions must be made swiftly. This approach harnesses the model's ability to reason and interact with external systems, providing robust solutions in dynamic scenarios.
