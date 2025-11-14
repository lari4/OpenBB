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
