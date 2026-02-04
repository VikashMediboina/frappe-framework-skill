# Bench Commands Reference

Bench is the CLI tool for managing Frappe sites, apps, and the development environment.

## Installation & Setup

```bash
# Install bench
pip install frappe-bench

# Initialize bench directory
bench init frappe-bench
cd frappe-bench

# Initialize with specific frappe version
bench init frappe-bench --frappe-branch version-15
```

---

## Site Management

### Creating Sites

```bash
# Create new site
bench new-site site_name

# With specific admin password
bench new-site site_name --admin-password secret123

# With database name
bench new-site site_name --db-name custom_db_name

# With MariaDB root password
bench new-site site_name --mariadb-root-password root_pass

# Install app during creation
bench new-site site_name --install-app erpnext

# From existing backup
bench new-site site_name --source_sql /path/to/backup.sql.gz
```

### Switching Sites

```bash
# Set default site
bench use site_name

# List all sites
bench --site all list-apps
```

### Deleting Sites

```bash
# Drop site
bench drop-site site_name

# Force drop (skip confirmation)
bench drop-site site_name --force

# Drop database as well
bench drop-site site_name --db-root-password root_pass
```

---

## App Management

### Creating Apps

```bash
# Create new app
bench new-app app_name

# Interactive prompts for app details
bench new-app my_app
# App Title: My App
# App Description: Description of my app
# App Publisher: Your Name
# App Email: email@example.com
```

### Getting Apps

```bash
# Get app from GitHub
bench get-app https://github.com/org/app_name

# From specific branch
bench get-app https://github.com/org/app_name --branch develop

# From local path
bench get-app /path/to/app
```

### Installing Apps

```bash
# Install app on site
bench --site site_name install-app app_name

# Install on all sites
bench --site all install-app app_name
```

### Uninstalling Apps

```bash
# Uninstall app
bench --site site_name uninstall-app app_name

# Force uninstall
bench --site site_name uninstall-app app_name --force

# Keep doctypes (don't delete data)
bench --site site_name uninstall-app app_name --no-delete
```

### Listing Apps

```bash
# List installed apps on site
bench --site site_name list-apps

# List all apps in bench
ls apps/
```

---

## Development

### Starting Development Server

```bash
# Start all services
bench start

# Start with specific port
bench start --port 8080

# Start only web server (no workers)
bench serve

# Start with debugger
bench serve --with-debugger
```

### Console Access

```bash
# Python console with site context
bench --site site_name console

# IPython console
bench --site site_name console --autoreload

# Database console (MariaDB)
bench --site site_name mariadb

# PostgreSQL console
bench --site site_name postgres
```

### Building Assets

```bash
# Build all assets
bench build

# Build specific app
bench build --app app_name

# Build with source maps (for debugging)
bench build --production

# Watch for changes and rebuild
bench watch
```

### Clearing Cache

```bash
# Clear cache
bench --site site_name clear-cache

# Clear website cache
bench --site site_name clear-website-cache
```

---

## Database Operations

### Migrations

```bash
# Run migrations (after updating code)
bench --site site_name migrate

# Skip failing patches
bench --site site_name migrate --skip-failing

# Rebuild search index
bench --site site_name migrate --rebuild-website
```

### Backup

```bash
# Full backup
bench --site site_name backup

# Backup with files
bench --site site_name backup --with-files

# Backup to specific path
bench --site site_name backup --backup-path /path/to/backups

# Exclude specific doctypes
bench --site site_name backup --exclude "Error Log"
```

### Restore

```bash
# Restore from backup
bench --site site_name restore /path/to/backup.sql.gz

# With admin password
bench --site site_name restore backup.sql.gz --admin-password new_pass

# Restore files too
bench --site site_name restore backup.sql.gz \
    --with-public-files /path/to/files.tar \
    --with-private-files /path/to/private.tar
```

### Reinstall

```bash
# Reinstall site (drops and recreates database)
bench --site site_name reinstall

# With yes to all prompts
bench --site site_name reinstall --yes
```

---

## User Management

```bash
# Add user
bench --site site_name add-user user@example.com --first-name John --last-name Doe

# Set password
bench --site site_name set-admin-password new_password

# Add role to user
bench --site site_name add-to-hosts
```

---

## Scheduler & Background Jobs

```bash
# Enable scheduler
bench --site site_name enable-scheduler

# Disable scheduler
bench --site site_name disable-scheduler

# Run scheduler
bench --site site_name scheduler resume
bench --site site_name scheduler pause

# Trigger scheduled jobs manually
bench --site site_name execute frappe.tasks.daily

# Clear pending jobs
bench --site site_name clear-log-table "Scheduled Job Log"
```

---

## Execute Commands

```bash
# Execute Python code
bench --site site_name execute app.module.function

# With arguments
bench --site site_name execute app.module.function --args '["arg1", "arg2"]'

# With kwargs
bench --site site_name execute app.module.function --kwargs '{"key": "value"}'

# Execute method
bench --site site_name run-tests --module app.module.doctype.doctype

# Console command
bench --site site_name console
>>> frappe.get_all('User', fields=['name', 'email'])
```

---

## Testing

```bash
# Run all tests
bench --site test_site run-tests --app app_name

# Run specific module tests
bench --site test_site run-tests --module app.module

# Run specific doctype tests
bench --site test_site run-tests --doctype "DocType Name"

# Run specific test file
bench --site test_site run-tests --module app.module.doctype.your_doctype.test_your_doctype

# With coverage
bench --site test_site run-tests --app app_name --coverage

# UI tests
bench --site test_site run-ui-tests --app app_name
```

---

## Production Setup

### Setup Commands

```bash
# Setup production (nginx + supervisor)
sudo bench setup production user_name

# Setup nginx only
bench setup nginx

# Setup supervisor only
bench setup supervisor

# Setup systemd
bench setup systemd

# Setup redis
bench setup redis

# Generate nginx config
bench setup nginx --yes
```

### SSL/HTTPS

```bash
# Setup Let's Encrypt
sudo bench setup lets-encrypt site_name

# With specific email
sudo bench setup lets-encrypt site_name --custom-domain example.com
```

### Multitenancy

```bash
# Enable DNS multitenancy
bench config dns_multitenant on

# Setup custom domain
bench setup add-domain --site site_name domain.com
```

---

## Configuration

### View/Set Config

```bash
# Show config
bench --site site_name show-config

# Set config value
bench --site site_name set-config key value

# Set config for bench
bench set-config -g key value

# Common configs
bench --site site_name set-config developer_mode 1
bench --site site_name set-config disable_website_cache 1
bench --site site_name set-config mute_emails 1
```

### Site Config Options

```python
# site_config.json
{
    "db_name": "database_name",
    "db_password": "database_password",
    "db_type": "mariadb",  # or "postgres"
    
    # Development
    "developer_mode": 1,
    "disable_website_cache": 1,
    
    # Email
    "mute_emails": 1,
    "mail_server": "smtp.example.com",
    "mail_port": 587,
    "mail_login": "user@example.com",
    "mail_password": "password",
    "use_ssl": 0,
    "use_tls": 1,
    
    # Rate limiting
    "rate_limit": {
        "limit": 100,
        "window": 60
    },
    
    # Scheduler
    "scheduler_tick_interval": 60,
    "pause_scheduler": 0
}
```

---

## Updates

```bash
# Update bench
bench update

# Update without migrations
bench update --no-migrate

# Update specific apps
bench update --apps frappe,erpnext

# Reset to upstream
bench update --reset

# Pull without update
bench update --pull

# Specific branch
bench switch-to-branch develop frappe erpnext
```

---

## Useful Commands

```bash
# Check versions
bench version

# Show all available commands
bench --help

# Get help for specific command
bench migrate --help

# Export fixtures
bench --site site_name export-fixtures

# Import translations
bench --site site_name import-translations /path/to/translations.csv

# Generate password hash
bench generate-password

# Download backup from remote
bench download-backups --site site_name

# Trim database (remove old logs)
bench --site site_name trim-database

# Optimize tables
bench --site site_name execute frappe.utils.bench_helper.optimize_tables
```

---

## Common Workflows

### Fresh Development Setup

```bash
# Initialize bench
bench init frappe-bench --frappe-branch version-15
cd frappe-bench

# Create site
bench new-site dev.local

# Get your app
bench get-app https://github.com/org/your_app

# Install app
bench --site dev.local install-app your_app

# Enable developer mode
bench --site dev.local set-config developer_mode 1

# Start development
bench start
```

### Deploy to Production

```bash
# On production server
cd /home/frappe/frappe-bench

# Get latest code
bench update --reset

# Run migrations
bench --site site_name migrate

# Build assets
bench build --production

# Restart services
sudo supervisorctl restart all
# or
sudo systemctl restart frappe-bench-web
sudo systemctl restart frappe-bench-workers
```

### Create and Deploy Custom App

```bash
# Development
bench new-app custom_app
# ... develop app ...

# Push to repository
cd apps/custom_app
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/org/custom_app
git push -u origin main

# On production
bench get-app https://github.com/org/custom_app
bench --site site_name install-app custom_app
bench --site site_name migrate
bench build --production
sudo supervisorctl restart all
```
