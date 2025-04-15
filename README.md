# MCP Deep-Dive

This document provides a comprehensive overview of the architecture and interaction between the **MCP Client** and **MCP Server**, highlighting the roles of **Tools**, **Resources**, and **Prompts** in the system.

---

## Overview

The **MCP (Model-Computer Protocol)** facilitates structured interactions between the MCP Client and Server through three primary components:

- **Tools** : Model-controlled
- **Resources** : Application-controlled
- **Prompts** : User-controlled

---


### Client-Server Interaction Diagram

```text
+-----------------------+                   +-------------------+
|    MCP Client         | <-------------->  |    MCP Server     |
|-----------------------|                   |-------------------|
| Invokes Tools         |                   | Exposes Tools     |
| Queries for Resources |                   | Exposes Resources |
| Interpolates Prompts  |                   | Exposes Prompts   |
+-----------------------+                   +-------------------+
```

## Deep Dive into MCP Components

### 1.Tools: Empowering AI with Actionable Functions

**What are Tools?**
- Executable functions that AI can invoke
- Designed to be automatically called by the AI model
- Provide concrete actions and computations

**Characteristics:**
- Model-controlled
- Can perform various actions like:
  - `Retrieve/search data`
  - `Send messages`
  - `Update database records`

### What Makes a Tool Powerful?

Tools are the executable functions that transform AI from a passive information processor to an active problem-solver. They are the bridge between computational thinking and practical action.

**Example (Math Calculator)**:
```python
@mcp.tool()
def add(a: float, b: float) -> float:
    """
    Add two numbers together.
    
    Args:
        a: First number
        b: Second number
        
    Returns:
        Sum of a and b
    """
    return a + b
```
```python
@mcp.tool()
def subtract(a: float, b: float) -> float:
    """
    Subtract the second number from the first.
    
    Args:
        a: Number to subtract from
        b: Number to subtract
        
    Returns:
        Difference between a and b
    """
    return a - b
```
```python
@mcp.tool()
def multiply(a: float, b: float) -> float:
    """
    Multiply two numbers together.
    
    Args:
        a: First number
        b: Second number
        
    Returns:
        Product of a and b
    """
    return a * b
```
```python
@mcp.tool()
def divide(a: float, b: float) -> str:
    """
    Divide the first number by the second.
    
    Args:
        a: Numerator
        b: Denominator
        
    Returns:
        Result of division or error message
    """
    if b == 0:
        return "Error: Division by zero is not allowed"
    return str(a / b)
```

### Tool Design Principles

1. **Atomic Functionality**
   - Each tool should do one thing well
   - Focus on a specific, well-defined task
   - Avoid complex, multi-step operations

2. **Robust Error Handling**
   - Division by zero prevention
   - Type-safe operations
   - Clear error messaging
   ```

3. **Type Annotations**
   - Use Python type hints
   - Clearly define input and output types
   - Enable automatic validation

### Advanced Tool Patterns

#### Async Tools
```python
@mcp.tool()
async def fetch_weather(location: str) -> dict:
    """Fetch real-time weather data"""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.weather.com/{location}")
        return response.json()
```

#### Tools with Complex Input Validation
```python
from pydantic import BaseModel, Field
from typing import List

class EmailAnalysisRequest(BaseModel):
    emails: List[str] = Field(..., min_items=1, max_items=100)
    analysis_type: str = Field(default="basic")

@mcp.tool()
def analyze_emails(request: EmailAnalysisRequest) -> dict:
    """Perform comprehensive email analysis"""
    # Validated input automatically handled
    results = {
        'total_emails': len(request.emails),
        'analysis_type': request.analysis_type
    }
    return results
```

-----

## 2.Resources: Data Access and Context
**What are Resources?**
- Expose data from various sources
- Controlled by the application
- Provide context and information

**Characteristics:**
- Application-controlled
- Can include various data types:
  - `Files`
  - `Database records`
  - `API responses`

### Resource Types and Patterns

1. **Static Resources**
```python
@mcp.resource("config://application")
def get_app_configuration() -> str:
    """Provide static application configuration"""
    return json.dumps({
        'version': '1.0.0',
        'environment': 'production',
        'features': ['authentication', 'logging']
    })
```

2. **Dynamic Resources**
```python
@mcp.resource("user://{user_id}/profile")
def get_user_profile(user_id: str) -> dict:
    """Dynamically fetch user profile based on ID"""
    # Implement database or API lookup
    return user_database.get_profile(user_id)
```

### Resource Access Patterns

1. **Direct Resource Retrieval**
```python
# Fetch a specific resource
user_profile = client.read_resource("user://123/profile")
```

2. **Resource Listing**
```python
# List available resources
available_resources = client.list_resources()
```

## Example (Math Calculator Help Resource):



```python
@mcp.resource("calculator://help")
def calculator_help() -> str:
    """Provide comprehensive calculator usage instructions"""
    return """
    # Basic Math Calculator Usage Guide

    ## Supported Operations
    - Addition: `add(a, b)` 
      - Adds two numbers together
    
    - Subtraction: `subtract(a, b)`
      - Subtracts the second number from the first
    
    - Multiplication: `multiply(a, b)`
      - Multiplies two numbers
    
    - Division: `divide(a, b)`
      - Divides first number by second
      - Prevents division by zero

    ## Example Usage
    - Add 5 and 3: `add(5, 3)` → Returns 8
    - Divide 10 by 2: `divide(10, 2)` → Returns 5.0
    """
```
----
### 3.Prompts: Guided AI Interactions

**What are Prompts?**
- Pre-defined interaction templates
- User-controlled workflow guidance
- Standardize complex interactions

**Characteristics:**
- User-controlled
- Provide structured interaction patterns

**Potential Prompt Examples**:
- `Document Q&A`
- `Code review templates`
- `Analysis guidelines`
- `Structured output generation`
- `Transcript summaries`
- `JSON output generation`

### Understanding Prompts

Prompts are structured templates that guide AI interactions, providing context, constraints, and workflow patterns.

#### Prompt Anatomy
```python
@mcp.prompt()
def math_problem_solving() -> List[Message]:
    """Guide users through mathematical problem-solving"""
    return [
        UserMessage("I need help solving a math problem."),
        AssistantMessage("I can help you use our calculator tools to solve it step by step."),
        SystemMessage("Encourage breaking down complex problems into simple operations.")
    ]
```

### Prompt Design Strategies

1. **Context Provision**
   - Include relevant background information
   - Set clear expectations
   - Provide step-by-step guidance

2. **Multi-stage Prompts**
```python
@mcp.prompt()
def troubleshooting_workflow(error_log: str) -> List[Message]:
    return [
        UserMessage(f"I'm experiencing this error: {error_log}"),
        AssistantMessage("Let's diagnose the issue step by step."),
        UserMessage("What have you already tried?"),
        SystemMessage("Guide the user through systematic troubleshooting.")
    ]
```

### Resource Integration in Prompts

```python
@mcp.prompt()
def project_analysis_prompt(project_files: List[Resource]) -> List[Message]:
    """Analyze project structure using multiple resources"""
    return [
        UserMessage("Please analyze the following project structure:"),
        ResourceMessage(project_files),
        AssistantMessage("I'll provide a comprehensive project assessment.")
    ]
```
