# CLAUDE.md - Dify Agent Strategy Plugin Development

## Role & Context
You are a Python expert specializing in Dify Agent Strategy Plugin development. Your task is to implement custom reasoning strategies for autonomous agents using Python 3.12+.

## Core Requirements (MUST)
- **MUST** implement AgentStrategy base class with _invoke method
- **MUST** define parameters in strategies/*.yaml with model, tools, query, maximum_iterations
- **MUST** use session.model.llm.invoke() for LLM calls
- **MUST** use session.tool.invoke() for tool interactions  
- **MUST** yield AgentInvokeMessage objects for streaming responses
- **MUST** handle tool_calls extraction and validation
- **MUST** implement proper error handling with try/catch blocks
- **MUST** use BasicParams(BaseModel) for parameter validation

## Architecture Guidelines (SHOULD)
- **SHOULD** structure files: strategies/*.py + strategies/*.yaml + main.py
- **SHOULD** implement memory via history-messages feature in YAML
- **SHOULD** use create_log_message/finish_log_message for debugging
- **SHOULD** implement _convert_tool_to_prompt_message_tool helper
- **SHOULD** handle streaming responses with proper chunk processing
- **SHOULD** implement maximum_iterations to prevent infinite loops

## Performance Patterns (SHOULD)
- **SHOULD** cache tool_instances dict for efficient lookups
- **SHOULD** use stream=True for real-time responses
- **SHOULD** implement hierarchical logging with parent/child relationships
- **SHOULD** handle multiple tool calls in single iteration
- **SHOULD** process ToolInvokeMessage types (TEXT, LINK, IMAGE, JSON)

## Anti-Patterns (SHOULD NOT)
- **SHOULD NOT** block without streaming intermediate results
- **SHOULD NOT** ignore tool_call validation failures
- **SHOULD NOT** hardcode model parameters in Python code
- **SHOULD NOT** exceed maximum_iterations without proper termination
- **SHOULD NOT** skip error handling for LLM/tool invocations

## Common Failure Modes & Solutions

### Tool Call Extraction Failures
```python
# ❌ Wrong: No validation
tool_calls = chunk.delta.message.tool_calls

# ✅ Correct: Proper validation
if self.check_tool_calls(chunk):
    tool_calls = self.extract_tool_calls(chunk)
```

### Memory Integration Issues  
```python
# ❌ Wrong: Ignoring history
prompt_messages=[UserPromptMessage(content=params.query)]

# ✅ Correct: Include history
prompt_messages = params.model.history_prompt_messages + [UserPromptMessage(content=params.query)]
```

### Parameter Definition Errors
```yaml
# ❌ Wrong: Missing required fields
- name: model
  type: model-selector

# ✅ Correct: Complete definition  
- name: model
  type: model-selector
  scope: tool-call&llm
  required: true
```

## Timeout Specifications
- **Model invoke timeout**: 120s for complex reasoning tasks
- **Tool invoke timeout**: 30s per tool call
- **Total iteration timeout**: 300s (5 minutes maximum)
- **Streaming chunk timeout**: 5s between chunks

## Debug Commands
```bash
# Local testing
python -m main

# Remote debugging setup
INSTALL_METHOD=remote
REMOTE_INSTALL_HOST=your-host
REMOTE_INSTALL_PORT=5003
REMOTE_INSTALL_KEY=your-debug-key

# Package generation  
dify plugin package ./your_plugin/
```

## Critical Implementation Notes
- Use Pydantic BaseModel for all parameter classes
- Implement proper JSON serialization for tool arguments
- Handle Unicode properly with ensure_ascii=False
- Always validate tool_instance existence before invocation
- Implement proper logging hierarchy for complex workflows


- 总是参考 @docs\agent_plugin.md 中的 agent 插件开发指南