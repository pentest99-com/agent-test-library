# Prompt: Install Agent Test Service → Discover → Generate → Package

Copy everything below the line and paste it into your coding agent (Cursor, Claude Code, Codex, etc.) or follow it manually in a terminal.

---

## Prompt (copy from here)

```
You are setting up and running the Agent Test Service on this machine.

GOAL
1. Install / place the agent-test-service and agent-test-library
2. Run discovery on the current project (or a path I specify)
3. Generate test cases for any agents found
4. Package everything into a shareable zip

CONTEXT
- agent-test-service.zip and agent-test-library.zip are the deliverables.
- The CLI is: agent-test-service/cli/agenttestctl.py
- It needs Python 3.9+ (stdlib only — no pip packages required).
- Output must be a zip the same shape as the static library:
    collection.json
    schemas/test_case.schema.json
    test_cases/*.json
    README.md

STEPS — execute in order

1. LOCATE OR UNPACK
   - If agent-test-service/ and agent-test-library/ already exist in the workspace, use them.
   - Otherwise unpack:
       unzip -o agent-test-service.zip
       unzip -o agent-test-library.zip
   - Confirm these paths exist:
       agent-test-service/cli/agenttestctl.py
       agent-test-library/schemas/test_case.schema.json
       agent-test-library/test_cases/

2. MAKE CLI EXECUTABLE
   chmod +x agent-test-service/cli/agenttestctl.py

3. CHOOSE PROJECT ROOT TO SCAN
   - Default: the current workspace root (.)
   - If I gave you a different path, use that instead.
   - Call this path PROJECT_ROOT.

4. RUN FULL PIPELINE
   python agent-test-service/cli/agenttestctl.py all \
     --root PROJECT_ROOT \
     --out ./agent-tests-output.zip \
     --name "Agent Tests – $(basename PROJECT_ROOT)" \
     --count-per-category 2

   If that fails, run the three steps separately:

   a) Discovery
      python agent-test-service/cli/agenttestctl.py discovery \
        --root PROJECT_ROOT \
        --out ./agents-discovered.json

   b) Generate tests
      python agent-test-service/cli/agenttestctl.py generate-tests \
        --agents ./agents-discovered.json \
        --out ./generated-tests \
        --count-per-category 2

   c) Package
      python agent-test-service/cli/agenttestctl.py package \
        --tests ./generated-tests \
        --out ./agent-tests-output.zip \
        --name "Agent Tests – $(basename PROJECT_ROOT)"

5. VERIFY
   - Confirm agent-tests-output.zip exists.
   - Run: unzip -l agent-tests-output.zip
   - Expect at least:
       collection.json
       schemas/test_case.schema.json
       test_cases/…json files
       README.md
   - Print a short summary:
       • how many agents were discovered
       • how many test cases were generated
       • zip path and size

6. REPORT
   Reply with:
   - Path to the final zip
   - Number of agents found
   - Number of test cases inside the zip
   - Any warnings from discovery or generation
   - The exact commands you ran

CONSTRAINTS
- Do not modify the agent-test-library schema.
- Do not install extra Python packages unless strictly required (the CLI is stdlib-only).
- If no agents are found, the CLI still generates a default placeholder set — that is OK; still package it.
- Prefer the `all` subcommand when possible.

START NOW.
```

---

## One-liner (manual terminal)

If you already have the zips in the current directory:

```bash
unzip -o agent-test-service.zip && unzip -o agent-test-library.zip && \
chmod +x agent-test-service/cli/agenttestctl.py && \
python agent-test-service/cli/agenttestctl.py all \
  --root . \
  --out ./agent-tests-output.zip \
  --name "Agent Tests"
```

Then check:

```bash
unzip -l agent-tests-output.zip | head -30
```
