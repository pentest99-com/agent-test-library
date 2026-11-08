# Run the Agent Test Pack on your repo

Paste the prompt below into your coding agent (Claude Code, Cursor, Codex, etc.) from the root of the repo you want to test. It downloads this pack, scans your repo for agents, generates test cases, and writes `agent-tests-output.zip`.

```
Download the Agent Test Pack and run it end-to-end on THIS repository to produce a shareable zip of agent test cases.

Do these steps now, without asking me questions:

1. Fetch the pack into a temp directory OUTSIDE this repo (so it isn't scanned or committed):
     PACK="$(mktemp -d)/agent-test-library"
     git clone --depth 1 https://github.com/pentest99-com/agent-test-library "$PACK"
   (If git is unavailable: download https://github.com/pentest99-com/agent-test-library/archive/refs/heads/main.zip and unzip it.)

2. Confirm these exist in $PACK:
   - agent-test-service/cli/agenttestctl.py
   - agent-test-library/schemas/test_case.schema.json
   - agent-test-library/test_cases/

3. From the root of THIS repo, run the full pipeline:
     python3 "$PACK/agent-test-service/cli/agenttestctl.py" all \
       --root . \
       --out ./agent-tests-output.zip \
       --name "Agent Tests" \
       --count-per-category 2
   If `all` fails, run the steps separately:
     discovery      --root . --out agents-discovered.json
     generate-tests --agents agents-discovered.json --out ./generated-tests
     package        --tests ./generated-tests --out ./agent-tests-output.zip

4. Verify: unzip -l agent-tests-output.zip
   Must contain collection.json, schemas/test_case.schema.json, test_cases/*.json, README.md

5. Report: zip path, agents discovered, test case count, warnings, exact commands run.

Rules:
- Python 3.9+ stdlib only; no pip installs.
- Do not modify this repo's source code or the pack's schema. Only add agent-tests-output.zip (and remove any temp files you create).
- Do not commit or push anything.
- If no agents are found, still package the default generated set.
- The pack contains test DEFINITIONS only; do not run them against any live agent.
```

The zip is the shareable unit to load into [PenTest99](https://www.pentest99.com).
