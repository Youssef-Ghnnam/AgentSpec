# Agent-of-Agents Scaffold v0.1

A minimal, production-ready Python framework for defining and executing AI agents with validation, quality assurance, and mock tools.

## Features

- **JSON-based configuration** - User-friendly agent intake forms
- **Comprehensive validation** - Clear, actionable error messages
- **Bounded execution loops** - Safe, controlled agent workflows
- **QA metrics** - Confidence scoring, consistency checks, rule compliance
- **Mock tools** - Search, fetch, summarize, write (ready to replace with real implementations)
- **Zero dependencies** - Pure Python 3.8+ standard library only
- **Structured logging** - Detailed execution traces

## Installation

No installation required! This is a single-file, stdlib-only Python script.

**Requirements:**
- Python 3.8 or higher
- No external dependencies

## Quick Start

### 1. Run with defaults (auto-generates example files)

```bash
python agent_scaffold.py
```

This creates:
- `agent_form.json` - Example agent configuration
- `run_input.json` - Example runtime parameters
- `out/` directory with execution results

### 2. Customize your agent

Edit `agent_form.json`:

```json
{
  "Name": "researcher",
  "Goal": "Find reliable sources on topic X and produce bullet points",
  "Focus": "reliability, modern solutions",
  "Display": {
    "Type": "bullet-points",
    "Add-ons": "include_sources, show_confidence"
  },
  "Tools": "search, fetch, summarize, write",
  "Complexity": "medium",
  "Autonomy": "medium"
}
```

### 3. Run your agent

```bash
python agent_scaffold.py --form agent_form.json --input run_input.json --out results
```

## Configuration Reference

### Agent Form Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | ✓ | Agent identifier (2-40 chars, alphanumeric + `-_`) |
| `Goal` | string | ✓ | Agent objective (1-240 chars, must contain action verb) |
| `Focus` | string | | Comma-separated focus areas/tags |
| `Display.Type` | string | | Output format: `bullet-points`, `summary`, `report` |
| `Display.Add-ons` | string | | CSV: `include_sources`, `show_confidence`, `group_by_theme` |
| `Inputs` | string | | Expected input parameters (CSV) |
| `Tools` | string | | Enabled tools (CSV from catalog) |
| `Specifics.Steps` | string | | Workflow steps (arrow or comma-separated) |
| `Specifics.Domains.whitelist` | string | | Allowed domains (CSV, supports `*.edu` wildcards) |
| `Specifics.Domains.blacklist` | string | | Blocked domains (CSV) |
| `Memory` | string | | `short`, `long`, `both`, `none` |
| `Complexity` | string | | `low` (4 steps), `medium` (8), `high` (12) |
| `Autonomy` | string | | `low` (approve plan+final), `medium` (approve final), `high` (none) |
| `Notes` | string | | Additional context (can include timeframe hints) |

### Tool Catalog

- `search` - Web search simulation
- `fetch` - Content retrieval
- `summarize` - Text summarization/clustering
- `write` - Markdown report generation
- `classify` - Content classification (stub)
- `extract` - Information extraction (stub)
- `browser` - Browser automation (stub)
- `file_read` - File reading (stub)
- `file_write` - File writing (stub)
- `codegen` - Code generation (stub)

### Run Input Parameters

```json
{
  "topic": "AI safety and elections",
  "depth": "brief",
  "timeframe": "24 months"
}
```

Parameters are agent-specific based on the `Inputs` field in your form.

## Output Structure

After execution, the output directory contains:

```
out/
├── brief.md              # Main output (or summary.md, report.md)
├── compiled_spec.json    # Normalized agent specification
├── qa.json              # Quality assurance metrics
├── bullets.json         # Structured bullet points
├── sources.json         # Source citations
└── logs.txt             # Execution trace
```

### QA Metrics

```json
{
  "p_action_success": 0.875,
  "self_consistency": 0.742,
  "rule_checks": 0.850,
  "chain_accuracy": 0.822
}
```

- **p_action_success**: Estimated probability that tool actions succeeded
- **self_consistency**: Coherence between different outputs (Jaccard similarity)
- **rule_checks**: Compliance with domain whitelist/blacklist constraints
- **chain_accuracy**: Weighted combination of above metrics (0.4/0.3/0.3)

## Command Line Options

```bash
python agent_scaffold.py [OPTIONS]

Options:
  --form FORM       Path to agent form JSON (default: agent_form.json)
  --input INPUT     Path to run input JSON (default: run_input.json)
  --out OUT         Output directory (default: out)
  --verbose         Enable verbose logging
  --version         Show version and exit
  --help            Show help message
```

## Validation Rules

The validator enforces:

- ✓ **Name**: 2-40 characters, letters/numbers/dash/underscore only
- ✓ **Goal**: 1-240 characters, must contain action verb (find, create, analyze, etc.)
- ✓ **Display.Type**: One of `bullet-points`, `summary`, `report`
- ✓ **Tools**: Only recognized tools from catalog
- ✓ **Memory**: One of `short`, `long`, `both`, `none`
- ✓ **Complexity**: One of `low`, `medium`, `high`
- ✓ **Autonomy**: One of `low`, `medium`, `high`

Validation errors provide clear, actionable messages:

```
Validation failed with the following errors:
  1. Name must be 2-40 characters, containing only letters, numbers, dashes, or underscores.
  2. Goal should contain an action verb (e.g., analyze, create, find, generate, prepare).
  3. Unknown tool 'webscrape'. Available: browser, classify, codegen, extract, fetch, file_read, file_write, search, summarize, write
```

## Extending with Real Tools

The mock tools are designed to be easily replaceable:

```python
class RealSearchTool(Tool):
    name = "search"
    
    def __call__(self, ctx: ToolContext, query: str = "") -> List[Dict[str, Any]]:
        # Replace with actual search API
        response = requests.get(f"https://api.search.com/q={query}")
        return response.json()["results"]

# In Orchestrator.__init__:
self.tools["search"] = RealSearchTool()
```

Each tool receives:
- `ctx`: ToolContext with spec, run_input, logging, and shared state
- Tool-specific parameters (e.g., `query`, `urls`, `docs`)

And should return tool-specific results (see docstrings for schemas).

## Architecture

```
┌─────────────┐
│ Agent Form  │ (JSON, user-friendly)
└──────┬──────┘
       │ load_form()
       ▼
┌─────────────┐
│  Validator  │ (validates structure & values)
└──────┬──────┘
       │ validate_form()
       ▼
┌─────────────┐
│  Compiler   │ (normalizes to AgentSpec)
└──────┬──────┘
       │ compile()
       ▼
┌─────────────┐
│ AgentSpec   │ (machine-readable)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Orchestrator │ (executes workflow)
└──────┬──────┘
       │
       ├──> ToolContext (shared state)
       ├──> SearchTool
       ├──> FetchTool
       ├──> SummarizeTool
       └──> WriteTool
       │
       ▼
┌─────────────┐
│     QA      │ (computes metrics)
└──────┬──────┘
       │
       ▼
   Artifacts
```

## Error Handling

The scaffold includes comprehensive error handling:

- **Validation errors**: Clear messages about what's wrong and how to fix it
- **File I/O errors**: Logged with specific file and error details
- **Execution errors**: Caught, logged, and included in execution trace
- **Missing tools**: Graceful degradation with warnings

All errors are logged to both console and `logs.txt`.

## Development Roadmap

### v0.1 (Current)
- ✅ Core validation and compilation
- ✅ Mock tool implementations
- ✅ QA metrics framework
- ✅ Structured logging

### Future Versions
- Real tool integrations (web search, LLM APIs)
- Streaming execution with progress updates
- Human-in-the-loop approval gates
- Persistent memory/state management
- Multi-agent coordination
- Plugin system for custom tools
- Web UI for configuration

## Testing

The code includes type hints and docstrings for all public APIs. To test:

```python
# Test validation
from agent_scaffold import AgentForm, Validator

form = AgentForm(name="test", goal="Find information")
is_valid, errors = Validator.validate_form(form)
assert is_valid

# Test compilation
from agent_scaffold import Compiler

spec = Compiler.compile(form)
assert spec.name == "test"
assert len(spec.tools) > 0

# Test execution
from agent_scaffold import Orchestrator
from pathlib import Path

orch = Orchestrator(spec, Path("test_out"))
result = orch.run({"topic": "test"})
assert "qa" in result
assert result["qa"]["chain_accuracy"] > 0.0
```

## License

This is example code provided as-is for educational and prototyping purposes.

## Support

For issues, questions, or contributions, please refer to the project repository.

---

**Version**: 0.1.0  
**Python**: 3.8+  
**Dependencies**: None (stdlib only)
