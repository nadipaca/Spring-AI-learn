# Spring AI - Complete Interview Preparation Guide 🚀

A comprehensive guide to Spring AI for interview preparation covering all essential concepts, patterns, and practical implementations.

## 📑 Table of Contents
- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
  - [What is Spring AI?](#what-is-spring-ai)
  - [AI Models](#ai-models)
  - [Chat Client](#chat-client)
  - [Prompts](#prompts)
  - [Prompt Templates](#prompt-templates)
  - [Roles in Spring AI](#roles-in-spring-ai)
- [Working with Outputs](#working-with-outputs)
  - [Standard Output Converter](#standard-output-converter)
  - [Structured Output Converter](#structured-output-converter)
- [Different Ways of Passing Prompts](#different-ways-of-passing-prompts)
- [RAG (Retrieval Augmented Generation)](#rag-retrieval-augmented-generation)
  - [Sentiment Analysis with RAG](#sentiment-analysis-with-rag)
- [AI Multimodal](#ai-multimodal)
- [UI Integration](#ui-integration)
  - [Feedback Form Example](#feedback-form-example)
- [Code Examples](#code-examples)
- [Interview Questions](#interview-questions)
- [Best Practices](#best-practices)

---

## Introduction

**Spring AI** is a framework that provides abstractions for integrating AI and Large Language Models (LLMs) into Spring applications. It simplifies the development of AI-powered applications by offering a consistent API across different AI providers (OpenAI, Azure OpenAI, Anthropic, etc.).

### Key Features:
- 🔌 Portable API across different AI providers
- 💬 Chat and Text generation
- 🖼️ Multimodal support (text, images, audio)
- 📝 Prompt engineering support
- 🎯 Structured output conversion
- 📚 RAG (Retrieval Augmented Generation)
- 🔄 Streaming responses
- 💾 Vector database integration

---

## Getting Started

### Maven Dependencies

```xml
<dependencies>
    <!-- Spring AI BOM -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-bom</artifactId>
        <version>1.0.0-M1</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
    
    <!-- OpenAI Integration -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>
    
    <!-- For RAG - Vector Store -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

### Configuration

```properties
# application.properties
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4
spring.ai.openai.chat.options.temperature=0.7
```

---

## Core Concepts

### What is Spring AI?

Spring AI is a framework that:
- Provides abstraction layer for AI/LLM integration
- Supports multiple AI providers through a unified API
- Offers Spring Boot auto-configuration
- Integrates seamlessly with Spring ecosystem

**Interview Key Point**: Spring AI uses the **portable service abstraction** pattern, allowing you to switch between AI providers without changing your business logic.

### AI Models

**Models** are the AI engines that process your requests. Different models have different capabilities:

| Model Type | Use Case | Example |
|-----------|----------|---------|
| **Chat Models** | Conversational AI | GPT-4, Claude, Gemini |
| **Text Generation** | Content creation | GPT-3.5-turbo |
| **Embedding Models** | Vector representations | text-embedding-ada-002 |
| **Image Models** | Image generation/analysis | DALL-E, GPT-4-Vision |

```java
@Service
public class AIModelService {
    
    @Autowired
    private ChatModel chatModel; // Injected by Spring Boot
    
    public String simpleChat(String userMessage) {
        return chatModel.call(userMessage);
    }
}
```

**Interview Key Point**: Models are injected as beans, and Spring Boot auto-configures them based on your dependencies and properties.

### Chat Client

The **ChatClient** is a fluent API for interacting with AI models. It provides a builder pattern for constructing requests.

```java
@Service
public class ChatService {
    
    private final ChatClient chatClient;
    
    public ChatService(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }
    
    public String chat(String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
}
```

**Key Methods**:
- `prompt()` - Start building a prompt
- `user(String)` - Add user message
- `system(String)` - Add system message
- `call()` - Execute the request
- `stream()` - Get streaming response
- `content()` - Extract response content

### Prompts

A **Prompt** is the input you send to an AI model. It can contain:
- User messages
- System messages (instructions for the AI)
- Context information
- Examples (few-shot learning)

```java
// Simple prompt
String response = chatClient.prompt()
    .user("What is Spring Boot?")
    .call()
    .content();

// Prompt with system message
String response = chatClient.prompt()
    .system("You are a helpful Java expert")
    .user("Explain dependency injection")
    .call()
    .content();
```

**Interview Key Point**: Prompts are immutable objects that encapsulate the conversation context.

### Prompt Templates

**Prompt Templates** allow you to create reusable prompt structures with placeholders.

```java
@Service
public class PromptTemplateService {
    
    private final ChatClient chatClient;
    
    public PromptTemplateService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }
    
    public String generateCode(String language, String task) {
        String template = """
            You are an expert {language} developer.
            Task: {task}
            Provide clean, production-ready code with comments.
            """;
        
        return chatClient.prompt()
            .user(u -> u.text(template)
                .param("language", language)
                .param("task", task))
            .call()
            .content();
    }
}
```

**Benefits**:
- ✅ Reusability
- ✅ Parameterization
- ✅ Consistency
- ✅ Maintainability

**Interview Key Point**: Templates support placeholder substitution using `{paramName}` syntax.

### Roles in Spring AI

Roles define who is sending each message in a conversation:

| Role | Purpose | Example |
|------|---------|---------|
| **SYSTEM** | Sets AI behavior and context | "You are a helpful assistant" |
| **USER** | User's input/questions | "What is Spring AI?" |
| **ASSISTANT** | AI's responses | "Spring AI is a framework..." |
| **FUNCTION** | Function call results | Return data from tools |

```java
public String conversationWithRoles() {
    return chatClient.prompt()
        .system("You are a Spring Boot expert specializing in microservices")
        .user("How do I implement circuit breaker pattern?")
        .call()
        .content();
}
```

**Interview Key Point**: System messages persist throughout the conversation and guide the AI's behavior.

---

## Working with Outputs

### Standard Output Converter

Standard output returns responses as simple strings.

```java
@Service
public class SimpleOutputService {
    
    private final ChatClient chatClient;
    
    public SimpleOutputService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }
    
    public String getSimpleResponse(String question) {
        return chatClient.prompt()
            .user(question)
            .call()
            .content(); // Returns String
    }
}
```

### Structured Output Converter

Convert AI responses into Java objects automatically.

```java
// Define your data model
public record Recipe(
    String name,
    List<String> ingredients,
    List<String> steps,
    int prepTimeMinutes
) {}

@Service
public class StructuredOutputService {
    
    private final ChatClient chatClient;
    
    public StructuredOutputService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }
    
    public Recipe getRecipe(String dish) {
        return chatClient.prompt()
            .user("Provide a recipe for " + dish)
            .call()
            .entity(Recipe.class); // Converts to Recipe object
    }
    
    public List<Recipe> getMultipleRecipes(String cuisine) {
        return chatClient.prompt()
            .user("Provide 3 recipes for " + cuisine + " cuisine")
            .call()
            .entity(new ParameterizedTypeReference<List<Recipe>>() {});
    }
}
```

**Interview Key Point**: Spring AI uses JSON schema and the AI's ability to generate structured JSON to map responses to Java objects.

---

## Different Ways of Passing Prompts

### 1. Direct String

```java
String response = chatClient.prompt()
    .user("What is Spring AI?")
    .call()
    .content();
```

### 2. Using Prompt Templates

```java
String template = "Translate '{text}' to {language}";
String response = chatClient.prompt()
    .user(u -> u.text(template)
        .param("text", "Hello World")
        .param("language", "Spanish"))
    .call()
    .content();
```

### 3. Using Message Objects

```java
import org.springframework.ai.chat.messages.*;

List<Message> messages = List.of(
    new SystemMessage("You are a helpful assistant"),
    new UserMessage("Explain microservices")
);

Prompt prompt = new Prompt(messages);
ChatResponse response = chatModel.call(prompt);
```

### 4. With Options/Parameters

```java
ChatResponse response = chatClient.prompt()
    .user("Write a poem about Spring")
    .options(ChatOptionsBuilder.builder()
        .withTemperature(0.9)
        .withMaxTokens(500)
        .build())
    .call()
    .chatResponse();
```

### 5. Using Advisors (Advanced)

```java
String response = chatClient.prompt()
    .user("Analyze this code...")
    .advisors(new MessageChatMemoryAdvisor(chatMemory))
    .call()
    .content();
```

**Interview Key Point**: Different approaches suit different scenarios - simple strings for quick queries, templates for reusability, and advisors for advanced features like memory and function calling.

---

## RAG (Retrieval Augmented Generation)

**RAG** enhances AI responses by retrieving relevant information from a knowledge base before generating a response.

### Architecture:
1. **Document Ingestion** → Split into chunks → Generate embeddings → Store in vector DB
2. **Query Time** → Convert query to embedding → Find similar documents → Use as context
3. **Generation** → Send context + query to LLM → Get informed response

### Basic RAG Implementation

```java
@Service
public class RAGService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    public RAGService(ChatClient.Builder builder, VectorStore vectorStore) {
        this.chatClient = builder.build();
        this.vectorStore = vectorStore;
    }
    
    // Advisor-based RAG (Recommended)
    public String queryWithRAG(String query) {
        return chatClient.prompt()
            .user(query)
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .content();
    }
    
    // Manual RAG implementation
    public String manualRAG(String query) {
        // 1. Search for relevant documents
        List<Document> similarDocs = vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(5)
        );
        
        // 2. Build context from documents
        String context = similarDocs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        // 3. Create prompt with context
        String promptText = """
            Use the following context to answer the question.
            
            Context:
            {context}
            
            Question: {question}
            
            Answer:
            """;
        
        return chatClient.prompt()
            .user(u -> u.text(promptText)
                .param("context", context)
                .param("question", query))
            .call()
            .content();
    }
}
```

### Document Ingestion

```java
@Service
public class DocumentIngestionService {
    
    private final VectorStore vectorStore;
    private final EmbeddingModel embeddingModel;
    
    public void ingestDocuments(List<String> texts) {
        List<Document> documents = texts.stream()
            .map(text -> new Document(text))
            .toList();
        
        vectorStore.add(documents);
    }
    
    public void ingestFromPDF(Resource pdfResource) {
        // Using PDF reader
        PagePdfDocumentReader reader = new PagePdfDocumentReader(
            pdfResource,
            PdfDocumentReaderConfig.builder()
                .withPageExtractedTextFormatter(
                    new ExtractedTextFormatter.Builder()
                        .withNumberOfBottomTextLinesToDelete(3)
                        .build())
                .build()
        );
        
        // Split into chunks
        TokenTextSplitter splitter = new TokenTextSplitter();
        List<Document> documents = splitter.apply(reader.get());
        
        // Store in vector database
        vectorStore.add(documents);
    }
}
```

### Sentiment Analysis with RAG

```java
public record SentimentAnalysis(
    String sentiment,  // POSITIVE, NEGATIVE, NEUTRAL
    double score,      // 0.0 to 1.0
    List<String> keywords,
    String reasoning
) {}

@Service
public class SentimentAnalysisService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    public SentimentAnalysis analyzeSentiment(String text) {
        // Use RAG to get relevant sentiment analysis examples/context
        return chatClient.prompt()
            .system("""
                You are a sentiment analysis expert. Analyze the sentiment 
                of the given text and provide structured output.
                """)
            .user(text)
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .entity(SentimentAnalysis.class);
    }
    
    public SentimentAnalysis analyzeFeedback(String feedback) {
        String prompt = """
            Analyze the sentiment of this customer feedback: "{feedback}"
            
            Provide:
            - Overall sentiment (POSITIVE/NEGATIVE/NEUTRAL)
            - Confidence score (0.0 to 1.0)
            - Key phrases that influenced your decision
            - Brief reasoning
            """;
        
        return chatClient.prompt()
            .user(u -> u.text(prompt).param("feedback", feedback))
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .entity(SentimentAnalysis.class);
    }
}
```

**Interview Key Point**: RAG is essential for:
- Reducing hallucinations
- Providing up-to-date information
- Grounding responses in factual data
- Building domain-specific AI applications

---

## AI Multimodal

**Multimodal AI** can process and generate multiple types of content (text, images, audio, video).

### Image Understanding (Vision)

```java
@Service
public class MultimodalService {
    
    private final ChatClient chatClient;
    
    public String analyzeImage(Resource imageResource) throws IOException {
        byte[] imageData = imageResource.getContentAsByteArray();
        
        return chatClient.prompt()
            .user(u -> u.text("What's in this image? Describe it in detail.")
                .media(MimeTypeUtils.IMAGE_PNG, imageData))
            .call()
            .content();
    }
    
    public String analyzeImageFromUrl(String imageUrl) throws MalformedURLException {
        // Validate URL to prevent SSRF attacks
        URL url = new URL(imageUrl);
        if (!url.getProtocol().equals("https")) {
            throw new IllegalArgumentException("Only HTTPS URLs are allowed");
        }
        
        return chatClient.prompt()
            .user(u -> u.text("Describe this image")
                .media(MimeTypeUtils.IMAGE_JPEG, url))
            .call()
            .content();
    }
}
```

### Multiple Images

```java
public String compareImages(Resource image1, Resource image2) throws IOException {
    return chatClient.prompt()
        .user(u -> u.text("Compare these two images and describe the differences")
            .media(MimeTypeUtils.IMAGE_PNG, image1.getContentAsByteArray())
            .media(MimeTypeUtils.IMAGE_PNG, image2.getContentAsByteArray()))
        .call()
        .content();
}
```

### Image Generation (DALL-E)

```java
@Service
public class ImageGenerationService {
    
    private final ImageModel imageModel;
    
    public ImageGenerationService(ImageModel imageModel) {
        this.imageModel = imageModel;
    }
    
    public String generateImage(String description) {
        ImagePrompt prompt = new ImagePrompt(description);
        ImageResponse response = imageModel.call(prompt);
        
        return response.getResult()
            .getOutput()
            .getUrl(); // Returns URL of generated image
    }
    
    public List<String> generateMultipleImages(String description, int count) {
        ImageOptions options = ImageOptionsBuilder.builder()
            .withN(count)
            .withWidth(1024)
            .withHeight(1024)
            .build();
        
        ImagePrompt prompt = new ImagePrompt(description, options);
        ImageResponse response = imageModel.call(prompt);
        
        return response.getResults().stream()
            .map(result -> result.getOutput().getUrl())
            .toList();
    }
}
```

### Audio/Speech Processing

```java
@Service
public class AudioService {
    
    private final ChatClient chatClient;
    
    // Transcribe audio to text
    public String transcribeAudio(Resource audioResource) {
        // Using OpenAI Whisper or similar
        return chatClient.prompt()
            .user(u -> u.text("Transcribe this audio")
                .media(MimeTypeUtils.parseMediaType("audio/mpeg"), 
                       audioResource))
            .call()
            .content();
    }
    
    // Text to speech
    public byte[] textToSpeech(String text) {
        // NOTE: This is a placeholder implementation
        // Actual implementation depends on the AI provider:
        // - OpenAI TTS API: Use OpenAI's text-to-speech endpoint
        // - Azure Cognitive Services: Use Azure Speech SDK
        // - Google Cloud: Use Google Text-to-Speech API
        // The implementation would call the provider's API and return audio bytes
        throw new UnsupportedOperationException(
            "Text-to-speech implementation depends on your AI provider"
        );
    }
}
```

**Interview Key Point**: Multimodal capabilities are provider-dependent. GPT-4-Vision supports image understanding, DALL-E for generation, Whisper for audio transcription.

---

## UI Integration

### Feedback Form Example

This example shows how to create a Spring Boot backend with REST API to handle feedback forms using AI.

#### Backend Controller

```java
@RestController
@RequestMapping("/api/feedback")
@CrossOrigin(origins = "*")
public class FeedbackController {
    
    private final FeedbackService feedbackService;
    
    public FeedbackController(FeedbackService feedbackService) {
        this.feedbackService = feedbackService;
    }
    
    @PostMapping("/submit")
    public ResponseEntity<FeedbackAnalysisResponse> submitFeedback(
            @RequestBody FeedbackRequest request) {
        
        FeedbackAnalysisResponse response = 
            feedbackService.processFeedback(request);
        
        return ResponseEntity.ok(response);
    }
    
    @PostMapping("/generate-response")
    public ResponseEntity<String> generateResponse(
            @RequestBody FeedbackRequest request) {
        
        String aiResponse = feedbackService.generateAIResponse(request);
        return ResponseEntity.ok(aiResponse);
    }
}
```

#### DTOs

```java
public record FeedbackRequest(
    String customerName,
    String email,
    String feedbackText,
    int rating  // 1-5
) {}

public record FeedbackAnalysisResponse(
    String sentiment,
    double sentimentScore,
    List<String> categories,
    List<String> suggestedActions,
    String aiGeneratedResponse,
    boolean requiresUrgentAttention
) {}
```

#### Service Layer

```java
@Service
public class FeedbackService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    public FeedbackService(ChatClient.Builder builder, VectorStore vectorStore) {
        this.chatClient = builder.build();
        this.vectorStore = vectorStore;
    }
    
    public FeedbackAnalysisResponse processFeedback(FeedbackRequest request) {
        // Analyze using structured output
        String analysisPrompt = """
            Analyze this customer feedback:
            
            Customer: {name}
            Rating: {rating}/5
            Feedback: {feedback}
            
            Provide:
            - Sentiment (POSITIVE/NEGATIVE/NEUTRAL)
            - Sentiment score (0.0 to 1.0)
            - Categories (list of relevant categories like PRODUCT, SERVICE, DELIVERY)
            - Suggested actions for the team
            - Whether this requires urgent attention
            """;
        
        return chatClient.prompt()
            .user(u -> u.text(analysisPrompt)
                .param("name", request.customerName())
                .param("rating", String.valueOf(request.rating()))
                .param("feedback", request.feedbackText()))
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .entity(FeedbackAnalysisResponse.class);
    }
    
    public String generateAIResponse(FeedbackRequest request) {
        String responsePrompt = """
            Generate a professional, empathetic response to this customer feedback:
            
            Customer: {name}
            Rating: {rating}/5
            Feedback: {feedback}
            
            The response should:
            - Thank the customer
            - Address their specific concerns
            - Offer solutions if applicable
            - Be warm and professional
            - Be under 150 words
            """;
        
        return chatClient.prompt()
            .user(u -> u.text(responsePrompt)
                .param("name", request.customerName())
                .param("rating", String.valueOf(request.rating()))
                .param("feedback", request.feedbackText()))
            .call()
            .content();
    }
}
```

#### Frontend Example (HTML + JavaScript)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI-Powered Feedback Form</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background: #f5f5f5;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        textarea {
            min-height: 120px;
            resize: vertical;
        }
        button {
            background: #007bff;
            color: white;
            padding: 12px 24px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background: #0056b3;
        }
        .result {
            margin-top: 20px;
            padding: 15px;
            background: white;
            border-radius: 4px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .loading {
            text-align: center;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>Customer Feedback Form</h1>
    
    <form id="feedbackForm">
        <div class="form-group">
            <label for="name">Name:</label>
            <input type="text" id="name" required>
        </div>
        
        <div class="form-group">
            <label for="email">Email:</label>
            <input type="email" id="email" required>
        </div>
        
        <div class="form-group">
            <label for="rating">Rating:</label>
            <select id="rating" required aria-label="Select your rating from 1 to 5 stars">
                <option value="">Select rating</option>
                <option value="5">⭐⭐⭐⭐⭐ Excellent</option>
                <option value="4">⭐⭐⭐⭐ Good</option>
                <option value="3">⭐⭐⭐ Average</option>
                <option value="2">⭐⭐ Poor</option>
                <option value="1">⭐ Terrible</option>
            </select>
        </div>
        
        <div class="form-group">
            <label for="feedback">Your Feedback:</label>
            <textarea id="feedback" required></textarea>
        </div>
        
        <button type="submit">Submit Feedback</button>
    </form>
    
    <div id="result" class="result" style="display: none;"></div>
    
    <script>
        document.getElementById('feedbackForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const resultDiv = document.getElementById('result');
            resultDiv.style.display = 'block';
            resultDiv.innerHTML = '<p class="loading">🤖 AI is analyzing your feedback...</p>';
            
            const feedbackData = {
                customerName: document.getElementById('name').value,
                email: document.getElementById('email').value,
                rating: parseInt(document.getElementById('rating').value),
                feedbackText: document.getElementById('feedback').value
            };
            
            try {
                const response = await fetch('http://localhost:8080/api/feedback/submit', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(feedbackData)
                });
                
                const result = await response.json();
                
                resultDiv.innerHTML = `
                    <h3>✅ Feedback Received!</h3>
                    <p><strong>Sentiment:</strong> ${result.sentiment} 
                       (Score: ${(result.sentimentScore * 100).toFixed(1)}%)</p>
                    <p><strong>Categories:</strong> ${result.categories.join(', ')}</p>
                    <p><strong>AI Response:</strong></p>
                    <p>${result.aiGeneratedResponse}</p>
                    ${result.requiresUrgentAttention ? 
                        '<p style="color: red;">⚠️ This feedback requires urgent attention!</p>' : ''}
                `;
            } catch (error) {
                resultDiv.innerHTML = `
                    <p style="color: red;">❌ Error: ${error.message}</p>
                `;
            }
        });
    </script>
</body>
</html>
```

**Interview Key Point**: This demonstrates a complete flow from UI → REST API → AI processing → Structured response → UI update.

---

## Code Examples

### Complete Spring Boot Application

```java
@SpringBootApplication
public class SpringAIApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringAIApplication.class, args);
    }
}

@Configuration
public class ChatClientConfig {
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder
            .defaultSystem("You are a helpful AI assistant")
            .build();
    }
}

@RestController
@RequestMapping("/api/ai")
public class AIController {
    
    private final ChatClient chatClient;
    private final ChatModel chatModel;
    
    public AIController(ChatClient chatClient, ChatModel chatModel) {
        this.chatClient = chatClient;
        this.chatModel = chatModel;
    }
    
    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
    
    @PostMapping("/structured")
    public Recipe getRecipe(@RequestBody RecipeRequest request) {
        return chatClient.prompt()
            .user("Create a recipe for " + request.dish())
            .call()
            .entity(Recipe.class);
    }
    
    @GetMapping("/stream")
    public Flux<String> streamResponse(@RequestParam String message) {
        return chatClient.prompt()
            .user(message)
            .stream()
            .content();
    }
}
```

### Function Calling Example

```java
@Configuration
public class FunctionConfig {
    
    @Bean
    @Description("Get the current weather for a location")
    public Function<WeatherRequest, WeatherResponse> currentWeather() {
        return request -> {
            // Call weather API
            return new WeatherResponse(
                request.location(),
                75.0,
                "Sunny"
            );
        };
    }
}

@Service
public class WeatherService {
    
    private final ChatClient chatClient;
    
    public String getWeatherInfo(String query) {
        return chatClient.prompt()
            .user(query)
            .functions("currentWeather") // AI will call function if needed
            .call()
            .content();
    }
}
```

---

## Interview Questions

### Basic Questions

**Q1: What is Spring AI?**
> Spring AI is a framework that provides abstractions and integrations for working with AI models and services in Spring applications. It offers a portable API across different AI providers.

**Q2: What are the main components of Spring AI?**
> - Chat/Text Models
> - Embedding Models
> - Image Models
> - Vector Stores
> - Prompt Templates
> - Output Converters
> - Function Calling

**Q3: How does Spring AI achieve portability?**
> Through abstraction interfaces (ChatModel, EmbeddingModel, etc.) that different providers implement, allowing you to switch providers by changing configuration without code changes.

### Intermediate Questions

**Q4: Explain the difference between ChatModel and ChatClient.**
> - **ChatModel**: Low-level interface for direct model interaction
> - **ChatClient**: Fluent API with builder pattern, offering higher-level abstractions and convenience methods

**Q5: What is the purpose of system messages in prompts?**
> System messages set the context, behavior, and persona for the AI. They persist throughout the conversation and guide how the AI responds.

**Q6: How does structured output conversion work?**
> Spring AI generates a JSON schema from your Java class and instructs the model to return JSON matching that schema. It then deserializes the response into your object.

**Q7: What is RAG and why is it important?**
> RAG (Retrieval Augmented Generation) retrieves relevant information from a knowledge base before generating responses. It's important for:
> - Reducing hallucinations
> - Providing current information
> - Grounding responses in facts
> - Building domain-specific applications

### Advanced Questions

**Q8: How would you implement conversation memory in Spring AI?**
> ```java
> @Bean
> public ChatMemory chatMemory() {
>     return new InMemoryChatMemory();
> }
> 
> public String chatWithMemory(String message, String conversationId) {
>     return chatClient.prompt()
>         .user(message)
>         .advisors(new MessageChatMemoryAdvisor(chatMemory, conversationId))
>         .call()
>         .content();
> }
> ```

**Q9: How do you handle streaming responses?**
> ```java
> public Flux<String> streamChat(String message) {
>     return chatClient.prompt()
>         .user(message)
>         .stream()
>         .content();
> }
> ```

**Q10: Explain function calling in Spring AI.**
> Function calling allows the AI to invoke predefined functions when needed. You define functions as Spring beans with @Description, and the AI decides when to call them based on the context.

**Q11: How would you implement rate limiting for AI API calls?**
> - Use Spring's @RateLimiter from Resilience4j
> - Implement a custom advisor for the ChatClient
> - Use API gateway rate limiting
> - Implement token bucket algorithm

**Q12: What strategies would you use to reduce AI API costs?**
> - Caching responses for common queries
> - Using appropriate model sizes (GPT-3.5 vs GPT-4)
> - Implementing request batching
> - Using RAG to reduce context size
> - Prompt optimization to reduce tokens
> - Streaming only when necessary

---

## Best Practices

### 1. Prompt Engineering
```java
// ❌ Bad: Vague prompt
"Tell me about Java"

// ✅ Good: Specific, structured prompt
"""
You are a Java expert.
Explain the concept of dependency injection in Spring Boot.
Include:
- Definition
- Benefits
- Code example
- Common use cases
Keep the explanation under 200 words.
"""
```

### 2. Error Handling
```java
@Service
public class RobustAIService {
    
    private final ChatClient chatClient;
    
    public String safeCall(String message) {
        try {
            return chatClient.prompt()
                .user(message)
                .call()
                .content();
        } catch (Exception e) {
            log.error("AI call failed", e);
            return "Sorry, I'm having trouble processing your request right now.";
        }
    }
}
```

### 3. Use Caching
```java
@Service
public class CachedAIService {
    
    private final ChatClient chatClient;
    
    @Cacheable(value = "ai-responses", key = "#message")
    public String getCachedResponse(String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

### 4. Implement Timeouts
```java
@Configuration
public class AIConfig {
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        // Configure timeouts for AI requests
        // Note: Timeout configuration may vary by AI provider
        return builder
            .defaultSystem("You are a helpful assistant")
            .build();
    }
    
    // For custom HTTP clients (if needed for certain providers)
    @Bean
    public RestClient restClient() {
        return RestClient.builder()
            .requestFactory(new SimpleClientHttpRequestFactory() {{
                setConnectTimeout(Duration.ofSeconds(10));
                setReadTimeout(Duration.ofSeconds(60));
            }})
            .build();
    }
}
```

### 5. Monitor and Log
```java
@Aspect
@Component
public class AICallLoggingAspect {
    
    @Around("@annotation(AICall)")
    public Object logAICall(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = pjp.proceed();
            long duration = System.currentTimeMillis() - start;
            log.info("AI call completed in {}ms", duration);
            return result;
        } catch (Exception e) {
            log.error("AI call failed after {}ms", 
                System.currentTimeMillis() - start, e);
            throw e;
        }
    }
}
```

### 6. Use Appropriate Models
- **GPT-4**: Complex reasoning, high accuracy (expensive)
- **GPT-3.5-turbo**: Fast, cost-effective for simple tasks
- **GPT-4-turbo**: Balanced performance and cost
- **Embedding models**: For RAG and similarity search

### 7. Implement Circuit Breakers
```java
@Service
public class ResilientAIService {
    
    private final ChatClient chatClient;
    private final CircuitBreaker circuitBreaker;
    
    public String resilientCall(String message) {
        return circuitBreaker.executeSupplier(() ->
            chatClient.prompt()
                .user(message)
                .call()
                .content()
        );
    }
}
```

### 8. Validate Input/Output
```java
public String validateAndCall(String userInput) {
    // Validate input
    if (userInput == null || userInput.length() > 4000) {
        throw new IllegalArgumentException("Invalid input");
    }
    
    String response = chatClient.prompt()
        .user(userInput)
        .call()
        .content();
    
    // Validate output
    if (response == null || response.isEmpty()) {
        throw new AIException("Empty response from AI");
    }
    
    return response;
}
```

---

## Additional Resources

### Official Documentation
- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [Spring AI GitHub](https://github.com/spring-projects/spring-ai)

### Sample Projects
- Create sample projects demonstrating each concept
- Include integration tests
- Add Docker compose for local development

### Vector Databases
- **PGVector**: PostgreSQL extension for vectors
- **Chroma**: Open-source embedding database
- **Pinecone**: Managed vector database
- **Weaviate**: Open-source vector search engine

### Key Takeaways for Interviews
1. ✅ Spring AI provides **portable abstractions** across AI providers
2. ✅ **ChatClient** offers fluent API for building AI interactions
3. ✅ **Prompt templates** enable reusable, parameterized prompts
4. ✅ **Structured output** converts AI responses to Java objects
5. ✅ **RAG** grounds AI responses in factual data
6. ✅ **Multimodal** support enables image, audio, and video processing
7. ✅ **Function calling** extends AI capabilities with custom logic
8. ✅ Always implement **error handling**, **caching**, and **monitoring**

---

## License
This is a learning resource for interview preparation.

## Contributing
Feel free to add more examples and scenarios as you learn!

---

**Happy Learning! 🚀**

*Last Updated: 2024*