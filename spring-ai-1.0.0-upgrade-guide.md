# Spring AI 1.0.0 GA Upgrade Guide

This document provides a comprehensive guide for upgrading to Spring AI 1.0.0 GA from previous versions, including all breaking changes, migration steps, and configuration updates.

## Table of Contents
- [Breaking Changes](#breaking-changes)
- [Dependency Changes](#dependency-changes)
- [Configuration Property Changes](#configuration-property-changes)
- [API Changes](#api-changes)
- [Removed Features](#removed-features)
- [Migration Steps](#migration-steps)

## Breaking Changes

### 1. Artifact Naming Convention Changes

Spring AI 1.0.0 GA introduces a new naming convention for Maven/Gradle artifacts:

#### Model Starters
**Before (M7 and earlier):**
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
```

**After (1.0.0 GA):**
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
</dependency>
```

#### Vector Store Starters
**Before:**
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

**After:**
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-vector-store-pgvector</artifactId>
</dependency>
```

### 2. Tool Calling API Changes

The tool calling API has been significantly refactored:

**Before (M7):**
```java
ChatClient chatClient = ChatClient.builder()
    .withChatModel(chatModel)
    .tools(List.of(weatherTool))
    .toolCallbacks(List.of(weatherCallback))
    .build();
```

**After (M8/1.0.0):**
```java
ChatClient chatClient = ChatClient.builder()
    .withChatModel(chatModel)
    .toolSpecifications(List.of(weatherToolSpec))
    .toolCallbacks(List.of(weatherCallback))
    .build();
```

### 3. ToolContext Changes

- `ToolContext` is now `final` and cannot be extended
- Custom context extensions must use composition instead of inheritance
- Tool resolution is now more explicit

**Before:**
```java
public class CustomToolContext extends ToolContext {
    // Custom implementation
}
```

**After:**
```java
public class CustomToolWrapper {
    private final ToolContext context;
    // Use composition
}
```

### 4. Observability Configuration Changes

Content observation has been replaced with logging handlers:

**Configuration Property Changes:**
```yaml
# Before
spring.ai.chat.client.observation.include-prompt: true
spring.ai.chat.client.observation.include-completion: true
spring.ai.chat.client.observation.include-query-response: true

# After
spring.ai.chat.client.observation.log-prompt: true
spring.ai.chat.client.observation.log-completion: true
spring.ai.chat.client.observation.log-query-response: true
```

### 5. Package Restructuring

Several packages have been reorganized:

- `org.springframework.ai.chat.prompt` → `org.springframework.ai.chat.messages`
- `org.springframework.ai.embedding.store` → `org.springframework.ai.vectorstore`
- Model-specific packages have been standardized

## Dependency Changes

### Updated Dependencies

| Component | Old Version | New Version |
|-----------|------------|-------------|
| Spring Boot | 3.2.x | 3.3.x+ |
| Spring Framework | 6.1.x | 6.2.x+ |
| Micrometer | 1.12.x | 1.13.x+ |

### Required Java Version

- **Minimum Java Version:** Java 17 (Java 21 recommended)

## Configuration Property Changes

### Chat Model Properties

```yaml
# Before
spring.ai.openai.chat.options.model: gpt-4
spring.ai.openai.chat.options.temperature: 0.7

# After
spring.ai.model.openai.chat.options.model: gpt-4
spring.ai.model.openai.chat.options.temperature: 0.7
```

### Embedding Model Properties

```yaml
# Before
spring.ai.openai.embedding.options.model: text-embedding-ada-002

# After
spring.ai.model.openai.embedding.options.model: text-embedding-3-small
```

### Vector Store Properties

```yaml
# Before
spring.ai.vectorstore.pgvector.index-type: HNSW
spring.ai.vectorstore.pgvector.distance-type: COSINE

# After
spring.ai.vector-store.pgvector.index-type: HNSW
spring.ai.vector-store.pgvector.distance-type: COSINE
```

## API Changes

### ChatClient API

**Method Renames:**
- `withSystemPrompt()` → `withSystemMessage()`
- `withUserPrompt()` → `withUserMessage()`
- `withAssistantPrompt()` → `withAssistantMessage()`

**New Features:**
- Streaming support improvements
- Better error handling
- Enhanced tool calling capabilities

### EmbeddingClient API

**Before:**
```java
EmbeddingResponse response = embeddingClient.embed(text);
```

**After:**
```java
EmbeddingResponse response = embeddingClient.embed(
    new EmbeddingRequest(List.of(text), 
    EmbeddingOptions.builder().build())
);
```

### VectorStore API

**Interface Changes:**
- `add()` method now requires `Document` objects with metadata
- `similaritySearch()` returns `List<Document>` instead of raw vectors
- New `filter()` capabilities for metadata-based filtering

## Removed Features

### Removed Model Implementations

The following model implementations have been removed in 1.0.0 GA:

1. **Watson AI** - No longer supported
2. **MoonShot** - Removed from core
3. **QianFan** - Removed from core
4. **ZhiPu AI** - Moved to community contributions

### Removed Vector Stores

1. **HanaDB Vector Store** - Removed
2. **SimpleVectorStore** - Replaced with in-memory implementation

### Deprecated APIs Removed

- `ChatPromptTemplate.create()` - Use `ChatPromptTemplate.builder()`
- `SystemPromptTemplate` - Use `SystemMessage`
- `UserPromptTemplate` - Use `UserMessage`

## Migration Steps

### Step 1: Update Dependencies

1. Update Spring Boot to 3.3.x or later
2. Replace old artifact names with new naming convention
3. Update transitive dependencies

### Step 2: Update Configuration

1. Rename all configuration properties according to new structure
2. Update logging configuration for observability
3. Adjust model-specific settings

### Step 3: Refactor Code

1. Update import statements for renamed packages
2. Replace deprecated API calls
3. Adjust tool calling implementations
4. Update vector store usage

### Step 4: Testing

1. Run existing tests to identify breaking changes
2. Update test configurations
3. Verify tool calling functionality
4. Test observability and logging

### Step 5: Performance Tuning

1. Review new default settings
2. Optimize batch sizes for embedding operations
3. Configure connection pools for vector stores
4. Adjust timeout settings

## Common Migration Issues and Solutions

### Issue 1: ClassNotFoundException

**Problem:** Classes not found after upgrade
**Solution:** Update import statements and check for removed features

### Issue 2: Tool Calling Failures

**Problem:** Tools not being invoked
**Solution:** Ensure tools are explicitly included in requests

### Issue 3: Configuration Not Applied

**Problem:** Old properties being ignored
**Solution:** Use new property naming convention

### Issue 4: Vector Store Connection Issues

**Problem:** Cannot connect to vector database
**Solution:** Update connection properties and driver versions

## Best Practices for Migration

1. **Incremental Migration:** Migrate one component at a time
2. **Use OpenRewrite:** Leverage automated migration recipes
3. **Test Thoroughly:** Ensure all functionality works after migration
4. **Monitor Performance:** Check for performance regressions
5. **Review Documentation:** Consult official Spring AI documentation

## Additional Resources

- [Spring AI Official Documentation](https://docs.spring.io/spring-ai/reference/)
- [Migration Examples](https://github.com/spring-projects/spring-ai/tree/main/spring-ai-examples)
- [Community Support](https://github.com/spring-projects/spring-ai/discussions)

## Version Compatibility Matrix

| Spring AI Version | Spring Boot | Java | OpenAI API | Ollama |
|------------------|-------------|------|------------|---------|
| 1.0.0 GA | 3.3.x+ | 17+ | v1 | 0.5.0+ |
| 1.0.0 M7 | 3.2.x+ | 17+ | v1 | 0.4.0+ |
| 0.8.x | 3.2.x | 17+ | v1 | 0.3.0+ |

## Conclusion

The upgrade to Spring AI 1.0.0 GA brings significant improvements in API consistency, performance, and feature set. While there are breaking changes, the migration path is well-defined and can be largely automated using OpenRewrite recipes. Following this guide and testing thoroughly will ensure a smooth transition to the new version.