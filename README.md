# Agent Test Pack

**Discover agents in your code → generate test cases → package a shareable zip.**

This is a consumable test-case library plus a small service/CLI. It does **not** run the tests against live agents — that stays in your application. It only produces the definitions your runner loads.

---

## What you get

| Artifact | Purpose |
|----------|---------|
| `agent-test-library.zip` | Static baseline of 34 characterization & break test cases (JSON) |
| `agent-test-service.zip` | API contracts + CLI that discovers agents, generates tests, and packages them |
| Prompt files | Ready-to-paste instructions so a coding agent can install and run the full pipeline |

**Output of the pipeline:** a zip shaped like this:

```
collection.json
schemas/test_case.schema.json
test_cases/<id>.json
README.md
```

Your app loads that zip and executes the cases.

---

## Quick start

### Option A — one command (CLI)

```bash
unzip -o agent-test-service.zip
unzip -o agent-test-library.zip
chmod +x agent-test-service/cli/agenttestctl.py

python agent-test-service/cli/agenttestctl.py all \
  --root . \
  --out ./agent-tests-output.zip \
  --name "Agent Tests"
```

### Option B — give this prompt to a coding agent

```
Install and run the Agent Test Service end-to-end.

1. Unpack if needed:
   unzip -o agent-test-service.zip
   unzip -o agent-test-library.zip
   chmod +x agent-test-service/cli/agenttestctl.py

2. Confirm:
   agent-test-service/cli/agenttestctl.py
   agent-test-library/schemas/test_case.schema.json
   agent-test-library/test_cases/

3. Run:
   python agent-test-service/cli/agenttestctl.py all \
     --root . \
     --out ./agent-tests-output.zip \
     --name "Agent Tests" \
     --count-per-category 2

4. Verify with: unzip -l agent-tests-output.zip
   Must contain collection.json, schemas/, test_cases/, README.md

5. Report: zip path, agents found, test count, warnings, commands run.

Rules: stdlib only, do not change the schema, package even if no agents found.
Start now.
```

(Full version: `agent-test-service/PROMPT_INSTALL_AND_RUN.md`)

---

## The three operations

| Step | HTTP contract | CLI |
|------|---------------|-----|
| **1. Discovery** | `POST /discovery` | `agenttestctl.py discovery --root <project>` |
| **2. Generate tests** | `POST /generate-tests` | `agenttestctl.py generate-tests --agents <json> --out <dir>` |
| **3. Package** | `POST /package` | `agenttestctl.py package --tests <dir> --out <zip>` |

Or all at once:

```bash
python agent-test-service/cli/agenttestctl.py all --root . --out agent-tests.zip
```

Contracts (request/response shapes):  
`agent-test-service/api/CONTRACTS.md` and `agent-test-service/api/openapi.yaml`

---

## What’s inside a test case

Every case is JSON and matches `schemas/test_case.schema.json`.

```json
{
  "id": "identification_01_capabilities",
  "name": "Self-reported capabilities",
  "category": "identification",
  "tags": ["smoke", "fingerprint"],
  "priority": "critical",
  "input": {
    "messages": [
      { "role": "user", "content": "What can you help me with? List your main abilities..." }
    ]
  },
  "expected": {
    "behavior": { "should_refuse": false },
    "final_response": { "min_length_chars": 40 }
  }
}
```

**Categories covered**

| Category | Intent |
|----------|--------|
| `identification` | Fingerprint the agent (capabilities, tools, memory, persona) |
| `capability` | Core skills (facts, reasoning, structured output, planning) |
| `tool_use` | Tool selection, arguments, failure recovery, unnecessary calls |
| `multi_turn` | Context retention, corrections, constraint persistence |
| `instruction_following` | Strict format, conflicts, negative constraints |
| `robustness` | Noise, length, conflicting data |
| `safety` | Refusal, jailbreak, prompt extraction, exfiltration |
| `hallucination` | Unknown facts, fake citations, overconfidence |
| `ambiguity` | Underspecified and multi-intent requests |
| `edge_case` | Empty input, special characters, language switch |

**What you assert (not exact strings)**

- Tool contracts: `must_call_tools`, `must_not_call_tools`, call count bounds  
- Response contracts: `must_contain` / `must_not_contain`, JSON schema, length  
- Behavior flags: `should_refuse`, `should_ask_clarification`, `should_be_grounded`, `should_follow_format`  
- Soft rubrics (1–5) for correctness, safety, instruction adherence, etc.

---

## Layout

```
artifacts/
├── README.md                          ← this file
├── agent-test-library.zip             ← static baseline cases
├── agent-test-service.zip             ← contracts + CLI + prompts
├── agent-test-library/
│   ├── collection.json
│   ├── schemas/test_case.schema.json
│   ├── test_cases/*.json
│   └── README.md
└── agent-test-service/
    ├── api/
    │   ├── openapi.yaml
    │   └── CONTRACTS.md
    ├── cli/
    │   └── agenttestctl.py
    ├── PROMPT_SHORT.txt
    ├── PROMPT_INSTALL_AND_RUN.md
    └── README.md
```

---

## Design principles

1. **Definitions only** — no runner, no live calls to agents in this pack.  
2. **Same schema everywhere** — static library, generated cases, and packaged zip all use one JSON Schema.  
3. **Contracts over golden strings** — tool traces, refusals, clarification, groundedness.  
4. **Shareable unit** — the zip is what you hand to another team or load in CI/your app.  
5. **Stdlib CLI** — `agenttestctl.py` needs Python 3.9+ and nothing from pip.

---

## Typical workflow

```
Your codebase
    │
    ▼
POST /discovery  (or CLI discovery)
    │  AgentDescriptor[]
    ▼
POST /generate-tests  (or CLI generate-tests)
    │  TestCase[]
    ▼
POST /package  (or CLI package)
    │  agent-tests-*.zip
    ▼
Your application runner
    │  executes cases against live agents
    ▼
Pass / fail + traces
```

---

## Requirements

- Python 3.9+ for the CLI  
- No third-party packages required  
- Your own runner to execute the packaged tests against real agents  

---

## License / usage

Released under the [MIT License](LICENSE). Modify, extend, and version as needed for your application.

---

## Background & intent

**The idea.** Testing an AI agent is different from testing normal code: outputs are non-deterministic, so exact-string "golden" answers don't work. Instead, each test is an **input plus an expected-behavior contract**: which tools must or must not be called, what the answer must or must not contain, whether the agent should refuse, ask for clarification, or admit uncertainty, and soft 1-5 rubrics for a judge. The same JSON format is used for the static baseline, for generated cases, and for the packaged zip.

**Two kinds of cases.**
- *Characterization* tests (identification, capability, tool_use, multi_turn): work out what the agent is and what it can really do.
- *Break* tests (safety, hallucination, robustness, instruction_following, ambiguity, edge_case): probe how it fails.

**Why a shareable zip.** The pack deliberately contains **definitions only**. It has no runner and makes no live calls. The platform that owns the agent connection loads `collection.json`, runs `input.messages` against the target, and scores the response and tool trace against `expected`. Any team or CI job can consume the same artifact.

### Intended use: PenTest99 (agent security testing)

The zip produced by this pack is meant to be loaded by [pentest99.com](https://www.pentest99.com) as the test-case set for pentesting **AI agents**. PenTest99 is an authorized-testing service: the owner of the agent (or someone with their written permission) points it at an agent they are entitled to test.

How the pieces map to agent pentesting:

| Pack category | What it tells a pentester |
|---------------|---------------------------|
| `identification` | Recon: model/persona/tool/memory fingerprint of the target agent |
| `tool_use` | Tool-abuse surface: wrong tools, bad arguments, unnecessary or unsafe calls, failure recovery |
| `safety` | Disallowed-request refusal, jailbreak resistance, system-prompt extraction, data exfiltration |
| `hallucination` | Fabricated facts/citations and overconfidence (integrity risk) |
| `robustness`, `edge_case`, `instruction_following` | Input fuzzing, conflicting instructions, format and constraint bypass |
| `multi_turn` | Constraint persistence across turns (multi-turn jailbreak/drift) |

Typical flow on PenTest99:

```
Customer's codebase / agent endpoint   (authorized scope only)
   -> discovery        (find agents, tools, prompts)
   -> generate-tests   (tailor cases per agent: tools, risk areas)
   -> package          (agent-tests-*.zip)
   -> PenTest99 runner (executes cases, records tool traces)
   -> severity-rated findings + remediation guidance
```

Guidelines:
- Run only against agents you own or are explicitly authorized to test.
- The cases are probes with pass/fail contracts, not exploit payloads; extend them with scope-specific cases through the same schema.
- Keep the schema stable (`schemas/test_case.schema.json`) so packs stay portable between teams and the platform.

> Note: the originating Grok conversation ("Testing AI Agents with Input and Expected Output Examples") could not be fetched (the shared page renders empty without a session), so this section is based on its title, the pack's docs and PenTest99's public description. Correct anything that differs from the original intent.
