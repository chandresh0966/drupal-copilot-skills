# Drupal Copilot Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Copilot](https://img.shields.io/badge/GitHub%20Copilot-Skills-blue)](https://code.visualstudio.com/docs/copilot/customization/overview)
[![Drupal](https://img.shields.io/badge/Drupal-10%2F11-009cde)](https://www.drupal.org/)

A comprehensive collection of GitHub Copilot customizations for Drupal development,
converted from [drupal-claude-skills](https://github.com/grasmash/drupal-claude-skills) by
[@grasmash](https://github.com/grasmash).

## What's Included

### 8 Instruction Files (`.github/instructions/`)

| File | Description |
|------|-------------|
| `drupal-at-your-fingertips` | 50+ Drupal topics — services, hooks, entities, forms, theming, caching, testing |
| `drupal-config-mgmt` | Configuration management — safe import/export, config splits, environment syncing |
| `drupal-contrib-mgmt` | Contrib module management — Composer updates, patch management, Drupal 11 compatibility |
| `drupal-ddev` | DDEV local development — setup, commands, database ops, Xdebug, performance |
| `drupal-security` | OWASP Top 10 security patterns — auth, access control, injection prevention, crypto |
| `drupal-simple-oauth` | OAuth2 with simple_oauth — TokenAuthUser permissions, scope/role matching, CSRF bypass |
| `drupal-search-api` | Search API — index configuration, boost processors, config management, reindexing |
| `drupal-canvas` | Drupal Canvas Code Components — scaffolding, Nebula template, Acquia Source Site Builder |

### 8 Custom Agents (`.github/agents/`)

| Agent | Description |
|-------|-------------|
| `quality-gate` | Pre-commit code review — security, performance, testing, regressions |
| `done-gate` | Completion validator — builds pass, tests run, deliverables exist |
| `drupal-specialist` | Drupal/PHP implementation — modules, hooks, services, Drush |
| `frontend-specialist` | Frontend — Twig, SCSS, JavaScript, responsive design, accessibility |
| `researcher` | Codebase exploration — architecture, patterns, execution paths |
| `reviewer` | Code review — bugs, security, quality, actionable feedback |
| `test-runner` | Test execution — PHP, JS, SCSS validation, build checks |
| `test-writer` | ExistingSite test writing — bug reproduction, DTT patterns |

### Always-on Instructions (`.github/copilot-instructions.md`)

Workspace-wide Drupal agent workflow guide including:
- Quality gate and done gate workflows
- Contrib/core patch policy
- CI test fix workflow
- Session completion checklist

## Installation

### Option 1: Clone into your Drupal project

```bash
# Clone this repo
git clone https://github.com/chandresh0966/drupal-copilot-skills.git /tmp/drupal-copilot-skills

# Copy instructions
mkdir -p .github/instructions
cp /tmp/drupal-copilot-skills/.github/instructions/*.instructions.md .github/instructions/

# Copy agents
mkdir -p .github/agents
cp /tmp/drupal-copilot-skills/.github/agents/*.agent.md .github/agents/

# Copy always-on instructions (merge with existing if needed)
cp /tmp/drupal-copilot-skills/.github/copilot-instructions.md .github/copilot-instructions.md

# Clean up
rm -rf /tmp/drupal-copilot-skills
```

### Option 2: Manual copy

Copy the files you need from this repo directly into your Drupal project's `.github/` folder.

## Usage

### Instructions (auto-applied)

Instructions files are automatically applied based on the `applyTo` glob pattern in their
frontmatter. For example:
- `drupal-security.instructions.md` applies to all `*.php` files
- `drupal-ddev.instructions.md` applies when working in `.ddev/` directories

You can also manually reference instructions in any chat by tagging `#drupal-config-mgmt` etc.

### Agents (invoked explicitly)

Switch to a custom agent in GitHub Copilot Chat:
1. Click the agent dropdown (default: "Ask", "Edit", "Agent")
2. Select an agent like `drupal-specialist` or `quality-gate`

Or invoke as subagents in chat:
```
@drupal-specialist Implement a new custom block that shows the latest articles
```

```
@quality-gate Review my recent changes before I commit
```

## Repository Structure

```
.github/
├── copilot-instructions.md          # Always-on workspace instructions
├── instructions/                    # File-based instruction files
│   ├── drupal-at-your-fingertips.instructions.md
│   ├── drupal-canvas.instructions.md
│   ├── drupal-config-mgmt.instructions.md
│   ├── drupal-contrib-mgmt.instructions.md
│   ├── drupal-ddev.instructions.md
│   ├── drupal-search-api.instructions.md
│   ├── drupal-security.instructions.md
│   └── drupal-simple-oauth.instructions.md
└── agents/                          # Custom agent definitions
    ├── done-gate.agent.md
    ├── drupal-specialist.agent.md
    ├── frontend-specialist.agent.md
    ├── quality-gate.agent.md
    ├── researcher.agent.md
    ├── reviewer.agent.md
    ├── test-runner.agent.md
    └── test-writer.agent.md
```

## Conversion Notes

This repo converts the [drupal-claude-skills](https://github.com/grasmash/drupal-claude-skills) format to GitHub Copilot's native VS Code customization format:

| Claude Code | GitHub Copilot |
|-------------|----------------|
| `skills/*/SKILL.md` | `.github/instructions/*.instructions.md` |
| `.claude/agents/*.md` | `.github/agents/*.agent.md` |
| `AGENTS.md` | `.github/copilot-instructions.md` |
| `.claude/settings.json` | VS Code settings / `github.copilot.*` |

## Credits

- [Matthew Grasmick (@grasmash)](https://github.com/grasmash) — Original drupal-claude-skills
- [Selwyn Polit](https://drupalatyourfingertips.com/) — Drupal at Your Fingertips
- [Ivan Grynenko](https://github.com/ivangrynenko/cursorrules) — Drupal security patterns
- Drupal Community — Ongoing contributions

## License

MIT — see [LICENSE](LICENSE)

## Related Resources

- [VS Code Copilot Customization Docs](https://code.visualstudio.com/docs/copilot/customization/overview)
- [drupal-claude-skills (original)](https://github.com/grasmash/drupal-claude-skills)
- [Drupal.org](https://drupal.org/)
- [DDEV Documentation](https://ddev.readthedocs.io/)
