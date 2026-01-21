---
name: run-tests
description: Execute the auto-harness plugin meta-test suite
arguments:
  - name: category
    description: Filter tests by category (e.g., smoke, commands, skills, agents)
    required: false
  - name: tag
    description: Filter tests by tag (e.g., smoke, critical, plugin)
    required: false
  - name: reset
    description: Reset test state and start fresh
    required: false
---

# Run auto-harness Meta-Test Suite

Execute the automated test suite to verify the auto-harness plugin works correctly using its own hook-driven testing framework.

## Initialization

<step name="init">
Run the test runner initialization:

```bash
./tests/functional/runner.sh init {{#if category}}--category {{category}}{{/if}} {{#if tag}}--tag {{tag}}{{/if}} {{#if reset}}--reset{{/if}}
```

Report the initialization result showing:
- Total tests to run
- Any active filters
- Instructions for proceeding
</step>

## Test Execution Mode

After initialization, the session enters **test mode**. The following commands become available:

| Command | Shortcut | Description |
|---------|----------|-------------|
| `next` | `n` | Get and execute the next test |
| `skip` | `s` | Skip the current test |
| `status` | - | Show progress summary |
| `report` | - | Generate test report |
| `vars` | - | Show captured variables |
| `abort` | `a` | Stop test run |

## Execution Flow

1. Type `next` to get the first test
2. Claude executes the test action
3. Copy Claude's response
4. Type `validate <response>` to validate
5. Repeat until all tests complete
6. Type `report` for final summary

## Test Categories

The meta-test suite includes:

- **smoke**: Basic functionality verification
- **commands**: /harness:init, /harness:validate command tests
- **skills**: test-authoring, hook-architecture skill tests
- **agents**: test-generator, setup-validator agent tests
- **templates**: runner.sh, hooks.json, tests.yaml template tests
- **integration**: Full plugin structure validation

## Example Session

```
User: /run-tests --category smoke

Claude: Test suite initialized.
        Total tests: 2 (filtered by category: smoke)
        Type 'next' to begin.

User: next

Claude: ## Test 1/2: smoke_basic
        **Action:** Confirm the test framework is operational

        Test framework operational.

User: validate Test framework operational

Claude: PASS: smoke_basic
        Type 'next' for the next test.
```

## Resuming Tests

If a session is interrupted, the state persists in `.claude/test-state.json`. Simply type `next` to continue from where you left off, or use `/run-tests --reset` to start fresh.
