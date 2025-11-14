# OpenBB AI Prompts Documentation

This document provides a comprehensive overview of all AI prompts used in the OpenBB application, organized by category and purpose.

## Table of Contents

1. [Development & Operations Prompts](#development--operations-prompts)
2. [MCP Server System Prompts](#mcp-server-system-prompts)
3. [MCP Server Financial Analysis Prompts](#mcp-server-financial-analysis-prompts)
4. [LangChain Integration Prompts](#langchain-integration-prompts)

---

## Development & Operations Prompts

### 1. Changelog Summarization Prompt

**Location:** `.github/scripts/summarize_changelog.py`

**Purpose:** This prompt is used to automatically generate concise, well-structured summaries for changelog entries in new releases. It processes Pull Request details from GitHub and creates a high-level overview that captures the essence of changes made in the release.

**AI Model:** OpenAI GPT-4

**Input Data:**
- Pull Request titles and descriptions
- Categorized changes (Breaking Changes, Enhancements, Bug Fixes, Documentation)
- Release version information

**Output:** A concise, story-like summary of the release changes

**Environment Requirements:**
- `OPENAI_API_KEY` environment variable must be set

**Prompt:**
```python
{
    "role": "system",
    "content": "Summarize the following text in a concise way to describe what happened in the new release. This will be used on top of the changelog to provide a high-level overview of the changes. Make sure it is well-written, concise, structured and that it captures the essence of the text. It should read like a concise story."
}
```

**Usage Context:**
This prompt is invoked during the release process when generating changelog documentation. The AI receives a compilation of all PR details and produces a human-readable summary that helps users quickly understand the scope and impact of the release.

---

## MCP Server System Prompts

The MCP (Model Context Protocol) Server is the primary AI integration point in OpenBB. It exposes OpenBB Platform's REST API endpoints as AI tools that can be used by various AI agents and LLM clients (Claude Desktop, Cursor, VS Code, etc.).

### Architecture Overview

**Location:** `openbb_platform/extensions/mcp_server/`

**Core Components:**
- **Prompt Models** (`openbb_mcp_server/models/prompts.py`) - StaticPrompt class for template rendering
- **Configuration** (`openbb_mcp_server/models/mcp_config.py`) - Validation models for prompt structures
- **Settings** (`openbb_mcp_server/models/settings.py`) - Environment and file-based configuration

### 1. System Prompt Configuration

**Location:** Configurable via `system_prompt_file` setting

**Purpose:** Provides system-level instructions to AI agents on how to interact with the OpenBB MCP server. This prompt establishes the context, capabilities, and usage guidelines for the AI agent.

**Configuration Methods:**
1. Command line: `--system-prompt /path/to/system_prompt.txt`
2. Environment variable: `OPENBB_MCP_SYSTEM_PROMPT_FILE=/path/to/system_prompt.txt`
3. Config file: `~/.openbb_platform/mcp_settings.json`

**File Format:** Plain text file (`.txt`)

**Exposure:**
- Available as resource: `resource://system_prompt`
- Discoverable via: `list_prompts` tool

**Usage Context:**
The system prompt is not automatically applied by clients. AI agents should be instructed to fetch and use it during their initialization/orientation phase.

**Example System Prompt Structure:**
```text
You are an AI assistant with access to OpenBB financial data platform.

Available Capabilities:
- Access to 100+ financial data endpoints
- Dynamic tool discovery and activation
- Financial analysis workflows via prompts

Guidelines:
1. Use tool discovery to find relevant data sources
2. Activate only the tools you need for the current task
3. Follow structured workflows provided in server prompts
4. Always cite data sources in your responses

Data Categories Available:
- Equity: Stock data, fundamentals, price history
- Economy: Economic indicators, GDP, employment
- News: Financial news from various sources
- Crypto: Cryptocurrency data and analysis
- Fixed Income: Bond data, rates, securities
- And more...

Best Practices:
- Start with minimal tools (admin category)
- Use available_categories to explore options
- Activate specific tools with activate_tools
- Deactivate unused tools to reduce context
```

### 2. Server Prompts Configuration

**Location:** Configurable via `server_prompts_file` setting

**Purpose:** Provides pre-built workflow prompts that guide AI agents through multi-step financial analysis tasks. These prompts combine multiple tools and data sources into coherent analytical workflows.

**Configuration Methods:**
1. Command line: `--server-prompts /path/to/prompts.json`
2. Environment variable: `OPENBB_MCP_SERVER_PROMPTS_FILE=/path/to/prompts.json`
3. Config file: `~/.openbb_platform/mcp_settings.json`

**File Format:** JSON file with array of prompt definitions

**JSON Schema:**
```json
[
  {
    "name": "string",          // Prompt identifier (required)
    "description": "string",   // Brief description (required)
    "content": "string",       // Template with {variable} placeholders (required)
    "arguments": [             // Optional list of arguments
      {
        "name": "string",      // Argument name (required)
        "type": "str|int|float|bool|list|dict",  // Type (default: "str")
        "default": "any",      // Default value (makes arg optional)
        "description": "string" // Argument description
      }
    ],
    "tags": ["string"]         // Optional tags for categorization
  }
]
```

**Validation:**
- Prompt names must be valid Python identifiers
- Content cannot be empty and must have balanced braces
- Arguments are validated for type and required fields
- Invalid prompts are logged and skipped (non-blocking)

**Discovery:**
- Listed via `list_prompts` tool
- Executed via `execute_prompt` tool
- Tagged with "server" category automatically

---

## MCP Server Financial Analysis Prompts

These are concrete examples of workflow prompts that guide AI agents through complex financial analysis tasks using multiple OpenBB tools.

### 1. Equity Analysis Workflow Prompt

**Location:** Server prompts configuration file (example from documentation)

**Purpose:** Guides AI agents through a comprehensive equity analysis process, combining multiple data sources including price performance, fundamental data, news, analyst estimates, and peer comparisons.

**Workflow Steps:**
1. Get basic stock quote and recent price performance
2. Retrieve fundamental data (financial statements, ratios, key metrics)
3. Gather recent news and analyst estimates
4. Compare valuation metrics with industry peers
5. Summarize findings with investment recommendation

**Parameters:**
- `symbol` (str, required) - Stock ticker symbol to analyze
- `analysis_period` (str, default: "last 12 months") - Time period for analysis
- `focus_areas` (str, default: "growth, profitability, valuation") - Specific analysis areas
- `risk_tolerance` (str, default: "moderate") - Risk tolerance level

**Tags:** equity, analysis, comprehensive

**Prompt:**
```json
{
  "name": "equity_analysis",
  "description": "Perform a comprehensive equity analysis using multiple data sources and metrics",
  "content": "Conduct a comprehensive analysis of {symbol} for {analysis_period}. Follow this workflow:\n1. First, get basic stock quote and recent price performance using equity_price_performance.\n2. Retrieve fundamental data including financial statements, ratios, and key metrics using [equity_fundamental_ratios, equity_fundamental_metrics, equity_fundamental_balance].\n3. Gather recent news and analyst estimates for the company using [news_company, equity_estimates_price_target].\n4. Compare valuation metrics with industry peers using equity_compare_peers.\n5. Summarize findings with investment recommendation.\n\nFocus areas: {focus_areas}\nRisk tolerance: {risk_tolerance}",
  "arguments": [
    {
      "name": "symbol",
      "type": "str",
      "description": "Stock ticker symbol to analyze (e.g., AAPL, TSLA)"
    },
    {
      "name": "analysis_period",
      "type": "str",
      "default": "last 12 months",
      "description": "Time period for the analysis"
    },
    {
      "name": "focus_areas",
      "type": "str",
      "default": "growth, profitability, valuation",
      "description": "Specific areas to focus on in the analysis"
    },
    {
      "name": "risk_tolerance",
      "type": "str",
      "default": "moderate",
      "description": "Risk tolerance level: conservative, moderate, or aggressive"
    }
  ],
  "tags": ["equity", "analysis", "comprehensive"]
}
```

**Usage Example:**
```json
{
  "prompt_name": "equity_analysis",
  "arguments": {
    "symbol": "AAPL",
    "analysis_period": "last 24 months",
    "focus_areas": "growth, innovation, market share",
    "risk_tolerance": "aggressive"
  }
}
```

### 2. GDP Summary Prompt (Inline Route Prompt)

**Location:** Inline route prompt example (`/economy/gdp` endpoint)

**Purpose:** Generates a concise summary of GDP data for a specific country over a specified time period. This is an example of an inline prompt attached to a specific API endpoint.

**Associated Tools:** `economy_gdp`

**Parameters:**
- `country` (str, required via endpoint) - Country name
- `years` (int, default: 5) - Number of years to summarize

**Tags:** economy, gdp, summary

**Prompt:**
```json
{
  "name": "gdp_summary_prompt",
  "description": "Generate a brief summary of GDP for a country.",
  "content": "Provide a concise summary of the GDP for {country} over the last {years} years.",
  "arguments": [
    {
      "name": "years",
      "type": "int",
      "default": 5,
      "description": "Number of years to summarize."
    }
  ],
  "tags": ["economy", "gdp", "summary"]
}
```

**Implementation Context:**
```python
@app.get(
    "/economy/gdp",
    openapi_extra={
        "mcp_config": {
            "prompts": [{
                "name": "gdp_summary_prompt",
                "description": "Generate a brief summary of GDP for a country.",
                "content": "Provide a concise summary of the GDP for {country} over the last {years} years.",
                "arguments": [
                    {
                        "name": "years",
                        "type": "int",
                        "default": 5,
                        "description": "Number of years to summarize."
                    }
                ],
                "tags": ["economy", "gdp", "summary"]
            }]
        }
    }
)
def get_gdp_data(country: str, period: Literal["annual", "quarterly"] = "annual"):
    """Get GDP data for a specific country."""
    return {"country": country, "period": period}
```

**Execution Example:**
```json
{
  "prompt_name": "gdp_summary_prompt",
  "arguments": {
    "years": 10,
    "country": "Japan"
  }
}
```

**Rendered Output:**
```json
{
  "description": "Generate a brief summary of GDP for a country.",
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Use the tool, economy_gdp, to perform the following task.\n\nProvide a concise summary of the GDP for Japan over the last 10 years."
      }
    }
  ]
}
```

### 3. GDP Comparison Prompt (Inline Route Prompt)

**Location:** Inline route prompt example (`/economy/gdp` endpoint)

**Purpose:** Compares GDP growth between two countries, facilitating comparative economic analysis.

**Associated Tools:** `economy_gdp`

**Parameters:**
- `country1` (str, required) - First country for comparison
- `country2` (str, required) - Second country for comparison

**Tags:** economy, gdp, comparison

**Prompt:**
```json
{
  "name": "gdp_comparison_prompt",
  "description": "Compare the GDP of two countries.",
  "content": "Compare the GDP growth of {country1} and {country2}.",
  "arguments": [
    {
      "name": "country1",
      "type": "str",
      "description": "First country for comparison."
    },
    {
      "name": "country2",
      "type": "str",
      "description": "Second country for comparison."
    }
  ],
  "tags": ["economy", "gdp", "comparison"]
}
```

**Implementation Context:**
```python
@app.get(
    "/economy/gdp",
    openapi_extra={
        "mcp_config": {
            "prompts": [
                # ... gdp_summary_prompt ...
                {
                    "name": "gdp_comparison_prompt",
                    "description": "Compare the GDP of two countries.",
                    "content": "Compare the GDP growth of {country1} and {country2}.",
                    "arguments": [
                        {
                            "name": "country1",
                            "type": "str",
                            "description": "First country for comparison."
                        },
                        {
                            "name": "country2",
                            "type": "str",
                            "description": "Second country for comparison."
                        }
                    ],
                    "tags": ["economy", "gdp", "comparison"]
                }
            ]
        }
    }
)
```

### 4. Prompt Template Rendering System

**Location:** `openbb_mcp_server/models/prompts.py`

**Purpose:** The `StaticPrompt` class provides the underlying mechanism for rendering prompt templates with user-provided arguments. It validates required arguments and formats the template content.

**Key Features:**
- Argument validation (required vs optional)
- Template variable substitution using Python's `.format()` method
- Error handling for missing or invalid arguments
- Returns MCP-compatible `PromptMessage` structures

**Implementation:**
```python
class StaticPrompt(Prompt):
    """A prompt that is a static string template."""

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

**Usage in MCP Server:**
When an AI agent calls `execute_prompt`, the server:
1. Looks up the prompt by name
2. Merges user-provided arguments with default values
3. Validates required arguments are present
4. Renders the template with the StaticPrompt.render() method
5. Returns the formatted message to the agent

---
