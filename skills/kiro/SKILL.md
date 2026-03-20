---
name: kiro-spec-bridge
description: Use this skill whenever the user wants to start from a specification before coding, asks to define requirements, acceptance criteria, or implementation plan first, or wants to use Kiro for planning while keeping Claude Code as the main coder. Trigger on phrases like "仕様から始めたい", "先にspec作って", "kiroで要件整理", "spec-firstで", "要件定義してから", or any request to create structured requirements before implementation. Even if the user just says "まず仕様を決めよう" or "要件をまとめてから実装したい" — use this skill. Claude Code handles implementation; this skill handles the spec generation phase via Kiro CLI.
---

# Kiro Spec Bridge

Generate a structured spec with Kiro CLI, then hand off to Claude Code for implementation.

## What this skill does

1. Clarifies the feature name (briefly, if not given)
2. Builds a focused prompt for Kiro using the required spec template
3. Runs `kiro-cli chat --no-interactive "<prompt>"` to generate a draft
4. Saves the result to `specs/<feature-slug>.md`
5. Reviews the spec for missing sections
6. Prepares a handoff prompt for Claude Code

## Preconditions

Check that `kiro-cli` is in PATH before running. If not available, skip Kiro and generate the spec directly using the same 8-section template — do not block the workflow.

## Step 1: Normalize the feature name

Turn the user's description into a filesystem-safe slug (e.g., `csv-dedup`, `oauth-login`, `invoice-export`). Output path: `specs/<slug>.md`.

## Step 2: Build and run the Kiro command

```bash
kiro-cli chat --no-interactive "<prompt>"
```

### Kiro prompt template

Fill in `<feature-description>` and use this prompt verbatim:

```
You are a specification designer. Generate a Markdown specification for: <feature-description>

Rules:
- Use ONLY these sections in this exact order:
  1. # Goal
  2. # Scope
  3. # Non-Goals
  4. # User Stories
  5. # Acceptance Criteria
  6. # Implementation Plan
  7. # Test Cases
  8. # Risks / Open Questions

- Acceptance Criteria: every item must be testable (pass/fail verifiable)
- Implementation Plan: steps must be actionable by a developer — not generic advice
- If you don't know something, put it under Risks / Open Questions — do NOT invent requirements
- Use bullet points, not long prose
- Be concrete and falsifiable throughout
```

Save the output to `specs/<slug>.md`.

## Step 3: Validate the spec

Check that all 8 sections exist in order. If any are missing:
- Regenerate with a tighter prompt, OR
- Add the missing section manually with `<!-- TODO: fill in -->` and move on

If the output is too vague, add these constraints to the prompt and rerun:
- "Use testable statements, not vague ones"
- "Name concrete edge cases"
- "Reference project constraints if any were provided"

## Step 4: Handoff to Claude Code

Produce this handoff prompt for the implementation phase:

```
Read specs/<slug>.md and implement the feature:
1. Summarize the spec in ≤5 bullet points
2. Implement according to # Implementation Plan
3. Write or update tests covering # Acceptance Criteria
4. Flag any deviations or unresolved items from # Risks / Open Questions
```

Save as `specs/<slug>.handoff.md` if the implementation is non-trivial.

## Failure modes

### `kiro-cli` fails or is missing
Capture stderr, explain briefly, then generate the spec yourself using the same 8-section template. Keep the workflow moving — Kiro is a tool, not a gate.

### Output is too vague after two retries
Patch the spec manually, mark assumptions clearly, and proceed.

## Examples

### CSV dedup feature
User: "CSVアップロードして重複行を削除する機能を、まず仕様にまとめてから実装したい"
→ Generates `specs/csv-dedup.md` covering duplicate-definition rules, edge cases, and test cases. Hands off to Claude Code.

### Auth feature
User: "ログイン機能を先に仕様化してから実装したい"
→ Creates `specs/login-auth.md` with MVP scope, non-goals, and success/failure flow criteria.

## Note

Keep Kiro in the **spec lane**. The value of this workflow:
- Clearer requirements upfront
- Testable acceptance criteria
- Lower implementation drift
