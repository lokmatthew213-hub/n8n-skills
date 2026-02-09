---
name: n8n-workflow-expert
description: Comprehensive guide for building production-ready n8n workflows using the n8n-mcp MCP server. Use when working with n8n workflows, including searching for nodes, writing expressions, configuring nodes, validating workflows, using Code nodes (JavaScript/Python), understanding workflow patterns, or troubleshooting validation errors. Covers expression syntax ({{}} patterns), MCP tool usage, proven workflow patterns, validation error fixing, operation-aware node configuration, and Code node best practices.
---

# n8n Workflow Expert

Complete guide for building flawless n8n workflows using the n8n-mcp MCP server. This skill integrates seven complementary areas of n8n expertise.

---

## Quick Navigation

Use this guide when working on:

1. **[Expression Syntax](#expression-syntax)** - Writing {{}} expressions, accessing $json/$node variables
2. **[MCP Tools](#mcp-tools)** - Using n8n-mcp server tools effectively
3. **[Workflow Patterns](#workflow-patterns)** - Proven architectural patterns
4. **[Validation](#validation)** - Interpreting and fixing validation errors
5. **[Node Configuration](#node-configuration)** - Operation-aware node setup
6. **[JavaScript Code](#javascript-code)** - Writing Code nodes in JavaScript
7. **[Python Code](#python-code)** - Writing Code nodes in Python

---

## Expression Syntax

### Core Format

All dynamic content uses **double curly braces**:

```javascript
✅ {{$json.email}}
✅ {{$json.body.name}}
✅ {{$node["HTTP Request"].json.data}}
❌ $json.email  // no braces
❌ {$json.email}  // single braces
```

### Core Variables

**$json** - Current node output:
```javascript
{{$json.fieldName}}
{{$json.nested.property}}
{{$json.items[0].name}}
```

**$node** - Reference other nodes:
```javascript
{{$node["Node Name"].json.fieldName}}
{{$node["HTTP Request"].json.data}}
{{$node["Webhook"].json.body.email}}
```

**$now** - Current timestamp:
```javascript
{{$now.toFormat('yyyy-MM-dd')}}
{{$now.plus({days: 7})}}
```

**$env** - Environment variables:
```javascript
{{$env.API_KEY}}
```

### 🚨 CRITICAL: Webhook Data Structure

**Most Common Mistake**: Webhook data is under `.body`, not at root!

```javascript
// Webhook output structure
{
  "headers": {...},
  "params": {...},
  "query": {...},
  "body": {           // ⚠️ USER DATA IS HERE!
    "name": "John",
    "email": "john@example.com"
  }
}

❌ WRONG: {{$json.name}}
✅ CORRECT: {{$json.body.name}}
```

### When NOT to Use Expressions

**Code Nodes** - Use direct JavaScript/Python access:
```javascript
// ❌ WRONG in Code node
const email = '={{$json.email}}';

// ✅ CORRECT in Code node
const email = $json.email;
const email = $input.item.json.email;
```

**Webhook Paths** - Use static paths only

**Credential Fields** - Use n8n credential system

### Common Expression Patterns

```javascript
// Nested fields
{{$json.user.email}}
{{$json.data[0].name}}

// Fields with spaces
{{$json['field name']}}
{{$node["HTTP Request"].json.data}}

// Concatenation
Hello {{$json.body.name}}!

// Conditional
{{$json.status === 'active' ? 'Active' : 'Inactive'}}

// Default values
{{$json.email || 'no-email@example.com'}}

// Date manipulation
{{$now.plus({days: 7}).toFormat('yyyy-MM-dd')}}
```

---

## MCP Tools

### Tool Categories

The n8n-mcp server provides:

1. **Node Discovery** - Search and understand nodes
2. **Configuration Validation** - Check node/workflow configs
3. **Workflow Management** - Create, update, activate workflows
4. **Template Library** - Access 2,700+ real workflows
5. **Documentation** - Tool docs, guides, Code node references

### Most Used Tools

| Tool | Use When | Speed |
|------|----------|-------|
| `search_nodes` | Finding nodes by keyword | <20ms |
| `get_node` | Understanding node operations | <10ms |
| `validate_node` | Checking configurations | <100ms |
| `n8n_create_workflow` | Creating workflows | 100-500ms |
| `n8n_update_partial_workflow` | Editing workflows (MOST USED!) | 50-200ms |
| `validate_workflow` | Checking complete workflow | 100-500ms |

### Tool Selection Workflows

**Finding the Right Node**:
```
1. search_nodes({query: "keyword"})
2. get_node({nodeType: "nodes-base.name"})
3. [Optional] get_node({nodeType: "nodes-base.name", mode: "docs"})
```

**Validating Configuration**:
```
1. validate_node({nodeType, config: {}, mode: "minimal"})
2. validate_node({nodeType, config, profile: "runtime"})
3. Fix errors, validate again
```

**Managing Workflows**:
```
1. n8n_create_workflow({name, nodes, connections})
2. n8n_validate_workflow({id})
3. n8n_update_partial_workflow({id, operations: [...]})
4. n8n_validate_workflow({id}) again
5. n8n_update_partial_workflow({id, operations: [{type: "activateWorkflow"}]})
```

### Critical: nodeType Formats

**Two different formats** for different tools:

**Format 1: Search/Validate Tools**
```javascript
// Use SHORT prefix: nodes-base.*
search_nodes({query: "slack"})
get_node({nodeType: "nodes-base.slack"})
validate_node({nodeType: "nodes-base.httpRequest"})
```

**Format 2: Workflow Management**
```javascript
// Use FULL prefix: n8n-nodes-base.*
{
  "type": "n8n-nodes-base.slack",
  "name": "Slack"
}
```

### Validation Profiles

Choose profile based on stage:

- **minimal** - Quick check, only required fields
- **runtime** - Full validation with auto-sanitization (recommended)
- **ai-friendly** - Detailed explanations for debugging
- **strict** - No auto-sanitization, all issues reported

### Smart Parameters

Some tools accept special parameters:

```javascript
// For IF/Switch nodes
get_node({
  nodeType: "nodes-base.if",
  branch: "true"  // Get true branch config
})

// For detail levels
get_node({
  nodeType: "nodes-base.slack",
  detail: "standard"  // Most used: 1-2K tokens, 95% coverage
})
```

---

## Workflow Patterns

### The 5 Core Patterns

Based on analysis of 2,653+ real n8n workflows:

**1. Webhook Processing** (Most Common)
```
Pattern: Webhook → Validate → Transform → Respond/Notify

Example:
Webhook → Code (validate) → IF → Slack/Email → Respond to Webhook

Use for: Form submissions, API webhooks, real-time events
```

**2. HTTP API Integration**
```
Pattern: Trigger → HTTP Request → Transform → Action

Example:
Schedule → HTTP Request → Code (parse) → Database/Slack

Use for: API polling, data sync, external service integration
```

**3. Database Operations**
```
Pattern: Trigger → Database Query → Transform → Action/Response

Example:
Webhook → Postgres → Code (format) → HTTP Response

Use for: CRUD operations, data retrieval, reporting
```

**4. AI Agent Workflows**
```
Pattern: Input → AI Agent → Tool Execution → Response

Example:
Webhook → AI Agent → [Tools: Database, HTTP, Code] → Response

Use for: Conversational AI, intelligent automation, decision-making
```

**5. Scheduled Tasks**
```
Pattern: Schedule → Fetch Data → Process → Notify/Store

Example:
Schedule → HTTP Request → Code (analyze) → Email + Database

Use for: Reports, monitoring, batch processing
```

### Workflow Creation Checklist

1. **Choose pattern** based on trigger type
2. **Search nodes** using `search_nodes`
3. **Get node details** with `get_node`
4. **Create workflow** with `n8n_create_workflow`
5. **Validate** using `n8n_validate_workflow`
6. **Iterate** with `n8n_update_partial_workflow`
7. **Activate** when validation passes

### Connection Best Practices

- **Linear flows**: Connect nodes sequentially
- **Branching**: Use IF/Switch nodes for conditional logic
- **Error handling**: Add Error Trigger nodes for critical workflows
- **Webhook responses**: Always include "Respond to Webhook" for webhook triggers
- **Data transformation**: Use Code nodes between incompatible node outputs

---

## Validation

### Validation Philosophy

**Validate early, validate often**

Expect 2-3 validate → fix cycles:
- Average: 23s thinking about errors, 58s fixing them
- Use validation profiles strategically
- Understand auto-sanitization behavior

### Validation Loop Workflow

```
1. validate_node or validate_workflow
2. Review errors/warnings
3. Identify error types (required field, type mismatch, etc.)
4. Apply fixes
5. Validate again
6. Repeat until clean
```

### Common Error Types

**Missing Required Fields**
```javascript
Error: "Missing required field 'operation'"
Fix: Add the required field to node configuration
```

**Type Mismatches**
```javascript
Error: "Expected number, got string"
Fix: Convert type or use correct format
```

**Invalid Node References**
```javascript
Error: "Referenced node 'HTTP Request' not found"
Fix: Check node name matches exactly (case-sensitive)
```

**Expression Syntax Errors**
```javascript
Error: "Invalid expression syntax"
Fix: Check {{ }} wrapping, quotes for spaces, proper nesting
```

### Auto-Sanitization Behavior

With `runtime` or `ai-friendly` profiles, validator auto-fixes:

- Adds default values for optional fields
- Converts compatible types (string ↔ number)
- Removes invalid/deprecated properties
- Normalizes field formats

**False Positives**: Sometimes validator reports errors that don't affect execution. Use `minimal` profile to reduce noise during development.

### Profile Selection Strategy

**Development**: `minimal` → quick feedback
**Pre-deployment**: `runtime` → catch real issues
**Debugging**: `ai-friendly` → detailed explanations
**Production**: `strict` → no surprises

---

## Node Configuration

### Configuration Philosophy

**Progressive disclosure**: Start minimal, add complexity as needed

Best practices:
- Use `get_node` with `detail: "standard"` (covers 95% of cases)
- Average 56s between configuration edits
- Understand property dependencies
- Know operation-specific requirements

### Property Dependencies

Some properties require others:

**HTTP Request Node**:
```javascript
{
  "sendBody": true,
  "contentType": "json"  // Required when sendBody is true
}
```

**Database Nodes**:
```javascript
{
  "operation": "executeQuery",
  "query": "SELECT * FROM users"  // Required for executeQuery
}
```

**AI Agent Node**:
```javascript
{
  "operation": "agent",
  "agent": "conversational",
  "tools": [...]  // Required for agent operation
}
```

### AI Connection Types

For AI Agent workflows, 8 connection types exist:

1. **ai_tool** - Custom tools
2. **ai_memory** - Conversation memory
3. **ai_chain** - Chain components
4. **ai_retriever** - Document retrieval
5. **ai_document** - Document loaders
6. **ai_embedding** - Embedding models
7. **ai_vectorStore** - Vector databases
8. **ai_outputParser** - Output parsing

### Common Configuration Patterns

**Webhook Node**:
```javascript
{
  "path": "webhook-path",
  "httpMethod": "POST",
  "responseMode": "responseNode"  // Use with "Respond to Webhook"
}
```

**Code Node**:
```javascript
{
  "mode": "runOnceForAllItems",  // Most common
  "jsCode": "// JavaScript code here"
}
```

**HTTP Request Node**:
```javascript
{
  "method": "POST",
  "url": "https://api.example.com/endpoint",
  "sendBody": true,
  "contentType": "json",
  "bodyParameters": {
    "parameters": [...]
  }
}
```

---

## JavaScript Code

### Quick Start Template

```javascript
// Basic template for Code nodes
const items = $input.all();

// Process data
const processed = items.map(item => ({
  json: {
    // Your transformed data
    field: item.json.field
  }
}));

return processed;
```

### Data Access Patterns

**$input API** (Recommended):
```javascript
// All items
const items = $input.all();

// First item only
const item = $input.first();

// Current item (in loop mode)
const item = $input.item;
```

**Direct access**:
```javascript
// Current item
const data = $json;

// From other nodes
const webhookData = $node["Webhook"].json;
```

### 🚨 CRITICAL: Webhook Data in Code Nodes

Same as expressions - webhook data is under `.body`:

```javascript
❌ WRONG:
const name = $json.name;

✅ CORRECT:
const name = $json.body.name;
const email = $json.body.email;
```

### Return Format

**Always return array of objects with `json` property**:

```javascript
// Single item
return [{
  json: {
    result: "value"
  }
}];

// Multiple items
return items.map(item => ({
  json: {
    processed: item.json.field
  }
}));
```

### Built-in Functions

**$helpers.httpRequest()** - Make HTTP requests:
```javascript
const response = await $helpers.httpRequest({
  method: 'POST',
  url: 'https://api.example.com/endpoint',
  body: {
    key: 'value'
  },
  headers: {
    'Authorization': 'Bearer token'
  }
});
```

**DateTime** - Date manipulation (Luxon):
```javascript
const now = DateTime.now();
const formatted = now.toFormat('yyyy-MM-dd');
const future = now.plus({days: 7});
```

**$jmespath()** - Query JSON:
```javascript
const result = $jmespath($json, 'data[?price > `100`].name');
```

### Top 5 Error Patterns

**1. Incorrect Return Format** (28% of errors)
```javascript
❌ return {result: value};
✅ return [{json: {result: value}}];
```

**2. Webhook .body Access** (18%)
```javascript
❌ const name = $json.name;
✅ const name = $json.body.name;
```

**3. Async/Await Missing** (12%)
```javascript
❌ const data = $helpers.httpRequest(...);
✅ const data = await $helpers.httpRequest(...);
```

**4. Undefined Property Access** (10%)
```javascript
❌ const email = $json.user.email;  // user might be undefined
✅ const email = $json.user?.email || 'default@example.com';
```

**5. Expression Syntax in Code** (8%)
```javascript
❌ const value = '={{$json.field}}';
✅ const value = $json.field;
```

### Production Patterns

**1. Data Transformation**:
```javascript
const items = $input.all();
return items.map(item => ({
  json: {
    id: item.json.id,
    name: item.json.name.toUpperCase(),
    timestamp: DateTime.now().toISO()
  }
}));
```

**2. API Request with Error Handling**:
```javascript
try {
  const response = await $helpers.httpRequest({
    method: 'GET',
    url: `https://api.example.com/users/${$json.body.userId}`
  });
  
  return [{json: response}];
} catch (error) {
  return [{json: {error: error.message}}];
}
```

**3. Webhook Data Processing**:
```javascript
const webhookData = $json.body;

// Validate
if (!webhookData.email) {
  throw new Error('Email is required');
}

// Transform
return [{
  json: {
    email: webhookData.email.toLowerCase(),
    name: webhookData.name,
    timestamp: DateTime.now().toISO()
  }
}];
```

**4. Array Processing**:
```javascript
const items = $input.all();
const filtered = items.filter(item => item.json.status === 'active');
const sorted = filtered.sort((a, b) => a.json.priority - b.json.priority);

return sorted.map(item => ({
  json: item.json
}));
```

**5. Combining Multiple Node Outputs**:
```javascript
const webhookData = $node["Webhook"].json.body;
const apiData = $node["HTTP Request"].json;

return [{
  json: {
    user: webhookData.email,
    result: apiData.data,
    processed: DateTime.now().toISO()
  }
}];
```

### Code Node Modes

**runOnceForAllItems** (Most common):
- Runs once with all items
- Use `$input.all()` to access items
- Best for aggregation, API calls

**runOnceForEachItem**:
- Runs separately for each item
- Use `$input.item` to access current item
- Best for item-specific processing

---

## Python Code

### ⚠️ Important: JavaScript First

**Recommendation**: Use **JavaScript for 95% of use cases**

Only use Python when:
- You need specific Python standard library functions
- You're significantly more comfortable with Python syntax
- Doing data transformations better suited to Python

**Why JavaScript is preferred:**
- Full n8n helper functions ($helpers.httpRequest, etc.)
- Better n8n integration
- More examples and community support
- Faster execution

### Python Data Access

**_input API**:
```python
# All items
items = _input.all()

# First item
item = _input.first()

# Current item (in loop mode)
item = _input.item
```

**Direct access**:
```python
# Current item
data = _json

# From other nodes
webhook_data = _node["Webhook"].json
```

### Return Format

Same as JavaScript - return list of dicts with `json` key:

```python
# Single item
return [{
  'json': {
    'result': 'value'
  }
}]

# Multiple items
return [
  {'json': {'processed': item['json']['field']}}
  for item in items
]
```

### 🚨 CRITICAL Limitation: No External Libraries

**Python Code nodes do NOT support external libraries**:

```python
❌ import requests  # NOT AVAILABLE
❌ import pandas    # NOT AVAILABLE
❌ import numpy     # NOT AVAILABLE

✅ import json      # Available (standard library)
✅ import datetime  # Available (standard library)
✅ import re        # Available (standard library)
```

### Available Standard Library

```python
import json         # JSON parsing
import datetime     # Date/time handling
import re           # Regular expressions
import math         # Math operations
import random       # Random numbers
import base64       # Base64 encoding
import hashlib      # Hashing
import urllib       # URL parsing
from collections import defaultdict, Counter
```

### Workarounds for Missing Libraries

**Instead of requests**:
```python
# ❌ Can't use requests library
# ✅ Use HTTP Request node instead
# Then access result in Code node:
api_data = _node["HTTP Request"].json
```

**Instead of pandas**:
```python
# ❌ Can't use pandas
# ✅ Use native Python data structures
items = _input.all()
filtered = [item for item in items if item['json']['status'] == 'active']
```

### Common Python Patterns

**1. Data Transformation**:
```python
items = _input.all()

processed = [
  {
    'json': {
      'id': item['json']['id'],
      'name': item['json']['name'].upper(),
      'timestamp': datetime.datetime.now().isoformat()
    }
  }
  for item in items
]

return processed
```

**2. Webhook Data Processing**:
```python
webhook_data = _json['body']

# Validate
if 'email' not in webhook_data:
  raise ValueError('Email is required')

# Transform
return [{
  'json': {
    'email': webhook_data['email'].lower(),
    'name': webhook_data['name'],
    'timestamp': datetime.datetime.now().isoformat()
  }
}]
```

**3. Text Processing**:
```python
import re

text = _json['body']['message']

# Clean and process
cleaned = re.sub(r'[^\w\s]', '', text)
words = cleaned.lower().split()
word_count = len(words)

return [{
  'json': {
    'original': text,
    'cleaned': cleaned,
    'word_count': word_count
  }
}]
```

---

## Skills Work Together

When you ask: **"Build and validate a webhook to Slack workflow"**

1. **Workflow Patterns** identifies webhook processing pattern
2. **MCP Tools** searches for webhook and Slack nodes
3. **Node Configuration** guides node setup
4. **JavaScript Code** helps process webhook data with proper .body access
5. **Expression Syntax** helps with data mapping in Slack node
6. **Validation** validates the final workflow

All areas compose seamlessly for complete workflow development!

---

## Quick Reference Summary

### Essential Rules

1. **Expressions**: Wrap in {{ }}, webhook data under `.body`
2. **Code Nodes**: No {{ }}, direct access, return [{json: {...}}]
3. **MCP Tools**: Use correct nodeType format (nodes-base.* vs n8n-nodes-base.*)
4. **Validation**: Iterate 2-3 times, use appropriate profile
5. **Patterns**: Choose from 5 core patterns based on trigger
6. **JavaScript First**: Use JavaScript for 95% of Code nodes

### Common Mistakes Quick Fixes

| Issue | Fix |
|-------|-----|
| `$json.field` in expression | `{{$json.field}}` |
| `{{$json.name}}` in webhook | `{{$json.body.name}}` |
| `'={{$json.field}}'` in Code | `$json.field` |
| Wrong nodeType format | Check tool category (search vs workflow) |
| Validation errors | Use appropriate profile, iterate |
| Code node return | Always `[{json: {...}}]` |

---

## Related Resources

For detailed guides on specific topics, refer to the original n8n-skills repository:
- [n8n-mcp GitHub](https://github.com/czlonkowski/n8n-mcp)
- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)

---

**Ready to build flawless n8n workflows!** 🚀
