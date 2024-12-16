### **Betalgo.Ranul.OpenAI Cheat Sheet**

---

#### **🔧 Installation**

- **Core Library:**
  ```shell
  Install-Package Betalgo.Ranul.OpenAI
  ```
- **Utilities (Experimental):**
  ```shell
  Install-Package Betalgo.OpenAI.Utilities
  ```

---

#### **🛠 Initialization**

- **Without Dependency Injection:**
  ```csharp
  var openAIService = new OpenAIService(new OpenAIOptions() {
      ApiKey = Environment.GetEnvironmentVariable("MY_OPEN_AI_API_KEY")
  });
  ```
- **With Dependency Injection:**
  ```csharp
  serviceCollection.AddOpenAIService();
  
  // Custom Configuration
  serviceCollection.AddOpenAIService(settings => {
      settings.ApiKey = Environment.GetEnvironmentVariable("MY_OPEN_AI_API_KEY");
  });
  ```
- **Secure API Keys:**
  - Use `secrets.json`:
    ```json
    "OpenAIServiceOptions": {
        "ApiKey": "Your api key",
        "Organization": "Org ID (optional)",
        "UseBeta": "true/false (optional)"
    }
    ```
  - Retrieve:
    ```csharp
    var openAiService = serviceProvider.GetRequiredService<IOpenAIService>();
    ```

---

#### **📚 Namespace Imports**

- Key Namespaces:
  - `Betalgo.OpenAI.Utilities.FunctionCalling`
  - `Betalgo.Ranul.OpenAI.Builders`
  - `Betalgo.Ranul.OpenAI.Extensions`
  - `Betalgo.Ranul.OpenAI.Interfaces`
  - `Betalgo.Ranul.OpenAI.Managers`
  - `Betalgo.Ranul.OpenAI.ObjectModels.RequestModels`
  - `Betalgo.Ranul.OpenAI.ObjectModels.SharedModels`
  - `Betalgo.Ranul.OpenAI`

---

#### **🔧 Function & Tool Definition**

- **Define Function:**
  ```csharp
  var function = new FunctionDefinitionBuilder("get_weather", "Get current weather")
      .AddParameter("location", PropertyDefinition.DefineString("City, e.g., San Francisco, CA"))
      .AddParameter("format", PropertyDefinition.DefineEnum(new[] { "celsius", "fahrenheit" }, "Temp unit"))
      .Validate()
      .Build();
  
  var tool = ToolDefinition.DefineFunction(function);
  ```
- **Property Definitions:**
  - String, Integer, Number, Boolean, Enum, Array, Object
  - Example:
    ```csharp
    PropertyDefinition.DefineString("desc", minLength:1, maxLength:100, pattern: @"^[a-zA-Z0-9]+$")
    ```

---

#### **⚙️ Attribute-Driven Tool Generation**

- **Attributes:**
  ```csharp
  [FunctionDescription(Name = "get_weather", Description = "Get current weather")]
  public WeatherInfo GetWeather(
      [ParameterDescription(Name = "location", Description = "City, e.g., 'San Francisco, CA'")] string location,
      [ParameterDescription(Name = "unit", Description = "Temp unit", Enum = "celsius,fahrenheit", Required = false)] string unit = "celsius")
  {
      // Implementation
  }
  ```
- **Generate Tools:**
  ```csharp
  var tools = FunctionCallingHelper.GetToolDefinitions(new WeatherService());
  ```

---

#### **💬 Multi-Turn Tool Calling**

- **Create Request:**
  ```csharp
  var request = new ChatCompletionCreateRequest
  {
      Messages = new List<ChatMessage>
      {
          ChatMessage.FromSystem("You are a helpful assistant."),
          ChatMessage.FromUser("What's the weather in New York?")
      },
      Tools = FunctionCallingHelper.GetToolDefinitions(new WeatherService()),
      ToolChoice = ToolChoice.Auto,
      Model = Models.Gpt_4o
  };
  ```
- **Handle Responses:**
  ```csharp
  var response = await openAIService.ChatCompletion.CreateCompletion(request);
  var message = response.Choices.First().Message;
  request.Messages.Add(ChatMessage.FromAssistant(message));

  while (message.ToolCalls != null)
  {
      foreach (var toolCall in message.ToolCalls)
      {
          var result = FunctionCallingHelper.CallFunction(toolCall.FunctionCall, new WeatherService());
          request.Messages.Add(ChatMessage.FromTool(result, toolCall.Id));
      }
      response = await openAIService.ChatCompletion.CreateCompletion(request);
      message = response.Choices.First().Message;
      request.Messages.Add(ChatMessage.FromAssistant(message));
  }

  Console.WriteLine("Assistant: " + message.Content);
  ```

---

#### **🚀 Advanced Tool Calling**

1. **Multi-Step Workflows:**
   ```csharp
   async Task ExecuteWorkflow(Node currentNode, ChatCompletionCreateRequest request) { /*...*/ }
   ```
2. **Dynamic Tool Selection:**
   ```csharp
   public List<ToolDefinition> SelectTools(string context) { /*...*/ }
   ```
3. **Recursive Tool Calls:**
   ```csharp
   public async Task ComplexOperation(string input) { /*...*/ }
   ```
4. **Error Handling & Caching:**
   ```csharp
   async Task ExecuteToolWithRetry(ToolDefinition tool, ChatCompletionCreateRequest request) { /*...*/ }
   string ExecuteCachedTool(ToolDefinition tool, string input) { /*...*/ }
   ```

---

#### **📊 Structured Outputs**

- **Define Schema:**
  ```csharp
  var schema = PropertyDefinition.DefineObject(
      new Dictionary<string, PropertyDefinition>
      {
          { "name", PropertyDefinition.DefineString("User's name") },
          { "age", PropertyDefinition.DefineInteger("User's age") },
          { "interests", PropertyDefinition.DefineArray(PropertyDefinition.DefineString("Interest"), 1, 5, true) }
      },
      new List<string> { "name", "age" },
      false,
      "User profile"
  );
  ```
- **Configure Request:**
  ```csharp
  var request = new ChatCompletionCreateRequest
  {
      Model = Models.Gpt_4o_2024_08_06,
      Messages = new List<ChatMessage>
      {
          ChatMessage.FromSystem("Generate a user profile."),
          ChatMessage.FromUser("Create a profile for John, 30, likes hiking.")
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
- **Handle Response:**
  ```csharp
  var result = await openAIService.ChatCompletion.CreateCompletion(request);
  if (result.Successful)
  {
      var profile = JsonSerializer.Deserialize<UserProfile>(result.Choices.First().Message.Content);
      Console.WriteLine($"Name: {profile.Name}, Age: {profile.Age}, Interests: {string.Join(", ", profile.Interests)}");
  }
  ```

---

#### **🔄 Workflow-Driven Task Execution**

- **Define Workflow Schema:**
  ```csharp
  var workflowSchema = PropertyDefinition.DefineObject(
      new Dictionary<string, PropertyDefinition>
      {
          { "task_name", PropertyDefinition.DefineString("Task name") },
          { "steps", PropertyDefinition.DefineArray(
              PropertyDefinition.DefineObject(
                  new Dictionary<string, PropertyDefinition>
                  {
                      { "step_name", PropertyDefinition.DefineString("Step name") },
                      { "status", PropertyDefinition.DefineEnum(new[] { "pending", "in_progress", "completed" }, "Step status") },
                      { "output", PropertyDefinition.DefineString("Step output") }
                  },
                  new List<string> { "step_name", "status" },
                  false,
                  "Step details"
              )
          )}
      },
      new List<string> { "task_name", "steps" },
      false,
      "Workflow structure"
  );
  ```
- **Execute Workflow with Tool Calls:**
  ```csharp
  var request = new ChatCompletionCreateRequest
  {
      Messages = new List<ChatMessage>
      {
          ChatMessage.FromSystem("Plan and execute a workflow."),
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
      },
      Tools = FunctionCallingHelper.GetToolDefinitions(new WorkflowService()), // Add relevant tools
      ToolChoice = ToolChoice.Auto
  };

  var result = await openAIService.ChatCompletion.CreateCompletion(request);

  if (result.Successful)
  {
      var workflow = JsonSerializer.Deserialize<Workflow>(result.Choices.First().Message.Content);

      foreach (var step in workflow.Steps)
      {
          // Add each step as a user message
          request.Messages.Add(ChatMessage.FromUser($"Execute step: {step.StepName}"));

          // Update the step status to "in_progress"
          step.Status = "in_progress";

          // Call the chat completion API to handle the tool execution
          var stepResponse = await openAIService.ChatCompletion.CreateCompletion(request);

          if (stepResponse.Successful)
          {
              var toolResults = stepResponse.Choices.First().Message.ToolCalls;

              foreach (var toolCall in toolResults)
              {
                  // Execute the tool and capture the output
                  var toolOutput = FunctionCallingHelper.CallFunction(toolCall.FunctionCall, new WorkflowService());

                  // Update the step status and output
                  step.Status = "completed";
                  step.Output = toolOutput;

                  // Add tool response as a message
                  request.Messages.Add(ChatMessage.FromTool(toolOutput, toolCall.Id));
              }
          }
          else
          {
              Console.WriteLine($"Error executing step '{step.StepName}': {stepResponse.Error.Message}");
              step.Status = "failed";
          }
      }

      // Update AI with completed workflow
      request.Messages.Add(ChatMessage.FromAssistant(JsonSerializer.Serialize(workflow)));

      var finalResponse = await openAIService.ChatCompletion.CreateCompletion(request);

      if (finalResponse.Successful)
      {
          Console.WriteLine("Workflow completed: " + finalResponse.Choices.First().Message.Content);
      }
  }
  else
  {
      Console.WriteLine($"Error planning workflow: {result.Error.Message}");
  }
  ```

---

#### **⚖️ Structured Outputs vs. Function Calling**

- **Structured Outputs:**
  - **Use When:**
    - Linear, predefined workflows
    - Standardized data formats
    - Predictable I/O patterns
  - **Benefits:** Consistency, repeatability, easy integration
  - **Example:** Automated ETL processes

- **Function Calling:**
  - **Use When:**
    - Dynamic, context-sensitive tasks
    - Real-time decisions and actions
    - Interaction with external APIs
  - **Benefits:** Flexibility, adaptability, external system integration
  - **Example:** Real-time customer support bots

---

#### **✅ Best Practices**

- **Schema Design:**
  - Keep schemas simple
  - Use strict mode for consistency
- **Error Handling:**
  - Implement retries and fallbacks
  - Validate responses against schemas
- **Testing:**
  - Use `PropertyDefinitionGenerator` for offline tests
  - Validate with mock data
- **Performance:**
  - Optimize API calls
  - Implement caching for frequent tools

---

#### **📌 Quick Code Snippets**

- **Set Default Model:**
  ```csharp
  openAIService.SetDefaultModelId(Models.Gpt_4o);
  ```
- **Create Chat Completion:**
  ```csharp
  var completion = await openAIService.ChatCompletion.CreateCompletion(new ChatCompletionCreateRequest { /*...*/ });
  if (completion.Successful)
      Console.WriteLine(completion.Choices.First().Message.Content);
  ```
- **Secure API Key Retrieval:**
  ```csharp
  ApiKey = Environment.GetEnvironmentVariable("MY_OPEN_AI_API_KEY")
  ```
