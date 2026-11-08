# Agent Characterization & Break Library

Version: 1.0.0

## Purpose

This package contains **only test-case definitions** in a strict JSON format.
It is designed to be loaded by your own application/runner so you can:

1. Identify / fingerprint what kind of agent you are testing
2. Map its real capabilities
3. Probe failure modes (robustness, safety, hallucination, instruction following, etc.)

No runner code, no framework, no external service dependencies.

## Structure

```
collection.json          ← master index of all test cases
schemas/
  test_case.schema.json  ← JSON Schema for every test case
test_cases/              ← one JSON file per test case
README.md                ← this file
```

## How to consume

1. Read `collection.json`
2. For each entry in `test_cases`, load the corresponding JSON file
3. Validate against `schemas/test_case.schema.json` (optional but recommended)
4. Execute the `input.messages` (and optional tools/metadata) against the target agent
5. Evaluate the agent response + tool trace against the `expected` contracts

## Categories

- `identification`  – fingerprint the agent (capabilities, tools, memory, persona)
- `capability`      – core skills (facts, reasoning, structured output, planning)
- `tool_use`        – tool selection, arguments, failure recovery, unnecessary calls
- `multi_turn`      – context retention, corrections, constraint persistence
- `instruction_following` – format, conflicts, negative constraints
- `robustness`      – noise, length, conflicting data
- `safety`          – refusal, jailbreak, prompt extraction, exfiltration
- `hallucination`   – unknown facts, fake citations, overconfidence
- `ambiguity`       – underspecified and multi-intent requests
- `edge_case`       – empty input, special characters, language switch

## Expected contract fields (summary)

| Field | Meaning |
|-------|---------|
| `must_call_tools` / `must_not_call_tools` | Tool trace assertions |
| `tool_call_count` | min/max number of tool calls |
| `final_response.must_contain` / `must_not_contain` | String checks on final answer |
| `final_response.json_schema` | Structured output validation |
| `behavior.should_refuse` | Expect a refusal |
| `behavior.should_ask_clarification` | Expect a clarifying question |
| `behavior.should_be_grounded` | Prefer uncertainty over invention |
| `behavior.should_follow_format` | Strict format compliance |
| `rubric.*` | Soft 1-5 scores (judge or human) |

## Notes for implementers

- Tool names in the test cases are illustrative. Map them to the real tool names of the agent under test.
- For multi-turn cases the conversation history is fully specified so the runner can replay it exactly.
- Safety and red-team cases are intentionally mild-to-moderate. Extend them as needed for your threat model.
- Empty or extremely long inputs can be expanded at runtime; the files show the structure.

## License / usage

Released under the [MIT License](LICENSE). Modify, extend, and version as needed for your application.
