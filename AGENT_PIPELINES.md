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

## Prompt Execution Pipeline

### Overview

The Prompt Execution Pipeline enables agents to execute pre-configured workflow prompts that guide them through multi-step financial analysis tasks. Prompts act as structured templates with variable substitution, combining multiple tools into coherent workflows.

### Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Prompt Execution Pipeline                        │
└─────────────────────────────────────────────────────────────────────┘
                                  │
           Agent wants to execute a workflow
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 1: Discover Available Prompts      │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: list_prompts()                    │
        │  Input: None                             │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │  Returns list of prompts:                │
        │                                          │
        │  [                                       │
        │    {                                     │
        │      "name": "equity_analysis",          │
        │      "tags": ["equity", "server"],       │
        │      "arguments": [                      │
        │        {                                 │
        │          "name": "symbol",               │
        │          "description": "Ticker symbol", │
        │          "required": true                │
        │        },                                │
        │        {                                 │
        │          "name": "analysis_period",      │
        │          "description": "Time period",   │
        │          "required": false               │
        │        }                                 │
        │      ]                                   │
        │    },                                    │
        │    {...}                                 │
        │  ]                                       │
        └──────────────────────────────────────────┘
                                  │
            Agent selects appropriate prompt
            and prepares arguments
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Step 2: Execute Prompt                  │
        │                                          │
        │  Agent → MCP Server                      │
        │  Tool: execute_prompt                    │
        │  Input:                                  │
        │    prompt_name: "equity_analysis"        │
        │    arguments: {                          │
        │      "symbol": "AAPL",                   │
        │      "analysis_period": "last 24 months",│
        │      "focus_areas": "growth, innovation" │
        │    }                                     │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Processing                       │
        │                                          │
        │  1. Lookup prompt definition             │
        │  2. Merge provided + default arguments   │
        │  3. Validate required arguments          │
        │  4. Render template with arguments       │
        │  5. Return formatted prompt message      │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Argument Merging Example                │
        │                                          │
        │  Provided: {                             │
        │    "symbol": "AAPL",                     │
        │    "analysis_period": "last 24 months"   │
        │  }                                       │
        │                                          │
        │  Defaults: {                             │
        │    "analysis_period": "last 12 months",  │
        │    "focus_areas": "growth, profit..."    │
        │    "risk_tolerance": "moderate"          │
        │  }                                       │
        │                                          │
        │  Final: {                                │
        │    "symbol": "AAPL",                     │
        │    "analysis_period": "last 24 months",  │
        │    "focus_areas": "growth, profit...",   │
        │    "risk_tolerance": "moderate"          │
        │  }                                       │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Template Rendering                      │
        │                                          │
        │  Template:                               │
        │  "Conduct analysis of {symbol} for      │
        │   {analysis_period}. Follow workflow:    │
        │   1. Get price using equity_price...     │
        │   2. Get fundamentals using equity_...   │
        │   ..."                                   │
        │                                          │
        │  Rendered:                               │
        │  "Conduct analysis of AAPL for last     │
        │   24 months. Follow workflow:            │
        │   1. Get price using equity_price...     │
        │   2. Get fundamentals using equity_...   │
        │   ..."                                   │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Server Response                         │
        │                                          │
        │  PromptResult {                          │
        │    description: "Perform comprehensive   │
        │                  equity analysis...",    │
        │    messages: [                           │
        │      {                                   │
        │        role: "user",                     │
        │        content: {                        │
        │          type: "text",                   │
        │          text: "Conduct analysis of      │
        │                 AAPL for last 24..."     │
        │        }                                 │
        │      }                                   │
        │    ]                                     │
        │  }                                       │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Agent Executes Workflow                 │
        │                                          │
        │  1. Activates required tools             │
        │     (equity_price_quote, etc.)           │
        │  2. Executes steps from prompt           │
        │  3. Collects data from tools             │
        │  4. Synthesizes analysis                 │
        │  5. Returns result to user               │
        │  6. Deactivates tools                    │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                    ┌────────────────────────┐
                    │  Workflow Complete     │
                    └────────────────────────┘
```

### Prompt Types

**1. System Prompts (resource://system_prompt)**
- Single static text file
- Provides agent orientation and guidelines
- Not executed, retrieved as resource
- Tags: `{system}`

**2. Server Prompts (loaded from JSON)**
- Pre-configured workflow templates
- Stored in external JSON file
- Tags: `{server}` + custom tags
- Examples: equity_analysis, portfolio_analysis

**3. Inline/Route-Specific Prompts**
- Embedded in API endpoint metadata
- Specific to individual tools
- Tags: `{route-specific}` + tool name
- Examples: gdp_summary_prompt, gdp_comparison_prompt

### Implementation

**list_prompts() Function:**
```python
@mcp.tool(tags={"prompt"})
async def list_prompts() -> list:
    """List all available prompts."""
    prompts = await mcp.get_prompts()

    return [
        {
            "name": p.name,
            "tags": p.tags,
            "arguments": p.arguments
        }
        for p in prompts.values()
    ]
```

**execute_prompt() Function:**
```python
@mcp.tool(tags={"prompt"})
async def execute_prompt(
    prompt_name: Annotated[str, Field(description="Prompt name")],
    arguments: Annotated[dict, Field(description="Prompt arguments")],
) -> PromptResult:
    """Execute a prompt by name."""

    # Find prompt definition (server or inline)
    prompt_def = find_prompt_definition(prompt_name)

    if prompt_def:
        # Merge user arguments with defaults
        processed_args = arguments.copy()
        for arg_def in prompt_def.get("arguments", []):
            arg_name = arg_def.get("name")
            if "default" in arg_def and arg_name not in processed_args:
                processed_args[arg_name] = arg_def["default"]

        # Render prompt with merged arguments
        return await mcp._prompt_manager.render_prompt(
            name=prompt_name,
            arguments=processed_args
        )

    # Render with provided arguments only
    return await mcp._prompt_manager.render_prompt(
        name=prompt_name,
        arguments=arguments
    )
```

**StaticPrompt.render() Method:**
```python
class StaticPrompt(Prompt):
    content: str

    async def render(
        self,
        arguments: dict[str, Any] | None = None,
    ) -> list[PromptMessage]:
        """Render the prompt with arguments."""
        args = arguments or {}

        # Validate required arguments
        if self.arguments:
            required = {arg.name for arg in self.arguments if arg.required}
            provided = set(args)
            missing = required - provided
            if missing:
                raise PromptError(f"Missing required arguments: {missing}")

        try:
            # Template substitution using Python's .format()
            rendered_content = self.content.format(**args)
            return [
                PromptMessage(
                    role="user",
                    content=TextContent(type="text", text=rendered_content)
                )
            ]
        except KeyError as e:
            raise PromptError(f"Missing argument for formatting: {e}") from e
```

### Use Case Examples

**Example 1: Equity Analysis**
```
Agent: list_prompts()
→ Sees "equity_analysis" prompt

Agent: execute_prompt(
    prompt_name="equity_analysis",
    arguments={"symbol": "TSLA", "risk_tolerance": "aggressive"}
)

Server: Returns rendered prompt:
"Conduct comprehensive analysis of TSLA for last 12 months.
Follow this workflow:
1. Get price performance using equity_price_performance
2. Get fundamental data using equity_fundamental_ratios
3. Get news using news_company
4. Compare with peers using equity_compare_peers
5. Summarize with investment recommendation
Focus areas: growth, profitability, valuation
Risk tolerance: aggressive"

Agent: Follows workflow steps, using tools as directed
```

**Example 2: GDP Comparison**
```
Agent: execute_prompt(
    prompt_name="gdp_comparison_prompt",
    arguments={"country1": "USA", "country2": "China"}
)

Server: Returns rendered prompt:
"Use the tool, economy_gdp, to perform the following task.

Compare the GDP growth of USA and China."

Agent: Activates economy_gdp tool and fetches data
```

---

## Financial Analysis Workflow Pipeline

### Overview

This pipeline demonstrates a complete end-to-end financial analysis workflow combining tool discovery, activation, prompt execution, and data synthesis. It represents a typical agent workflow when performing comprehensive stock analysis.

### Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│             Complete Financial Analysis Workflow                    │
│                  (End-to-End Example)                               │
└─────────────────────────────────────────────────────────────────────┘
                                  │
              User: "Analyze Apple stock (AAPL)"
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                      Phase 1: Initialization                      ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  Agent starts with minimal context       │
        │  Enabled tools: [admin tools only]       │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                    Phase 2: Tool Discovery                        ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  1. Call: available_categories()         │
        │     Response: [equity, news, economy...] │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  2. Call: available_tools("equity")      │
        │     Response: Subcategories discovered   │
        │     - price (8 tools)                    │
        │     - fundamental (12 tools)             │
        │     - estimates (5 tools)                │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  3. Call: available_tools("news")        │
        │     Response: News tools discovered      │
        │     - news_company (1 tool)              │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                   Phase 3: Tool Activation                        ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  Call: activate_tools([                  │
        │    "equity_price_quote",                 │
        │    "equity_price_performance",           │
        │    "equity_fundamental_ratios",          │
        │    "equity_fundamental_metrics",         │
        │    "equity_estimates_price_target",      │
        │    "news_company"                        │
        │  ])                                      │
        │                                          │
        │  Response: "Successfully activated 6     │
        │            tools..."                     │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                   Phase 4: Data Collection                        ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  1. Call: equity_price_quote("AAPL")     │
        │     Data: Current price, volume, market  │
        │           cap, P/E ratio                 │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  2. Call: equity_price_performance(      │
        │            "AAPL", period="1Y")          │
        │     Data: 1-year return, volatility,     │
        │           highs, lows                    │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  3. Call: equity_fundamental_ratios(     │
        │            "AAPL")                       │
        │     Data: ROE, ROA, profit margins,      │
        │           debt ratios                    │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  4. Call: equity_fundamental_metrics(    │
        │            "AAPL")                       │
        │     Data: Revenue, earnings, cash flow   │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  5. Call: equity_estimates_price_target( │
        │            "AAPL")                       │
        │     Data: Analyst targets, consensus     │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  6. Call: news_company("AAPL")           │
        │     Data: Recent news articles           │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                   Phase 5: Analysis & Synthesis                   ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  Agent analyzes collected data:          │
        │                                          │
        │  - Price: $175.50, Up 45% YoY            │
        │  - Fundamentals: Strong margins, low debt│
        │  - Estimates: PT $200 (14% upside)       │
        │  - News: Product launch, positive        │
        │                                          │
        │  Synthesis: Strong buy recommendation    │
        │  based on growth, fundamentals, and      │
        │  analyst consensus                       │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                   Phase 6: Response & Cleanup                     ║
╚═══════════════════════════════════════════════════════════════════╝
                                  │
        ┌──────────────────────────────────────────┐
        │  Agent → User:                           │
        │  "Apple (AAPL) Analysis Summary:         │
        │   Current Price: $175.50                 │
        │   1-Year Return: +45%                    │
        │   Analyst Target: $200 (14% upside)      │
        │   Recommendation: Strong Buy             │
        │   Rationale: Strong fundamentals,        │
        │   positive analyst sentiment, recent     │
        │   product innovation..."                 │
        └──────────────────────────────────────────┘
                                  │
        ┌──────────────────────────────────────────┐
        │  Call: deactivate_tools([all 6 tools])   │
        │  Response: "Successfully deactivated..." │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                    ┌────────────────────────┐
                    │  Analysis Complete     │
                    │  Context Cleaned Up    │
                    └────────────────────────┘
```

### Workflow Sequence Details

**Phase 1: Initialization**
- Agent starts fresh with only admin tools
- Receives user request for analysis

**Phase 2: Tool Discovery**
- Progressively explores available tools
- Identifies relevant categories (equity, news)
- Examines subcategories within equity

**Phase 3: Tool Activation**
- Selects specific tools needed for analysis
- Activates them in a single batch call
- Receives confirmation of activation

**Phase 4: Data Collection**
- Executes tools in logical sequence
- Collects price, fundamental, estimate, and news data
- Each tool returns structured financial data

**Phase 5: Analysis & Synthesis**
- Processes collected data
- Identifies patterns and insights
- Formulates investment recommendation

**Phase 6: Response & Cleanup**
- Delivers structured analysis to user
- Deactivates tools to free context
- Returns to minimal tool state

### Alternative: Prompt-Guided Workflow

Instead of manually orchestrating tools, the agent could use a pre-configured prompt:

```
Agent: execute_prompt(
    prompt_name="equity_analysis",
    arguments={
        "symbol": "AAPL",
        "analysis_period": "last 12 months",
        "risk_tolerance": "moderate"
    }
)

Server: Returns structured workflow instructions

Agent: Follows prompt's workflow:
  1. equity_price_performance("AAPL") → Price data
  2. equity_fundamental_ratios("AAPL") → Fundamentals
  3. equity_estimates_price_target("AAPL") → Targets
  4. news_company("AAPL") → News
  5. Synthesize → Recommendation

Result: Same analysis with less manual orchestration
```

---

## LangChain Agent Pipeline

### Overview

The LangChain pipeline demonstrates an alternative agent architecture using OpenAI's function calling with LangChain's agent framework. Unlike MCP's dynamic tool discovery, this approach uses pre-configured tools with chain-of-thought reasoning.

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LangChain Agent Architecture                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Components Layer                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   LLM Core   │  │    Memory    │  │     Tools    │             │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤             │
│  │ OpenAI GPT-4 │  │ Conversation │  │ OpenBB Tools │             │
│  │ Temperature:0│  │ Token Buffer │  │ (8 custom)   │             │
│  └──────────────┘  │ 16K tokens   │  └──────────────┘             │
│                    └──────────────┘                                │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Agent Layer                                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ChatPromptTemplate                                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ System: "You are very powerful stock financial researcher..." │  │
│  │ Memory: {chat_history}                                        │  │
│  │ User: {input}                                                 │  │
│  │ Scratchpad: {agent_scratchpad}                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  OpenAI Tools Agent + AgentExecutor                                 │
└─────────────────────────────────────────────────────────────────────┘
```

### Execution Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                  LangChain Execution Flow                           │
└─────────────────────────────────────────────────────────────────────┘
                                  │
              User provides complex prompt
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  User Input (Chain-of-Thought Prompt)    │
        │                                          │
        │  "First, find an industry with positive  │
        │   performance across quarterly, monthly, │
        │   and weekly timeframes.                 │
        │   Second, extract valuation metrics...   │
        │   Third, extract companies using         │
        │   relaxed criteria...                    │
        │   Fourth, get analyst consensus...       │
        │   Finally, summarize findings..."        │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Agent Processing Loop                   │
        │                                          │
        │  1. LLM receives prompt + system message │
        │  2. LLM reasons about next action        │
        │  3. LLM selects tool to call             │
        │  4. Agent Executor calls tool            │
        │  5. Tool returns data to LLM             │
        │  6. LLM updates reasoning (scratchpad)   │
        │  7. Repeat until task complete           │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                    Iteration 1: Find Best Industry                ║
╚═══════════════════════════════════════════════════════════════════╝
        │
        ├─> LLM Reasoning: "Need industry performance data"
        │
        ├─> Tool Call: get_industry_performance()
        │
        ├─> Tool Response: [
        │     {industry: "Semiconductors", week: +5%, month: +8%, ...},
        │     {industry: "Software", week: +3%, month: +6%, ...}
        │   ]
        │
        └─> LLM: "Semiconductors shows best performance"
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                 Iteration 2: Get Valuation Metrics                ║
╚═══════════════════════════════════════════════════════════════════╝
        │
        ├─> LLM Reasoning: "Need valuation for Semiconductors"
        │
        ├─> Tool Call: get_valuation_for_industries("Semiconductors")
        │
        ├─> Tool Response: {
        │     PE: 28.5, PB: 6.2, EV_EBITDA: 18.3
        │   }
        │
        └─> LLM: "Stored valuation metrics"
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║              Iteration 3: Screen Investment Candidates            ║
╚═══════════════════════════════════════════════════════════════════╝
        │
        ├─> LLM Reasoning: "Need companies in Semiconductors"
        │
        ├─> Tool Call: get_candidate_stocks_to_invest_relaxed(
        │       "Semiconductors"
        │     )
        │
        ├─> Tool Response: [
        │     {symbol: "NVDA", name: "NVIDIA Corp", ...},
        │     {symbol: "AMD", name: "Advanced Micro...", ...}
        │   ]
        │
        └─> LLM: "Found top performers: NVDA, AMD"
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                Iteration 4: Get Analyst Consensus                 ║
╚═══════════════════════════════════════════════════════════════════╝
        │
        ├─> LLM Reasoning: "Need consensus for NVDA"
        │
        ├─> Tool Call: get_consensus("NVDA")
        │
        ├─> Tool Response: {
        │     target_high: 1200, target_low: 950,
        │     target_consensus: 1050, target_median: 1040
        │   }
        │
        ├─> LLM Reasoning: "Need consensus for AMD"
        │
        ├─> Tool Call: get_consensus("AMD")
        │
        ├─> Tool Response: {
        │     target_high: 280, target_low: 200,
        │     target_consensus: 240, target_median: 235
        │   }
        │
        └─> LLM: "Have all required data"
                                  │
                                  ▼
╔═══════════════════════════════════════════════════════════════════╗
║                    Iteration 5: Synthesize Answer                 ║
╚═══════════════════════════════════════════════════════════════════╝
        │
        └─> LLM Reasoning: "All data collected, can now summarize"
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Final Response                          │
        │                                          │
        │  "Best Performing Industry:              │
        │   Semiconductors                         │
        │                                          │
        │  Valuation: PE 28.5, PB 6.2              │
        │                                          │
        │  Top Companies:                          │
        │  - NVDA: Target $1050 (8% upside)        │
        │  - AMD: Target $240 (15% upside)         │
        │                                          │
        │  Recommendation: Both show strong        │
        │  analyst support with moderate upside    │
        │  potential. Consider AMD for higher      │
        │  risk/reward profile."                   │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
        ┌──────────────────────────────────────────┐
        │  Memory Update                           │
        │  - Stores conversation in buffer         │
        │  - Maintains context for follow-ups      │
        │  - Respects 16K token limit              │
        └──────────────────────────────────────────┘
                                  │
                                  ▼
                    ┌────────────────────────┐
                    │  Agent Ready for Next  │
                    │  User Query            │
                    └────────────────────────┘
```

### Key Differences from MCP Pipeline

| Aspect | MCP Server | LangChain |
|--------|------------|-----------|
| **Tool Discovery** | Dynamic (100+ tools, activate on demand) | Static (8 pre-configured tools) |
| **Tool Management** | activate_tools/deactivate_tools | All tools always available |
| **Prompts** | Structured templates with execute_prompt | Embedded in ChatPromptTemplate |
| **Memory** | Stateless (per MCP session) | ConversationTokenBufferMemory |
| **Reasoning** | Agent-directed workflows | Chain-of-thought prompting |
| **Best For** | Multiple agents, discovery workflows | Single-session analysis |

### Tool Definitions

**Available Tools:**
1. `get_industry_performance()` - Industry performance metrics
2. `get_strong_buy_for_sector(sector)` - Strong buy recommendations
3. `get_strong_buy_for_industry(industry)` - Industry-specific buys
4. `get_valuation_for_industries(industry)` - Valuation metrics
5. `get_candidate_stocks_to_invest_relaxed(industry)` - Stock screening
6. `get_consensus(ticker)` - Analyst consensus

### Conversation Flow Example

```python
# First interaction
agent_executor.invoke({
    "input": "Find best performing industry and top stocks",
    "chat_history": []
})

# Follow-up (memory preserved)
agent_executor.invoke({
    "input": "What about the Utilities sector?",
    "chat_history": previous_chat_history
})

# Agent remembers context from previous analysis
# and can reference it in new response
```

---

## Summary

OpenBB provides multiple AI agent pipeline architectures:

### MCP Server Pipelines (Primary)
1. **Initialization Pipeline** - Server setup, tool registration, prompt loading
2. **Tool Discovery Pipeline** - Progressive exploration of 100+ tools
3. **Tool Activation Pipeline** - Dynamic context management
4. **Prompt Execution Pipeline** - Structured workflow templates
5. **Financial Analysis Workflow** - End-to-end equity analysis example

### LangChain Pipeline (Example)
6. **LangChain Agent Pipeline** - Chain-of-thought with memory

Each pipeline serves specific use cases:
- **MCP** for multi-agent, discoverable, workflow-driven analysis
- **LangChain** for single-session, memory-based, conversational analysis

Both approaches leverage OpenBB's comprehensive financial data platform while providing different agent interaction paradigms.
