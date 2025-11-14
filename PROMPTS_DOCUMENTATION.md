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
