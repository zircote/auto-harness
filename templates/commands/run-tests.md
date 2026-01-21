---
name: run-tests
description: Execute the hook-driven test suite with optional filtering by category or tag
arguments:
  - name: category
    description: Filter tests by category (e.g., crud, search, commands)
    required: false
  - name: tag
    description: Filter tests by tag (e.g., smoke, critical, regression)
    required: false
  - name: reset
    description: Reset test state and start fresh
    required: false
---

# Run Test Suite

Execute the automated test suite using the hook-driven testing framework.

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

## Example Session

```
User: /run-tests --category smoke

Claude: Test suite initialized.
        Total tests: 12 (filtered by category: smoke)
        Type 'next' to begin.

User: next

Claude: ## Test 1/12: init_basic
        **Action:** Call subcog_init with include_recall: true

        [Executes the test...]

        Result: Session initialized successfully...

User: validate Session initialized successfully. Status: healthy

Claude: ✅ PASS: init_basic
        Type 'next' for the next test.

User: next
...

User: report

Claude: # Test Report
        **Passed:** 11 | **Failed:** 1 | **Skipped:** 0

        ## Failed Tests
        - search_empty: Missing 'no results'
```

## Filtering Options

### By Category
```
/run-tests --category crud
/run-tests --category commands
/run-tests --category integration
```

### By Tag
```
/run-tests --tag smoke
/run-tests --tag critical
/run-tests --tag regression
```

### Combined
```
/run-tests --category crud --tag critical
```

## Resuming Tests

If a session is interrupted, the state persists in `.claude/test-state.json`. Simply type `next` to continue from where you left off, or use `/run-tests --reset` to start fresh.
