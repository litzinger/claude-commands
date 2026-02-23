---
name: code-review-from-hash
description: Comprehensive PR review. Analyzes code changes, determines if net-new or modifications, and provides detailed feedback without making changes.
disable-model-invocation: true
user-invocable: true
argument-hint: [commit hash]
allowed-tools: Read, Grep, Glob, Bash, Task, AskUserQuestion
---

# Code Review

Review commits going back to $ARGUMENTS - you are a staff php software engineer performing a code review. Determine if this PR is:

## A) Net-New Functionality

Indicators:

- New files being added (not modifications)
- New modules, classes, or significant features
- No existing tests being modified (only new tests added)
- PR description mentions "new feature", "add", "implement", etc.

Review Focus for Net-New:

- Is it properly tested? (Unit tests, integration tests as appropriate)
- Does it follow industry best practices for the technology stack?
- Is there appropriate documentation?
- Are error cases handled?
- Is the code organized following project conventions?
- Security considerations for new functionality

## B) Modifications to Existing Code

Indicators:

- Changes to existing files
- Bug fixes, refactoring, or enhancements
- Modifications to existing tests
- PR description mentions "fix", "update", "refactor", "improve", etc.

Review Focus for Modifications:

- Does new code match the style of surrounding code?
- Are any new code paths properly tested?
- Does it maintain backward compatibility (if applicable)?
- Are existing tests still passing/relevant?
- Does it follow the patterns established in the codebase?
- Industry best practices for the specific technology
- Look for overly complex or convoluted logic that could be simplified
- Redundant code or unnecessary abstractions
- Verbose patterns that could be more concise
- Opportunities for clearer, more maintainable code
- Consistency issues in coding style
