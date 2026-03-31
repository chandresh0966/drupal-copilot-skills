---
name: drupal-config-mgmt
description: Drupal configuration management including config import/export, config splits (complete and partial), syncing config across environments, drush commands for config management, config:import, config:export, config-split commands
applyTo: "**"
---

# Drupal Configuration Management

Comprehensive guide for Drupal configuration management including imports, exports, config
splits, and environment syncing.

## Problem: Avoid Accidental Config Imports

**CRITICAL**: Remote drush commands may default to `--yes` depending on your hosting
setup. This means commands like `config:import` or `cim` can AUTO-CONFIRM and import
configuration even when you only want to inspect differences.

### Dangerous vs Safe Patterns

❌ **DANGEROUS** - May auto-import without confirmation:
```bash
ssh user@remote.server "cd /path/to/drupal && drush cim --diff"
```

✅ **SAFE** - Shows diff without importing:
```bash
ssh user@remote.server "cd /path/to/drupal && drush cim --no --diff"
```

✅ **SAFEST** - Read-only commands:
```bash
ssh user@remote.server "cd /path/to/drupal && drush config:get config.name"
ssh user@remote.server "cd /path/to/drupal && drush config:status"
```

## Preferred Prod Config Merge Workflow

**Purpose**: Safely merge production config changes while preserving local feature work.

### The Process

**Step 1: Commit local changes first**
```bash
git add config/default/your-new-field.yml docroot/modules/custom/your_module/your_module.module
git commit -m "feat: add new feature"
```

**Step 2: Pull production database**
```bash
ddev pull --environment=live
```

**Step 3: Export config from prod DB**
```bash
ddev drush config:export -y
```

**Step 4: Review git diff on config directory**
```bash
git diff --stat config/           # Summary of changes
git status --short config/        # See added/modified/deleted
```

**Step 5: Identify files to revert vs keep**

Look for these patterns:
- **D (Deleted)** - Your new feature files deleted by prod export → **REVERT**
- **M (Modified)** - UUID changes from prod → **KEEP**
- **M (Modified)** - Actual prod config changes → **KEEP**
- **M (Modified)** - Local changes overwritten → **REVERT** (case by case)

**Step 6: Restore your local feature files**
```bash
git checkout HEAD -- config/default/field.storage.node.your_new_field.yml
git checkout HEAD -- config/default/field.field.node.bundle.your_new_field.yml
```

**Step 7: Verify and commit prod config**
```bash
git status --short config/
git add config/
git commit -m "chore: sync config from production"
```

## Configuration Import & Export Basics

### Exporting Configuration

```bash
# Export ALL configuration
ddev drush config:export
ddev drush cex

# Export a SINGLE config object
ddev drush config:get views.view.content --format=yaml > config/default/views.view.content.yml
```

### Importing Configuration

```bash
# Import ALL configuration
ddev drush config:import
ddev drush cim

# Always preview changes first
ddev drush config:import --no --diff
```

## Config Splits Overview

Config splits allow **environment-specific configuration** that doesn't get deployed everywhere.

### Complete Splits (Recommended for most cases)

- Config in the split is **ONLY active** when split is enabled
- When disabled, config is **completely removed** from active config
- Use for: development modules (devel, kint), environment-specific modules

### Partial Splits (Conditional Overrides)

- Base config exists in `config/default/`
- Split contains **overrides** merged when active
- Use for: API keys, email settings, cache settings, search server URLs

### Choosing Complete vs Partial

**Use Complete when**: Config should NOT exist in other environments (modules, views, blocks)
**Use Partial when**: Config exists everywhere but with different VALUES (API URLs, credentials)

## Config Split Commands

```bash
# Export ALL config including active splits
ddev drush config:export

# Import ALL config including active splits
ddev drush config:import

# Check split status
ddev drush config-split:status
ddev drush css

# Activate/Deactivate splits
ddev drush config-split:activate {split-name}
ddev drush config-split:deactivate {split-name}

# Export/import specific split
ddev drush csex {split-name}
ddev drush csim {split-name}
```

## Best Practices

1. **Always inspect before importing** - Use `config:get` and `--no --diff` flags
2. **Manual edits preferred** - Edit config files directly for precision
3. **One config type per commit** - Separate concerns for clean history
4. **Clear commit messages** - Reference source environment
5. **Use config splits** - Keep environment-specific config separate

## Troubleshooting

### Config deleted from config/default on export

**Root cause (99% of cases)**: Config is in `complete_list` instead of `partial_list`!

**Diagnosis**:
```bash
grep -A10 "complete_list:" config/default/config_split.config_split.local.yml
grep -A10 "partial_list:" config/default/config_split.config_split.local.yml
```

**Solution**: Move from `complete_list` to `partial_list`

### Related Commands

**Read-only**: `config:get`, `config:status`
**Exports**: `config:export` (alias: `cex`)
**Imports**: `config:import` (alias: `cim`) - Use with `--no --diff` to preview
**Splits**: `config-split:status`, `csex`, `csim`, `config-split:activate`
