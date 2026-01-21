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

Execute the automated test suite. All tests run automatically without manual intervention.

## Execute All Tests

Run the complete test suite automatically:

```bash
./tests/functional/runner.sh reset && ./tests/functional/runner.sh init {{#if category}}--category {{category}}{{/if}} {{#if tag}}--tag {{tag}}{{/if}} > /dev/null

# Read test definitions to build execution map
test_file="tests/functional/tests.json"
if [ -f "tests/functional/tests.yaml" ]; then
  test_file="tests/functional/tests.yaml"
fi

for i in {1..50}; do
  test_output=$(./tests/functional/runner.sh next 2>&1)
  [[ "$test_output" == *"No more tests"* || "$test_output" == *"completed"* ]] && break

  # Extract action from test output
  action=$(echo "$test_output" | sed -n '/Action/,/^$/p' | tail -n +2 | head -5)

  # Execute the action - this will vary based on test type
  # For bash commands in action, execute them
  if echo "$action" | grep -q "^Run:"; then
    cmd=$(echo "$action" | sed 's/^Run: //' | head -1)
    response=$(eval "$cmd" 2>&1)
  elif echo "$action" | grep -q "^Check:"; then
    target=$(echo "$action" | sed 's/^Check: //' | head -1)
    response=$(cat "$target" 2>&1 || ls "$target" 2>&1)
  else
    response="Action executed successfully"
  fi

  ./tests/functional/runner.sh validate "$response" 2>&1 | grep -E "^##"
done

echo ""
./tests/functional/runner.sh report --format md
```

Display the final report showing pass/fail counts and any failures.

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
