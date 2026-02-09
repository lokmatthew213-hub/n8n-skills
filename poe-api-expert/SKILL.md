---
name: poe-api-expert
description: Expert guide for using the Poe API MCP server to access 327+ AI models including GPT-5, Claude 4.5, Gemini 3, Grok 4, DeepSeek, and more. Use when calling specific Poe models, getting model recommendations, listing available models, or executing tasks with specialized AI models through the poe-api MCP server.
license: MIT
---

# Poe API Expert

Guide for effectively using the Poe API MCP server to access 327+ AI models through Manus.

## Overview

The `poe-api` MCP server provides access to a vast collection of AI models from multiple providers. Use this skill when you need to leverage specific models for specialized tasks or when the default Manus models are not optimal.

## Available MCP Tools

### 1. `poe_chat`

Send chat requests to any Poe AI model.

**Parameters:**
- `model` (required): Model name (e.g., `Claude-Sonnet-4`, `gpt-5`, `deepseek-v3`)
- `message` (required): User message to send
- `system_prompt` (optional): System prompt to guide model behavior

**Example usage:**
```bash
manus-mcp-cli tool call poe_chat --server poe-api --input '{
  "model": "Claude-Sonnet-4",
  "message": "Explain quantum computing in simple terms",
  "system_prompt": "You are a physics professor"
}'
```

### 2. `poe_list_models`

Get categorized list of all available models.

**Parameters:** None

**Returns:** JSON object with models grouped by provider (openai, anthropic, google, xai, other)

**Example usage:**
```bash
manus-mcp-cli tool call poe_list_models --server poe-api --input '{}'
```

### 3. `poe_recommend_models`

Get intelligent model recommendations based on task description.

**Parameters:**
- `task` (required): Description of the task (e.g., "write Python code", "translate to Chinese")

**Returns:** 3 recommended models with reasons and strengths

**Example usage:**
```bash
manus-mcp-cli tool call poe_recommend_models --server poe-api --input '{
  "task": "write a Python sorting algorithm"
}'
```

## Model Categories and Use Cases

### Coding Tasks
**Best models:** `deepseek-v3`, `Claude-Sonnet-4`, `gpt-5`
- Use `deepseek-v3` for: Code generation, debugging, algorithm implementation
- Use `Claude-Sonnet-4` for: Code with explanations, learning-focused code
- Use `gpt-5` for: Latest tech stack, general programming

### Translation Tasks
**Best models:** `gemini-3-pro`, `Claude-Sonnet-4`, `gpt-5`
- Use `gemini-3-pro` for: Multi-language support, cultural context
- Use `Claude-Sonnet-4` for: Technical terminology, natural flow
- Use `gpt-5` for: High-volume translation

### Creative Writing
**Best models:** `Claude-Opus-4.5`, `gpt-5`, `Claude-Sonnet-4`
- Use `Claude-Opus-4.5` for: Literary quality, emotional depth
- Use `gpt-5` for: Diverse styles, various genres
- Use `Claude-Sonnet-4` for: Business writing, technical articles

### Reasoning and Analysis
**Best models:** `o3`, `deepseek-r1`, `Claude-Opus-4.5`
- Use `o3` for: Complex reasoning, math problems, logic puzzles
- Use `deepseek-r1` for: Step-by-step reasoning, learning
- Use `Claude-Opus-4.5` for: Multi-perspective analysis

### Fast Responses
**Best models:** `gemini-3-flash`, `gpt-4o-mini`, `Claude-Haiku-4.5`
- Use `gemini-3-flash` for: Low latency, simple questions
- Use `gpt-4o-mini` for: Cost-effective, high volume
- Use `Claude-Haiku-4.5` for: Speed-quality balance

## Complete Model List

### OpenAI
- `gpt-5`, `gpt-5.2`, `gpt-5.2-pro`
- `gpt-4o`, `gpt-4o-mini`
- `o1`, `o3`, `o4-mini`

### Anthropic
- `Claude-Sonnet-4.5`, `Claude-Opus-4.5`, `Claude-Haiku-4.5`
- `Claude-Sonnet-4`, `Claude-Opus-4`

### Google
- `gemini-3-pro`, `gemini-3-flash`
- `gemini-2.5-pro`, `gemini-2.0-flash`
- `nano-banana-pro`

### XAI
- `grok-4`, `grok-3`, `grok-4-fast-reasoning`

### Other
- `minimax-m2.1`
- `deepseek-r1`, `deepseek-v3`
- `qwen3-max`
- `llama-4-scout-t`

## Workflow Patterns

### Pattern 1: Direct Model Call

When you know which model to use:

1. Identify the task requirements
2. Select appropriate model from categories above
3. Call `poe_chat` with model and message
4. Return results to user

### Pattern 2: Get Recommendations First

When unsure which model to use:

1. Call `poe_recommend_models` with task description
2. Review recommendations and select best model
3. Call `poe_chat` with selected model
4. Return results to user

### Pattern 3: Model Comparison

When quality is critical:

1. Call `poe_recommend_models` to get top 3 models
2. Call `poe_chat` with each recommended model
3. Compare outputs
4. Present best result or all results to user

## Best Practices

### Model Selection
- **Start with recommendations** if unsure which model to use
- **Use specialized models** for domain-specific tasks (e.g., `deepseek-v3` for code)
- **Consider speed vs quality** trade-offs based on task urgency
- **Test multiple models** for critical tasks

### System Prompts
- Use system prompts to guide model behavior and output format
- Keep system prompts concise and specific
- Include role definition (e.g., "You are a Python expert")
- Specify output requirements (e.g., "Provide code with comments")

### Error Handling
- If a model returns an error, try an alternative model from the same category
- Check API key configuration if all models fail
- Verify model name spelling (case-sensitive)

## Common Use Cases

### Use Case 1: Code Generation
```bash
# Get recommendation
manus-mcp-cli tool call poe_recommend_models --server poe-api --input '{
  "task": "write a Python function to sort a list"
}'

# Use recommended model
manus-mcp-cli tool call poe_chat --server poe-api --input '{
  "model": "deepseek-v3",
  "message": "Write a Python function that implements quicksort with comments",
  "system_prompt": "You are a Python expert. Provide clean, well-commented code."
}'
```

### Use Case 2: Translation
```bash
manus-mcp-cli tool call poe_chat --server poe-api --input '{
  "model": "gemini-3-pro",
  "message": "Translate this to Chinese: The quick brown fox jumps over the lazy dog",
  "system_prompt": "You are a professional translator. Provide natural, idiomatic translations."
}'
```

### Use Case 3: Creative Writing
```bash
manus-mcp-cli tool call poe_chat --server poe-api --input '{
  "model": "Claude-Opus-4.5",
  "message": "Write a short story about a robot learning to feel emotions",
  "system_prompt": "You are a creative writer. Write engaging, emotionally rich stories."
}'
```

## Troubleshooting

### Issue: "POE_API_KEY environment variable is not set"
**Solution:** Ensure the MCP server is running with the API key configured in environment variables.

### Issue: Model returns empty response
**Solution:** 
- Check if the model name is correct (case-sensitive)
- Try a different model from the same category
- Simplify the message or system prompt

### Issue: Slow response
**Solution:** Use faster models like `gemini-3-flash`, `gpt-4o-mini`, or `Claude-Haiku-4.5`

## Notes

- All models are accessed through the same MCP server endpoint
- API key is configured on the server side, not passed in requests
- Model availability depends on Poe API subscription level
- Response times vary by model (see "Fast Responses" category)
