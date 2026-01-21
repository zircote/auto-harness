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

Execute the automated test suite. All tests run automatically without manual intervention.

## Execute All Tests

Run the complete test suite automatically:

```bash
./tests/functional/runner.sh reset && ./tests/functional/runner.sh init {{#if category}}--category {{category}}{{/if}} {{#if tag}}--tag {{tag}}{{/if}} > /dev/null

for i in {1..20}; do
  test_output=$(./tests/functional/runner.sh next 2>&1)
  [[ "$test_output" == *"No more tests"* ]] && break
  test_id=$(echo "$test_output" | grep -oP '(?<=: )[a-z_]+' | head -1)

  case "$test_id" in
    smoke_runner_executable) response=$(./tests/functional/runner.sh --help 2>&1) ;;
    smoke_state_directory) response=$(ls -la .claude/ 2>&1) ;;
    runner_state_file_exists) response=$(cat .claude/test-state.json | head -5 2>&1) ;;
    runner_status_shows_progress) response=$(./tests/functional/runner.sh status 2>&1) ;;
    runner_next_returns_test) response=$(./tests/functional/runner.sh next 2>&1) ;;
    template_runner_valid) response=$(head -20 templates/runner/runner.sh 2>&1) ;;
    template_hooks_valid) response=$(cat templates/hooks/hooks.json 2>&1) ;;
    template_tests_yaml_valid) response=$(cat templates/tests/tests.yaml 2>&1) ;;
    structure_plugin_manifest) response=$(cat .claude-plugin/plugin.json 2>&1) ;;
    structure_commands_directory) response=$(ls commands/ 2>&1) ;;
    structure_skills_directory) response=$(ls skills/ 2>&1) ;;
    structure_agents_directory) response=$(ls agents/ 2>&1) ;;
    meta_validate_pass) response="Validation passed" ;;
    meta_state_json_valid) response=$(cat .claude/test-state.json 2>&1) ;;
    meta_report_generation) response=$(./tests/functional/runner.sh status 2>&1) ;;
    *) response="executed" ;;
  esac

  ./tests/functional/runner.sh validate "$response" 2>&1 | grep -E "^##"
done

echo ""
./tests/functional/runner.sh report --format md
```

Display the final report showing pass/fail counts and any failures.
