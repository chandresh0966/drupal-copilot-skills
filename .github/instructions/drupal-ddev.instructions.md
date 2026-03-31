---
name: drupal-ddev
description: DDEV local development environment patterns for Drupal, including configuration, commands, database management, debugging tools, and performance optimization.
applyTo: "**/.ddev/**,**/ddev/**"
---

# DDEV for Drupal Development

Comprehensive patterns for using DDEV as your local Drupal development environment,
including setup, configuration, workflow optimization, and troubleshooting.

## Quick Reference

### Essential Commands

```bash
ddev start                            # Start project
ddev stop                             # Stop project
ddev restart                          # Restart services
ddev ssh                              # SSH into web container

# Run Drush commands
ddev drush cr
ddev drush status
ddev drush config:status

# Run Composer
ddev composer require drupal/module_name
ddev composer update

# Database operations
ddev import-db --file=backup.sql.gz
ddev export-db --file=backup.sql.gz
ddev snapshot
ddev snapshot restore --name=before-testing

# View logs
ddev logs
ddev logs -f    # Follow mode

ddev describe   # Describe project
ddev launch     # Open site in browser
```

### Basic .ddev/config.yaml

```yaml
name: myproject
type: drupal10
docroot: web
php_version: "8.3"
webserver_type: nginx-fpm
database:
  type: mariadb
  version: "10.6"
nodejs_version: "20"
performance_mode: mutagen  # For macOS
```

## Common Workflows

### New Drupal Project

```bash
mkdir myproject && cd myproject
ddev config --project-type=drupal10 --docroot=web --php-version=8.3
ddev composer create drupal/recommended-project
ddev composer require drush/drush
ddev start
ddev drush site:install standard --site-name="My Site" --account-name=admin
ddev launch
```

### Import Existing Project

```bash
git clone repo-url myproject && cd myproject
ddev start
ddev composer install
ddev import-db --file=path/to/backup.sql.gz
ddev drush updb -y
ddev drush cr
ddev launch
```

### Database Sync from Remote

```bash
# Option 1: Download backup and import
scp user@remote.server:/path/to/backup.sql.gz backup.sql.gz
ddev import-db --file=backup.sql.gz

# Option 2: Using DDEV pull
ddev pull --environment=live

# Run updates after import
ddev drush updb -y
ddev drush cr
```

### Daily Development Workflow

```bash
ddev start
git pull origin main
ddev composer install
ddev drush cr
# ... work on features ...
ddev snapshot --name=before-testing  # Create snapshot before testing
# ... test changes ...
ddev snapshot restore --name=before-testing  # If needed, restore
ddev stop
```

## Debugging with Xdebug

```bash
ddev xdebug on     # Enable Xdebug
ddev xdebug off    # Disable when done (improves performance)
ddev xdebug status
```

**VSCode launch.json**:
```json
{
  "name": "Listen for Xdebug",
  "type": "php",
  "request": "launch",
  "port": 9003,
  "pathMappings": {
    "/var/www/html": "${workspaceFolder}"
  }
}
```

## Performance Optimization

### macOS Performance (Mutagen)

```yaml
# .ddev/config.yaml
performance_mode: mutagen
```

```bash
ddev restart  # After config change
```

### Database Tuning

```yaml
# .ddev/config.yaml — create .ddev/mysql/my.cnf:
[mysqld]
innodb_buffer_pool_size = 512M
innodb_log_file_size = 128M
```

## Custom Commands

Create project-specific commands in `.ddev/commands/web/`:

```bash
#!/bin/bash
## Description: Fresh Drupal install from scratch
## Usage: fresh-install

set -e
drush sql-drop -y
drush site:install standard --site-name="My Site" --account-name=admin --account-pass=admin -y
if [ -d /var/www/html/config/default ]; then drush config:import -y; fi
drush cr
```

```bash
chmod +x .ddev/commands/web/fresh-install
ddev fresh-install
```

## Troubleshooting

### Site Not Loading

```bash
ddev restart
ddev describe
ddev logs
ddev drush cr
```

### Slow Performance on macOS

```bash
ddev config --performance-mode=mutagen
ddev restart
```

### PHP Deprecation Warnings in Drush

Create **`.ddev/php/drush.ini`**:
```ini
[PHP]
error_reporting = 22527
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /tmp/php-errors.log
```
Then: `ddev restart`

### Docker overlay2 I/O Errors

A normal quit and restart of Docker Desktop is **not sufficient**. You must **force quit ALL Docker processes** (via Activity Monitor or `killall -9 Docker`), then relaunch Docker Desktop.

### Unhealthy Containers / Mutagen Sync Hanging

```bash
# 1. Full power off
ddev poweroff
# 2. Start fresh
ddev start

# If still failing:
ddev stop
~/.ddev/bin/mutagen daemon stop
~/.ddev/bin/mutagen daemon start
ddev mutagen reset
ddev start
```

## Best Practices

1. **Commit .ddev/config.yaml** - Share config with team
2. **Use ddev composer** instead of local composer
3. **Don't commit database snapshots** - Too large
4. **Create snapshots before risky operations**
5. **Disable Xdebug when not debugging** - Performance impact
6. **Use mutagen on macOS** - Much faster file sync
7. **Version pin services** - PHP, database, Node.js

**Official Documentation**: https://ddev.readthedocs.io
