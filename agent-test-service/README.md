# Agent Test Service

Implements the three operations you asked for:

| Operation | HTTP (contract) | CLI |
|-----------|-----------------|-----|
| Search for agents in code | `POST /discovery` | `agenttestctl.py discovery` |
| Create test cases | `POST /generate-tests` | `agenttestctl.py generate-tests` |
| Package tests for sharing | `POST /package` | `agenttestctl.py package` |

Also: `agenttestctl.py all` runs the full pipeline in one shot.

## Layout

```
agent-test-service/
├── api/
│   ├── openapi.yaml      ← full OpenAPI 3.1 contract
│   └── CONTRACTS.md      ← human-readable request/response examples
├── cli/
│   └── agenttestctl.py   ← working local implementation
└── README.md
```

The CLI reuses the static library at `../agent-test-library` (schema + baseline cases).

## Quick start (CLI)

```bash
# 1. Discover
python cli/agenttestctl.py discovery --root /path/to/your/project --out agents.json

# 2. Generate tests
python cli/agenttestctl.py generate-tests --agents agents.json --out ./generated-tests

# 3. Package
python cli/agenttestctl.py package --tests ./generated-tests --out agent-tests.zip

# Or everything:
python cli/agenttestctl.py all --root /path/to/your/project --out agent-tests.zip
```

## What the CLI actually does

**discovery**
- Walks source files (`*.py`, `*.ts`, `*.js`, …)
- Finds classes/functions whose names look like agents
- Detects framework hints (LangChain, CrewAI, OpenAI Agents, AutoGen, LlamaIndex)
- Extracts rough tool lists and whether a system prompt is present
- Returns `AgentDescriptor[]` matching the API contract

**generate-tests**
- Builds `TestCase` objects (same schema as the static library)
- Categories: identification, tool_use, safety, hallucination, ambiguity, instruction_following, multi_turn, capability, robustness, edge_case
- Optionally merges a baseline set from `agent-test-library/test_cases`

**package**
- Writes a zip with:
  - `collection.json`
  - `schemas/test_case.schema.json`
  - `test_cases/<id>.json`
  - `README.md`
- Same layout as the hand-built library zip, so your consumer app can load either

## HTTP layer

The OpenAPI file is the contract your application (or a thin FastAPI/Flask wrapper) should implement. The CLI is the reference behaviour for the same three operations.

Your runner stays outside this service: it loads the packaged zip and executes tests against live agents.
