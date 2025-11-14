# OpenBB AI Agent Pipelines Documentation

This document provides comprehensive documentation of all AI agent workflows and pipelines in the OpenBB application. Each pipeline is described with ASCII diagrams showing data flow, prompt sequences, and tool interactions.

## Table of Contents

1. [MCP Server Initialization Pipeline](#mcp-server-initialization-pipeline)
2. [Tool Discovery Pipeline](#tool-discovery-pipeline)
3. [Tool Activation/Deactivation Pipeline](#tool-activationdeactivation-pipeline)
4. [Prompt Execution Pipeline](#prompt-execution-pipeline)
5. [Financial Analysis Workflow Pipeline](#financial-analysis-workflow-pipeline)
6. [LangChain Agent Pipeline](#langchain-agent-pipeline)

---

## MCP Server Initialization Pipeline

### Overview

The MCP Server initialization pipeline sets up the OpenBB platform as an AI-accessible service through the Model Context Protocol. This pipeline processes FastAPI routes, configures prompts, and creates discoverable tools for AI agents.

### Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                     MCP Server Initialization                       │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  1. Load MCPSettings Configuration       │
        │     - CLI arguments (highest priority)   │
        │     - Environment variables              │
        │     - ~/.openbb_platform/mcp_settings    │
        │     - Default values                     │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  2. Create Tool Registry                 │
        │     - Initialize empty registry          │
        │     - Prepare for tool categorization    │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  3. Process FastAPI Routes               │
        │     - Scan all API endpoints             │
        │     - Extract route metadata             │
        │     - Build route lookup dictionary      │
        │     - Extract inline prompt definitions  │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  4. Create FastMCP Server                │
        │     - Convert routes to MCP components   │
        │     - Apply customizations per route     │
        │     - Categorize and tag tools           │
        │     - Register tools in registry         │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  5. Load System Prompt (if configured)   │
        │     - Read from text file                │
        │     - Register as prompt + resource      │
        │     - Tag with "system"                  │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  6. Load Server Prompts (if configured)  │
        │     - Read JSON file                     │
        │     - Validate each prompt definition    │
        │     - Create StaticPrompt instances      │
        │     - Tag with "server"                  │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  7. Register Inline Prompts              │
        │     - Process extracted route prompts    │
        │     - Create StaticPrompt instances      │
        │     - Tag with "route-specific"          │
        │     - Associate with parent tools        │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  8. Register Admin Tools (if enabled)    │
        │     - available_categories()             │
        │     - available_tools()                  │
        │     - activate_tools()                   │
        │     - deactivate_tools()                 │
        │     Tag: "admin"                         │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  9. Register Prompt Tools                │
        │     - list_prompts()                     │
        │     - execute_prompt()                   │
        │     Tag: "prompt"                        │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  10. Start Server                        │
        │     Transport options:                   │
        │     - stdio (for Claude Desktop)         │
        │     - http/sse (for web clients)         │
        │     - streamable-http (default)          │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │  Server Ready for Agents │
                    └──────────────────────────┘
```

### Key Components

**Input Data:**
- MCPSettings configuration
- FastAPI application instance
- System prompt file (optional)
- Server prompts JSON file (optional)

**Processing Steps:**
1. Configuration loading with priority order
2. Route introspection and metadata extraction
3. Tool categorization and naming
4. Prompt validation and registration
5. Dynamic tool enabling/disabling based on settings

**Output:**
- Running MCP server
- Registered tools (100+ financial data endpoints)
- Registered prompts (system, server, and inline)
- Admin tools for discovery
- Tool registry for dynamic management

**Configuration Priority:**
```
CLI Arguments > Environment Variables > Config File > Defaults
```

---

## Tool Discovery Pipeline

### Overview

The Tool Discovery Pipeline enables AI agents to explore available financial data tools dynamically. Instead of being overwhelmed with 100+ tools at once, agents can discover categories, examine subcategories, and activate only the tools they need for specific tasks.

### Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Tool Discovery Workflow                         │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                   Agent starts with minimal tools
                   (only "admin" category enabled)
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 1: Call available_categories()     │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: available_categories              │
        │  Input: None                             │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │  Returns list of CategoryInfo:           │
        │                                          │
        │  [                                       │
        │    {                                     │
        │      "name": "equity",                   │
        │      "total_tools": 45,                  │
        │      "subcategories": [                  │
        │        {"name": "price", "tool_count": 8}│
        │        {"name": "fundamental", ...}      │
        │      ]                                   │
        │    },                                    │
        │    {"name": "economy", ...},             │
        │    {"name": "crypto", ...}               │
        │  ]                                       │
        └──────────────────────────────────────────┘
                                  │
                   Agent analyzes categories and
                   decides which to explore further
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 2: Call available_tools()          │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: available_tools                   │
        │  Input:                                  │
        │    category: "equity"                    │
        │    subcategory: "price" (optional)       │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │  Returns list of ToolInfo:               │
        │                                          │
        │  [                                       │
        │    {                                     │
        │      "name": "equity_price_quote",       │
        │      "active": false,                    │
        │      "description": "Get current quote"  │
        │    },                                    │
        │    {                                     │
        │      "name": "equity_price_historical",  │
        │      "active": false,                    │
        │      "description": "Get historical..."  │
        │    }                                     │
        │  ]                                       │
        └──────────────────────────────────────────┘
                                  │
                   Agent identifies needed tools
                                  │
                                  ▼
                    ┌────────────────────────┐
                    │  Discovery Complete    │
                    │  Ready to Activate     │
                    └────────────────────────┘
```

### Tool Registry Structure

```
ToolRegistry
├── categories/
│   ├── equity/
│   │   ├── price/
│   │   │   ├── equity_price_quote → OpenAPITool
│   │   │   ├── equity_price_historical → OpenAPITool
│   │   │   └── equity_price_performance → OpenAPITool
│   │   ├── fundamental/
│   │   │   ├── equity_fundamental_ratios → OpenAPITool
│   │   │   ├── equity_fundamental_metrics → OpenAPITool
│   │   │   └── equity_fundamental_balance → OpenAPITool
│   │   └── estimates/
│   │       ├── equity_estimates_price_target → OpenAPITool
│   │       └── equity_estimates_consensus → OpenAPITool
│   ├── economy/
│   │   ├── general/
│   │   │   ├── economy_gdp → OpenAPITool
│   │   │   ├── economy_cpi → OpenAPITool
│   │   │   └── economy_unemployment → OpenAPITool
│   │   └── indicators/
│   │       └── ...
│   ├── crypto/
│   │   └── ...
│   └── news/
│       └── ...
└── enabled_tools: Set[str]  # Currently active tools
```

### Data Flow

**Request: available_categories()**
```python
# Agent calls
available_categories()

# Server executes
def available_categories() -> list[CategoryInfo]:
    categories = tool_registry.get_categories()
    return [
        CategoryInfo(
            name=category_name,
            subcategories=[
                SubcategoryInfo(name=subcat_name, tool_count=len(tools))
                for subcat_name, tools in sorted(subcategories.items())
            ],
            total_tools=sum(len(tools) for tools in subcategories.values())
        )
        for category_name, subcategories in sorted(categories.items())
    ]

# Returns structured category overview
```

**Request: available_tools(category, subcategory)**
```python
# Agent calls
available_tools(category="equity", subcategory="price")

# Server executes
def available_tools(category: str, subcategory: str = None) -> list[ToolInfo]:
    # Get category data
    category_data = tool_registry.get_category_subcategories(category)

    # If category not found, return error with available categories
    if not category_data:
        raise ValueError(f"Category '{category}' not found...")

    # Filter by subcategory if provided
    if subcategory:
        tools_dict = tool_registry.get_category_tools(category, subcategory)
        if not tools_dict:
            raise ValueError(f"Subcategory '{subcategory}' not found...")
    else:
        tools_dict = tool_registry.get_category_tools(category)

    # Return tool information
    return [
        ToolInfo(
            name=name,
            active=tool.enabled,
            description=_extract_brief_description(tool.description or "")
        )
        for name, tool in sorted(tools_dict.items())
    ]

# Returns list of tools with activation status
```

### Use Case Example

**Scenario:** Agent needs to analyze Apple stock

```
1. Agent: available_categories()
   → Sees "equity" category with 45 tools

2. Agent: available_tools(category="equity")
   → Sees subcategories: "price", "fundamental", "estimates"

3. Agent: available_tools(category="equity", subcategory="price")
   → Finds: equity_price_quote, equity_price_historical, equity_price_performance

4. Agent: available_tools(category="equity", subcategory="fundamental")
   → Finds: equity_fundamental_ratios, equity_fundamental_metrics

5. Agent decides which tools to activate based on task requirements
   → Proceeds to Tool Activation Pipeline
```

### Key Features

- **Progressive Discovery:** Agents explore in stages (category → subcategory → tools)
- **Error Handling:** Helpful error messages with available options
- **Tool Metadata:** Each tool includes name, activation status, and description
- **Filtered Views:** Subcategory filtering reduces information overload
- **Dynamic State:** Tool activation status reflects current server state

---
