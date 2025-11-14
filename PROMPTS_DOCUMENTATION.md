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
