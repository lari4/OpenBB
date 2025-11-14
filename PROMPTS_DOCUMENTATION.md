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

## LangChain Integration Prompts

The LangChain integration demonstrates how OpenBB can be used as a tool provider for LangChain agents, enabling chain-of-thought reasoning and multi-step financial analysis workflows.

### Architecture Overview

**Location:** `examples/openbb_vs_langchain.ipynb`

**AI Model:** OpenAI GPT-4.1

**Components:**
- **LLM:** ChatOpenAI with temperature=0 for deterministic responses
- **Tools:** Custom OpenBB-powered tools decorated with `@tool`
- **Memory:** ConversationTokenBufferMemory (16,000 token limit)
- **Agent:** OpenAI Tools Agent with conversation history
- **Executor:** AgentExecutor for managing tool calls and reasoning

### 1. System Prompt for Stock Financial Researcher

**Location:** `examples/openbb_vs_langchain.ipynb` (ChatPromptTemplate)

**Purpose:** Establishes the AI agent as a stock financial researcher with access to specific OpenBB tools. Guides the agent on how to use tools efficiently and when to call them.

**Role:** System-level instruction

**Key Directives:**
- Take user questions and answer using available tools
- Use returned data to formulate answers
- Call each function only once
- Don't call functions if information is already available

**Prompt:**
```python
"""You are very powerful stock financial researcher.
You will take the user questions and answer using the tools available.
Once you have the information you need, you will answer user's questions using the data returned.
Use the following tools to answer user queries:
- get_strong_buy_for_sector to find strong buy recommendations for a sector
- get_strong_buy_for_industry to find strong buy recommendations for an industry
- get_industry_performance to find the performance for an industry
- get_valuation_for_industries to find valuation metrics for industries
- get_candidate_stocks_to_invest_relaxed to fetch all companies using relaxed criteria
- get_consensus(ticker:str) - to find analyst consensus for a company
You should call each function only once, and you should not call the function if you already have the information you need."""
```

**Implementation Context:**
```python
MEMORY_KEY = "chat_history"
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are very powerful stock financial researcher..."""),
    MessagesPlaceholder(variable_name=MEMORY_KEY),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])
```

### 2. Available Custom Tools

The LangChain agent has access to the following OpenBB-powered tools:

#### Tool: get_industry_performance
**Purpose:** Return performance metrics by industry across multiple timeframes
**Returns:** Performance data for last week, month, quarter, half year, and year
**Implementation:**
```python
@tool
def get_industry_performance() -> list:
    """Return performance by industry for last week, last month, last quarter,
    last half year and last year"""
    return obb.equity.compare.groups(group='industry', metric='performance').to_llm()
```

#### Tool: get_strong_buy_for_sector
**Purpose:** Find stocks with strong buy recommendations in a specific sector
**Parameters:** `sector` (str) - Sector name
**Implementation:**
```python
@tool
def get_strong_buy_for_sector(sector: str) -> list:
    """Return the strong buy recommendation for a given sector"""
    new_sector = '_'.join(sector.lower().split()).lower()
    data = obb.equity.screener(provider='finviz', sector=new_sector,
                                recommendation='buy')
    return data.to_llm()
```

#### Tool: get_strong_buy_for_industry
**Purpose:** Find stocks with strong buy recommendations in a specific industry
**Parameters:** `industry` (str) - Industry name
**Implementation:**
```python
@tool
def get_strong_buy_for_industry(industry: str) -> list:
    """Return the strong buy recommendation for a given industry"""
    data = obb.equity.screener(provider='finviz', industry=industry,
                                recommendation='buy')
    return data.to_llm()
```

#### Tool: get_valuation_for_industries
**Purpose:** Get valuation metrics (P/E, P/B, EV/EBITDA) for an industry
**Parameters:** `input` (str) - Industry name
**Returns:** JSON with valuation metrics
**Implementation:**
```python
@tool
def get_valuation_for_industries(input: str) -> list:
    """Return valuation metrics for the industry provided as input"""
    data = obb.equity.compare.groups(group='industry', metric='valuation',
                                     provider='finviz').to_df()
    filtered = data[data.name == input]
    return filtered.to_json(orient="records", date_format="iso", date_unit="s")
```

#### Tool: get_candidate_stocks_to_invest_relaxed
**Purpose:** Screen for investment candidates using relaxed criteria
**Parameters:** `industry` (str) - Industry name
**Screening Criteria:**
- Market Cap: Over $300M
- Average Volume: Over 200K
- Institutional Ownership: Under 60%
- Current Ratio: Over 1.5
- Debt/Equity: Over 0.3

**Implementation:**
```python
@tool
def get_candidate_stocks_to_invest_relaxed(industry: str) -> list:
    '''Use relaxed criteria to find best companies in an industry
    which are worth investing into'''
    desc_filters = {
        'Market Cap.': '+Small (over $300mln)',
        'Average Volume': 'Over 200K',
    }
    fund_filters = {
        'InstitutionalOwnership': 'Under 60%',
        'Current Ratio': 'Over 1.5',
        'Debt/Equity': 'Over 0.3',
    }
    desc_filters.update(fund_filters)

    try:
        data = obb.equity.screener(provider='finviz', industry=industry,
                                   filters_dict=desc_filters)
        return data.to_llm()
    except Exception as e:
        logging.info(f'No data found:{str(e)}')
        return []
```

#### Tool: get_consensus
**Purpose:** Get analyst consensus and price targets for a stock
**Parameters:** `ticker` (str) - Stock ticker symbol
**Returns:** JSON with target_high, target_low, target_consensus, target_median
**Implementation:**
```python
@tool
def get_consensus(ticker: str) -> list:
    """Return analyst consensus for the ticker provided
    It returns the following fields:
    - target_high: float, High target of the price target consensus.
    - target_low: float Low target of the price target consensus.
    - target_consensus: float Consensus target of the price target consensus.
    - target_median: float Median target of the price target consensus
    """
    data = obb.equity.estimates.consensus(symbol=ticker, limit=3,
                                          provider='yfinance').to_df()
    return data.to_json(orient="records", date_format="iso", date_unit="s")
```

### 3. Chain-of-Thought Workflow Example

**Purpose:** Multi-step industry and stock analysis using sequential reasoning

**User Prompt:**
```python
input1 = '''
First, find an industry that has consistently shown positive performance across
quarterly, monthly, and weekly timeframes.
Second, once you have identified the industry, extract its relevant valuation
metrics (e.g., P/E, P/B, EV/EBITDA).
Third, extract companies from the selected industry using relaxed criteria.
Fourth, for the best performing companies get the analyst consensus
Finally, summarize your findings in no more than 80 words detailing:
- Best performing industry
- Best performing companies in industry
- A table displaying the analyst consensus for each of the companies you
  found at previous step
'''
```

**Workflow Steps:**
1. Call `get_industry_performance()` - Find consistently positive industry
2. Call `get_valuation_for_industries(industry)` - Extract valuation metrics
3. Call `get_candidate_stocks_to_invest_relaxed(industry)` - Screen companies
4. Call `get_consensus(ticker)` for each top company - Get analyst targets
5. Synthesize and summarize findings in structured format

**Agent Execution:**
```python
result = agent_executor.invoke({"input": input1, "chat_history": chat_history})
print(result['output'])
```

### 4. Sector Analysis Workflow Example

**Purpose:** Find and analyze strong buy recommendations in a specific sector

**User Prompt:**
```python
input1 = '''
First, find the stocks recommended for strong buy in the Utilities Sector
Second, find the valuation metrics for this stock.
Third, summarize your findings in a short paragraph.
'''
```

**Workflow Steps:**
1. Call `get_strong_buy_for_sector("Utilities")` - Get strong buy stocks
2. Call `get_valuation_for_industries("Utilities")` - Get sector valuation
3. Synthesize data into paragraph summary

**Agent Execution:**
```python
result = agent_executor.invoke({"input": input1, "chat_history": chat_history})
print(result['output'])
```

### 5. Conversation Memory Configuration

**Purpose:** Maintain conversation context across multiple interactions

**Configuration:**
```python
from langchain.memory import ConversationTokenBufferMemory

MEMORY_KEY = "chat_history"
memory = ConversationTokenBufferMemory(
    llm=llm,                    # Required for token counting
    max_token_limit=16000,      # Leave buffer for functions + responses
    memory_key="chat_history",  # Must match prompt's key
    return_messages=True
)
```

**Benefits:**
- Tracks conversation history
- Prevents context overflow (16K token limit)
- Enables multi-turn conversations
- Allows agent to reference previous findings

---

## Summary

OpenBB provides three main categories of AI prompts:

1. **Development & Operations**
   - Changelog summarization for release automation

2. **MCP Server Integration** (Primary AI Interface)
   - System prompts for agent configuration
   - Server prompts for pre-built workflows
   - Inline prompts for endpoint-specific guidance
   - Template rendering system for dynamic content

3. **LangChain Integration** (Example Implementation)
   - System prompt for researcher persona
   - Custom tool definitions with OpenBB data
   - Chain-of-thought workflow examples
   - Conversation memory management

All prompts are designed to guide AI agents through complex financial analysis tasks while maintaining efficiency, accuracy, and proper tool usage patterns.
