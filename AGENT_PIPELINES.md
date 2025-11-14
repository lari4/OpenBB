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

## Tool Activation/Deactivation Pipeline

### Overview

After discovering tools, agents can dynamically activate or deactivate them to manage their working context. This prevents context overflow and ensures agents only have access to relevant tools for their current task.

### Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│              Tool Activation/Deactivation Pipeline                  │
└─────────────────────────────────────────────────────────────────────┘
                                  │
         Agent has discovered needed tools
         (from Tool Discovery Pipeline)
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 1: Call activate_tools()           │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: activate_tools                    │
        │  Input:                                  │
        │    tool_names: [                         │
        │      "equity_price_quote",               │
        │      "equity_fundamental_ratios",        │
        │      "news_company"                      │
        │    ]                                     │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Processing                       │
        │                                          │
        │  1. Validate tool names exist            │
        │  2. Lookup tools in registry             │
        │  3. Call tool.enable() on each           │
        │  4. Update enabled_tools set             │
        │  5. Generate response message            │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │                                          │
        │  "Successfully activated 3 tools:        │
        │   - equity_price_quote                   │
        │   - equity_fundamental_ratios            │
        │   - news_company"                        │
        │                                          │
        │  Or if errors:                           │
        │  "Activated 2 tools. Not found: [...]"   │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                ┌──────────────────────────────────┐
                │  Agent Uses Activated Tools      │
                │  (tools now appear in context)   │
                └──────────────────────────────────┘
                                  │
                  Task complete, cleanup needed
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 2: Call deactivate_tools()         │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: deactivate_tools                  │
        │  Input:                                  │
        │    tool_names: [                         │
        │      "equity_price_quote",               │
        │      "equity_fundamental_ratios",        │
        │      "news_company"                      │
        │    ]                                     │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Processing                       │
        │                                          │
        │  1. Validate tool names exist            │
        │  2. Lookup tools in registry             │
        │  3. Call tool.disable() on each          │
        │  4. Remove from enabled_tools set        │
        │  5. Generate response message            │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │                                          │
        │  "Successfully deactivated 3 tools:      │
        │   - equity_price_quote                   │
        │   - equity_fundamental_ratios            │
        │   - news_company"                        │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                ┌──────────────────────────────────┐
                │  Tools Removed from Context      │
                │  Agent Context Reduced           │
                └──────────────────────────────────┘
```

### Implementation Details

**activate_tools() Function:**
```python
@mcp.tool(tags={"admin"})
def activate_tools(
    tool_names: Annotated[
        list[str], Field(description="Names of tools to activate")
    ],
) -> str:
    """Activate a tool for use."""
    return tool_registry.toggle_tools(tool_names, enable=True).message
```

**deactivate_tools() Function:**
```python
@mcp.tool(tags={"admin"})
def deactivate_tools(
    tool_names: Annotated[
        list[str], Field(description="Names of tools to deactivate")
    ],
) -> str:
    """Deactivate a tool for use."""
    return tool_registry.toggle_tools(tool_names, enable=False).message
```

**ToolRegistry.toggle_tools() Logic:**
```python
def toggle_tools(self, tool_names: list[str], enable: bool) -> ToggleResult:
    """Toggle tools on or off by name."""
    activated = []
    not_found = []

    for tool_name in tool_names:
        # Search across all categories and subcategories
        tool = self._find_tool_by_name(tool_name)

        if tool:
            if enable:
                tool.enable()      # Makes tool available to agent
            else:
                tool.disable()     # Removes tool from agent context

            activated.append(tool_name)
        else:
            not_found.append(tool_name)

    # Generate informative message
    action = "activated" if enable else "deactivated"

    if not_found:
        message = f"{action.capitalize()} {len(activated)} tools. "
        message += f"Not found: {not_found}"
    else:
        message = f"Successfully {action} {len(activated)} tools: "
        message += ", ".join(activated)

    return ToggleResult(
        activated=activated,
        not_found=not_found,
        message=message
    )
```

### State Management

**Tool States:**
```
┌──────────────────┐
│ Tool States      │
├──────────────────┤
│ enabled=True     │  ← Tool appears in agent's tool list
│ enabled=False    │  ← Tool hidden from agent
└──────────────────┘
```

**Global vs. Session State:**
- **Important:** Tool activation is GLOBAL on the MCP server
- Changes affect ALL connected agents (single-user limitation)
- For multi-user scenarios, disable tool discovery and use fixed tool sets

### Use Case Examples

**Example 1: Stock Analysis Task**
```
1. Agent needs stock data
   → activate_tools(["equity_price_quote", "equity_price_historical"])

2. Agent fetches price data
   → Uses equity_price_quote("AAPL")
   → Uses equity_price_historical("AAPL", start_date="2024-01-01")

3. Task complete
   → deactivate_tools(["equity_price_quote", "equity_price_historical"])
```

**Example 2: Comprehensive Analysis**
```
1. Agent starts with minimal tools
   → Only admin tools enabled

2. Discovers equity category
   → available_tools(category="equity", subcategory="price")

3. Activates price tools
   → activate_tools(["equity_price_quote", "equity_price_performance"])

4. Needs fundamental data
   → available_tools(category="equity", subcategory="fundamental")
   → activate_tools(["equity_fundamental_ratios", "equity_fundamental_metrics"])

5. Needs news context
   → available_tools(category="news")
   → activate_tools(["news_company"])

6. Performs analysis with all tools

7. Cleanup
   → deactivate_tools([
       "equity_price_quote",
       "equity_price_performance",
       "equity_fundamental_ratios",
       "equity_fundamental_metrics",
       "news_company"
     ])
```

### Benefits

- **Context Management:** Prevents agent from being overwhelmed with too many tools
- **Focused Analysis:** Only relevant tools are active for current task
- **Dynamic Workflow:** Agents can adapt their toolset as tasks evolve
- **Resource Efficiency:** Reduces token usage and cognitive load
- **Error Reduction:** Fewer tools means less chance of using wrong tool

### Configuration Note

**Fixed Tool Sets (Multi-User):**
```bash
# Disable tool discovery for fixed toolset
openbb-mcp --no-tool-discovery --default-categories equity,news,economy

# All specified categories are permanently enabled
# activate_tools/deactivate_tools are NOT available
```

---
