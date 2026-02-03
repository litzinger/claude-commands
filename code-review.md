---
name: code-review
description: Comprehensive PR review using expert agents. Analyzes code changes, determines if net-new or modifications, and provides detailed feedback without making changes.
disable-model-invocation: true
user-invocable: true
argument-hint: [pr-number]
allowed-tools: Read, Grep, Glob, Bash, Task, AskUserQuestion
---

# Pull Request Review

Review pull request #$ARGUMENTS comprehensively using expert agents in parallel.

---

## CRITICAL: READ-ONLY REVIEW

**DO NOT MODIFY ANY FILES. DO NOT MAKE ANY CODE CHANGES.**

This is a review-only command. Your job is to:
1. Analyze the PR
2. Gather expert feedback
3. Provide a summary to the user
4. Optionally offer to post comments via GitHub CLI

**NEVER edit, write, or modify any files in the repository.**

---

## Step 1: Gather PR Information

Use the GitHub CLI to fetch complete PR details:

```bash
# Get PR metadata and description
gh pr view $ARGUMENTS --json title,body,author,baseRefName,headRefName,files,additions,deletions,changedFiles

# Get the full diff
gh pr diff $ARGUMENTS

# Get existing review comments (if any)
gh pr view $ARGUMENTS --comments
```

## Step 2: Analyze Change Type

Determine if this PR is:

### A) Net-New Functionality
Indicators:
- New files being added (not modifications)
- New modules, classes, or significant features
- No existing tests being modified (only new tests added)
- PR description mentions "new feature", "add", "implement", etc.

**Review Focus for Net-New:**
- Is it properly tested? (Unit tests, integration tests as appropriate)
- Does it follow industry best practices for the technology stack?
- Is there appropriate documentation?
- Are error cases handled?
- Is the code organized following project conventions?
- Security considerations for new functionality

### B) Modifications to Existing Code
Indicators:
- Changes to existing files
- Bug fixes, refactoring, or enhancements
- Modifications to existing tests
- PR description mentions "fix", "update", "refactor", "improve", etc.

**Review Focus for Modifications:**
- Does new code match the style of surrounding code?
- Are any new code paths properly tested?
- Does it maintain backward compatibility (if applicable)?
- Are existing tests still passing/relevant?
- Does it follow the patterns established in the codebase?
- Industry best practices for the specific technology

## Step 3: Launch Expert Agent Reviews in Parallel

Based on the technologies detected in the PR, launch relevant expert agents **in parallel** using the Task tool.

### Discover Available Agents

**IMPORTANT:** The user adds new expert agents over time. Before launching reviews, check what agents are currently available by reviewing:
1. The system prompt's list of available `subagent_type` values for the Task tool
2. Any custom agents in `~/.claude/` directories

### Common Agent Mappings (Examples - Not Exhaustive)

These are examples of agent types that may be available. **Always check for additional relevant agents** beyond this list:

- `.ts`, `.tsx`, `.js`, `.jsx` → typescript-architect agent
- `.py` → python-expert agent
- `.swift` → swift-expert agent
- `.java`, `.kt` → general-purpose agent with Java/Kotlin focus
- React/React Native files → react-architecture-expert or react-native-architect
- Architecture/structure → architecture-reviewer agent
- AWS/Cloud → aws-bedrock-expert or similar cloud agents
- Testing → any test-focused agents available

**Always include:**
- `architecture-reviewer` agent (or equivalent) for structural/design review
- `code-simplifier:code-simplifier` agent for code clarity and maintainability review (NOTE: this agent uses scoped naming with a colon)
- Technology-specific expert agent(s) based on file types
- **Any other relevant expert agents you discover** that match the PR's technology stack

### Launching Reviews

**Example parallel Task calls (adapt based on available agents):**
```
Task 1: "Review PR #X diff for TypeScript best practices, type safety, and modern patterns. This is a READ-ONLY review - provide feedback only, no code changes. Focus on: [net-new criteria OR modification criteria based on Step 2]"

Task 2: "Review PR #X diff for architectural concerns, code organization, SOLID principles, and design patterns. This is a READ-ONLY review - provide feedback only. Focus on: [net-new criteria OR modification criteria based on Step 2]"

Task 3 (code-simplifier:code-simplifier): "Review all code changes in PR #X for simplification opportunities. This is a READ-ONLY review - provide feedback only, no code changes. Analyze: code clarity, unnecessary complexity, redundant logic, overly verbose patterns, opportunities for cleaner abstractions, and maintainability improvements. Report findings as suggestions, not changes."
```

### Code Simplifier Review (Required)

The `code-simplifier:code-simplifier` agent **must always be included** in every PR review. This agent specifically looks for:
- Overly complex or convoluted logic that could be simplified
- Redundant code or unnecessary abstractions
- Verbose patterns that could be more concise
- Opportunities for clearer, more maintainable code
- Consistency issues in coding style

Launch this agent in parallel with architecture and technology-specific reviews. The code-simplifier:code-simplifier agent focuses on the "how" of the implementation - making sure the code is as clean and simple as possible while preserving functionality.

Pass the PR diff content to each agent so they have full context.

## Step 4: Compile Review Summary

After all expert agents complete, compile their findings into a structured review:

### Review Summary Format

```markdown
## PR Review: #[number] - [title]

**Author:** [author]
**Branch:** [head] → [base]
**Change Type:** [Net-New Functionality | Modification to Existing Code]
**Files Changed:** [count] (+[additions] / -[deletions])

---

### Technology Stack Detected
- [List of technologies/frameworks identified]

---

### Expert Reviews

#### Architecture Review
[Summary from architecture-reviewer agent]

#### Code Simplifier Review
[Summary from code-simplifier agent - complexity, clarity, maintainability findings]

#### [Technology] Review
[Summary from technology-specific agent]

---

### Key Findings

#### Critical Issues (Must Fix)
- [List any blocking issues]

#### Recommendations (Should Consider)
- [List suggested improvements]

#### Positive Observations
- [What was done well]

---

### Test Coverage Assessment
- [Assessment of test coverage for changes]
- [Missing test scenarios if any]

---

### Industry Best Practices Compliance
- [How well does this follow best practices for the stack?]

---

### Final Recommendation
[ ] Approve
[ ] Approve with suggestions
[ ] Request changes

[Brief rationale]
```

## Step 5: Offer to Post Review

After presenting the summary, use AskUserQuestion to ask:

"Would you like me to post this review as a comment on the PR?"

Options:
1. **Post full review** - Post the complete review summary as a PR comment
2. **Post summary only** - Post just the key findings and recommendation
3. **Copy to clipboard** - Just copy the review so you can post it manually
4. **No thanks** - Don't post, just keep the review in this conversation

If user chooses to post, use:
```bash
gh pr review $ARGUMENTS --comment --body "[review content]"
# OR for approve/request-changes:
gh pr review $ARGUMENTS --approve --body "[review content]"
gh pr review $ARGUMENTS --request-changes --body "[review content]"
```

---

## Important Reminders

1. **This is READ-ONLY** - Never modify files, only analyze and report
2. **Use parallel agents** - Launch multiple Task agents simultaneously for efficiency
3. **Be constructive** - Focus on actionable feedback, not nitpicking
4. **Respect project conventions** - Check CLAUDE.md and existing patterns before critiquing style
5. **Confidence matters** - Only report high-confidence issues (following code-reviewer pattern of 80%+ confidence)
