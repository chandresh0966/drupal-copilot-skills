# Drupal Copilot Skills

This repository provides reusable GitHub Copilot customizations for Drupal development,
converted from [drupal-claude-skills](https://github.com/grasmash/drupal-claude-skills).

## Agent Workflow Guide

### Parallelization

Maximize parallel work. Use multiple subagents when tasks are independent. Use agent
teams for features spanning Drupal + frontend. Research areas concurrently before
implementing.

### Quality Gate (mandatory before commit)

Before committing code changes, invoke the `quality-gate` agent to review the diff. It
checks security, performance, testing, and regressions. Proactively run the review
before reaching the commit step — don't wait for a reminder. For agent teams, the team
lead should invoke quality-gate after all teammates finish, not each teammate
individually.

### Done Gate (mandatory before declaring done)

After quality-gate passes and before telling the user work is complete, invoke the
`done-gate` agent. It validates that the work actually works — builds succeed, tests
pass, deliverables match the plan. Quality-gate checks code quality; done-gate checks
functional correctness. Both must pass before declaring done.

### Local Execution Verification (mandatory)

For any fix or feature that changes runtime behavior (cron jobs, drush commands, data
processing, API endpoints, service logic), you MUST execute the changed code path
locally via DDEV and verify the results before declaring done. Unit tests passing is
not sufficient — run the actual command/service and confirm the real-world outcome.

### CI Test Fix Workflow

When CI fails with test errors, do NOT iterate by pushing commits and re-running the
full suite. Instead:

1. Identify the failing tests from the CI logs
2. Reproduce locally — run the specific failing tests in your local environment
3. Fix each failing test and run it individually until it passes (e.g., `vendor/bin/phpunit --filter ClassName::testMethod`)
4. Commit the fix to the branch
5. Only re-run the full CI suite after all individual tests pass locally

This avoids wasting CI minutes on known-broken tests.

### Contrib/Core Patch Policy

NEVER directly edit files in `docroot/modules/contrib/` or `docroot/core/`. All changes
to contributed modules and Drupal core MUST use patches:

1. Create a `.patch` file in `patches/`
2. Register it in `composer.json` under `extra.patches`
3. Run `composer install` to apply

Direct edits get overwritten by `composer install`/`composer update`. See the
`drupal-contrib-mgmt` instructions for full patch management workflows.

### Session Completion (Landing the Plane)

When ending a work session, complete these steps:

1. File issues for remaining work — Create issues for anything that needs follow-up
2. Run quality gates (if code changed) — Tests, linters, builds
3. Update issue status — Close finished work, update in-progress items
4. Commit changes — Ensure all work is committed locally
5. Ask before pushing — Ask the user if they want you to push to remote. Do NOT push automatically
6. Hand off — Provide context for next session
