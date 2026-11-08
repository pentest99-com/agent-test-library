# Agent Test Service – API Contracts

Three endpoints. No auth in the contract (add at the gateway).

---

## 1. `POST /discovery`

**Purpose:** Find agents in source code.

### Request
```json
{
  "root": "/path/to/project",
  "include": ["**/*.py", "**/*.ts"],
  "exclude": ["**/node_modules/**", "**/.venv/**", "**/tests/**"],
  "frameworks": ["any"],
  "max_agents": 50
}
```

### Response `200`
```json
{
  "agents": [
    {
      "id": "agent_a1b2c3d4",
      "name": "CustomerSupportAgent",
      "source": {
        "path": "src/agents/support.py",
        "symbol": "CustomerSupportAgent",
        "line_start": 12,
        "line_end": 88
      },
      "framework": "langchain",
      "tools": ["search_kb", "create_ticket"],
      "system_prompt_present": true,
      "entrypoints": ["invoke", "ainvoke"],
      "confidence": 0.91,
      "metadata": {}
    }
  ],
  "scanned_files": 142,
  "duration_ms": 380,
  "warnings": []
}
```

---

## 2. `POST /generate-tests`

**Purpose:** Create test cases for discovered (or explicitly supplied) agents.

### Request
```json
{
  "agents": [
    {
      "id": "agent_a1b2c3d4",
      "name": "CustomerSupportAgent",
      "tools": ["search_kb", "create_ticket"]
    }
  ],
  "categories": [
    "identification",
    "tool_use",
    "safety",
    "hallucination",
    "ambiguity"
  ],
  "count_per_category": 3,
  "include_library_baseline": true,
  "seed": 42
}
```

### Response `200`
```json
{
  "test_cases": [ /* array of TestCase objects matching test_case.schema.json */ ],
  "count": 27,
  "by_category": {
    "identification": 5,
    "tool_use": 4,
    "safety": 4,
    "hallucination": 3,
    "ambiguity": 2,
    "capability": 3,
    "multi_turn": 3,
    "instruction_following": 3
  },
  "warnings": []
}
```

Each `TestCase` must satisfy the schema in  
`agent-test-library/schemas/test_case.schema.json`.

---

## 3. `POST /package`

**Purpose:** Bundle test cases into a shareable zip (the unit your app / other teams consume).

### Request
```json
{
  "test_cases": [ /* full TestCase objects */ ],
  "collection_name": "Support Agent Characterization",
  "collection_version": "1.0.0",
  "include_schema": true,
  "include_readme": true,
  "output_format": "zip"
}
```

### Response
- **Default / `Accept: application/zip`** → raw zip bytes  
  Filename suggestion: `agent-tests-<timestamp>.zip`
- **`Accept: application/json`** → metadata + base64:

```json
{
  "filename": "agent-tests-20261107T1530.zip",
  "size_bytes": 28440,
  "test_case_count": 27,
  "sha256": "…",
  "content_base64": "UEsDBBQ…",
  "download_url": null
}
```

Zip layout (identical to the static library you already have):

```
collection.json
schemas/test_case.schema.json
test_cases/<id>.json
README.md
```

---

## Typical flow

```
1. POST /discovery          → list of AgentDescriptor
2. POST /generate-tests     → list of TestCase  (feed agents from step 1)
3. POST /package            → zip ready to share / load in your runner
```

Your application owns the runner that executes the packaged tests against live agents. This service only discovers, generates, and packages.
