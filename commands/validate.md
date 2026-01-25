---
name: harness:validate
description: Validate an existing hook-driven test framework setup. Checks configuration, tests syntax, and hook integration.
arguments:
  - name: project-path
    description: Path to project with test harness (defaults to current directory)
    required: false
  - name: fix
    description: Attempt to automatically fix detected issues
    required: false
allowed-tools: Read, Write, Bash, Glob, Grep
---

# Test Framework Validation

This command validates an existing hook-driven test framework setup and optionally fixes detected issues.

## Pre-Validation: Recall from Mnemonic

Before validating, check mnemonic for prior validation insights:

```bash
# Search for validation-related memories
rg -i "validation|harness|hook|test-framework" ~/.claude/mnemonic/ --glob "*.memory.md" -l | head -5

# Check learnings namespace for prior issues
rg -l "." ~/.claude/mnemonic/*/learnings/ --glob "*.memory.md" 2>/dev/null | xargs grep -l -i "harness\|hook" 2>/dev/null | head -5
```

Apply recalled insights:
- Common validation failures and their root causes
- Quick fixes that worked previously
- Configuration patterns that prevent issues

## Validation Checks

### 1. Structure Validation
Verify all required files exist:

```
Required files:
├── .claude/commands/
│   └── run-tests.md
├── tests/functional/
│   ├── runner.sh
│   └── tests.{json|yaml}
└── hooks/
    ├── test-wrapper.sh
    └── hooks.json
```

### 2. Runner Script Validation
Check `tests/functional/runner.sh`:
- File exists and is executable (`chmod +x`)
- Contains required functions: `cmd_init`, `cmd_next`, `cmd_validate`, `cmd_status`, `cmd_report`
- Uses proper JSON escaping (Python-based)
- State file path is correct
- No syntax errors (bash -n check)

### 3. Test Definitions Validation
Check `tests/functional/tests.{json|yaml}`:
- Valid JSON/YAML syntax
- Required schema fields present for each test:
  - `id` (unique, non-empty string)
  - `description` (non-empty string)
  - `action` (non-empty string)
  - `expect` (array with at least one expectation)
- Expectation types are valid: `contains`, `not_contains`, `regex`
- Regex patterns are valid (compile without errors)
- Variable references (`${var}`) have corresponding `save_as` definitions
- `depends_on` references exist
- No duplicate test IDs
- Tags are arrays of strings

### 4. Hook Configuration Validation
Check `hooks/hooks.json`:
- Valid JSON syntax
- Proper wrapper structure: `{"hooks": {...}}`
- UserPromptSubmit hook is registered
- Hook command paths use `${CLAUDE_PLUGIN_ROOT}` or are absolute
- Referenced scripts exist
- Timeout values are reasonable (10-120 seconds)

### 5. Hook Script Validation
Check `hooks/test-wrapper.sh`:
- File exists and is executable
- Contains state file check function
- Handles all test commands: next, skip, validate, status, report, abort
- Properly delegates to runner.sh
- Returns valid JSON with `{"replace": "..."}` structure
- Has passthrough for non-test-mode prompts

### 6. Command Validation
Check `.claude/commands/run-tests.md`:
- Valid YAML frontmatter
- Has name and description
- Instructions reference correct runner commands

## Validation Process

<step name="locate-files">
Find all test framework files in the target project.
Report which files are found and which are missing.
</step>

<step name="validate-structure">
For each required file, check:
1. File exists
2. File is readable
3. File has correct permissions (executable for .sh files)

Collect all structural issues.
</step>

<step name="validate-syntax">
Parse configuration files and check syntax:
1. JSON/YAML test definitions
2. JSON hook configuration
3. Bash scripts (syntax check only)

Report any parsing errors with line numbers.
</step>

<step name="validate-schema">
Deep validation of test definitions:
1. Required fields present
2. Field types correct
3. Cross-references valid
4. Patterns compile

Report schema violations with specific test IDs.
</step>

<step name="validate-integration">
Check that components work together:
1. Hook paths resolve correctly
2. Runner commands match hook expectations
3. State file location consistent
4. Variable substitution chains valid

Report integration issues.
</step>

<step name="report-results">
Generate validation report:

```
# Test Framework Validation Report

## Summary
- Total checks: 47
- Passed: 44
- Warnings: 2
- Errors: 1

## Errors (must fix)
1. [ERROR] tests/functional/tests.yaml:45
   Invalid regex pattern in test 'search_basic': unclosed group

## Warnings (should fix)
1. [WARN] hooks/test-wrapper.sh
   No timeout specified, using default 60s

2. [WARN] tests/functional/tests.yaml
   Test 'crud_update' has no tags

## Passed Checks
- ✓ All required files present
- ✓ Runner script executable
- ✓ Hook configuration valid
...
```
</step>

<step name="auto-fix">
If `--fix` flag provided or user requests fixes:

Fixable issues:
- Missing execute permissions → `chmod +x`
- Missing optional fields → Add defaults
- Deprecated syntax → Update to current
- Incorrect paths → Correct based on project structure

Non-fixable issues (require user action):
- Invalid regex patterns
- Missing required fields
- Broken dependencies
- Syntax errors in scripts
</step>

## Usage Examples

```
# Basic validation
/harness:validate

# Validate specific project
/harness:validate --project-path ../my-plugin

# Validate and auto-fix
/harness:validate --fix
```

## Exit Conditions

Validation succeeds if:
- All required files exist
- No syntax errors in any configuration
- All test definitions are valid
- Hook integration is correct

Validation fails if:
- Any required file missing
- Syntax errors in configuration
- Invalid test definitions
- Hook integration broken

---

## Post-Validation: Capture Learnings to Mnemonic

If validation reveals novel issues or successful fixes, capture to mnemonic:

```bash
# Check if similar insight exists
rg -i "{issue-type}" ~/.claude/mnemonic/ --glob "*.memory.md"

# Capture novel validation learnings
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

cat > ~/.claude/mnemonic/default/learnings/user/${UUID}-harness-validation.memory.md << 'MEMORY'
---
id: ${UUID}
type: episodic
namespace: learnings/user
created: ${DATE}
modified: ${DATE}
title: "Harness Validation: {Issue Type}"
tags:
  - auto-harness
  - validation
  - testing
temporal:
  valid_from: ${DATE}
  recorded_at: ${DATE}
provenance:
  source_type: validation-session
  agent: claude-opus-4
  confidence: 0.8
---

# Validation Learning

## Issue Identified
{Description of the validation issue}

## Root Cause
{Why this occurred}

## Resolution
{How it was fixed}

## Prevention
{How to avoid this in future setups}
MEMORY
```
