# Electromagnetic Tissue Simulator — Project Plan

## Goals

## Scope

### In scope

### Out of scope

## Decisions

## Physics & modelling

### Governing equations

### Tissue models & material properties

### Numerical methods

### Validation & verification

## Architecture

## Technical plan

## Environments

## Engineering workflow

### Branching

### CI gates on pull requests

Every pull request to `main` must pass three checks before it can merge. The checks are jobs in a CI workflow (`.github/workflows/ci.yml`, not yet written), and a branch ruleset on `main` makes them required.

| Check (job name) | What it does | Fails when |
|---|---|---|
| `unit-tests` | Runs the unit test suite | Any test fails |
| `coverage` | Runs the tests with coverage measurement | Coverage is below the threshold (TBD, e.g. 80%) |
| `static-analysis` | Runs the linter and type checker | Any error is reported |

- The coverage threshold is enforced inside the workflow, not by the ruleset. The ruleset only sees whether the `coverage` check passed.
- The specific tools (test runner, coverage tool, linter, type checker) depend on the language, which is still to be decided. Write the workflow once it is.
- Ruleset setup (**Settings → Rules → Rulesets**, targeting the default branch):
  - Require a pull request before merging, with 0 required approvals. PRs opened from Claude sessions are authored as the repo owner, who can't approve their own PR; the owner merging is the approval.
  - Restrict deletions and block force pushes.
  - Require status checks to pass: `unit-tests`, `coverage`, `static-analysis`. Each job must have run at least once before GitHub offers it as a required check.
  - Optional: require branches to be up to date before merging.
  - Optional: require code scanning results (CodeQL). Needs a CodeQL workflow, and GitHub Advanced Security on a private repo.

### Dependency updates

## Code style & conventions

### Code

### Tests

### Copy and docs

## Milestones

## Open questions

## Future ideas
