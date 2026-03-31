---
name: drupal-contrib-mgmt
description: Comprehensive guide for managing Drupal contributed modules via Composer, including updates, patches, version compatibility, and Drupal 11 upgrades. Use when updating modules or resolving dependency issues.
applyTo: "**"
---

# Drupal Contrib Module Management

## Core Update Workflow

### Standard Module Update

```bash
# Update a single module
composer require drupal/module_name --with-all-dependencies

# Update to specific version
composer require drupal/module_name:^3.0 --with-all-dependencies

# After any update, ALWAYS run database updates
drush updb -y

# Clear cache
drush cr

# CRITICAL: Test by visiting pages to check for fatal errors
```

## Checking Drupal 11 Compatibility

### Method 1: Check .info.yml File (Fastest, Most Reliable)

```bash
cat docroot/modules/contrib/MODULE_NAME/MODULE_NAME.info.yml | grep core_version_requirement
```

**What to look for**:
```yaml
core_version_requirement: ^9.5 || ^10 || ^11     # ✅ D11 compatible
core_version_requirement: ^9 || ^10                # ❌ Not D11 compatible yet
```

### Method 2: Use Composer Commands

```bash
composer show drupal/MODULE_NAME --all | grep -A5 "^versions"
```

## Patch Management (cweagans/composer-patches)

**IMPORTANT**: Use version 2.x for reliable patch application.

### Patch Configuration

```json
{
  "require": { "cweagans/composer-patches": "^2.0" },
  "config": { "allow-plugins": { "cweagans/composer-patches": true } },
  "extra": {
    "composer-exit-on-patch-failure": true,
    "patches": {
      "drupal/module_name": {
        "Description of patch": "https://www.drupal.org/files/issues/2024-01-15/module-issue-1234567-8.patch",
        "Local patch": "patches/custom-fix.patch"
      }
    }
  }
}
```

### Finding Patches — Search FIRST Before Creating

**CRITICAL WORKFLOW**: When encountering Drupal errors, ALWAYS search for existing patches before creating your own.

1. Extract the exact error signature from the error message
2. Search the Drupal.org issue queue: `https://www.drupal.org/project/issues/MODULE_NAME?text=ERROR_STRING`
3. Find existing patches — look for RTBC (Reviewed & tested by the community) status
4. Download and apply official patch
5. Only create a custom patch if none exists

**Patch Naming Convention**: `module-issue-NODEID-COMMENT.patch`

### Creating Local Patches

```bash
# Clone the module repo to a separate directory
cd ~/Sites
git clone git@git.drupal.org:project/module_name.git module_name-contrib

# Checkout the exact version you have installed
cd ~/Sites/module_name-contrib
git checkout 1.0.3

# Make your changes, then generate the patch
git diff > ~/Sites/your-project/patches/module_name-custom-fix.patch

# Add to composer.json and apply
composer reinstall drupal/module_name
```

### Patch Application

```bash
composer install          # Install with patches
composer reinstall drupal/module_name  # Re-patch a single module
```

## Drupal Lenient Plugin

Allows installing modules that haven't updated their version requirements:

```json
{
  "require": { "mglaman/composer-drupal-lenient": "^1.0" },
  "config": { "allow-plugins": { "mglaman/composer-drupal-lenient": true } },
  "extra": {
    "drupal-lenient": {
      "allowed-list": ["drupal/module_name"]
    }
  }
}
```

## Drupal 11 Compatibility Workflow

```bash
# Step 1: Analyze readiness
drush upgrade_status:analyze --all

# Step 2: Fix custom code deprecations
# REQUEST_TIME → \Drupal::time()->getRequestTime()
# user_roles() → \Drupal\user\Entity\Role::loadMultiple()

# Step 3: Re-scan to confirm fixes
drush upgrade_status:analyze module_name
```

## Production Deployment

```bash
# CRITICAL: Always use these flags for production
composer install --no-dev -o
# --no-dev: Excludes development dependencies
# -o: Optimizes autoloader for performance
```

## Best Practices

1. **Always use `--with-all-dependencies`** for module updates
2. **Always run `drush updb`** after composer updates
3. **Test immediately** after updates (visit pages, check logs)
4. **Keep patches organized** in a `patches/` directory
5. **Check issue queues first** before creating custom patches
6. **Use upgrade_status** to validate D11 compatibility
7. **Commit atomically**: one module update per commit

## Complete Update Checklist

- [ ] Check current module version: `composer show drupal/module_name`
- [ ] Search issue queue for known issues
- [ ] Check if module is D11 compatible
- [ ] Update composer.json with new version
- [ ] Add to drupal-lenient if needed
- [ ] Search for and apply necessary patches
- [ ] Run `composer require drupal/module_name:^X.0 --with-all-dependencies`
- [ ] Run `drush updb -y`
- [ ] Run `drush cr`
- [ ] Run `drush upgrade_status:analyze module_name`
- [ ] Test module functionality
- [ ] Check for PHP errors/warnings in logs
- [ ] Commit changes with descriptive message

## Contributing Back to drupal.org

```bash
# Create issue fork on drupal.org, then:
git remote add module_name-XXXXXXX git@git.drupal.org:issue/module_name-XXXXXXX.git
git checkout -b 'XXXXXXX-short-description' --track module_name-XXXXXXX/'XXXXXXX-short-description'

# Make changes, commit with proper format:
git commit -m "Issue #XXXXXXX: Short description"
git push module_name-XXXXXXX XXXXXXX-short-description
```

## Reference Links

- **Composer Patches**: https://github.com/cweagans/composer-patches
- **Drupal Lenient**: https://github.com/mglaman/composer-drupal-lenient
- **Upgrade Status Module**: https://www.drupal.org/project/upgrade_status
